---
title: netdataLinux性能实时监测工具
date: 2016-11-17 11:03:37
categories: linux
tags: 监控
---
Netdata是一个高度优化的Linux守护进程，它为Linux系统，应用程序，SNMP服务等提供实时的性能监测。
它用可视化的手段，将被监测者最细微的细节，展现了出来。这样，你便可以清晰地了解你的系统和应用程序此时的状况


![](https://cloud.githubusercontent.com/assets/2662304/14253734/536bd4c2-fa95-11e5-8872-81eed5178e4b.gif)
# 地址
https://github.com/firehol/netdata


<!--more-->
# 检测内容
1.CPU的使用率,中断，软中断和频率(总量和每个单核)
 
2.RAM，互换和内核内存的使用率（包括KSM和内核内存deduper）
 
3.硬盘输入/输出(每个硬盘的带宽，操作，整理，利用等)

4.IPv4网络（数据包，错误，分片）：
 
TCP：连接，数据包，错误，握手
 
UDP:数据包，错误
 
广播：带宽，数据包
 
组播：带宽，数据包
 
5.Netfilter/iptables Linux防火墙(连接，连接跟踪事件，错误等)
 
6.进程(运行，受阻，分叉，活动等)
 
7.熵
 
8.NFS文件服务器，v2,v3,v4(输入/输出，缓存，预读，RPC调用)
 
9.网络服务质量（唯一一个可实时可视化网络状况的工具）

10.应用程序，通过对进程树进行分组（CPU,内存，硬盘读取，硬盘写入，交换，线程，管道，套接字等）


