---
title: mysql存储emoji表情问题
date: 2016-07-13 18:19:25
categories: 数据库
tags: mysql
---
## 前言
在Android手机或者iPhone的各种输入法键盘中，会自带一些Emoji表情符号
![](http://ww3.sinaimg.cn/mw690/69045600gw1f5s2l401lsj20e00ao451.jpg) 
文本内容时包含了这种Emoji表情符号，通过接口传递到服务器端，服务器端再存入MySQL数据库
<!--more-->
- gbk 字符集的数据库，写入数据库的数据，在回显时，变成 ‘口口’ 无法回显
- utf8 字符集的数据库，则根本无法写入数据库，程序直接报出异常信息 ```java.sql.SQLException: Incorrect string value: '\xF0\x9F\x92\x94' for column 'name' at row```

## 原因
unicode emoji是4个字节的，utf8 字符集只支持1-3个字节的字符，存不进MySQL里

## 解决方案
- 遍历输入的文本，把四字节长度的字符，修正为自定义的字符替换掉
- 数据库字符集从utf8 修改为支持1-4 个字节字符的utf8mb4，数据库连接也需要改为utf8mb4
>修改数据库字符集character-set-server=utf8mb4 重启数据库生效
修改database 的字符集为 utf8mb4 alter database dbname character set utf8mb4
修改表的字符集 为utf8mb4 ， alter table character set = utf8mb4
```
- 修改my.cnf
[mysqld]
character-set-server=utf8mb4
[mysql]
default-character-set=utf8mb4
重启Mysql

- 修改环境变量
set character_set_client = utf8mb4;
set character_set_connection = utf8mb4;
set character_set_database = utf8mb4;
set character_set_results = utf8mb4;
set character_set_server = utf8mb4;
- 将已经建好的表也转换成utf8mb4
alter table TABLE_NAME convert to character set utf8mb4 collate utf8mb4_bin
- 查看是否修改成功，执行如下sql语句
SHOW VARIABLES WHERE variable_name LIKE 'character\_set\_%' OR Variable_name LIKE 'collation%'；
```

## 升级
从MySQL 5.5.3版本开始，数据库可支持4个字节的utf8mb4 字符集，一个字符最多可以有4个字节，所以能支持更多的字符集，故能存储Emoji表情符号。从 mysql 5.5.3 之后版本基本可以无缝升级到 utf8mb4 字符集。同时，utf8mb4兼容utf8字符集，utf8 字符的编码、位置、存储在utf8mb4与utf8字符集里一样的，不会对有现有数据带来损坏。