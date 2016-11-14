---
title: linux系统用户切换
date: 2016-11-13 11:34:28
categories: linux
tags: 命令
---

# 语法
```
su [-fmp] [-c command] [-s shell] [--help] [--version] [-] [USER [ARG]]
```

# 参数说明
```
-f ， –fast：不必读启动文件（如 csh.cshrc 等），仅用于csh或tcsh两种Shell。
-l ， –login：加了这个参数之后，就好像是重新登陆一样，大部分环境变量(例如HOME、SHELL和USER等)都是以该使用者(USER)为主，并
且工作目录也会改变。如果没有指定USER，缺省情况是root。 
-m， -p ，–preserve-environment：执行su时不改变环境变数。
-c command：变更账号为USER的使用者，并执行指令（command）后再变回原来使用者。
–help 显示说明文件
–version 显示版本资讯
USER：欲变更的使用者账号，
ARG: 传入新的Shell参数。
```
<!--more-->

# su [user] 和 su - [user]的区别
## su [user]切换到其他用户，但是不切换环境变量
用户还在这个目录
```
[root@2fe7d559fffb ~]# pwd
/root
[root@2fe7d559fffb ~]# su liubo
[liubo@2fe7d559fffb root]$ pwd
/root
```
## su - [user]则是完整的切换到新的用户环境
切换到新用户目录
```
[root@2fe7d559fffb ~]# pwd
/root
[root@2fe7d559fffb ~]# su - liubo
Last login: Thu Nov 10 09:43:52 UTC 2016
[liubo@2fe7d559fffb ~]$ pwd
/home/liubo
```
