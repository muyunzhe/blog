---
title: Hive编程-HiveQL数据导入与导出
comments: true
categories: technology
tags: database
date: 2016-05-24 20:11:04
---

HiveQL,也就是hive查询语言，关注与向表中装载数据和从表中抽取收据到文件系统的数据操作。
<!--more-->
# 数据导入
Hive没有行级别的数据插入、数据更新和删除操作，那么往表中装载数据的唯一途径就是使用一种“大量”的数据装载操作。
1.从本地文件系统中导入数据
在hive环境下运行：
```
LOAD DATA LOCAL INPATH '${env:HOME}/california-employee'
OVERWRITE INTO TABLE employees
PARTITION (country = 'US',stat = 'CA');
```
提示：
> LOAD DATA LOCAL... 拷贝本地数据到位于分布式文件系统上的目标位置，而LOAD DATA...转移数据到目标位置

如果分区目录不存在的话，这个命令会先创建分区目录，然后再将数据拷贝到该目录下。
如果目标是非分区表，那么与剧中应该省略PARTITION子句。
通常情况下指定的路径应该是一个目录而不是单个独立的文件。
HIve要求源文件和目标文件以及目录应该在同一个文件系统中。
如果用户指定了OVERWRITE关键字，那么目标文件夹中之前的数据将会被先删除掉。如果没有该关键字，仅仅会把新增的文件增加到目标文件夹中而不会删除之前的数据，然而如果目标文件夹中已经存在和装载的文件同名的文件，那么旧的文件将被覆盖掉。

2.从HSFS上导数据到hive表
新建hdfs文件目录
`hadoop fs -mkdir /tmp/input;`
将本地数据放到集群中
`hadoop fs -put ${env:HOME}/california-employee /tmp/input/california-employee;`
查看文件是否复制成功
`dfs -cat /tmp/input/california-employee;`
导入集群数据
`load data inpath '/tmp/input/california-employee' overwrite into table employees partition(openingtime=201508);`

3.从别的Hive表中导入数据到Hive表中
创建数据表
```
create table if not exists temp.employees(id int,name string,tel string)
row format delimited
fields terminated by ','
stored as textfile
partitioned by (age int)
CLUSTERED BY (id) INTO 256 BUCKETS STORED AS ORC;
```
插入数据
```
from ext.employees w
insert overwrite table temp.employees
partition (age=25)
select w.id,w.name,w.tel where w.age=25
```
也可以采用动态分区
```
from ext.employees w
insert overwrite table temp.employees
partition (age)
select w.id,w.name,w.tel where w.age=25
```
在这里age分区将会根据w.age的值，被动态的创建。

多表插入，是一种高效的插入方式，同时插入多个分区
```
from ext.employees w
insert overwrite table temp.employees
    select w.id,w.name,w.tel where w.age=25
insert overwrite table temp.employees
    select w.id,w.name,w.tel where w.age=27;
```

4.创建Hive表的同时导入查询数据

```
create table temp.employees
       as select id,name,tel,age from ext.employees where age=25;
```

5.使用sqoop从关系数据库导入数据到Hive表
这部分另说

# 数据导出
1.将查询的结果写入文件系统
语法：
```
Standard syntax:
INSERT OVERWRITE [LOCAL] DIRECTORY directory1
  [ROW FORMAT row_format] [STORED AS file_format] (Note: Only available starting with Hive 0.11.0)
  SELECT ... FROM ...

Hive extension (multiple inserts):
FROM from_statement
INSERT OVERWRITE [LOCAL] DIRECTORY directory1 select_statement1
[INSERT OVERWRITE [LOCAL] DIRECTORY directory2 select_statement2] ...


row_format
  : DELIMITED [FIELDS TERMINATED BY char [ESCAPED BY char]] [COLLECTION ITEMS TERMINATED BY char]
        [MAP KEYS TERMINATED BY char] [LINES TERMINATED BY char]
        [NULL DEFINED AS char] (Note: Only available starting with Hive 0.13)
```
如果使用LOCAL，则数据会写入到本地.如果不使用LOCAL,则数据会写到指定的HDFS中，如果没写全路径，则使用Hadoop的配置项fs.default.name （NameNode的URI）。
例子：
1.导出到本地
`insert overwrite local directory  '/data/tmp/score' select * from score;`
这个导出的结果会被分区
2.导出到HDFS
`insert overwrite directory  '/data/tmp/score' select * from score;`
3.导出到另一个表中
`insert into table test partition (age='25') select id, name, tel from wyp;`
4.用hive命令带-e或-f参数来导出数据，导出的结果是用\t分割的
`hive -e "select * from wyp" >> local/wyp.txt`
