---
title: 白话docker
date: 2016-08-15 12:56:57
categories: 容器
tags: docker
---

集装箱
# vm和docker
Docker 是一种容器技术，它可以将应用和环境等进行打包，形成一个独立的，类似于 iOS 的 APP 形式的「应用」，这个应用可以直接被分发到任意一个支持 Docker 的环境中，通过简单的命令即可启动运行。Docker 是一种最流行的容器化实现方案。和虚拟化技术类似，它极大的方便了应用服务的部署；又与虚拟化技术不同，它以一种更轻量的方式实现了应用服务的打包。使用 Docker 可以让每个应用彼此相互隔离，在同一台机器上同时运行多个应用，不过他们彼此之间共享同一个操作系统。Docker 的优势在于，它可以在更细的粒度上进行资源的管理，也比虚拟化技术更加节约资源。

<!--more-->

![](http://ww1.sinaimg.cn/mw690/69045600gw1f6mc0u81fmj20sz0gk0us.jpg)

镜像：我们可以理解为一个预配置的系统光盘，这个光盘插入电脑后就可以启动一个操作系统。当然由于是光盘，所以你无法修改它或者保存数据，每次重启都是一个原样全新的系统。Docker 里面镜像基本上和这个差不多。
 
容器：同样一个镜像，我们可以同时启动运行多个，运行期间的产生的这个实例就是容器。把容器内的操作和启动它的镜像进行合并，就可以产生一个新的镜像


# 出现之前
docker出现之前，我要部署java应用
1. 安装jdk环境
1. 安装tomcat
1. 安装mysql
1. 安装redis

如果有很多机器，那么。。
如果要换个机器，那么。。

# 出现之后
按照一种格式，把jdk，tomcat，mysql，redis应用和代码封装成image，可以在docker上解析
就像封装成光盘，可以在播放器播放一样

在docker中，镜像是无法直接运行的，一个软件的某个具体版本只会打包成一个镜像。如果镜像可以配置，
运行的话，在使用过程中很可能会对镜像造成破坏。
再加一层，也就是相当于用了分身术，只要本尊没问题，分身怎么扑街都不会真正的跪掉。多加的这一层分身，
就叫容器（container），这个名字也挺形象，它就像个瓶子这样的容器一样，你的应用在里面运行，
而且多了一层安全机制。你想使用服务或把你的应用跑起来的话，只需要使用镜像新创建一个容器就可以了
（也是一条命令搞定），而镜像还放在那里不动，没办法，金贵嘛。

# 镜像分类

## 基础独立镜像
一个独立的服务，最独立的服务镜像，莫过于各种精简的操作系统了。它们被封装在镜像中，作为最基础的服务，
基本上所有镜像都离不开它
比如
```
#基础的操作系统
From ubuntu:latest
#构建其他需要的其他服务，以及我们自己的代码
RUN apt-get install mysql
COPY .  /src/   
```

##  组合镜像

建立在一个独立镜像的基础上，在上面组合了其他服务，比如python服务，或是mysql之类的服务。
作为比较顶层的应用开发者，我们一般会直接使用这种组合好后的服务镜像。
```
# 基础python服务
From python:2.7.10 

# 我们自己的代码
RUN apt-get install mysql

COPY .  /src/
```

其实像上面的python服务，其本身也是建立在一个基础的操作系统之上的。如果我们查看dockerhub
上的python的Dockerfile，就可以明确这一点：
```
FROM buildpack-deps:jessie    # jessie就是一个精简的debian操作系统

# remove several traces of debian python
RUN apt-get purge -y python.*
```
而我们自己构建的镜像，也可以称之为一种组合镜像。我们一般使用Dockerfile将自己的应用代码，
加上上面的某些具体的服务镜像（比如python）再组合起来，就可以构建我们自己的应用了

# Docker 究竟做了什么简化
docker正是在部署过程中，将上面那些重复的部分，由docker自动化完成。只需要在第一次部署时，
构建完可用的docker镜像。然后在以后使用的过程中，短短的几行命令，就可以直接拉取镜像，
根据这个镜像创建出一个容器，把服务跑起来了。所需要的仅仅是安装了docker的服务器，
一个Dockerfile文件，以及比较流畅的网络而已。真可谓，『一次构建，到处部署』。
需要python3环境？直接 from python:3.x 搞定。
需要迁移服务器？ 直接把应用连带Dockerfile，volumes数据拷贝到新服务器上，几条命令又搞定
需要作为服务给别人使用？Dockerfile即是最清晰的部署文档，维护一个官方镜像即可，谁需要就直接拉下来几条命令部署上就行了。

直接给你一个标准docker货件，它可能是Dockerfile，或者直接就是镜像，这个标准件不仅包括了代码本身，
还包括了代码运行的OS等各种整体环境。
于是，谁想用我的服务，直接拉取镜像，实例化一个容器就可以了，能直接提供你所要的服务，
不再像之前那样有繁复的安装过程

# 统一的管理服务

使用docker部署的应用，都会在docker的管理范围之内。这也是docker的另一个非常大的优点，
它提供了一种隔离的空间，把服务器上的零散的部署应用集中起来进行管理


传统的部署模式是：安装(包管理工具或者源码包编译)->配置->运行；
Docker的部署模式是：复制->运行。

1）标准化应用发布，docker容器包含了运行环境和可执行程序，可以跨平台和主机使用；
2）节约时间，快速部署和启动，VM启动一般是分钟级，docker容器启动是秒级；
3）方便构建基于SOA架构或微服务架构的系统，通过服务编排，更好的松耦合；
4）节约成本，以前一个虚拟机至少需要几个G的磁盘空间，docker容器可以减少到MB级；
5）方便持续集成，通过与代码进行关联使持续集成非常方便；
6）可以作为集群系统的轻量主机或节点，在IaaS平台上，已经出现了CaaS，通过容器替代原来的主机。

# 实际使用
# 运行一个容器
使用 Docker 最关键的一步就是从镜像创建容器。有两种方式可以创建一个容器：使用 docker create 命令创建容器，或者使用 docker run 命令运行一个新容器。两个命令并没有太大差别，只是前者创建后并不会立即启动容器
```
docker run -it ubuntu:latest sh -c '/bin/bash'
```
启动一个新容器，将当前终端连接为这个 Ubuntu 的 bash shell
参数 -i 表示这是一个交互容器，会把当前标准输入重定向到容器的标准输入中，而不是终止程序运行，-t 指为这个容器分配一个终端
按 Ctrl+D 可以退出这个容器

在容器运行期间，我们可以通过 docker ps 命令看到所有当前正在运行的容器。添加-a参数可以看到所有创建的容器
```
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
redis               latest              e5181bd07b8e        4 days ago          185 MB
ubuntu              latest              2fa927b5cdd3        10 weeks ago        122 MB
 
hzlbo@yfzx-hzlbo MINGW64 ~
$ docker run -it ubuntu:latest sh -c '/bin/bash'
root@28803fe48d54:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@28803fe48d54:/# ^C
root@28803fe48d54:/# exit
 
hzlbo@yfzx-hzlbo MINGW64 ~
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
 
hzlbo@yfzx-hzlbo MINGW64 ~
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED                  STATUS                       PORTS               NAMES
28803fe48d54        ubuntu:latest       "sh -c /bin/bash"   Less than a second ago   Exited (130) 7 seconds ago                       cranky_lamarr
```
每个容器都有一个唯一的 ID 标识，通过 ID 可以对这个容器进行管理和操作。在创建容器时，我们可以通过 `--name` 参数指定一个容器名称，如果没有指定系统将会分配一个，就像这里的「cranky_lamarr」。

按 Ctrl+D 退出容器时，命令执行完了，所以容器也就退出了。要重新启动这个容器，可以使用 docker start
```
docker start -i cranky_lamarr
```
注意：**每次执行 docker run 命令都会创建新的容器，建议一次创建后，使用 docker start/stop 来启动和停用容器**

# 存储
在 Docker 容器运行期间，对文件系统的所有修改都会以增量的方式反映在容器使用的联合文件系统中，并不是真正的对只读层数据信息修改。每次运行容器对它的修改，都可以理解成对夹心饼干又添加了一层奶油。这层奶油仅供当前容器使用。当删除 Docker 容器，或通过该镜像重新启动时，之前的更改将会丢失。这样做并不便于我们持久化和共享数据。Docker 的数据卷这个东西可以帮到我们。

在创建容器时，通过 -v 参数可以指定将容器内的某个目录作为数据卷加载
```
docker run -it -v /home/www ubuntu:latest sh -c '/bin/bash'
```
在容器中会多一个 /home/www 挂载点，在这个挂载点存储数据会绕过联合文件系统。我们可以通过下面的命令来找到这个数据卷在主机上真正的存储位置：
```
docker inspect -f {{.Volumes}} cranky_lamarr
```
你会看到输出了一个指向到/var/lib/docker/vfs/dir/...的目录。CD 进入后你会发现在容器中对 /home/www 的读写创建，都会反映到这儿。事实上，/home/www 就是挂载自这个位置
有时候，我们需要将本地的文件目录挂载到容器内的位置，同样是使用数据卷这一个特性，-v参数格式为：
```
#host_dir是主机的目录，container_dir是容器的目录。
docker run -it -v [host_dir]:[container_dir]
```
容器和容器之间是可以共享数据卷的，我们可以单独创建一些容器，存放如数据库持久化存储、配置文件一类的东西，然而这些容器并不需要运行
```
docker run --name dbdata ubuntu echo "Data container."
```
在需要使用这个数据容器的容器创建时 --volumes-from [容器名] 的方式来使用这个数据共享容器

# 网络
Docker 容器内的系统工作起来就像是一个虚拟机，容器内开放的端口并不会真正开放在主机上。可以使我们的容器更加安全，而且不会产生容器间端口的争用。想要将 Docker 容器的端口开放到主机上，可以使用类似端口映射的方式
在 Docker 容器创建时，通过指定 -p 参数可以暴露容器的端口在主机上：
```
docker run -it -p 22 ubuntu sh -c '/bin/bash'
```
现在我们就将容器的 22 端口开放在了主机上，注意主机上对应端口是自动分配的。如果想要指定某个端口，可以通过 -p [主机端口]:[容器端口] 参数指定
容器和容器之间想要网络通讯，可以直接使用 --link 参数将两个容器连接起来。例如 WordPress 容器对 some-mysql 的连接：
```
docker run --name some-wordpress --link some-mysql:mysql -p 8080:80 -d wordpress
```
# 环境变量
通过 Docker 打包的应用，对外就像是一个密闭的 exe 可执行文件。有时候我们希望 Docker 能够使用一些外部的参数来使用容器，这时候参数可以通过环境变量传递进去，通常情况下用来传递比如 MySQL 数据库连接这种的东西。环境变量通过 -e 参数向容器传递
```
docker run --name some-wordpress -e WORDPRESS_DB_HOST=10.1.2.3:3306 \
    -e WORDPRESS_DB_USER=... -e WORDPRESS_DB_PASSWORD=... -d wordpress
```

# 创建镜像
Docker 强大的威力在于可以把自己开发的应用随同各种依赖环境一起打包、分发、运行。要创建一个新的 Docker 镜像，通常基于一个已有的 Docker 镜像来创建。Docker 提供了两种方式来创建镜像：把容器创建为一个新的镜像、使用 Dockerfile 创建镜像
## 将容器创建为镜像
为了创建一个新的镜像，我们先创建一个新的容器作为基底
```
docker run -it ubuntu:latest sh -c '/bin/bash'
```
现在我们可以对这个容器进行修改了，例如我们可以配置 PHP 环境、将我们的项目代码部署在里面等
```
apt-get install php
# some other opreations ...
```
当执行完操作之后，我们按 Ctrl+D 退出容器，接下来使用 docker ps -a 来查找我们刚刚创建的容器 ID
```
docker ps -a
```
可以看到我们最后操作的那个 Ubuntu 容器。这时候只需要使用 docker commit 即可把这个容器变为一个镜像了
```
docker commit 28803fe48d54 ubuntu:liubo-ubuntu
```
这时候 docker 容器会被创建为一个新的 Ubuntu 镜像，版本名称为 liuboubuntu。以后我们可以随时使用这个镜像来创建容器了，新的容器将自动包含上面对容器的操作

如果我们要在另外一台机器上使用这个镜像，可以将一个镜像导出
```
docker save -o liuboubuntu.tar.gz ubuntu:liuboubuntu
```
现在我们可以把刚才创建的镜像打包为一个文件分发和迁移了。要在一台机器上导入镜像，只需要
```
docker import liuboubuntu.tar.gz
```
这样在新机器上就拥有了这个镜像。
注意：通过导入导出的方式分发镜像并不是 Docker 的最佳实践，因为我们有 Docker Hub。

## 使用 Dockerfile 创建镜像
使用命令行的方式创建 Docker 镜像通常难以自动化操作。在更多的时候，我们使用 Dockerfile 来创建 Docker 镜像。Dockerfile 是一个纯文本文件，它记载了从一个镜像创建另一个新镜像的步骤。撰写好 Dockerfile 文件之后，我们就可以轻而易举的使用 docker build 命令来创建镜像了


下面是一个 Dockerfile 的例子：
```
FROM ubuntu:14.04
MAINTAINER liubo <917551811@qq.com>
RUN apt-get update && apt-get install -y ***

```
在准备好 Dockerfile 之后，我们就可以创建镜像了
```
docker build -t liubo6/tomcat:server_jre_8
```

新手经常会有疑问的是关于 Docker 打包的粒度，比如 MySQL 要不要放在镜像中？最佳实践是根据应用的规模和可预见的扩展性来确定 Docker 打包的粒度。例如某小型项目管理系统使用 LAMP 环境，由于团队规模和使用人数并不会有太大的变化（可预计的团队规模范围是几人到几千人），数据库也不会承受无法承载的记录数（生命周期内可能一个表最多会有数十万条记录），并且客户最关心的是快速部署使用。那么这时候把 MySQL 作为依赖放在镜像里是一种不错的选择。当然如果你在为一个互联网产品打包，那最好就是把 MySQL 独立出来，因为 MySQL 很可能会单独做优化做集群等。
