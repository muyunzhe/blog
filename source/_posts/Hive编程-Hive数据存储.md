---
title: Hive编程-Hive数据存储
comments: true
categories: [cloud-computing,storage]
tags: [storage,hive]
keywords: '牧云者,人工智能,云计算,数据挖掘,hexo,blog'
date: 2016-05-26 09:56:28
---
Hive本身是没有专门的数据存储格式，也没有为数据建立索引，只需要在创建表的时候告诉Hive数据中的列分隔符和行分隔符，Hive就可以解析数据。所以往Hive表里面导入数据只是简单的将数据移动到表所在的目录中。
<!--more-->
Hive的数据分为表数据和元数据，表数据是Hive中表格(table)具有的数据;而元数据是用来存储表的名字，表的列和分区及其属性，表的属性(是否为外部表等)，表的数据所在目录等。下面分别来介绍。
# Hive的数据存储
Hive中主要包含以下几种数据模型：Table(表)，External Table(外部表)，Partition(分区)，Bucket(桶)
1.Table(表)
首先是数据库的存储，每个库在HDFS中都有相应的目录用来存储，这个目录可以通过${HIVE_HOME}/conf/hive-site.xml配置文件中的 hive.metastore.warehouse.dir属性来配置，这个属性默认的值是/user/hive/warehouse(这个目录在 HDFS上)，我们可以根据实际的情况来修改这个配置。并且每个库对应的文件夹都是以db结尾，比如系统下有个数据库叫temp，那么它在HDFS里的存储路径就为：
`hdfs://user/hive/warehouse/temp.db`
该数据库下表的数据存放在上述目录下的子文件夹中，如表temp_table存储于：
`hdfs://user/hive/warehouse/temp.db/temp_table`
文件中。
2.External Table(外部表)
Hive中的外部表和表很类似，但是其数据不是放在自己表所属的目录中，而是存放到别处，这样的好处是如果你要删除这个外部表，该外部表所指向的数据是不会被删除的，它只会删除外部表对应的元数据;而如果你要删除表，该表对应的所有数据包括元数据都会被删除。
3.Partition(分区)
在Hive中，表的每一个分区对应表下的相应目录，所有分区的数据都是存储在对应的目录中。比如wyp 表有dt和city两个分区，则对应dt=20131218,city=BJ对应表的目录为/user/hive/warehouse /dt=20131218/city=BJ，所有属于这个分区的数据都存放在这个目录中。
4.Bucket(桶)
对指定的列计算其hash，根据hash值切分数据，目的是为了并行，每一个桶对应一个文件(注意和分区的区别)。比如将wyp表id列分散至16个桶中，首先对id列的值计算hash，对应hash值为0和16的数据存储的HDFS目录为：/user /hive/warehouse/wyp/part-00000;而hash值为2的数据存储的HDFS 目录为：/user/hive/warehouse/wyp/part-00002。
# Hive的元数据存储
Hive中的元数据包括表的名字，表的列和分区及其属性，表的属性(是否为外部表等)，表的数据所在目录等。 目前Hive将元数据存储在RDBMS中，如Mysql、Derby中。
元数据中主要存储一下几类数据：
* Database相关
存储Hive Database的元数据信息，DB_ID是数据库ID,NAME是库名,DB_LOCATION_URI是数据库在HDFS中的位置，DESC为数据库的描述信息。
* Table相关
TBLS 存储Hive Table的元数据信息,每个表有唯一的TBL_ID,SD_ID外键指向所属的Database,SD_IID关联SDS表的主键。
* 数据存储相关SDS
SDS表保存了Hive数据仓库所有的HDFS数据文件信息，每个SD_ID唯一标记一个数据存储记录,CD_ID关联COLUMN_V2.CD_ID，指定该数据的字段信息,SERDE_ID关联SERDES.SERDE_ID，指定该数据的序列化信息
* COLUMN相关
该表只有一个字段CD_ID，永远存储整个Hive数据仓库中的CD_ID.
* (序列化)
* Partition相关(分区)
Hive表分区信息
* SKEW相关(数据倾斜)
* BUCKET相关(分桶)
* PRIVS相关(权限管理)
