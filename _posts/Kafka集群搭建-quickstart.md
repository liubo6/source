---
title: Kafka集群搭建-quickstart
date: 2016-06-16 10:52:14
tags: kafka
---

## 下载安装kafka
```
#下载二进制安装包
wget  http://apache.fayea.com/kafka/0.10.0.0/kafka_2.11-0.10.0.0.tgz
#解压到特定目录
tar -zxvf kafka_2.11-0.10.0.0.tgz  -C /opt/

```
##  配置Kafka
```
vim  /opt/kafka_2.11-0.10.0.0/config/server.properties
#内容
broker.id=1
log.dirs=/data/kafka/logs
zookeeper.connect=zookeeper.connect=192.168.129.171:2181,192.168.129.172:2181,192.168.129.173:2181
#拷贝到其他节点，在其他节点上对应修改 broker.id 即可// 修改为2，3
```

<!--more-->


### 启动kafka
```
#在每个节点上执行
./bin/kafka-server-start.sh  config/server.properties &
```
## 验证安装

###  创建一个topic
```
bin/kafka-topics.sh --create --zookeeper 192.168.129.171:2181,192.168.129.172:2181,192.168.129.173:2181 --replication-factor 3 --partitions 1 --topic liubo-topic
```
###  查看创建的topic
```
[root@localhost kafka_2.11-0.10.0.0]# bin/kafka-topics.sh --list --zookeeper 192.168.129.171:2181
liubo-topic
```
###  查看创建的topic详细信息
```
[root@localhost kafka_2.11-0.10.0.0]# bin/kafka-topics.sh  --describe --zookeeper 192.168.129.171:2181
Topic:liubo-topic    PartitionCount:1    ReplicationFactor:3    Configs:
    Topic: liubo-topic    Partition: 0    Leader: 1    Replicas: 2,1,3    Isr: 1
#此时 1 节点被选为 Leader ， topic 有三个备份分别在我集群的3个节点上
#leader：负责处理消息的读和写，leader是从所有节点中随机选择的.
#replicas：列出了所有的副本节点，不管节点是否在服务中.
#isr：是正在服务中的节点.
```
### 生产消息
```
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic liubo-topic
#输入消息 hello,welcome,nice to meet you 
```

### 消费消息
```
bin/kafka-console-consumer.sh --zookeeper 192.168.129.171:2181  --from-beginning --topic liubo-topic
# 收到消息
```

## 高可用测试

```
[root@localhost kafka_2.11-0.10.0.0]# bin/kafka-topics.sh  --describe --zookeeper 192.168.129.171:2181
Topic:liubo-topic    PartitionCount:1    ReplicationFactor:3    Configs:
    Topic: liubo-topic    Partition: 0    Leader: 1    Replicas: 2,1,3    Isr: 1

```
上面是目前集群状态，关闭1节点，查看集群状态
```
[root@localhost kafka_2.11-0.10.0.0]# bin/kafka-topics.sh  --describe --zookeeper 192.168.129.171:2181
Topic:liubo-topic    PartitionCount:1    ReplicationFactor:3    Configs:
    Topic: liubo-topic    Partition: 0    Leader: 2    Replicas: 2,1,3    Isr: 2
```
2被选举为 新的leader 。


















