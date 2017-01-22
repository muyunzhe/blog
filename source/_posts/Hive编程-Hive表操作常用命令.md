---
title: Hive编程-Hive库操作
comments: true
categories: [cloud-computing,storage]
tags: [storage,hive]
keywords: '牧云者,人工智能,云计算,数据挖掘,hexo,blog'
date: 2016-05-25 22:37:54
---
Hive作为数据管理工具，这通常意味着必须有组织和结构。否则，我们的数据库会成为一个永久存储无限制数据的死区，没人能够或者想要使用这些数据。当然Hive数据库跟传统数据库有很多不同的地方，下面就来说说Hive数据库到底如何来管理。
<!--more-->
# 数据库创建
```
CREATE (DATABASE|SCHEMA) [IF NOT EXISTS] database_name
  [COMMENT database_comment]
  [LOCATION hdfs_path]
  [WITH DBPROPERTIES (property_name=property_value, ...)];
```
The uses of SCHEMA and DATABASE are interchangeable – they mean the same thing.
IF NOT EXISTS是一个可选子句，通知用户已经存在相同名称的数据库。
然后可以通过
`SHOW DATABASES;`
查询数据库列表
# 数据库修改
用户可以通过使用alter DATABASE命令为某个数据库的DBPROPERTIES设置键值对属性值，来描述这个数据库的属性信息。数据库的其他属性是不可更改的，包括数据库名和数据库所在的目录位置：
`ALTER (DATABASE|SCHEMA) database_name SET DBPROPERTIES (property_name=property_value, ...);`
# 数据库删除
DROP DATABASE是删除所有的表并删除数据库的语句。它的语法如下：
```
DROP (DATABASE|SCHEMA) [IF EXISTS] database_name
[RESTRICT|CASCADE];
```
使用CASCADE查询删除数据库。这意味着要全部删除相应的表在删除数据库之前。
而如果使用RESTRICT的话，那么就和默认情况一下，也就是，如果想删除某个数据库，那么必须先删除该数据库里的所有表。如果某个数据库被删除了，那么其对应的目录也就同时也被删除。
