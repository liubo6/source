---
title: RabbitMQ作用、模型、应用场景
date: 2016-07-08 09:41:52
categories: 消息队列
tags: 消息队列
---
## RabbitMQ 作用
- 异步处理
- 应用解耦
- 消息缓冲
- 流量削峰
- 消息通信
- 消息分发

<!--more-->

消息队列本质上，是把多个应用给连接了起来，相互之间做异步消息通讯用。连接的彼此，可能都是一套独立的生态系统，连接后，就产生了更为强大的软件程序

## 应用场景
### 异步处理
![](http://ww4.sinaimg.cn/mw690/69045600gw1f5mak1yerzj20ef03o74o.jpg)
![](http://ww4.sinaimg.cn/mw690/69045600gw1f5mak2c7n0j20b305wq3c.jpg)
![](http://ww2.sinaimg.cn/mw690/69045600gw1f5mak2n42vj20fu0593z2.jpg)

### 应用解耦
![](http://ww1.sinaimg.cn/mw690/69045600gw1f5mak35y8zj207y031jrf.jpg)
![](http://ww1.sinaimg.cn/mw690/69045600gw1f5mak3yv93j209m04zdfz.jpg)

### 流量削峰
![](http://ww2.sinaimg.cn/mw690/69045600gw1f5mak4dandj20c4035glt.jpg)

### 日志处理
![](http://ww2.sinaimg.cn/mw690/69045600gw1f5mak4kb34j20co0313yq.jpg)
![](http://ww4.sinaimg.cn/mw690/69045600gw1f5mak4y3uyj20up0bxdhg.jpg)

### 消息通讯
![](http://ww4.sinaimg.cn/mw690/69045600gw1f5mak5fyvmj20ba031wen.jpg)
![](http://ww1.sinaimg.cn/mw690/69045600gw1f5mak5oaz1j20ba039jrm.jpg)

###  电商系统
![](http://ww4.sinaimg.cn/mw690/69045600gw1f5mak6dbxkj20b704s3z2.jpg) 

### 日志收集
![](http://ww2.sinaimg.cn/mw690/69045600gw1f5mak6vh02j20f505kdgs.jpg)


## 其他常见消息队列
- ActiveMQ
- ZeroMQ
- Kafka
- MetaMQ
- RocketMQ


## AMQP协议
一种二进制协议。默认启动端口 5672
![](http://ww3.sinaimg.cn/mw690/69045600jw1f5m9nockcij20ci09ewf8.jpg)

- 左侧 P 代表 生产者，也就是往 RabbitMQ 发消息的程序。
- 中间即是 RabbitMQ，其中包括了 交换机 和 队列。
- 右侧 C 代表 消费者，也就是往 RabbitMQ 拿消息的程序。

## 虚拟主机，交换机，队列，和绑定
- 虚拟主机：一个虚拟主机持有一组交换机、队列和绑定。为什么需要多个虚拟主机呢？很简单，RabbitMQ当中，用户只能在虚拟主机的粒度进行权限控制。 因此，如果需要禁止A组访问B组的交换机/队列/绑定，必须为A和B分别创建一个虚拟主机。每一个RabbitMQ服务器都有一个默认的虚拟主机“/”。
- 交换机：Exchange 用于转发消息，但是它不会做存储 ，如果没有 Queue bind 到 Exchange 的话，它会直接丢弃掉 Producer 发送过来的消息。常用交换机又分为3种- - 类型：Direct Exchange，Topic Exchange，Fanout Exchange。
- 绑定：也就是交换机需要和队列相绑定，这其中如上图所示，是多对多的关系。

## Direct Exchange
主要根据 路由键 来分发消息到不同的队列中。路由键是消息的一个属性，由生产者加在消息头中。这里首先会有一个 binding key 的概念
![](http://ww3.sinaimg.cn/mw690/69045600jw1f5m9nor3nhj20m5084q3x.jpg))
![](http://ww2.sinaimg.cn/mw690/69045600gw1f5m9u0975bj20bc04raaa.jpg)
第一个 X - Q1 就有一个 binding key，名字为 orange； X - Q2 就有 2 个 binding key，名字为 black 和 green。当消息中的 路由键 和 这个 binding key 对应上的时候，那么就知道了该消息去到哪一个队列中

## Topic Exchange
Topic Exchange 转发消息主要是根据通配符。 在这种交换机下，队列和交换机的绑定会定义一种路由模式，那么，通配符就要在这种路由模式和路由键之间匹配后交换机才能转发消息。
在这种交换机模式下：
 
- 路由键必须是一串字符，用句号（.） 隔开，比如说 agreements.us，或者 agreements.eu.stockholm 等。
- 路由模式必须包含一个 星号（*），主要用于匹配路由键指定位置的一个单词，比如说，一个路由模式是这样子：agreements.*.b.*，那么就只能匹配路由键是这样子的：第一个单词是 agreements，第三个单词是 b。 井号（#）就表示相当于一个或者多个单词，例如一个匹配模式是agreements.eu.berlin.#，那么，以agreements.eu.berlin开头的路由键都是可以的。
![](http://ww1.sinaimg.cn/mw690/69045600jw1f5m9nnwkahj20cc08zgn3.jpg)

## Fanout Exchange
Fanout Exchange 不管路由键或者是路由模式，会把消息发给绑定给它的全部队列。如果不同的consumer需要对同样的消息进行不同的处理，那么这种方式是很有用的。
![](http://ww3.sinaimg.cn/mw690/69045600jw1f5m9np2r68j20cl08mjs1.jpg)


