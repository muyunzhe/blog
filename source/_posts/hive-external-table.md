---
title: hive 外表与分区
comments: true
toc: true
mathjax2: true
keywords: '牧云者,人工智能,云计算,数据挖掘,hexo,blog'
date: 2017-01-22 14:30:46
categories: [cloud-computing, storage]
tags: [storage, hive]
---
在大数据分析中，我们经常会用到外部接入的数据源，因此使用外部表可直接将外部数据源挂载进我们的hive数据库中。
 <!--more-->
## 外部表的创建
```
create external table if not exists test(
  username String,
  work string)
PARTITIONED BY(year String, month String, day String)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
LOCATION '/tmp/test/';
```

关键字external告诉hive这个表是外部的。而后面的location...子句则用户告诉hive数据位于哪个路径下。
如果创建的外部表不是分区表，则location...子句是必需的，而如果创建的外部表是分区表，则location...子句是可选的，可以在后面add分区的时候指定。


## 分区
外部表是建议分区的，这样可以减小数据风险，并且可以缩短查询范围以及逻辑管理能力等。
可以通过
```
alter table test add partition (year='2010', month='04', day='18')
location '2010/04/18';  
```

子句添加分区。hive不关心一个分区对应的分区目录是否存在或者分区目录下是否有文件。如果分区下没有文件或者分区不存在，则针对该分区的查询语句返回的结果将是空，并不报错。

### 删除分区
```
ALTER TABLE test DROP IF EXISTS PARTITION (year='2010', month='04', day='18');
```
### 修改分区
```
ALTER TABLE test PARTITION (dt='2008-08-08') SET LOCATION "new location";
ALTER TABLE test PARTITION (dt='2008-08-08') RENAME TO PARTITION (dt='20080808');
```

## 删除
因为表是外部的，所以hive并非完全拥有这份数据，因此在删除的时候，删除表并不会删除这份数据，不过描述表的元数据信息会被删除掉。
不过这里要注意，如果hive表当前操作用户对该表对应的目录没有写权限只有读权限，则创建表是没有问题的，但是创建完之后，要删除表就难了，会提示权限错误。
