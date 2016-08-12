---
title: Docker_02_命令
date: 2016-08-10 13:51:03
categories: 容器
tags: docker
---
手动创建镜像时会用到创建、提交、搜索、pull和push的功能
## 显示docker信息
```
docker info
检查Docker的安装是否正确

```

<!--more-->

## 查看所有镜像的列表
```
docker images
```

## 查找镜像
```
docker search (image-name)
```


## 拉取镜像 
```
docker pull busybox 
```

## 运行镜像
```
#打印"Hello Docker"
docker run busybox /bin/echo Hello Docker
```

## 停止容器
```
docker stop name
```

## 重启容器
```
docker restart containerName
```

## 移除容器
```
docker stop name //停止容器
docker rm name//删除容器
```

## 将容器的状态保存为镜像
```
#镜像名称只能取字符[a-z]和数字[0-9]
docker commit job job1
```

## 查看镜像的历史版本
```
docker history (image_name)
```

## 将镜像推送到registry
```
docker push (image_name)
```

存储库不是根存储库，它应该使用此格式`(user)/(repo_name)`

## 后台进程运行
Docker daemon是一个用于管理容器的后台进程。一般情况下，守护进程是一个长期运行的用来处理请求的进程服务。-d参数用于运行后台进程

## 构建镜像
`build` 可以使用Dockerfile来构建镜像。简单的构建命令
```
docker build [options] PATH | URL

还有一些Docker提供的额外选项，如：
--rm=true表示构建成功后，移除所有中间容器
--no-cache=false表示在构建过程中不使用缓存
```

## 容器交互
Docker允许使用~~attach~~命令与运行中的容器交互，并且可以随时观察容器內进程的运行状况。退出容器可以通过两种方式来完成：
Ctrl+C 直接退出
Ctrl-\ 退出并显示堆栈信息（stack trace）

docker exec java /bin/bash 代替
语法
```
~~docker attach container~~
```



## 展示
Docker提供了一个非常强大的命令diff，它可以列出容器内发生变化的文件和目录。这些变化包括添加（A-add）、删除（D-delete）、修改（C-change）。该命令便于Debug，并支持快速的共享环境。
语法
```
docker diff container
```

## 事件
events 打印指定时间内的容器的实时系统事件

## 导入
Docker可以导入远程文件、本地文件和目录。使用HTTP的URL从远程位置导入，而本地文件或目录的导入需要使用-参数。从远程位置导入的语法
```
docker import http://example.com/example.tar
```
本地导入
```
docker import - image.txt
```

## 导出
export命令用于将容器的系统文件打包成tar文件

## 复制
从容器内复制文件到指定的路径上
语法
```
docker cp container:path hostpath.
```

## 登录
登录到Docker registry服务器
语法
```
docker login [options] [server]
```
如要登录自己主机的registry请使用
```
docker login localhost:8080
```
## 查看信息
Docker inspect命令可以收集有关容器和镜像的底层信息。这些信息包括：
容器实例的IP地址
端口绑定列表
特定端口映射的搜索
收集配置的详细信息
语法
```
docker inspect container/image
```

## 杀死进程
发送SIGKILL信号来停止容器的主进程。语法是：
docker kill [options] container

## 删除镜像
该命令可以移除一个或者多个镜像，语法如下：
docker rmi image
镜像可以有多个标签链接到它。在删除镜像时，你应该确保删除所有相关的标签以避免错误

## 阻塞
阻塞对指定容器的其它调用方法，直到容器停止后退出阻塞

## 加载
该命令从tar文件中载入镜像或仓库到STDIN

## 保存
类似于load，该命令保存镜像为tar文件并发送到STDOUT。语法
```
docker save image
```

