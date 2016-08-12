---
title: Docker_01_介绍
date: 2016-08-9 09:24:23
categories: 容器
tags: docker
---
Docker是一个新的容器化的技术,它轻巧,且易移植，号称"build once, configure once and run anywhere"

# 特性
- 速度飞快以及优雅的隔离框架
- 物美价廉
- CPU/内存的低消耗
- 快速开/关机
- 跨云计算基础构架
<!--more-->

# docker和虚拟机区别
![](http://ww1.sinaimg.cn/mw690/69045600gw1f6mc0u81fmj20sz0gk0us.jpg)

# Docker组件与元素
## 三个组件
- `Docker Client` 是用户界面，它支持用户与`Docker Daemon`之间通信
- `Docker Daemon`运行于主机上，处理服务请求
- `Docker Index`是中央registry，支持拥有公有与私有访问权限的Docker容器镜像的备份

## 三个元素
- `Docker Containers`负责应用程序的运行，包括操作系统、用户添加的文件以及元数据
- `Docker Images`是一个只读模板，用来运行Docker容器
- `DockerFile`是文件指令集，用来说明如何自动创建Docker镜像
![](http://ww2.sinaimg.cn/mw690/69045600gw1f6isg373ynj20ld0c1t96.jpg)

Docker使用以下操作系统的功能来提高容器技术效率
- `Namespaces` 充当隔离的第一级。确保一个容器中运行一个进程而且不能看到或影响容器外的其它进程
- `Control Groups`是LXC的重要组成部分，具有资源核算与限制的关键功能
- `UnionFS`（文件系统）作为容器的构建块。为了支持Docker的轻量级以及速度快的特性，它创建了用户层

# 运行程序
## 构建镜像
`Docker Image`是一个构建容器的只读模板，它包含了容器启动所需的所有信息，包括**运行程序和配置数据**。
每个镜像都源于一个基本的镜像，然后根据Dockerfile中的指令创建模板
一旦镜像创建完成，就可以将它们推送到中央`registry：Docker Index`，以供他人使用。然而，`Docker Index`为镜像提供了两个级别的访问权限：公有访问和私有访问。你可以将镜像存储在私有仓库，Docker官网有私有仓库的套餐可以供你选择。总之，公有仓库是可搜索和可重复使用的，而私有仓库只能给那些拥有访问权限的成员使用。`Docker Client`可用于`Docker Index`内的镜像搜索

## 运行容器
运行容器源于我们在第一步中创建的镜像。当容器被启动后，一个读写层会被添加到镜像的顶层。当分配到合适的网络和IP地址后，需要的应用程序就可以在容器中运行了。

# 安装Docker
## Ubuntu16.04
1. 查看内核版本
```
liubo@ubuntu:~$ uanme -r
4.4.0-22-generic
```
### 检查APT系统的HTTPS兼容性
sudo apt-get install apt-transport-https ca-certificates 
### 在本地添加Docker Repo密钥
sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D 

### 添加Docker Repository到APT源列表。 
```
sudo vim  /etc/apt/sources.list.d/docker.list
添加上
deb https://apt.dockerproject.org/repo ubuntu-xenial main
添加成功后
sudo apt-get update
```
### 安装docker-engine程序包。
```
apt-get install -y linux-image-extra-$(uname -r)
sudo apt-get install docker-engine
```
### 验证所安装的内容
```
sudo docker run -i -t ubuntu /bin/bash
exit //退出
```

