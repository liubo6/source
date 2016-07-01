---
title: centos 安装 mysql5.7
date: 2016-07-01 13:11:24
categories: 数据库
tags: mysql
---
## Step1: 检测系统是否自带安装mysql
```
yum list installed | grep mysql
```
## Step2: 删除系统自带的mysql及其依赖
```
yum -y remove mysql-libs.x86_64
```
<!--more-->

## Step3: 给CentOS添加rpm源，并且选择较新的源
```
wget dev.mysql.com/get/mysql-community-release-el6-5.noarch.rpm
yum localinstall mysql-community-release-el6-5.noarch.rpm -y
yum repolist all | grep mysql 
yum-config-manager --disable mysql55-community 
yum-config-manager --disable mysql56-community 
yum-config-manager --enable mysql57-community-dmr 
yum repolist enabled | grep mysql
```


## Step4:安装mysql 服务器
```
yum install mysql-community-server -y
```
## Step5: 启动mysql
```
service mysqld start
```
## Step6: 查看mysql是否自启动,并且设置开启自启动
```
chkconfig --list | grep mysqld
chkconfig mysqld on
```
## Step7: 找到随机密码
```
grep 'temporary password' /var/log/mysqld.log
```

## Step8: mysql安全设置
```
mysql_secure_installation
```
##  Step9:设置新密码
```
Liubo@68
```

##  Step10: 赋权限
```
GRANT ALL PRIVILEGES ON *.* TO 'liubo'@'%' IDENTIFIED BY 'liubo';
FLUSH PRIVILEGES;
```

---
##  Step11:  慢日志
```
mysql> show VARIABLES like "%slow%";
+---------------------------+--------------------------------------+
| Variable_name             | Value                                |
+---------------------------+--------------------------------------+
| log_slow_admin_statements | OFF                                  |
| log_slow_slave_statements | OFF                                  |
| slow_launch_time          | 2                                    |
| slow_query_log            | OFF                                  |
| slow_query_log_file       | /var/lib/mysql/fdsgg6qpzZ-slow.log |
+---------------------------+--------------------------------------+
5 行于数据集 (0.37 秒)
#开启
set global log_slow_queries = ON;
set global slow_query_log = ON;
set global long_query_time=0.1; #设置大于0.1s的sql语句记录下来
select sleep(0.13);
设置0.13秒延迟，然后这条语句按照预期(因为之前设置超过0.1秒)会被记录到日志文件中去
```
## 登录
```
mysql -h 主机名 -u 用户名 -p 
mysql -D 所选择的数据库名 -h 主机名 -u 用户名 -p
 -h : 该命令用于指定客户端所要登录的MySQL主机名, 登录当前机器该参数可以省略; 
-u : 所要登录的用户名; 
-p : 告诉服务器将会使用一个密码来登录, 如果所要登录的用户名密码为空, 可以忽略此选项。
```
## 基础操作
```
#添加列
alter table 表名 add 列名 列数据类型 [after 插入位置];
#修改列
alter table 表名 change 列名称 列新名称 新数据类型;
#删除列
alter table 表名 drop 列名称;
#重命名表
alter table 表名 rename 新表名;
#mysql启用时间
show status like 'uptime';
#查询次数
show status like 'com_select';
#添加次数
show status like 'com_insert';
#更新次数
show status like 'com_update'
#删除次数
show status like 'com_delete'
#连接次数
show status like 'connections';
#慢查询次数
show status like 'slow_queries';
#查询慢查询时间
show variables like 'long_query_tiem';

```
## 数据库备份
```
# 备份数据库
# -l
# -F 刷新bin-log日志
# -d 没有数据,只导出表结构
# --add-drop-table 在每个create语句之前增加一个drop table
/usr/local/mysql/bin/mysqldump -h127.0.0.1 -uroot -p密码 数据库名 -l -F > /data/test.sql
/usr/local/mysql/bin/mysqldump -h127.0.0.1 -uroot -p密码 -d --add-drop-table 数据库名 > /data/test.sql
# 导入数据库
# -v 查看导入详细信息
# -f 遇到错误直接跳过，继续执行
/usr/local/mysql/bin/mysql -h127.0.0.1 -uroot -pliubo test -v -f </data/test.sql
# 回复bin-log日志数据到数据库
# --start-position  开始位置
# --stop-position   结束位置
/usr/local/mysql/bin/mysqlbinlog --no-defaults mysql-bin.000008 |/usr/local/mysql/bin/mysql -uroot -pliubo  test
/usr/local/mysql/bin/mysqlbinlog --no-defaults --start-position="500" --stop-position="600" mysql-bin.000008 |/usr/local/mysql/bin/mysql -uroot -pliubo test
# 查看big-log日志
/usr/local/mysql/bin/mysqlbinlog --no-defaults mysql-bin.000008
# 刷新日志
MySQL > flush logs;
# 查看bin-log日志
MySQL > show master status;
```

