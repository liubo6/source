---
title: Docker_04_安全
date: 2016-08-12 17:31:01
categories: 容器
tags: docker
---
Docker是LXC的延伸，它也很容易使用LXC的安全特性
`docker run`命令可以用来运行容器,执行后发生了以下任务
<!--more-->

docker run命令初始化。
Docker 运行 lxc-start 来执行run命令。
lxc-start 在容器中创建了一组namespace和Control Groups。

namespace是隔离的第一级，容器是相互隔离的，一个容器是看不到其它容器内部运行的进程情况,每个容器都分配了单独的网络栈，因此一个容器不可能访问另一容器的sockets。为了支持容器之间的IP通信，您必须指定容器的公网IP端口。

Control Groups是非常重要的组件，具有以下功能：
负责资源核算和限制。
提供CPU、内存、I/O和网络相关的指标。
避免某种DoS攻击。
支持多租户平台
```

## Docker Daemon的攻击面
Docker Daemon以root权限运行，这意味着有一些问题需要格外小心。
下面介绍一些需要注意的地方：
当Docker允许与访客容器目录共享而不限制其访问权限时，Docker Daemon的控制权应该只给授权用户。
REST API支持Unix sockets，从而防止了cross-site-scripting攻击。
REST API的HTTP接口应该在可信网络或者VPN下使用。
在服务器上单独运行Docker时，需要与其它服务隔离

一些关键的Docker安全特性包括：
容器以非特权用户运行。
Apparmor、SELinux、GRSEC解决方案，可用于额外的安全层。
可以使用其它容器系统的安全功能

## Docker.io API
用于管理与授权和安全相关的几个进程，Docker提供REST API

## 容器状态
![](http://ww2.sinaimg.cn/mw690/69045600gw1f6ix4t2ml8j20vd0fzgng.jpg)

```

## 开发环境
开发环境代码一直在变动，而且多人通过git协作，于是代码都是放在外面，构建一个运行环境的image，然后代码部分用volume映射进去，方便随时调整
-v是数据卷，可以主机和容器之间共享/home/Docker/workspace这个目录，在容器里面就是/opt/tomcat8/webapps。其实这就是我们的代码目录，调试代码就是靠共享目录实现的，很方便
```
docker run -it -p 8080:8080 liu/jdk8-tomcat8 /bin/bash 看看是否有错误
```
进入容器,启动tomcat
```
docker run -it -p 8080:8080 --name web liubo/tomcat8:v2

外部访问 http://localhost:8080/
```
项目在IDE中经常修改，所以可以将打包项目的目录直接挂载到容器内 tomcat 的 webapps 目录下面，启动容器是就会自动启动应用
```
docker run --name liubo-tomcat-01  -d -p 8080:8080 -v /opt/funds/target:/opt/tomcat8/webapps liu/jdk8-tomcat8
```
--name  设置容器名称
-p 外端口:容器内端口
-v 挂载数据卷

## 生产环境
代码都构建在image里面，这样直接在客户那边把镜像运行起来就行

或者和开发环境一样也可以

版本稳定的服务可以放到image

使用在docker run的时候使用--restart参数来设置。
no 不重启
on-failure  退出状态非0时重启
always:始终重启

开启启动时候,容器是否需要自动启动
