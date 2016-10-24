---
title: mysql中的时间存储
date: 2016-10-23 11:04:21
categories: 数据库
tags: 日期
---

在mysql中存储时间，我们可以用datetime 格式，timestamp格式，也可以用bigint格式

# DATETIME
```
DATETIME格式，默认是"YYYY-MM-DD HH:MM:SS",这19个字符表示的，从1000-01-01 00:00:00-9999-12-31 23:59:59 。
```
<!--more-->

# TIMESTAMP
```
TIMESTAMP格式也是'YYYY-MM-DD HH:MM:SS'这样的，与DATETIME不同的地方是，它的年份取值范围是1970-2037。
mysql官方文档解释：timestamp的范围：1970-01-01 00:00:01UTC～2038/01/19 3:14:07UTC
东八区 1970-01-01 8:00:01UTC～2038/01/19 11:14:07UTC
```
## 区间测试
```
mysql> insert into timestamp_test(createDate) values('1970-01-01 00:00:00');
Incorrect datetime value: '1970-01-01 00:00:00' for column 'createDate' at row 1
mysql> insert into timestamp_test(createDate) values('1970-01-01 00:00:01');
Query OK, 1 rows affected (0.10 秒)

mysql> insert into timestamp_test(createDate) values('2038-01-19 03:14:07');
Query OK, 1 rows affected (0.11 秒)
mysql> insert into timestamp_test(createDate) values('2038-01-19 03:14:08');
Incorrect datetime value: '2038-01-19 03:14:08' for column 'createDate' at row 1

mysql> select unix_timestamp('2038-01-19 03:14:07');
+---------------------------------------+
| unix_timestamp('2038-01-19 03:14:07') |
+---------------------------------------+
| 2147483647                            |
+---------------------------------------+
1 行于数据集 (0.10 秒)

2147483647就是"2038/01/19 3:14:07UTC"减去"1970-01-01 8:00:01 UTC"的秒数
```
## 特点
```
timestamp 是自带时区转换的
东八区现在是 2016-10-10 00:11:23， 东九区为 2016-10-10 01:11:23 数据库记录了时间，取出来之后，
对于东八区的中国时间是 2016-10-10 00:11:23,对于东九区的日本来说就是 2016-10-10 01:11:23


```

#  bigint
```
bigint的格式就是整数的形式，它可以控制位数，一般我们设置成13位就可以
直接比较大小就可以比较时间
```

# 总结
- timestamp 记录经常变化的更新/创建/发布/日志时间/购买时间/登录时间/注册时间等，并且是近来的时间，
够用，时区自动处理，比如说做海外购或者业务可能拓展到海外
- datetime 记录固定时间如服务器执行计划任务时间/健身锻炼计划时间等，在任何时区都是需要一个固定的时间要做某个事情。
超出 timestamp 的时间，如果需要时区必须记得时区处理