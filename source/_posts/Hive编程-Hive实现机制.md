---
title: Hive编程-Hive实现机制
comments: true
categories: [cloud-computing,storage]
tags: [storage,hive]
keywords: '牧云者,人工智能,云计算,数据挖掘,hexo,blog'
date: 2016-05-24 21:02:16
---
Hive是基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张数据库表，并提供完整的SQL查询功能，可以将SQL语句转换为MapReduce任务运行。
<!--more-->
1.Hive介绍
Hive 定义了简单的类 SQL 查询语言，称为 HQL，它允许熟悉 SQL 的用户查询数据。同时，这个语言也允许熟悉 MapReduce 开发者的开发自定义的 mapper 和 reducer 来处理内建的 mapper 和 reducer 无法完成的复杂的分析工作。
Hive是建立在Hadoop基础上的数据仓库软件包,在开始阶段,它被Facebook用于处理大量的用户数据和日志数据.它现在是Hadoop的子项目并有许多贡献者.其目标用户仍然是习惯SQL的数据分析师,他们需要在Hadoop规模的数据上做即席查询/汇总和数据分析.通过称为HiveQL的类SQL语言,你可以发起一个查询来实现与Hive的交互.

2.实现机制
利用hadoop Map/Reduce程序和Hive实现hadoop word count的对比
![img](img/chapter0000.jpeg)
>借助于Hadoop和HDFS的大数据存储能力，数据仍然存储于Hadoop的HDFS中，Hive提供了一种类SQL的查询语言：HiveQL（HQL），对数据进行管理和分析，开发人员可以近乎sql的方式来实现逻辑，从而加快应用开发效率。

HQL经过解析和编译，最终会生成基于Hadoop平台的Map Reduce任务，Hadoop通过执行这些任务来完成HQL的执行。
Hive的组件总体上可以分为以下几个部分：用户接口（UI）、驱动、编译器、元数据（Hive系统参数数据）和执行引擎。
![img](img/chapter00020.png)
>1.对外的接口UI包括以下几种：命令行CLI，Web界面、JDBC/ODBC接口；
2.驱动：接收用户提交的查询HQL；
3.编译器：解析查询语句，执行语法分析，生成执行计划；
4.元数据Metadata：存放系统的表、分区、列、列类型等所有信息，以及对应的HDFS文件信息等；
5.执行引擎：执行执行计划，执行计划是一个有向无环图，执行引擎按照各个任务的依赖关系选择执行任务（Job）。

3.Hive的数据模型
Hive没有专门的数据存储格式，也没有为数据建立索引，用于可以非常自由的组织Hive中的表，只需要在创建表的时候定义好表的schema即可。Hive中包含4中数据模型：Tabel、ExternalTable、Partition、Bucket。
![img](img/chapter00030.png)
>* Table：类似与传统数据库中的Table，每一个Table在Hive中都有一个相应的目录来存储数据。例如：一个表t，它在HDFS中的路径为：/user/hive/warehouse/t。
* Partition：类似于传统数据库中划分列的索引。在Hive中，表中的一个Partition对应于表下的一个目录，所有的Partition数据都存储在对应的目录中。例如：t表中包含ds和city两个Partition，则对应于ds=2014，city=beijing的HDFS子目录为：/user/hive/warehouse/t/ds=2014/city=Beijing； 需要注意的是，分区列是表的伪列，表数据文件中并不存在这个分区列的数据。
* Buckets：对指定列计算的hash，根据hash值切分数据，目的是为了便于并行，每一个Buckets对应一个文件。将user列分数至32个Bucket上，首先对user列的值计算hash，比如，对应hash=0的HDFS目录为：/user/hive/warehouse/t/ds=2014/city=Beijing/part-00000；对应hash=20的目录为：/user/hive/warehouse/t/ds=2014/city=Beijing/part-00020。
* External Table指向已存在HDFS中的数据，可创建Partition。Managed Table创建和数据加载过程，可以用统一语句实现，实际数据被转移到数据仓库目录中，之后对数据的访问将会直接在数据仓库的目录中完成。删除表时，表中的数据和元数据都会删除。External Table只有一个过程，因为加载数据和创建表是同时完成。数据是存储在Location后面指定的HDFS路径中的，并不会移动到数据仓库中。

4.Hive转化为mapReduce
Hive编译器将HQL代码转换成一组操作符（operator），操作符是Hive的最小操作单元，每个操作符代表了一种HDFS操作或者MapReduce作业。
如下列查询语句：
```
INSERT OVERWRITE TABLE read_log_tmp
SELECT a.userid,a.bookid,b.author,b.categoryid
FROM user_read_log a JOIN book_info b ON a.bookid = b.bookid;
```
最终的编译顺序为：
![img](img/chapter080008.png)
