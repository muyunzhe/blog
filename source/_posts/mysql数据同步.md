---
title: mysql数据同步
comments: true
categories: technology
tags: database
keywords: 'mysql, binlog， kafka， redis'
date: 2016-12-19 15:24:46
---
阿里巴巴开发的canal，通过模拟slave向master发送请求，获取mysql改动记录，将同步消息发送给redis或者kafka，实现mysql数据同步
 <!--more-->
## canal的工作原理
### mysql主备复制实现
1. master将改变记录到二进制日志(binary log)中（这些记录叫做二进制日志事件，binary log events，可以通过show binlog events进行查看）；
2. slave将master的binary log events拷贝到它的中继日志(relay log)；
3. slave重做中继日志中的事件，将改变反映它自己的数据。

### canal的工作原理
1. canal模拟mysql slave的交互协议，伪装自己为mysql slave，向mysql master发送dump协议
2. mysql master收到dump请求，开始推送binary log给slave(也就是canal)
3. canal解析binary log对象(原始为byte流)

所以，Canal的原理是模拟Slave向Master发送请求，Canal解析binlog，但不将解析结果持久化，而是保存在内存中，每次有客户端读取一次消息，就删除该消息。这里所说的客户端，就需要我们写一个连接Canal的程序，持续从Canal获取数据。

## 架构
![canal架构](/img/canal架构.jpg)
server代表一个canal运行实例，对应于一个jvm
instance对应于一个数据队列  （1个server对应1..n个instance)
instance模块：
  eventParser (数据源接入，模拟slave协议和master进行交互，协议解析)
  eventSink (Parser和Store链接器，进行数据过滤，加工，分发的工作)
  eventStore (数据存储)
  metaManager (增量订阅&消费信息管理器)

mysql的binlog是多文件存储，定位一个LogEvent需要通过binlog filename +  binlog position，进行定位
mysql的binlog数据格式，按照生成的方式，主要分为：statement-based、row-based、mixed。
目前canal只能支持row模式的增量订阅(statement只有sql，没有数据，所以无法获取原始的变更日志)

### EventParser
![EventParser](/img/EventParser.jpg)

### EventSink
![EventSink](/img/EventSink.jpg)

说明：
数据过滤：支持通配符的过滤模式，表名，字段内容等
数据路由/分发：解决1:n (1个parser对应多个store的模式)
数据归并：解决n:1 (多个parser对应1个store)
数据加工：在进入store之前进行额外的处理，比如join

### EventStore
![EventStore](/img/EventStore.jpg)

定义了3个cursor
Put :  Sink模块进行数据存储的最后一次写入位置
Get :  数据订阅获取的最后一次提取位置
Ack :  数据消费成功的最后一次消费位置

## 增量订阅/消费设计
![增量订阅/消费设计](/img/增量订阅与消费设计.jpg)


## 可靠性与可用性
1，虽然canal服务端解析binlog后不会把数据持久化，但canal服务端会记录每次客户端消费的位置(客户端每次ack时服务端会记录pos点)。如果数据正在更新时，canal服务端挂掉，客户端也会跟着挂掉，mysql依然在插入数据，而redis则因为客户端的关闭而停止更新，造成mysql和redis的数据不一致。解决办法是，只要重启canal服务端和客户端就可以了，虽然canal服务端因为重启之前解析数据清空，但因为canal服务端记录的是客户端最后一次获取的pos点，canal服务端再从这个pos点开始解析，客户端更新至redis，以达到数据的一致。
2，如果只有一个canal服务端和一个客户端，肯定存在可用性低的问题，一种做法是用程序来监控canal服务端和客户端，如果挂掉，再重启；一种做法是多个canal服务端+zk，将canal服务端的配置文件放在zk，任何一个canal服务端挂掉后，切换到其他canal服务端，读到的配置文件的内容就是一致的（还有记录的消费pos点），保证业务的高可用，客户端可使用相同的做法。
