# Kafka基础



[toc]



## 消息队列的作用

削峰：

解耦：不同模块通过MQ传递消息。

异步通信：



## 消息队列两种模式

点对点模式：消息消费后被删除。

发布订阅模式：消息消费后不被删除。



## AMQP模型

包含三大实体：队列、信箱和绑定。

消息被放在信箱中，根据绑定的路由规则，将消息发送到队列中。

AMQP特点：支持事务、数据一致性高。



## TOPIC

Record存放在topic中，topic类似DB的table。



## PARTITION

一个TOPIC包含多个PARTITION。通常不同的PARTITION存放在不同的服务器上。这样就可以线性拓展Kafka。

Record存放到分区后，就不可变更。Kafka可以通过偏移量提取某个Partition中的消息。



Kafka通过副本机制，保证分区的可靠性。例如：`replication-factor = 3`表示1个Partition有2个副本。

Kafka会选择一个Partition作为Leader，其他副本作为Follower。数据的读写都发生在Leader上。Follower只负责从Leader中复制数据，保证数据一致性。



## RECORD

一个Record是一个键值对。若不指定key，则key为空。

相同key的Record会被放在同一个Partition。key为null的Record会被放在每一个Partition中。



## BROKER

Kafka集群由多个Broker组成，Broker负责读写Record，并将数据写入磁盘中。通常一个服务器上有一个Broker实例。



## 实战

创建./etc

将config下的zookeeper.properties拷贝到etc。

搭建3个节点。将config下的server.properties拷贝到etc下，分别命名为server-0.properties，server-1.properties，server-2.properties，并写上broker-id=0，log.dirs=/tmp/kafka-logs-0，listeners=PLAINTEXT://:9092（依次写）。

启动 zookeeper：bin目录下执行`./zookeeper-server-start.sh ../etc/zookeeper.properties`

启动3个kafka：`./kafka-server-start.sh ../etc/server-0.properties`和`./kafka-server-start.sh ../etc/server-0.properties`和`./kafka-server-start.sh ../etc/server-0.properties`



bin目录下执行`./kafka-topics.sh --zookeeper localhost:2181 --create --topic test --partitions 3 --replication-factor 2`

bin目录下执行`./kafka-console-consumer.sh --bootstrap-server localhost:9092, localhost:9093, localhost:9094 --topic test`

bin目录下执行`./kafka-console-producer.sh --broker-list localhost:9092, localhost:9093, localhost:9094 --topic test`，然后生产者可以发送消息了。





Server.properties配置文件重要配置：

broke.id

log.dirs

zookeeper.connect

Listener:指定broker启动时本机的监听名称和端口。

Advertised.listeners

