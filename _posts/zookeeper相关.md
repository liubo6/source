---
title: zookeeper相关
date: 2016-08-14 22:55:02
categories: 分布式
tags: zookeeper
---
```
Apache ZooKeeper is an effort to develop and maintain an open-source server 
which enables highly reliable distributed coordination.
```
由官网解释可知
>ZooKeeper致力于提供一个高性能、高可用、且具有严格的顺序访问控制能力的分布式协调服务。
分布式应用可以基于它实现诸如数据发布/订阅、负载均衡、命名服务、分布式协调/通知，
集群管理、Master 选举、分布式锁和分布式队列等功能。

<!--more-->

## 角色
在Zookeeper集群中，主要分为三者角色，而每一个节点同时只能扮演一种角色
- Leader：接受所有Follower的提案请求并统一协调发起提案的投票，负责与所有的Follower进行内部的数据交换(同步);
- Follower：直接为客户端服务并参与提案的投票，同时与Leader进行数据交换(同步);
- Observer：直接为客户端服务但并不参与提案的投票，同时也与Leader进行数据交换(同步);

## ZooKeeper的数据模型
看做一个具有高可用性的文件系统。但这个文件系统中没有文件和目录，而是统一使用节点（node）的概念，
称为znode。znode既可以保存数据（如同文件），也可以保存其他znode（如同目录）。
所有的znode构成一个层次化的数据结构，Znode中的数据可以有多个版本，
比如某一个路径下存有多个数据版本，那么查询这个路径下的数据就需要带上版本

###  Persistent Nodes
永久有效地节点,除非client显式的删除,否则一直存在
###  Ephemeral Nodes 
临时节点,仅在创建该节点client保持连接期间有效,一旦连接丢失,zookeeper会自动删除该节点，
而且EPHEMERAL类型的节点不能有子节点
###  Sequence Nodes
顺序节点,client申请创建该节点时,zk会自动在节点路径末尾添加递增序号,这种类型是实现分布式锁,分布式queue等特殊功能的关键

## 典型使用场景
### 数据发布和订阅
发布与订阅即所谓的配置管理，顾名思义就是将数据发布到zk节点上，供订阅者动态获取数据，
实现配置信息的集中式管理和动态更新。例如：全局的配置信息、地址列表等。

### 命名服务
作为分布式命名服务，通过调用zk的create node api，能够很容易创建一个全局唯一
的path，可以将这个path作为一个名称

### 分布通知/协调
ZooKeeper中特有的watcher注册于异步通知机制，能够很好的实现分布式环境下不同系统之间的
通知与协调，实现对数据变更的实时处理。使用方法通常是不同系统都对zk上同一个znode进行注册，
监听znode的变化（包括znode本身内容及子节点内容），其中一个系统update了znode，那么
另一个系统能够收到通知，并做出相应处理。使用ZooKeeper来进行分布式通知和协调能够大大降低系统之间的耦合
>  心跳检测机制：检测系统和被测系统之间并不直接关联起来，而是通过zk上某个节点关联，大大减少系统耦合。
系统调度模式：某系统有控制台和推送系统两部分组成，控制台的职责是控制推送系统进行相应的推送工作。管理人员在控制台做的一些操作，实际上是修改了zk上某些节点的状态，而zk就把这些变化通知给它们注册watcher的客户端，即推送系统，于是，做出相应的推送任务。
工作汇报模式：一些类似于任务分发系统，子任务启动后，到zk来注册一个临时节点，并定时将自己的进度进行汇报（将进度写回这个临时节点），这样任务管理者就能够实时指导任务进度。

### 分布式锁
主要得益于ZooKeeper保证数据的强一致性，即zk集群中任意节点（一个zk server）上系统znoe的数据一定相同。
锁服务可以分为两类：
保持独占锁：所有试图来获取这个锁的客户端，最终只有一个可以成功获得这把锁。通常的做法是把zk上的一个znode
看做是一把锁，通过create znode的方式来实现。所有客户端都去创建/distribute_lock节点，最终成功创建的那个客户端
也即拥有了这把锁。
控制时序锁：所有试图来获取这个锁的客户端，最终都是会被安排执行，只是有个全局时序了。与保持独占锁的做法类似，
不同点是/distribute_lock已经预先存在，客户端在它下面创建临时有序节点（可以通过节点控制属性控制：
CreateMode.EPHEMERAL_SEQUENTIAL来指定）。zk的父节点(/distribute_lock)维持一份sequence，
保证子节点创建的时序性，从而形成每个客户端的全局时序

### 集群管理
> 集群机器监控：这通常用于那种对集群中机器状态、机器在线率有较高要求的场景，能够快速对集群中机器变化做出响应。
这样的场景中，往往有一个监控系统，实时监测集群机器是否存活。过去的做法通常是：监控系统通过某种手段（比如ping）
定时检测每个机器、或每个机器定时向监控系统发送心跳信息。这种做法存在两个弊端：1.集群中机器有变动的时候，
牵连修改的东西比较多。2.有一定的延迟。利用ZooKeeper，可以实现另一种集群机器存活性监控系统：
a.客户端在节点x上注册watcher，如果x的子节点发生变化，会通知该客户端。b.创建EPHEMERAL类型的节点，
一旦客户端和服务器的会话结束或过期，该节点就会消失。例如：监控系统在/clusterServers节点上注册一个watcher，
以后每动态加机器，就往/culsterServer下创建一个EPHEMERAL类型的节点：/clusterServer/{hostname}。
这样，监控系统就能实时知道机器的增减情况，至于后续处理就是监控系统的业务了。
Master选举：在分布式环境中，相同的业务应用分布在不同的机器上，有些业务逻辑（例如一些耗时的计算、网络I/O处理）
，往往需要让整个集群中的某一台机器进行执行，其余机器可以共享这个结果，这样可以减少重复劳动、提高性能。
利用ZooKeeper的强一致性，能够保证在分布式高并发情况下节点创建的全局唯一性，即：同时有多个客户端请求
创建/currentMaster节点，最终一定只有一个客户端请求能够创建成功。利用这个特性，就能很轻易的在分布式环境中
进行集群选取了。另外，这种场景演化一下，就是动态Master选举。这就要用到 EPHEMERAL_SEQUENTIAL类型节点
的特性了。上文中提到，所有客户端创建请求，最终只有一个能够创建成功。在这里稍微变化下，就是允许所有请求都
能够创建成功，但是得有个创建顺序，于是所有的请求最终在zk上创建结果的一种可能情况是这样：
 /currentMaster/{sessionId}-1、/currentMaster/{sessionId}-2、/currentMaster/{sessionId}-3……。每次选取序列号
最小的那个机器作为Master，如果这个机器挂了，由于他创建的节点会马上消失，那么之后最小的那个机器就是Master了。

### 分布式队列
队列方面，有两种方式：一种是常规的先进先出队列，另一种是要等到队列成员聚齐之后的才统一按序执行。
对于先进先出队列，和分布式锁服务中的控制时序场景基本原理一致
第二种队列其实是在FIFO队列的基础上作了一个增强。通常可以在/queue这个znode下预先建立一个/queue/
num节点，并且赋值为n（或者直接给/queue赋值n），表示队列大小，之后每次有队列成员加入后，
就判断下是否已经到达队列大小，决定是否可以开始执行了。这种用法的典型场景是，分布式环境中，一个大任务Task 
A，需要在很多子任务完成（或条件就绪）情况下才能进行。这个时候，凡是其中一个子任务完成（就绪），那么就去/
taskList下建立自己的临时时序节点（CreateMode.EPHEMERAL_SEQUENTIAL），当/
taskList发现自己下面的子节点满足指定个数，就可以进行下一步按序进行处理了。


