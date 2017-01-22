---
title: 分布式消息系统：kafka
comments: true
categories: [cloud-computing,architecture]
tags: [architecture,kafka]
keywords: '牧云者,人工智能,云计算,数据挖掘,hexo,blog'
date: 2016-10-11 20:49:57
---
Kafka是分布式发布-订阅消息系统。
 <!--more-->
# 特性
* 高吞吐量、低延迟：kafka每秒可以处理几十万条消息，它的延迟最低只有几毫秒
* 可扩展性：kafka集群支持热扩展
* 持久性、可靠性：消息被持久化到本地磁盘，并且支持数据备份防止数据丢失
* 容错性：允许集群中节点失败（若副本数量为n,则允许n-1个节点失败）
* 高并发：支持数千个客户端同时读写

# 架构
kafka的集群有多个Broker服务器组成，每个类型的消息被定义为topic，同一topic内部的消息按照一定的key和算法被分区(partition)存储在不同的Broker上，消息生产者producer和消费者consumer可以在多个Broker上生产/消费topic
![Kafka拓扑结构](/img/KafkaArchitecture.png)

## Topic
　　每条发布到Kafka集群的消息都有一个类别，这个类别被称为Topic。（物理上不同Topic的消息分开存储，逻辑上一个Topic的消息虽然保存于一个或多个broker上但用户只需指定消息的Topic即可生产或消费数据而不必关心数据存于何处）
## Partition
　　Parition是物理上的概念,为了使得Kafka的吞吐率可以线性提高，物理上把Topic分成一个或多个Partition，每个Partition在物理上对应一个文件夹，该文件夹下存储这个Partition的所有消息和索引文件。因为每条消息都被append到该Partition中，属于顺序写磁盘，因此效率非常高（经验证，顺序写磁盘效率比随机写内存还要高，这是Kafka高吞吐率的一个很重要的保证）。
![](/img/log_anatomy.png)

## Producer
　　负责发布消息到Kafka broker
## Consumer
　　消息消费者，向Kafka broker读取消息的客户端。
## Consumer Group
　　每个Consumer属于一个特定的Consumer Group（可为每个Consumer指定group name，若不指定group name则属于默认的group）。


因为Kafka读取特定消息的时间复杂度为O(1)，即与文件大小无关，所以这里删除过期文件与提高Kafka性能无关。选择怎样的删除策略只与磁盘以及具体的需求有关。另外，Kafka会为每一个Consumer Group保留一些metadata信息——当前消费的消息的position，也即offset。这个offset由Consumer控制。正常情况下Consumer会在消费完一条消息后递增该offset。当然，Consumer也可将offset设成一个较小的值，重新消费一些消息。因为offet由Consumer控制，所以Kafka broker是无状态的，它不需要标记哪些消息被哪些消费过，也不需要通过broker去保证同一个Consumer Group只有一个Consumer能消费某一条消息，因此也就不需要锁机制，这也为Kafka的高吞吐率提供了有力保障。
Producer发送消息到broker时，会根据Paritition机制选择将其存储到哪一个Partition。如果Partition机制设置合理，所有消息可以均匀分布到不同的Partition里，这样就实现了负载均衡。　
作为一个消息系统，Kafka遵循了传统的方式，选择由Producer向broker push消息并由Consumer从broker pull消息。

# 优势
1. 吞吐量
数据磁盘持久化：消息不在内存中cache，直接写入到磁盘，充分利用磁盘的顺序读写性能
zero-copy：减少IO操作步骤
数据批量发送
数据压缩
Topic划分为多个partition，提高parallelism
2. 负载均衡
producer根据用户指定的算法，将消息发送到指定的partition
存在多个partiiton，每个partition有自己的replica，每个replica分布在不同的Broker节点上
多个partition需要选取出lead partition，lead partition负责读写，并由zookeeper负责fail over
通过zookeeper管理broker与consumer的动态加入与离开
3. push vs pull系统
由于kafka broker会持久化数据，broker没有内存压力，因此，consumer非常适合采取pull的方式消费数据，具有以下几点好处：
(1)简化kafka设计
(2)consumer根据消费能力自主控制消息拉取速度
(3)consumer根据自身情况自主选择消费模式，例如批量，重复消费，从尾端开始消费等
4. 可扩展性
当需要增加broker结点时，新增的broker会向zookeeper注册，而producer及consumer会根据注册在zookeeper上的watcher感知这些变化，并及时作出调整。

# 设计要点
1. 直接使用linux 文件系统的cache，来高效缓存数据。
2. 采用linux Zero-Copy提高发送性能。传统的数据发送需要发送4次上下文切换，采用sendfile系统调用之后，数据直接在内核态交换，系统上下文切换减少为2次。根据测试结果，可以提高60%的数据发送性能。
3. 数据在磁盘上存取代价为O(1)。kafka以topic来进行消息管理，每个topic包含多个part（ition），每个part对应一个逻辑log，有多个segment组成。每个segment中存储多条消息，消息id由其逻辑位置决定，即从消息id可直接定位到消息的存储位置，避免id到位置的额外映射。每个part在内存中对应一个index，记录每个segment中的第一条消息偏移。发布者发到某个topic的消息会被均匀的分布到多个part上（随机或根据用户指定的回调函数进行分布），broker收到发布消息往对应part的最后一个segment上添加该消息，当某个segment上的消息条数达到配置值或消息发布时间超过阈值时，segment上的消息会被flush到磁盘，只有flush到磁盘上的消息订阅者才能订阅到，segment达到一定的大小后将不会再往该segment写数据，broker会创建新的segment。
4. 显式分布式，即所有的producer、broker和consumer都会有多个，均为分布式的。Producer和broker之间没有负载均衡机制。broker和consumer之间利用zookeeper进行负载均衡。所有broker和consumer都会在zookeeper中进行注册，且zookeeper会保存他们的一些元数据信息。如果某个broker和consumer发生了变化，所有其他的broker和consumer都会得到通知。

# 关键点
1. zookeeper在kafka的作用是什么？
管理broker和consumer，实现负载均衡
2. kafka中几乎不允许对消息进行“随机读写”的原因是什么？
顺序读写效率更高
3. kafka集群consumer和producer状态信息是如何保存的？
因为offet由Consumer控制，所以Kafka broker是无状态的，它不需要标记哪些消息被哪些消费过，也不需要通过broker去保证同一个Consumer Group只有一个Consumer能消费某一条消息，因此也就不需要锁机制，这也为Kafka的高吞吐率提供了有力保障。 　
4. partitions设计的目的的根本原因是什么？
并行
