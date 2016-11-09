---
title: mysql压缩包精简
date: 2016-11-08 15:23:35
categories: 数据库
tags: mysql
---

# 官网下载mysql
mysql5.7.16下载
```
http://cdn.mysql.com//Downloads/MySQL-5.7/mysql-5.7.16-winx64.zip
http://cdn.mysql.com//Downloads/MySQL-5.7/mysql-5.7.16-winx64.zip
http://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.16-linux-glibc2.5-x86_64.tar
http://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.16-osx10.11-x86_64.tar.gz
```
<!--more-->
# mysql 目录结构
```
bin
docs
include
lib
share
COPYING
my-default.ini
README
```

mysql-5.7.16-winx64.zip 压缩包大小 348M
解压后大小 1.77G
# 精简
- 目录中仅保留bin，data和share 目录，其他目录文件删除，删除data中test数据文件
- bin目录下，保留 mysqld.exe、 mysqladmin.exe、mysql.exe（如果要使用客户端请保留）三个文件
- share 目录下仅保留 charsets 和 english 两个子目录，其他子目录及文件全部删除