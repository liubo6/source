---
title: MySQL事务autocommit自动提交
date: 2016-10-25 11:48:13
categories: 数据库
tags: mysql
---
MySQL默认操作模式就是autocommit自动提交模式,其对mysql的性能有一定影响,这就表示除非显式地开始一个事务，
否则每个查询都被当做一个单独的事务自动执行。我们可以通过设置autocommit的值改变是否是自动提交autocommit模式。
>例子，如果你插入了1000条数据，mysql会commit1000次的，如果我们把autocommit关闭掉，
通过程序来控制，只要一次commit就可以了


# 查看当前autocommit模式
```
mysql> show variables like 'autocommit';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| autocommit    | ON    |
+---------------+-------+
1 row in set
```
<!--more-->

Value的值是ON，表示autocommit开启。
我们可以通过以下SQL语句改变这个模式
# 改变autocommit模式
## 命令方式改变
```
mysql> set autocommit = 0;
```
值0和OFF都是一样的，当然，1也就表示ON。通过以上设置autocommit=0，则用户将一直处于某个事务中，直到执行一条commit
提交或rollback语句才会结束当前事务重新开始一个新的事务。

## 配置文件改变(注意用户权限问题)
mysql的配置文件my.cnf来关闭autocommit
```
[mysqld] 
init_connect='SET autocommit=0' 
```

# innodb 的事务处理
## 用begin,rollback,commit来实现
```
begin 开始一个事务
rollback 事务回滚
commit 事务确认
```
## 用set来改变mysql的自动提交模式
```
MYSQL默认是自动提交的，也就是你提交一个QUERY，它就直接执行！我们可以通过
set autocommit=0 禁止自动提交
set autocommit=1 开启自动提交
来实现事务的处理。
当你用 set autocommit=0 的时候，你以后所有的SQL都将做为事务处理，直到你用commit确认或rollback结束
```
