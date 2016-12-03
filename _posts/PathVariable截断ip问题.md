---
title: '@PathVariable截断ip问题'
date: 2016-12-03 14:19:51
categories:
tags:
---
# 问题
路径传参ip被截断

@PathVariable出现点号&quot;.&quot;时导致路径参数截断获取不全的解决办法
SpringMVC项目中通过下面的ＵＲＬ进行GET请求，ip地址包含多个.号，192.168.1.1 会截为 192.168.1 丢失部分

在@RequestMapping的value中使用SpEL来表示，value中的{ip}换成{ip:.+}。


