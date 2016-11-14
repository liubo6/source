---
title: centos搭建L2TP
date: 2016-11-12 16:52:49
categories: 阿里云
tags: vpn
---
# 检查是否支持TUN
```
cat /dev/net/tun
```
如果返回信息为：cat: /dev/net/tun: File descriptor in bad state 说明正常

检测是否支持ppp模块

<!--more-->
```
# cat /dev/ppp
```
如果返回信息为：cat: /dev/ppp: No such device or address 说明正常

# 运行
```
wget --no-check-certificate https://raw.githubusercontent.com/teddysun/across/master/l2tp.sh
chmod +x l2tp.sh
./l2tp.sh

```

```
ServerIP:47.88.24.205
PSK:liubo.loan
Username:liubo_vpn
Password:liubo_vpn_pwd
```

mac 下连接必须选中，通过vpn连接发送所有流量

