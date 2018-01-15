---
title: hive 数据倾斜解决方案
date: 2017-07-21 16:23:28
comments: true
categories: [cloud-computing,storage]
tags: [storage,hive]
keywords: '牧云者,人工智能,云计算,数据挖掘,hexo,blog'
---
数据倾斜是hive工程中常见的情况。
 <!--more-->
 对sum，count,max,min来说，不存在数据倾斜问题。
 在开始之前，先把MR的流程图帖出来（摘自Hadoop权威指南），方便后面对照。
 ![mr](/img/mr.jpg)

## 表现
任务进度长时间维持在99%（或100%），查看任务监控页面，发现只有少量（1个或几个）reduce子任务未完成。

## 原因
  1)、key分布不均匀
  2)、业务数据本身的特性
  3)、建表时考虑不周
  4)、某些SQL语句本身就有数据倾斜

## map阶段的优化
* mapred.min.split.size指的是数据的最小分割单元大小。
* mapred.max.split.size指的是数据的最大分割单元大小。
* dfs.block.size指的是HDFS设置的数据块大小。
一般来说dfs.block.size这个值是一个已经指定好的值，而且这个参数hive是识别不到的,所以实际上只有mapred.min.split.size和mapred.max.split.size这两个参数来决定map数量。在hive中min的默认值是1B，max的默认值是256MB.
所以如果不做修改的话，就是1个map task处理256MB数据，我们就以调整max为主。通过调整max可以起到调整map数的作用，减小max可以增加map数，增大max可以减少map数。需要提醒的是，直接调整mapred.map.tasks这个参数是没有效果的。
调整大小的时机根据查询的不同而不同，总的来讲可以通过观察map task的完成时间来确定是否需要增加map资源。如果map task的完成时间都是接近1分钟，甚至几分钟了，那么往往增加map数量，使得每个map task处理的数据量减少，能够让map task更快完成；而如果map task的运行时间已经很少了，比如10-20秒，这个时候增加map不太可能让map task更快完成，反而可能因为map需要的初始化时间反而让job总体速度变慢，这个时候反而需要考虑是否可以把map的数量减少，这样可以节省更多资源给其他Job。

## reduce阶段的优化
与map优化不同的是，reduce优化时，可以直接设置mapred.reduce.tasks参数从而直接指定reduce的个数。当然直接指定reduce个数虽然比较方便，但是不利于自动扩展。Reduce数的设置虽然相较map更灵活，但是也可以像map一样设定一个自动生成规则，这样运行定时job的时候就不用担心原来设置的固定reduce数会由于数据量的变化而不合适。

## map与reduce之间的优化
map phase和reduce phase之间主要有3道工序。首先要把map输出的结果进行排序后做成中间文件，其次这个中间文件就能分发到各个reduce，最后reduce端在执行reduce phase之前把收集到的排序子文件合并成一个排序文件。需要强调的是，虽然这个部分可以调的参数挺多，但是大部分在一般情况下都是不要调整的，除非能精准的定位到这个部分有问题。

spill
在spill阶段，由于内存不够，数据可能没办法在内存中一次性排序完成，那么就只能把局部排序的文件先保存到磁盘上，这个动作叫spill，然后spill出来的多个文件可以在最后进行merge。如果发生spill，可以通过设置io.sort.mb来增大mapper输出buffer的大小，避免spill的发生。另外合并时可以通过设置io.sort.factor来使得一次性能够合并更多的数据，默认值为10，也就是一次归并10个文件。调试参数的时候，一个要看spill的时间成本，一个要看merge的时间成本，还需要注意不要撑爆内存（io.sort.mb是算在map的内存里面的）。Reduce端的merge也是一样可以用io.sort.factor。一般情况下这两个参数很少需要调整，除非很明确知道这个地方是瓶颈。比如如果map端的输出太大，考虑到map数不一定能很方便的调整，那么这个时候就要考虑调大io.sort.mb（不过即使调大也要注意不能超过jvm heap size）。而map端的输出很大，要么是每个map读入了很大的文件（比如不能split的大gz压缩文件），要么是计算逻辑导致输出膨胀了很多倍，都是比较少见的情况。

Copy
这里说的copy，一般叫做shuffle更加常见。但是由于一开始的配图以及MR job的web监控页对这个环节都是叫copy phase，指代更加精确，所以这里称为copy。

copy阶段是把文件从map端copy到reduce端。默认情况下在5%的map完成的情况下reduce就开始启动copy，这个有时候是很浪费资源的，因为reduce一旦启动就被占用，一直等到map全部完成，收集到所有数据才可以进行后面的动作，所以我们可以等比较多的map完成之后再启动reduce流程，这个比例可以通过mapred.reduce.slowstart.completed.maps去调整，他的默认值就是5%。如果觉得这么做会减慢reduce端copy的进度，可以把copy过程的线程增大。tasktracker.http.threads可以决定作为server端的map用于提供数据传输服务的线程，mapred.reduce.parallel.copies可以决定作为client端的reduce同时从map端拉取数据的并行度（一次同时从多少个map拉数据），修改参数的时候这两个注意协调一下，server端能处理client端的请求即可。

另外，在shuffle阶段可能会出现的OOM问题，原因比较复杂，一般认为是内存分配不合理，GC无法及时释放内存导致。对于这个问题，可以尝试调低shuffle buffer的控制参数mapred.job.shuffle.input.buffer.percent这个比例值，默认值0.7，即shuffle buffer占到reduce task heap size的70%。另外也可以直接尝试增加reduce数量。

## 解决方案
### group by类
hive.map.aggr=true; #Map 端部分聚合，相当于Combiner，也就是在mapper里面做聚合。这个方法不同于直接写mapreduce的时候可以实现的combiner，但是却实现了类似combiner的效果。
hive.groupby.skewindata; 意思是做reduce操作的时候，拿到的key并不是所有相同值给同一个reduce，而是随机分发，然后reduce做聚合，做完之后再做一轮MR，拿前面聚合过的数据再算结果。所以这个参数其实跟hive.map.aggr做的是类似的事情，只是拿到reduce端来做，而且要额外启动一轮job，所以其实不怎么推荐用，效果不明显。

### count distinct类
使用disticnt函数，所有的数据只会shuffle到一个reducer上，导致reducer数据倾斜严重。
count(*) 可以在多个reduce上执行。
直接改写sql
```
/*改写前*/
select a, count(distinct b) as c from tbl group by a;

/*改写后*/
select a, count(*) as c from (select a, b from tbl group by a, b) group by a;
```

### Join类
#### 大表与大表
1. 设置参数
set hive.optimize.skewjoin = true;
set hive.skewjoin.key=100000; 这个是join的键对应的记录条数超过这个值则会进行优化。
有数据倾斜的时候进行负载均衡，当选项设定为 true，生成的查询计划会有两个 MR Job。第一个 MR Job 中，Map 的输出结果集合会随机分布到 Reduce 中，每个 Reduce 做部分聚合操作，并输出结果到hdfs，这样处理的结果是相同的 Group By Key 有可能被分发到不同的 Reduce 中，从而达到负载均衡的目的；第二个 MR Job 再根据预处理的数据结果按照 Group By Key 分布到 Reduce 中（这个过程可以保证相同的 Group By Key 被分布到同一个 Reduce 中），最后完成最终的聚合操作。
![skew_join](/img/skew_join.jpg)

2. 加随机数
就是在join的时候增加一个随机数，随机数的取值范围n相当于将item给分散到n个reducer。
```
select a.*, b.*
from (select *, cast(rand() * 10 as int) as r_id from logs)a
join (select *, r_id from items
lateral view explode(range_list(1,10)) rl as r_id)b
on a.item_id = b.item_id and a.r_id = b.r_id
```
上面的写法里，对行为表的每条记录生成一个1-10的随机整数，对于item属性表，每个item生成10条记录，随机key分别也是1-10，这样就能保证行为表关联上属性表。其中range_list(1,10)代表用udf实现的一个返回1-10整数序列的方法。这个做法是一个解决join倾斜比较根本性的通用思路，就是如何用随机数将key进行分散。当然，可以根据具体的业务场景做实现上的简化或变化。

#### 小表与大表
如何Join：
关于驱动表的选取，选用join key分布最均匀的表作为驱动表
做好列裁剪和filter操作，以达到两表做join的时候，数据量相对变小的效果。
大小表Join：
使用map join让小的维度表（1000条以下的记录条数,小于25m） 先进内存。在map端完成reduce.
```
select /*+mapjoin(a)*/ count(1) from tb_a a left outer join tb_b b on a.uid=b.uid；
```

#### 有空值的key：
把空值的key变成一个字符串加上随机数，把倾斜的数据分到不同的reduce上，由于null值关联不上，处理后并不影响最终结果。
count distinct大量相同特殊值
count distinct时，将值为空的情况单独处理，如果是计算count distinct，可以不用处理，直接过滤，在最后结果中加1。如果还有其他计算，需要进行group by，可以先将值为空的记录单独处理，再和其他计算结果进行union。
group by维度过小：
采用sum() group by的方式来替换count(distinct)完成计算。
特殊情况特殊处理：
在业务逻辑优化效果的不大情况下，有些时候是可以将倾斜的数据单独拿出来处理。最后union回去。
如：
```
select *
  from log a
  left outer join users b
  on case when a.user_id is null then concat(‘hive’,rand() ) else a.user_id end = b.user_id;
```

#### 不同数据类型关联产生数据倾斜
用户表中user_id字段为int，log表中user_id字段既有string类型也有int类型。当按照user_id进行两个表的Join操作时，默认的Hash操作会按int型的id来进行分配，这样会导致所有string类型id的记录都分配到一个Reducer中。
```
select * from users a
  left outer join logs b
  on a.usr_id = cast(b.user_id as string)
```

#### 小表不小不大，怎么用 map join 解决倾斜问题
使用 map join 解决小表(记录数少)关联大表的数据倾斜问题，这个方法使用的频率非常高，但如果小表很大，大到不能放入内存处理，那么把小表分发到不同的map也是个不小的开销。
```
select * from log a
  left outer join (
    select  d.*
      from ( select distinct user_id from log ) c
      join users d
      on c.user_id = d.user_id
    ) x
  on a.user_id = b.user_id;
```


### distribute by类
distribute by是控制在map端如何拆分数据给reduce端的。hive会根据distribute by后面列，对应reduce的个数进行分发，默认是采用hash算法。sort by为每个reduce产生一个排序文件。在有些情况下，你需要控制某个特定行应该到哪个reducer，这通常是为了进行后续的聚集操作。distribute by刚好可以做这件事。因此，distribute by经常和sort by配合使用。

### sql整体优化
#### job间并行
设置job间并行的参数是hive.exec.parallel，将其设为true即可。默认的并行度为8，也就是最多允许sql中8个job并行。如果想要更高的并行度，可以通过hive.exec.parallel. thread.number参数进行设置，但要避免设置过大而占用过多资源。

#### 减少job数量
通过代码优化

#### 控制map数
两种方式控制Map数
减少map数可以通过合并小文件来实现，这点是对文件数据源来讲，下面介绍。
增加map数的可以通过控制上一个job的reduer数来控制.

#### 控制reduce数
有多少个reduce,就会有多少个输出文件。

#### 合并小文件
是否合并Map输出文件：hive.merge.mapfiles=true（默认值为真）
是否合并Reduce 端输出文件：hive.merge.mapredfiles=false（默认值为假）
Map输入合并：
set mapred.max.split.size=1000000000;
set mapred.min.split.size.per.node=1000000000;
set mapred.min.split.size.per.rack=1000000000;
set hive.merge.size.per.task=25610001000（默认值为 256000000）

上面部分是针对从很多小文件的表里读取的时候用的，并不能合并小文件。
以下配置在合并小文件时亲测有效
hive结果合并：
set hive.merge.mapfiles=true;
set hive.merge.mapredfiles=true;
set hive.merge.smallfiles.avgsize=256000000;
set hive.merge.size.per.task=256000000;


以下是个小集合：
```
-- 动态分区
SET hive.exec.dynamic.partition=true;
SET hive.exec.dynamic.partition.mode=nonstrict;
SET hive.exec.max.dynamic.partitions.pernode=10000;
SET hive.exec.max.dynamic.partitions=100000;
SET hive.exec.max.created.files=1000000;

-- map读入合并
set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat;
set mapred.max.split.size=4294967296;
set mapred.min.split.size.per.node=4294967296;
set mapred.min.split.size.per.rack=4294967296;

-- hive输出合并
set hive.merge.mapfiles=true;
set hive.merge.mapredfiles=true;
set hive.merge.smallfiles.avgsize=256000000;
set hive.merge.size.per.task=256000000;

-- 输出压缩
SET mapred.output.compress=true;
SET mapred.output.compression.type=BLOCK;
SET mapred.output.compression.codec=org.apache.hadoop.io.compress.Lz4Codec;
SET mapred.map.output.compression.codec=org.apache.hadoop.io.compress.Lz4Codec;
```
