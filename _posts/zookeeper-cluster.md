---
title: zookeeper集群配置
date: 2016-05-19 11:35:04
tags: 集群
---

## 下载解压 ##
```
wget http://mirrors.noc.im/apache/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz
tar -zxvf zookeeper-3.4.6.tar.gz
```

## Zookeeper的配置文件 ##
进入zk conf 目录，将zoo_sample.cfg复制一份，命名为zoo.cfg，此即为Zookeeper的配置文件
```
cp zoo_sample.cfg zoo.cfg
```

<!--more-->

## 常见目录存放数据和日志 ##
```
mkdir -p /data/zk/zk1/data
mkdir -p /data/zk/zk1/logs
```

## 编辑zoo.cfg ##
```
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
dataDir=/data/zk/zk1/data
dataLogDir=/data/zk/zk1/logs
# the port at which the clients will connect
clientPort=2181
server.1=192.168.129.171:4001:4002
server.2=192.168.129.172:4001:4002
server.3=192.168.129.173:4001:4002
#port1是node间通信使用的端口，port2是node选举使用的端口，需确保三台主机的这两个端口都是互通的
#在另外两台主机上执行同样的操作，安装并配置zookeeper
```

## 创建PID ##
>分别在三台主机的dataDir路径下创建一个文件名为myid的文件，文件内容为该zk节点的编号。例如在第一台主机上建立的myid文件内容是1，第二台是2,第三台是3

## 三个节点都启动 ##
```
bin/zkServer.sh start
#启动成功
JMX enabled by default
Using config: /opt/zookeeper-3.4.6/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
```
## 查看集群状态 ##
```
bin/zkServer.sh status
#171 Mode: follower
#172 Mode: leader
#173 Mode: follower
```
## 停止172，然后查看集群状态 ##
```
./bin/zkServer.sh stop
#查看171，173状态
172 Mode: follower
173 Mode: leader
```

## zookeeper集群的安装和高可用性验证完成 ##
>Mode: leader 或 Mode: follower
3个节点中，应有1个leader和两个follower
停掉server1上的zookeeper服务会发现此时server2（也有可能是server3）为leader，另一个为follower

## zookeeper.out ##
>Zookeeper默认会将控制台信息输出到启动路径下的zookeeper.out中，显然在生产环境中我们不能允许Zookeeper这样做，通过如下方法，可以让Zookeeper输出按尺寸切分的日志文件：修改conf/log4j.properties文件，将
zookeeper.root.logger=INFO, CONSOLE
改为zookeeper.root.logger=INFO, ROLLINGFILE
修改bin/zkEnv.sh文件，
将ZOO_LOG4J_PROP="INFO,CONSOLE"改为
ZOO_LOG4J_PROP="INFO,ROLLINGFILE"然后重启zookeeper，就ok了
    
---



