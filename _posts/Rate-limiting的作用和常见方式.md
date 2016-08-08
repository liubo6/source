---
title: Rate_limiting的作用和常见方式
date: 2016-08-08 15:30:19
categories: 接口限制
tags: 缓存
---
Rate limiting 在 Web 架构中非常重要，是互联网架构可靠性保证之一
从整个架构的稳定性角度看，一般 SOA 架构的每个接口的有限资源的情况下，所能提供的单位时间服务能力是有限的。假如超过服务能力，一般会造成整个接口服务停顿，或者应用 Crash，或者带来连锁反应，将延迟传递给服务调用方造成整个系统的服务能力丧失。有必要在服务能力超限的情况下 Fail Fast。

<!--more-->

根据排队论，由于 API 接口服务具有延迟随着请求量提升迅速提升的特点，为了保证 SLA 的低延迟，需要控制单位时间的请求量

![](http://ww4.sinaimg.cn/mw690/69045600gw1f6ipxwtfajj20h40c3tac.jpg)

公开 API 接口服务，Rate limiting 应该是一个必备的功能，否则公开的接口不知道哪一天就会被服务调用方有意无意的打垮。
提供资源能够支撑的服务，将过载请求快速抛弃对整个系统架构的稳定性非常重要。这就要求在应用层实现 Rate limiting 限制。

# Rate limiting 的实现方式

## Proxy 层的实现，针对部分 URL 或者 API 接口进行访问频率限制
### nginx模块
```
====语法====
limit_req_zone
语法: limit_req_zone $variable zone=name:size rate=rate;
默认值: none
配置段: http
====语法====
#用法
limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;
server {
    location /search/ {
        limit_req zone=one burst=5;
    }
#说明：区域名称为one，大小为10m，平均处理的请求频率不能超过每秒一次
```
