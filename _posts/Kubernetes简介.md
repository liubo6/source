---
title: Kubernetes简介
date: 2016-08-15 13:20:48
categories: 容器
tags: docker
---
# Google 容器化之路
谷歌从2000年初开始使用容器，但是它所使用的是自研的一种叫做lmctfy的容器格式，其实是Let Me ContainThat For You几个单词首字母的缩写。谷歌最早使用容器的初衷之一是节省物理资源，通过用容器取代虚拟化层（hypervisor和每个虚拟机所占用的物理资源）来极大地节省计算成本谷歌在2014年将精力转入了开源容器集群管理技术，即[Kubernetes](https://github.com/kubernetes/kubernetes)
谷歌在2015年已经达到了80多个数据中心，超过一百万台机器。其中大家所熟知的search，youtube，maps，gmail等所有产品，包括很多我们使用的内部工具，都是基于容器在运行。这个容器的数量是巨大的，官方称之为每周运行20亿个容器.
<!--more-->

# Kubernetes扫盲
Kubernetes(经常被缩写成 k8s)是Google开源的容器集群管理系统。它构建在Docker之上，为容器话的应用提供资源调度、部署运行、服务发现、
扩容缩容等一整套功能，本质上可以看做是基于容器技术的Micro-PaaS平台，是第三代PaaS技术的代表之作

# 概念
Kubernetes是Google基于内部Borg开源的容器调度系统，被称为OpenStack拥抱容器的实质性一步的Magnum
项目就是使用Kubernetes来调度的Kubernetes是Google开源的容器调度系统，众所周知Google十几年前已经在生产环境使用容器技术，
并且向Linux社区贡献出Cgroups、Namespaces这样的容器核心代码。经过这些年的实践，加上近年来Docker的迅速推广，
Google启动了新项目来重写内部容器调度系统Borg，这就是Kubernetes。Kubernetes 1.3发布，支持跨集群联合服务和有状态服务
# Kubernetes的核心概念
## Pod
Pod是若干相关容器的组合，Pod包含的容器运行在同一台宿主机上，这些容器使用相同的网络命名空间、IP地址和端口，相互之间能通过localhost来发现和通信。这些容器还可以共享一块存储卷空间。在Kubernetes中创建、调度和管理的最小单元就是Pod，而不是容器，Pod通过更高层次的抽象，提供了更加灵活的部署和管理模式。
 
## Replication Controller
Replication Controller用来控制管理Pod副本，它确保任何时候Kubernetes集群中有指定数量的Pod副本在运行。如果少于指定数量的Pod副本，它会启动新的Pod副本，反之它会杀死多余的副本以保证数量不变。而且，Replication Controller是弹性伸缩、滚动升级的实现核心
 
## Service
Service是真实应用服务的抽象，定义了Pod的逻辑集合和访问这个Pod集合的策略。Service将代理Pod对外表现为一个单一的访问接口，外部不需要了解后端Pod是如何运行的，这给扩展和维护带来很多好处，提供了一套简化的服务代理和发现机制
 
## Label
Label是用于区分Pod、Service、Replication Controller和Key/Value对，实际上，Kubernetes中的任意API对象都可以通过Label进行标识。每个API对象可以有多个Label，但是每个Label的Key只能对应一个Value。Label是Service和Replication Controller运行的基础，他们都通过Label来关联Pod，相比于强绑定模型，这是一种非常好的松耦合关系。
 
## Node
Kubernetes属于主从分布式集群架构，Kubernetes Node运行并管理容器。Node作为Kubernetes的操作单元，用来分配给Pod进行绑定，Pod最终运行在Node上，Node可以认为是Pod的宿主机

# Kubernetes的一些理念
1. 用户不需要关心需要多少台机器，只需要关心软件（服务）运行所需的环境。 
1. 以服务为中心，你需要关心的是api，如何把大服务拆分成小服务，如何使用api去整合它们。 
1. 保证系统总是按照用户指定的状态去运行。 
1. 不仅仅提给你供容器服务，同样提供一种软件系统升级的方式；在保持HA的前提下去升级系统是很多用户最想要的功能，也是最难实现的。 
1. 那些需要担心和不需要担心的事情。
1. 更好的支持微服务理念，划分、细分服务之间的边界，比如lablel、pod等概念的引入。

# Kubernetes的主要特性
网络、服务发现、负载均衡、资源管理、高可用、存储、安全、监控等
容器的自动化部署
自动化扩展或者缩容
自动化应用/服务升级
容器成组，对外提供服务，支持负载均衡
服务的健康检查，自动重启

## 网络
Kubernetes的网络方式主要解决以下几个问题：
a. 紧耦合的容器之间通信，通过 Pod 和 localhost 访问解决。 
b. Pod之间通信，建立通信子网，比如隧道、路由，Flannel、Open vSwitch、Weave。 
c. Pod和Service，以及外部系统和Service的通信，引入Service解决。

Kubernetes的网络会给每个Pod分配一个IP地址，不需要在Pod之间建立链接，也基本不需要去处理容器和主机之间的端口映射。
注意：Pod重建后，IP会被重新分配，所以内网通信不要依赖Pod IP；通过Service环境变量或者DNS解决。

## 服务发现及负载均衡
kube-proxy和DNS， 在v1之前，Service含有字段portalip 和publicIPs， 分别指定了服务的虚拟ip和服务的出口机ip，
publicIPs可任意指定成集群中任意包含kube-proxy的节点，可多个。portalIp 通过NAT的方式跳转到container的内网地址。
在v1版本中，publicIPS被约定废除，标记为deprecatedPublicIPs，仅用作向后兼容，portalIp也改为ClusterIp, 
而在service port 定义列表里，增加了nodePort项，即对应node上映射的服务端口。
DNS服务以addon的方式，需要安装skydns和kube2dns。kube2dns会通过读取Kubernetes API获取服务的clusterIP和port信息，
同时以watch的方式检查service的变动，及时收集变动信息，并将对于的ip信息提交给etcd存档，而skydns通过etcd内的DNS记录信息，
开启53端口对外提供服务。大概的DNS的域名记录是servicename.namespace.tenx.domain, "tenx.domain"是提前设置的主域名。

注意：kube-proxy 在集群规模较大以后，可能会有访问的性能问题，可以考虑用其他方式替换，比如HAProxy，
直接导流到Service 的endpints 或者 Pods上。Kubernetes官方也在修复这个问题。

## 资源管理
有3 个层次的资源限制方式，分别在Container、Pod、Namespace 层次。Container层次主要利用容器本身的支持，
比如Docker 对CPU、内存、磁盘、网络等的支持；Pod方面可以限制系统内创建Pod的资源范围，比如最大或者最小的CPU、
memory需求；Namespace层次就是对用户级别的资源限额了，包括CPU、内存，还可以限定Pod、rc、service的数量。

资源管理模型 －》 简单、通用、准确，并可扩展
目前的资源分配计算也相对简单，没有什么资源抢占之类的强大功能，通过每个节点上的资源总量、以及已经使用的各种资源
加权和，来计算某个Pod优先非配到哪些节点，还没有加入对节点实际可用资源的评估，需要自己的scheduler plugin来支持。
其实kubelet已经可以拿到节点的资源，只要进行收集计算即可，相信Kubernetes的后续版本会有支持。

## 高可用
主要是指Master节点的 HA方式 官方推荐 利用etcd实现master 选举，从多个Master中得到一个kube-apiserver 
保证至少有一个master可用，实现high availability。对外以loadbalancer的方式提供入口。这种方式可以用作ha，
但仍未成熟，据了解，未来会更新升级ha的功能。

## 存储
容器本身一般不会对数据进行持久化处理，在Kubernetes中，容器异常退出，kubelet也只是简单的基于原有镜像
重启一个新的容器。另外，如果我们在同一个Pod中运行多个容器，经常会需要在这些容器之间进行共享一些数据。
Kuberenetes 的 Volume就是主要来解决上面两个基础问题的。

Docker 也有Volume的概念，但是相对简单，而且目前的支持很有限，Kubernetes对Volume则有着清晰定义和广泛的支持。
其中最核心的理念：Volume只是一个目录，并可以被在同一个Pod中的所有容器访问。而这个目录会是什么样，
后端用什么介质和里面的内容则由使用的特定Volume类型决定

## 安全
一些主要原则： 基础设施模块应该通过API server交换数据、修改系统状态，而且只有API server可以访问后端存储（etcd）。
 把用户分为不同的角色：Developers/Project Admins/Administrators。 允许Developers定义secrets 对象，并在pod启动时关联到相关容器。

以secret 为例，如果kubelet要去pull 私有镜像，那么Kubernetes支持以下方式： 通过docker login 生成 .dockercfg 文件，
进行全局授权。 通过在每个namespace上创建用户的secret对象，在创建Pod时指定 imagePullSecrets 属性（也可以统一设置在serviceAcouunt 上），进行授权。

认证 （Authentication） API server 支持证书、token、和基本信息三种认证方式。

授权 （Authorization） 通过apiserver的安全端口，authorization会应用到所有http的请求上 AlwaysDeny、AlwaysAllow、
ABAC三种模式，其他需求可以自己实现Authorizer接口。

## 监控
Kubernetes集群范围内的监控主要由kubelet、heapster和storage backend（如influxdb）构建。
Heapster可以在集群范围获取metrics和事件数据。它可以以pod的方式运行在k8s平台里，也可以单独运行以standalone的方式。

# 具体实践使用
使用Docker可实现对环境和资源的隔离,采用的是Docker＋Kubernetes＋Jenkins的技术方案实现CI/CD，团队的开发、测试和生产环境的部署工作都
Docker下进行。该方案实现PB级数据处理，帮助了新业务的技术方案落地.

##技术栈
Java + MySQL + Redis + ELK
持续集成和持续部署方案 GitLab + Jenkins + Docker + Kubernetes
![](http://ww3.sinaimg.cn/mw690/69045600gw1f6t1rjhsgvj20ku0frgmr.jpg)

## 工作流程
首先，开发人员提交代码代码提交；随后，GitLab 会自动触发Jenkins job，Jenkins job
会构建相应的镜像，放在一个Kubernetes的Pod里面；接下来，Kubernetes的Pod会把模块需要的其他依赖都包含
在其内部（比如MySQL、Redis、MongoDB等），运行robot测试用例，测试用例的结果最后会反馈到Jenkins中；
所有测试通过之后，GitLab把代码Merge到Master分支，然后触发部署，构建生产环境所需的Docker镜像并上传
到镜像仓库。最后，生产环境的Kubernetes拉取最新的镜像做更新.
