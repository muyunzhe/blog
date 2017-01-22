---
title: mysql binlog原理
comments: true
categories: [cloud-computing,storage]
tags: [storage,mysql]
keywords: '牧云者,人工智能,云计算,数据挖掘,hexo,blog'
date: 2016-12-19 15:01:55
---
Mysql 的binlog，是mysql执行改动产生的二进制日志文件，记录对数据发生或潜在发生更改的SQL语句，并以二进制的形式保存在磁盘中。其主要作用有两个：* 数据恢复*和*主从数据库*。用于slave端执行增删改，保持与master同步。
 <!--more-->
## binlog概述
文件位置：默认存放位置为数据库文件所在目录下

文件的命名方式： 名称为hostname-bin.xxxxx （重启mysql一次将会自动生成一个新的binlog）

状态的查看：mysql> show variables like '%log_bin%';
```
mysql> show variables like '%log_bin%';

+---------------------------------+-------+

| Variable_name | Value |

+---------------------------------+-------+

| log_bin | ON | //表示当前已开启二进制日志//

| log_bin_trust_function_creators | OFF |

| sql_log_bin | ON |

+---------------------------------+-------+

3 rows in set (0.00 sec)
```

## binlog查看
主要保存日志文件名、开始偏移、事件类型、结束偏移、时间戳，info，是否自动提交事务等信息
1. 在客户端中使用 show binlog events in 'mysql_bin.000001' 语句进行查看
2. 用mysql自带的工具mysqlbinlog

## 利用binlog进行数据恢复
见 mysql数据同步
