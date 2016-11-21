---
title: BlackNurse
date: 2016-11-21 15:41:06
categories: DDos
tags: DDos
---
![](http://ww3.sinaimg.cn/mw690/69045600gw1f9ty12mkgdj20p00dwt9b.jpg)
BlackNurse的简单攻击方式，能够让独立入侵者能用有限的资源（一个有15Mbps带宽的笔记本）驱动大规模DDoS攻击，直接将大型服务器踢下线。
发现BlackNurse攻击的是一个家叫做TDC的丹麦安全运营中心，据说BlackNurse能够攻击大型防火墙保护下的服务器，包括Cisco Systems, 
Palo Alto Networks, SonicWall和Zyxel的防火墙

<!--more-->
将这种攻击方式命名为BlackNurse，它不是仅仅建立在网络连接上的单纯ICMP（控制报文协议Internet Control Message Protocol）泛洪攻击，
要知道传统的ICMP泛洪攻击是通过高频向目标发送ICMP请求来实现的，而BlackNurse攻击则是基于ICMP Type3 Code3的包，而这种包通常被路由器
和网络设备用来发送和接受错误信息。”
 
ICMP是TCP/IP的一个子协议，大部分常见的ICMP攻击都是基于Type8 Code0的，即泛洪攻击。Type8 Code0是Echo request——回显请求（Ping请求），
Ping的原理是向网络上的另一个主机系统发送ICMP报文请求，如果指定系统获得报文，它会回送应答报文，这类似潜水艇声纳系统中使用的发声装置。
 
而BlackNurse攻击基于Type3（Destination Unreachable） Code3（Port Unreachable）——端口不可达，当目标端口不可达，所发出的ICMP
包都会返回源。攻击者可以通过发这种特定的ICMP包令大多数服务器防火墙的CPU过载。一旦设备抛弃的包到了临界值15Mbps至18Mbps（每秒4万到
5万个包），服务器就会直接下线

BlackNurse攻击能吸引我们注意力的主要原因是它能够以这样低的频率源源不断地攻击目标，在我们所知的所有抗DDoS方案中，如此低的流量和每秒发
包数也是十分罕见的。BlackNurse攻击甚至对有着海量带宽的企业防火墙也有效。当然我们也期望有专业的防火墙能够应对这种类型的攻击
这种攻击方式跟你有没有1Gbit/s的网络连接没关系，它最大的影响在于令各类防火墙的CPU过载。在遭受攻击时，本地局域网的用户将不能通过网络发送
和接受数据。一旦停止攻击就能看到防火墙恢复功能。

这样低容量的DDoS攻击能够起效的主要原因是它不像泛洪攻击那样以流量取胜，而是增加CPU的负荷，这样即使网站还有很大的流量依然会被踢下线
Netresec的安专家也支持了TDC的分析，确定这种攻击针对一些主要的防火墙制造商，包括Cisco Systems, Palo Alto Networks, SonicWall和Zyxel。
TDC证实容易被BlackNurse攻击的防火墙主要有以下几种型号
```
Cisco ASA 5506, 5515, 5525 (default settings)
 
Cisco ASA 5550 (Legacy) and 5515-X (latest generation)
 
Cisco Router 897 (unless rate-limited)
 
Palo Alto (unverified)
 
SonicWall (if misconfigured)
 
Zyxel NWA3560-N (wireless attack from LAN Side)
 
Zyxel Zywall USG50
```
