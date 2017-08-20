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
 对sum，count来说，不存在数据倾斜问题。
## 数据倾斜的情况
### Join
 1. 其中一个表较小，但是key集中，分发到某一个或几个Reduce上的数据远高于平均值
 2. 大表与大表，但是分桶的判断字段0值或空值过多，这些空值都由一个reduce处理，灰常慢

### group by
 group by 维度过小，某值的数量过多，处理某值的reduce灰常耗时

### Count Distinct
 某特殊值过多，处理此特殊值的reduce耗时

## 原因
  1)、key分布不均匀
  2)、业务数据本身的特性
  3)、建表时考虑不周
  4)、某些SQL语句本身就有数据倾斜

## 表现
 任务进度长时间维持在99%（或100%），查看任务监控页面，发现只有少量（1个或几个）reduce子任务未完成。

## 解决方案
### 调参
hive.map.aggr=true
Map 端部分聚合，相当于Combiner

hive.groupby.skewindata=true
有数据倾斜的时候进行负载均衡，当选项设定为 true，生成的查询计划会有两个 MR Job。第一个 MR Job 中，Map 的输出结果集合会随机分布到 Reduce 中，每个 Reduce 做部分聚合操作，并输出结果，这样处理的结果是相同的 Group By Key 有可能被分发到不同的 Reduce 中，从而达到负载均衡的目的；第二个 MR Job 再根据预处理的数据结果按照 Group By Key 分布到 Reduce 中（这个过程可以保证相同的 Group By Key 被分布到同一个 Reduce 中），最后完成最终的聚合操作。

### 修改sql逻辑
如何Join：
关于驱动表的选取，选用join key分布最均匀的表作为驱动表
做好列裁剪和filter操作，以达到两表做join的时候，数据量相对变小的效果。
大小表Join：
使用map join让小的维度表（1000条以下的记录条数） 先进内存。在map端完成reduce.
大表Join大表：
把空值的key变成一个字符串加上随机数，把倾斜的数据分到不同的reduce上，由于null值关联不上，处理后并不影响最终结果。
count distinct大量相同特殊值
count distinct时，将值为空的情况单独处理，如果是计算count distinct，可以不用处理，直接过滤，在最后结果中加1。如果还有其他计算，需要进行group by，可以先将值为空的记录单独处理，再和其他计算结果进行union。
group by维度过小：
采用sum() group by的方式来替换count(distinct)完成计算。
特殊情况特殊处理：
在业务逻辑优化效果的不大情况下，有些时候是可以将倾斜的数据单独拿出来处理。最后union回去。

## 解决案例
1. 空值产生的数据倾斜，空值会被分配到一个reduce
```
select *
  from log a
  left outer join users b
  on case when a.user_id is null then concat(‘hive’,rand() ) else a.user_id end = b.user_id;
```

2. 不同数据类型关联产生数据倾斜
用户表中user_id字段为int，log表中user_id字段既有string类型也有int类型。当按照user_id进行两个表的Join操作时，默认的Hash操作会按int型的id来进行分配，这样会导致所有string类型id的记录都分配到一个Reducer中。
```
select * from users a
  left outer join logs b
  on a.usr_id = cast(b.user_id as string)
```

3. 小表不小不大，怎么用 map join 解决倾斜问题
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

4. count(distinct ip) 转成 count(*) from **** group by ip
使用disticnt函数，所有的数据只会shuffle到一个reducer上，导致reducer数据倾斜严重。
count(*) 可以在多个reduce上执行

5. distribute by
distribute by是控制在map端如何拆分数据给reduce端的。hive会根据distribute by后面列，对应reduce的个数进行分发，默认是采用hash算法。sort by为每个reduce产生一个排序文件。在有些情况下，你需要控制某个特定行应该到哪个reducer，这通常是为了进行后续的聚集操作。distribute by刚好可以做这件事。因此，distribute by经常和sort by配合使用。
