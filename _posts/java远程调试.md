---
title: java远程调试
date: 2016-10-22 19:42:09
categories: debug
tags: java
---
# 原理
>Java远程调试的原理是两个VM之间通过debug协议进行通信，然后以达到远程调试的目的。两者之间可以通过socket进行通信。
首先被debug程序的虚拟机在启动时要开启debug模式，启动debug监听程序。jdwp是Java Debug Wire Protocol的缩写。


<!--more-->
# 使用
## DEBUG选项参数
```
-XDebug 启用调试；
-Xrunjdwp 加载JDWP的JPDA参考执行实例；
transport 用于在调试程序和VM使用的进程之间通讯；
dt_socket 套接字传输；
server=y/n VM是否需要作为调试服务器执行；
address=56791 调试服务器监听的端口号；
suspend=y/n 是否在调试客户端建立连接之后启动 VM 
```
## jdk1.3.x 或者更早
```
-Xnoagent -Djava.compiler=NONE -Xdebug 
-Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=5005
```

## jdk1.4.x
```
-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=5005
```
## jdk1.7之后
```
-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005
```

## 普通java应用
服务器端
```
java -agentlib:jdwp=transport=dt_socket,address=5005,suspend=n,server=y  -jar demo-liubo.jar  
```

本地 IDEA
RUN->Edit Configurations->+Romote->
选择服务器配置
host 设置服务器的ip
port设置服务器配置的56791端口
选择应用demo-liubo
ok

## Tomcat应用
服务器端
apach/bin/startup.sh开始处中增加如下内容
```
JAVA_OPTS="$JAVA_OPTS -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=5005”  
```
客户端同上

还有一种
JPDA(Java Platform Debugger Architecture) 是 Java 平台调试体系结构的缩写，通过 JPDA 提供的 API，
开发人员可以方便灵活的搭建 Java 调试应用程序。JPDA 主要由三个部分组成：Java 虚拟机工具接口（JVMTI），
Java 调试线协议（JDWP），以及 Java 调试接口（JDI）
而像Eclipse和IDEA这种开发工具提供的图形界面的调试工具，其实就是实现了JDI

tomcat使用jpda
启动jpda
```
./catalina.sh jpda start
```
默认情况下，远程调试的默认端口为8000，可以通过JPDA_ADDRESS进行配置，指定自定义的端口，
另外，还有两个可以配置的参数
JPDA_TRANSPORT：即调试器和虚拟机之间数据的传输方式，默认值是dt_socket
JPDA_SUSPEND：即JVM启动后是否立即挂起，默认是n
可以在catalina.sh中进行配置
```
JPDA_TRANSPORT=dt_socket  
JPDA_ADDRESS=5005  
JPAD_SUSPEND=n 
```
或者通过JPDA_OPTS进行配置
```
JPDA_OPTS='-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005’  
```
这样启动之后 ，就可以通过Eclise或IDEA进行远程调试了










