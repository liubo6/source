---
title: 美西aliyun.centos 安装vpn
date: 2016-11-11 10:00:59
categories: 阿里云
tags: vpn
---

# 美西阿里云vpn

##  环境检查

```
linux内核版本 等于或高于 2.6.15 ，内核集成了MPPE
#modprobe ppp-compress-18 && echo ok
结果为ok则通过

#cat /dev/net/tun
结果为 File descriptor in bad state ，则通过
Centos 6.4内核版本在2.6.15以上，都默认集成了MPPE和PPP
```

## 安装ppp
```
yum install -y perl ppp iptables pptpd
```
<!--more-->

##  修改配置文件
```
vim /etc/ppp/options.pptpd
末尾添加
ms-dns 8.8.8.8
ms-dns 8.8.4.4
```

```
vim  /etc/ppp/chap-secrets
liubo_vpn  pptpd  liubo_vpn_pwd *
#用户名，协议，密码 ，任何ip
```

```
vim /etc/pptpd.conf
取消注释
localip 192.168.0.1
remoteip 192.168.0.234-238,192.168.0.245
```

```
vim /etc/sysctl.conf
将net.ipv4.ip_forward = 0 改成 net.ipv4.ip_forward = 1
#保存
/sbin/sysctl -p

```

```
/sbin/service pptpd start
或者service pptpd start
```
我们的VPN已经可以拨号登录，但是还不能访问任何网页
47.88.18.25 （自己真实地址）
## 启动iptables和nat转发功能
```
/sbin/service iptables start
iptables -t nat -A POSTROUTING    -s 192.168.0.0/24 -j SNAT --to-source  47.88.25.215
#保存iptables的转发规则
/etc/init.d/iptables save
#重新启动iptables
/sbin/service iptables restart 
```

```
service pptpd restart
```

