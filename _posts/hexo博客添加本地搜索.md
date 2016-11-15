---
title: hexo博客添加本地搜索
date: 2016-11-15 14:14:40
categories: 博客
tags: hexo
---
# 安装LocalSearch
```
npm install hexo-generator-search --save
```

# 启用LocalSearch
修改主站点配置文件_config.yml添加以下代码
```
search:
  path: search.xml
  field: post
```
ok 尝试下

<!--more-->

