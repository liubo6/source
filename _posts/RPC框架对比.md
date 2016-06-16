---
title: RPC框架对比
date: 2016-05-19 16:31:25
tags: RPC
---
## RPC框架比较

|语言|协议|服务治理|社区|机构
---|---|---|---|---|---|---
Hessian|多语言|hessian|-|不活跃|Caucho
Thrift|多语言|thrift|-|活跃|Apache
Finagle|Java/Scala|多协议|支持|活跃|Twitter
TChannel|多语言|thrift|支持|活跃|Uber
Dubbo|Java|支持扩展|支持|停滞|alibaba


<!--more-->

## 详细介绍
### Hassian

>Hassian被广泛用做二进制编码协议，但本身也是一个Web Service框架对RPC调用提供支持，功能简单，使用起来也方便。有些像《RPC》介绍的SimpleRPCFramework，只是把编码协议从Java Serialization替换为hassian。



### Thrift

>Thrift是一个跨语言的RPC框架，自带的代码生成引擎大幅提高了开发效率，从而使它如此流行。

最初由Facebook团队设计开发，现在已贡献给Apache。

Thrift基于接口描述语言定义的Model(契约)，Thrift Compiler进行解析并自动生成目标语言Client和Server端的代码框架，直接基于此框架开发服务或者与现有系统集成都可以较方便实现。

基于Thrift开发远程服务调用，后续Model的改动会导致上下游产生较大升级成本，因此服务双方应先仔细定义Model。

公司内部所有Model应由专门的系统进行统一版本管理、编译、发布。



### Finagle

>Finagle是一个专门为Java/Scala设计的、成熟的RPC框架。

简单、高效、支持基本服务治理功能，所有服务治理功能均在Client端集成：服务发现、负载均衡、容错、日志收集等。

使用Scala开发，国内没有听说过有团队使用此框架。



### TChannel

>TChannel是一个支持多语言的RPC框架，主要编码协议为thrift，同时又参考Finagle基于Dapper设计自己的集群监控实现。

服务治理与Finagle不同，由类似系统总线的独立的路由系统进行服务治理，本身这种系统结构存在单点故障风险，已经被多数公司放弃。但优点显而易见，功能集中在一个系统中，对于系统维护和升级要方便得多，极大降低降低各语言终端的开发成本和框架升级引入的潜在风险。



### Dubbo

>Dubbo是阿里巴巴开源的专门为Java设计的、成熟的RPC框架。

支持基本的服务治理，所有服务治理功能均在Client端集成：服务发现、负载均衡、容错、监控等。

采用微内核+插件体系，所有组件都可以遵循一定的规则进行替换，拥有较强的拓展性。

主流的搭配： 

- zookeeper作服务中心 

- thrift/dubbo作为编码协议

- netty作为底层通信模块

Dubbo定义了自己的spring schema，与spring项目进行集成相当方便。

如果是Java系统推荐使用Dubbo，如果是多语言系统可以为Dubbo添加thrift原生协议支持（Dubbo本身不支持原生的thrift协议），通过thrift代码生成引擎支持多语言调用。

