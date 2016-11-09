---
title: mysql主从复制配置
date: 2016-11-09 09:39:00
categories: 数据库
tags: mysql
---
# 版本
mysql-5.7.16-master
mysql-5.7.16-slave

# 安装服务并启动
```
进入各自bin目录 
mysqld --install mysql3306
mysqld --install mysql3307
管理员方式启动cmd.exe
```

<!--more-->

# 初始化密码
```
mysqld --initialize --user=mysql --console，执行后进行初始化，此时会生成root的初始密码
生成的临时密码，必须用客户端client来修改
mysql -uroot -pXXXX -P3306
set password =password('123456');
```

# 忘记密码
```
mysqld --skip-grant-tables //启动MySQL服务的时候跳过权限表认证
重新打开一个cmd，输入mysql 即可，
use mysql;
修改密码
update user set authentication_string=password("123456") where user="root";
#更新权限
flush privileges;
mysql -u root -p -P3306，新密码123456登录
修改密码 mysqladmin -u用户名 -p旧密码 password 新密码
```

# 主从复制的用途
1. 灾难备份，防止主库数据丢失；
2. 故障切换，主库挂断，可以切换到从库，不影响业务；
3. 读写分离的基础，从库分担读的压力，主库只有写的压力。

# 配置主从
master:localhost:3306
slaves1:localhost:3307

# 主库配置
```
[mysqld]
#服务端使用的字符集默认为utf8
character-set-server=utf8
 
#设置3306端口
port = 3306
 
# 设置mysql的目录
basedir = D:\MySQL\mysql-5.7.16-master
datadir = D:\MySQL\mysql-5.7.16-master\data
 
# 允许最大连接数
max_connections=500
 
server_id =1
log-bin=mysql-log-bin
 
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB 

#配置说明
#server-id=  #服务id，一般为ip后三位
#log-bin=    #二进制日志的路径
#binlog-do-db=test   #要同步的数据库名
```

## 分配从库复制的账号
```
grant replication slave on *.* to 'salves1' @'localhost' identified by '123456';
FLUSH PRIVILEGES;
salves1是从库复制的账号,localhost是从库的ip, 123456是密码
```

show master status;

主库状态


# 配置从库
## 修改从库配置文件
```
[mysqld]
#服务端使用的字符集默认为utf8
character-set-server=utf8
 
#设置3306端口
port = 3307
 
# 设置mysql的目录
basedir = D:\MySQL\mysql-5.7.16-slave
datadir = D:\MySQL\mysql-5.7.16-slave\data
 
# 允许最大连接数
max_connections=500
 
server_id =2
#不是必须的二进制日志
log-bin=mysql-log-bin
# a-b-c
log_slave_updates=1
 
#设置从服务器为读写状态read_only=0，read_only=1只读，普通的应用用户进行insert、update、delete等会产生数据变化的DML操作时，
#都会报出数据库处于只读模式不能发生数据变化的错误，但具有super权限的用户，例如在本地或远程通过root用户登录到数据库，
#还是可以进行数据变化的DML操作；
read_only=1
#配置中继日志
#从服务器I/O线程将主服务器的二进制日志读取过来记录到从服务器本地文件，然后SQL线程会读取relay-log日志的内容并应用到从服务器
relay_log= mysql-relay-bin
 
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB 
#server-id=2                                 #从库自己的id，不可与主库重复      
#replicate-do-db=test                                  #要同步的数据库名
#relay_log = "D:/MySQL/MySQL56/data/mysql-relay-bin"   #中继日志的路径
```

## 设置与主库相关的信息
```
执行命令：
change master to master_host='localhost',master_user = 'salves1',master_password ='123456',master_port=3306,master_log_file='mysql-log-bin.000007',master_log_pos=1412;   
//主库的ip,复制的账号,复制账号的密码
```
启动从库复制线程
start slave
show slave status
主要检查两个参数：Slave_IO_Running和Slave_Sql_Running。这两个值为Yes，OK从库配置好了

# 主从复制测试
主库里添加一条数据
从库里能看到刚才添加的数据，主从复制OK了



# 故障案例
1. 主从同步报错Fatal error: The slave I/O thread stops because master and slave have equal MySQL server
>mysql 5.6的复制引入了uuid的概念，各个复制结构中的server_uuid得保证不一样，但是查看到直接copy
data文件夹后server_uuid是相同的，show variables like ‘%server_uuid%’;
解决方法
找到data文件夹下的auto.cnf文件，修改里面的uuid值，保证各个db的uuid不一样，重启db即可

# 原理
Mysql复制是通过将mysql的某一台主机的数据复制到其他主机（slaves）上，并且在slaves上重新执行一遍来实现。
主服务器每次数据操作都会将更新记录到二进制日志文件，并维护文件的一个索引跟踪日志循环，slaves服务器通过
获取主服务器的二进制日志来更新同步数据。当一个从服务器连接主服务器时，它通知主服务器从服务器的日志中读取
的最后一次成功更新的为止
注意：当进行主从复制时，所有对表的更新必须在主服务器上进行，否则会造成冲突

## mysql支持的复制类型
1. 基于语句的复制(Statement-Based Replication)：在主服务器上执行SQL语句，在从服务器上执行同样的SQL语句（默认采用的），如果没法精确复制，就会自动选择基于行的复制
1. 基于行的复制(Row-Based Replication)：把改变的内容复制过去，而不是把命令在从服务器执行一遍，从mysql5.0开始支持
1. 混合类型，默认采用基于语句的复制，一旦发现基于语句的无法精确的复制时，采用基于行的复制

## 工作方式
1. master服务器将数据更新记录到二进制日志文件中（主服务器上必须开启log-bin）
1. slave将master的二进制日志拷贝到它的中继日志（relay log）
1. slave通过SQL线程从relay log中读取二进制日志，重新执行一遍，以改变自己的数据
>1）master服务器上开启二进制日志，每个事务更新数据完成之前，master都会把数据变化记录到二进制日志文件中（串行写入）
2）slave上配置change master to，将master的binary日志拷贝到它自己的中继日志（I/O线程）
3）SQL线程从中继日志中读取事件，并重放其中的事件更新slave的数据，使得和master的数据一样
复制过程有一个很重要的限制——复制在slave上是串行化的，也就是说master上的并行更新操作不能在slave上并行操作。

![mysql复制工作方式](http://liubo.qiniudn.com/mysql_rep.png)

备份服务器数据
```
mysqldump -uroot -p123456 -A -x > /server/backup/mysql_`date +%F`.sql
```

## 发送复制事件到其它slave,a->b->c 模式
当设置log_slave_updates时，你可以让slave扮演其它slave的master。此时，slave把SQL线程执行的事件写进行自己的二进制日志
(binary log)，然后，它的slave可以获取这些事件并执行它。如下：
![](http://liubo.qiniudn.com/mysql03-5.JPG)

## 复制过滤(Replication Filters)
复制过滤可以让你只复制服务器中的一部分数据，有两种复制过滤：在master上过滤二进制日志中的事件；在slave上过滤中继日志中的事件
![](http://liubo.qiniudn.com/mysql03-6.JPG)

# 复制的常用拓扑结构
复制的体系结构有以下一些基本原则：
1. 每个slave只能有一个master；
1. 每个slave只能有一个唯一的服务器ID；
1. 每个master可以有很多slave；
1. 如果你设置log_slave_updates，slave可以是其它slave的master，从而扩散master的更新。
MySQL不支持多主服务器复制(Multimaster Replication)——即一个slave可以有多个master

## 单一master和多slave
一个master和一个slave组成复制系统是最简单的情况。Slave之间并不相互通信，只能与master进行通信
如果写操作较少，而读操作很时，可以采取这种结构。你可以将读操作分布到其它的slave，从而减小master的压力。但是，当slave增加到一定数量时，slave对master的负载以及网络带宽都会成为一个严重的问题。
这种结构虽然简单，但是，它却非常灵活，足够满足大多数应用需求。一些建议：
1. 不同的slave扮演不同的作用(例如使用不同的索引，或者不同的存储引擎)；
1. 用一个slave作为备用master，只进行复制；
1. 用一个远程的slave，用于灾难恢复；

## 主动模式的Master-Master(Master-Master in Active-Active Mode)
Master-Master复制的两台服务器，既是master，又是另一台服务器的slave

## 主动-被动模式的Master-Master(Master-Master in Active-Passive Mode)
这是master-master结构变化而来的，它避免了M-M的缺点，实际上，这是一种具有容错和高可用性的系统。它的不同点在于其中一个服务只能进行只读操作

## Master-Master结构(Master-Master with Slaves)
这种结构的优点就是提供了冗余。在地理上分布的复制结构，它不存在单一节点故障问题，而且还可以将读密集型的请求放到slave上

# 备份
做灾难恢复：对损坏的数据进行恢复和还原
需求改变：因需求改变而需要把数据还原到改变以前
测试：测试新功能是否可用

冷备（cold backup）：需要关mysql服务，读写请求均不允许状态下进行；
温备（warm backup）： 服务在线，但仅支持读请求，不允许写请求；
热备（hot backup）：备份的同时，业务不受影响。
