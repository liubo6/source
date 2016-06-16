---
title: redis-cluster集群搭建
date: 2016-05-19 15:08:05
tags: redis
---
### 下载
```
wget http://download.redis.io/releases/redis-3.0.7.tar.gz
```


### 复制到每个机器
scp到 每个机器 
```
scp redis-3.0.7.tar.gz  root@192.168.129.171:/opt/
scp redis-3.0.7.tar.gz  root@192.168.129.172:/opt/
scp redis-3.0.7.tar.gz  root@192.168.129.173:/opt/
```
<!--more-->

###  解压到指定目录
```
tar -zxvf redis-3.0.7.tar.gz  -C /opt/
```


###  安装
```
make PREFIX=/usr/local/redis install
#则目录/usr/local/redis/bin下会有执行文件
#make   报错请 make MALLOC=libc
```
### 创建配置文件目录
```
mkdir -p /usr/local/redis/etc
mkdir -p /usr/local/redis/logs
mkdir -p /usr/local/redis/data
```

### 作为服务启动
>redis-3.0.7/utils 目录下 redis_init_script 启动脚本修改，目录等
在启动脚本开头添加如下两行注释以修改其运行级别：
redis服务必须在运行级2，3，4，5下被启动或关闭，启动的优先级是90，关闭的优先级是10

```
# chkconfig:   2345 90 10
# description:  Redis is a persistent key-value database
```

### 添加开机启动
```
cp  redis_init_script  /etc/init.d/redis  
chkconfig redis on
```

### 客户端sharding
>客户端sharding技术其优势在于服务端的Redis实例彼此独立，相互无关联，每个Redis实例像单服务器一样运行，非常容易线性扩展，系统的灵活性很强。其不足之处在于： 由于sharding处理放到客户端，规模进步扩大时给运维带来挑战。 服务端Redis实例群拓扑结构有变化时，每个客户端都需要更新调整。 连接不能共享，当应用规模增大时，资源浪费制约优化

### 服务端sharding
>服务端sharding的Redis Cluster其优势在于服务端Redis集群拓扑结构变化时，客户端不需要感知，客户端像使用单Redis服务器一样使用Redis集群，运维管理也比较方便。 不过Redis Cluster正式版推出时间不长，系统稳定性、性能等都需要时间检验，尤其在大规模使用场合



### 搭建主从
```
第一台服务器 192.168.129.171  6379
第二台服务器 192.168.129.172  6379
第三台服务器 192.168.129.173  6379

171 master     配置 requirepass liubo123!@#456$%^  ，bind 192.168.129.171  127.0.0.1
172 slave      配置 slaveof 192.168.129.171 6379 ，masterauth liubo123!@#456$%^
173 slave      配置 slaveof 192.168.129.171 6379 ，masterauth liubo123!@#456$%^
```
### 查看主从信息

#### 171  info 命令
```
role:master
connected_slaves:2
slave0:ip=192.168.129.172,port=6379,state=online,offset=239,lag=1
slave1:ip=192.168.129.173,port=6379,state=online,offset=239,lag=1
```

#### 172 info 命令
```
# Replication
role:slave
master_host:192.168.129.171
master_port:6379
master_link_status:up
master_last_io_seconds_ago:4
master_sync_in_progress:0
slave_repl_offset:491
slave_priority:100
slave_read_only:1
```

#### 173 info 命令
```
# Replication
role:slave
master_host:192.168.129.171
master_port:6379
master_link_status:up
master_last_io_seconds_ago:5
master_sync_in_progress:0
slave_repl_offset:631
slave_priority:100
slave_read_only:1
```

### 搭建Redis Cluster
>了增加集群的可访问性，官方推荐的方案是将node配置成主从结构，即一个master主节点，挂n个slave从节点。这时，如果主节点失效，Redis Cluster会根据选举算法从slave节点中选择一个上升为主节点，整个集群继续对外提供服务。这非常类似前篇文章提到的Redis Sharding场景下服务器节点通过Sentinel监控架构成主从结构，只是Redis Cluster本身提供了故障转移容错的能力。

### 建立6个redis实例
```
修改上面三个实例配置改为
第一台服务器 192.168.129.171  6379（主）  192.168.129.171  6380（复）
第二台服务器 192.168.129.172  6379（主）  192.168.129.172  6380（复）
第三台服务器 192.168.129.173  6379（主）  192.168.129.173  6380（复）
```
### 配置文件修改
服务启动了并不是集群服务，还需要开启集群才可以分别对6个节点，开启集群，配置文件修改 取消注释
```
cluster-enabled yes
cluster-config-file  nodes-6379.conf
cluster-node-timeout 15000
```
### 查看服务状态
重启服务看到redis为集群状态了
```
[root@localhost bin]# ps -ef|grep redis
root      2872     1  0 13:17 ?        00:00:00 /usr/local/redis/bin/redis-server 192.168.129.171:6379 [cluster]
[root@localhost etc]# ps -ef|grep redis
root      2533     1  0 21:16 ?        00:00:00 /usr/local/redis/bin/redis-server 192.168.129.172:6379 [cluster]
[root@localhost redis6380]# ps -ef|grep redis
root      1801     1  0 21:17 ?        00:00:00 /usr/local/redis/bin/redis-server 192.168.129.173:6379 [cluster]
```

>这时候集群没有搭建成功，只是每个服务器启动成功了，还需要client相关，才可以联系起来，组成集群，访问单个服务会有错误信息
```
Cannot open value tab :Cannot load key name,connection error occurred :-ClUSTERDOWN the cluster is down
```

### 安装redis client需要的依赖
```
yum -y install ruby rubygems   //安装ruby rubygems
gem install redis
会安装在源码目录下 redis-trib.rb
``` 
```

### 脚本创建redis集群 

```
redis-trib.rb create --replicas 1 192.168.129.171:6379 192.168.129.171:6380 192.168.129.172:6379 192.168.129.172:6380 192.168.129.173:6379 192.168.129.173:6380
```


### 开启过程
```
[root@localhost src]# redis-trib.rb create --replicas 1 192.168.129.171:6379 192.168.129.171:6380 192.168.129.172:6379 192.168.129.172:6380 192.168.129.173:6379 192.168.129.173:6380
Creating cluster
Performing hash slots allocation on 6 nodes...
Using 3 masters:
192.168.129.173:6379
192.168.129.172:6379
192.168.129.171:6379
Adding replica 192.168.129.172:6380 to 192.168.129.173:6379
Adding replica 192.168.129.173:6380 to 192.168.129.172:6379
Adding replica 192.168.129.171:6380 to 192.168.129.171:6379
M: ed4c0fbdf5729337d340bbe35b7a36c1638ead48 192.168.129.171:6379
   slots:10923-16383 (5461 slots) master
S: 17c5b191a8ddc1ca096bdf510fcbf34dd6042445 192.168.129.171:6380
   replicates ed4c0fbdf5729337d340bbe35b7a36c1638ead48
M: 8d76f878a5c71f17a004def887f2f8b7f842bcb3 192.168.129.172:6379
   slots:5461-10922 (5462 slots) master
S: b4dac8102245c1367004d5591497705f7edf8d53 192.168.129.172:6380
   replicates f4bbad5333be8c52c15f5c46f02479ed1263c5e5
M: f4bbad5333be8c52c15f5c46f02479ed1263c5e5 192.168.129.173:6379
   slots:0-5460 (5461 slots) master
S: 7da8367c17006eef4acd91d82dbef53523eb7f3c 192.168.129.173:6380
   replicates 8d76f878a5c71f17a004def887f2f8b7f842bcb3
Can I set the above configuration? (type 'yes' to accept): yes
 Nodes configuration updated
 Assign a different config epoch to each node
 Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join..
 Performing Cluster Check (using node 192.168.129.171:6379)
M: ed4c0fbdf5729337d340bbe35b7a36c1638ead48 192.168.129.171:6379
   slots:10923-16383 (5461 slots) master
M: 17c5b191a8ddc1ca096bdf510fcbf34dd6042445 192.168.129.171:6380
   slots: (0 slots) master
   replicates ed4c0fbdf5729337d340bbe35b7a36c1638ead48
M: 8d76f878a5c71f17a004def887f2f8b7f842bcb3 192.168.129.172:6379
   slots:5461-10922 (5462 slots) master
M: b4dac8102245c1367004d5591497705f7edf8d53 192.168.129.172:6380
   slots: (0 slots) master
   replicates f4bbad5333be8c52c15f5c46f02479ed1263c5e5
M: f4bbad5333be8c52c15f5c46f02479ed1263c5e5 192.168.129.173:6379
   slots:0-5460 (5461 slots) master
M: 7da8367c17006eef4acd91d82dbef53523eb7f3c 192.168.129.173:6380
   slots: (0 slots) master
   replicates 8d76f878a5c71f17a004def887f2f8b7f842bcb3
[OK] All nodes agree about slots configuration.
 Check for open slots...
 Check slots coverage...
[OK] All 16384 slots covered.

```


### 可能出现的错误
>[ERR] Sorry, can't connect to node
原因是版本太低
下载新的 http://cache.ruby-lang.org/pub/ruby/2.3/ruby-2.3.0.tar.gz
解压 安装

```
./configure --prefix=/usr/local/ruby-2.3.0
make && make install
gem install redis
```
 

###  检查集群状态

```
redis-trib.rb check 192.168.129.171:6379
[root@localhost src]# redis-trib.rb check 192.168.129.171:6379
 Performing Cluster Check (using node 192.168.129.171:6379)
M: ed4c0fbdf5729337d340bbe35b7a36c1638ead48 192.168.129.171:6379
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
S: b4dac8102245c1367004d5591497705f7edf8d53 192.168.129.172:6380
   slots: (0 slots) slave
   replicates f4bbad5333be8c52c15f5c46f02479ed1263c5e5
M: 8d76f878a5c71f17a004def887f2f8b7f842bcb3 192.168.129.172:6379
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
S: 17c5b191a8ddc1ca096bdf510fcbf34dd6042445 192.168.129.171:6380
   slots: (0 slots) slave
   replicates ed4c0fbdf5729337d340bbe35b7a36c1638ead48
S: 7da8367c17006eef4acd91d82dbef53523eb7f3c 192.168.129.173:6380
   slots: (0 slots) slave
   replicates 8d76f878a5c71f17a004def887f2f8b7f842bcb3
M: f4bbad5333be8c52c15f5c46f02479ed1263c5e5 192.168.129.173:6379
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
 Check for open slots...
 Check slots coverage...
[OK] All 16384 slots covered
```


###  修复集群

>如果出现  [ERR] Not all 16384 slots are covered by nodes

```
redis-trib.rb fix   192.168.129.171:6379
```

### 使用集群

>使用了集群之后，不能像以前一样使用命令行客户端了, 需要加参数`-c`
```
redis-cli -c
127.0.0.1:6379> set a b
-> Redirected to slot [15495] located at 127.0.0.1:6381
OK
```

### 查询集群状态

```
#cluster info
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:7
cluster_my_epoch:7
cluster_stats_messages_sent:5906
cluster_stats_messages_received:3773
```

### 集群命令

```
CLUSTER INFO 打印集群的信息
CLUSTER NODES 列出集群当前已知的所有节点（node），以及这些节点的相关信息。
节点
CLUSTER MEET <ip> <port> 将 ip 和 port 所指定的节点添加到集群当中，让它成为集群的一份子。
CLUSTER FORGET <node_id> 从集群中移除 node_id 指定的节点。
CLUSTER REPLICATE <node_id> 将当前节点设置为 node_id 指定的节点的从节点。
CLUSTER SAVECONFIG 将节点的配置文件保存到硬盘里面。
槽(slot)
CLUSTER ADDSLOTS <slot> [slot ...] 将一个或多个槽（slot）指派（assign）给当前节点。
CLUSTER DELSLOTS <slot> [slot ...] 移除一个或多个槽对当前节点的指派。
CLUSTER FLUSHSLOTS 移除指派给当前节点的所有槽，让当前节点变成一个没有指派任何槽的节点。
CLUSTER SETSLOT <slot> NODE <node_id> 将槽 slot 指派给 node_id 指定的节点，如果槽已经指派给另一个节点，那么先让另一个节点删除该槽>，然后再进行指派。
CLUSTER SETSLOT <slot> MIGRATING <node_id> 将本节点的槽 slot 迁移到 node_id 指定的节点中。
CLUSTER SETSLOT <slot> IMPORTING <node_id> 从 node_id 指定的节点中导入槽 slot 到本节点。
CLUSTER SETSLOT <slot> STABLE 取消对槽 slot 的导入（import）或者迁移（migrate）。
键
CLUSTER KEYSLOT <key> 计算键 key 应该被放置在哪个槽上。
CLUSTER COUNTKEYSINSLOT <slot> 返回槽 slot 目前包含的键值对数量。
CLUSTER GETKEYSINSLOT <slot> <count> 返回 count 个 slot 槽中的键。
```

###  info memory 命令

```
127.0.0.1:6379> info memory
# Memory
used_memory:2978624
used_memory_human:2.84M
used_memory_rss:5431296
used_memory_peak:2980488
used_memory_peak_human:2.84M
used_memory_lua:36864
mem_fragmentation_ratio:1.82
mem_allocator:libc
used_memory: redis #当前数据使用的内存，有可能包括SWAP虚拟内存。
used_memory_rss # redis当前占用的物理内存，包括内存碎片。此值和用top命令查看的进程占用内存一致
```



### info stats 命令查看统计信息

```
127.0.0.1:6379> info stats
# Stats
total_connections_received:3
total_commands_processed:6893
instantaneous_ops_per_sec:0   //每秒操作数，查看当前并发压力
total_net_input_bytes:96702
total_net_output_bytes:2615508
instantaneous_input_kbps:0.00
instantaneous_output_kbps:0.00
rejected_connections:0
sync_full:0
sync_partial_ok:0
sync_partial_err:0
expired_keys:0
evicted_keys:0
keyspace_hits:2
keyspace_misses:0
pubsub_channels:0
pubsub_patterns:0
latest_fork_usec:0
migrate_cached_sockets:0
```

### redis 性能测试

```
[root@localhost bin]# ./redis-benchmark
#基本一秒处理十万请求
100.00% <= 0 milliseconds
102564.10 requests per second
```

### 配置

>--replicas 1 表示我们希望为集群中的每个主节点创建一个从节点，由于redis的集群最少需要3个主节点，如果我们每个主节点需要一个从节点，那么最少需要6台机器（或者说6个实例）
```
daemonize yes            # redis默认不是后台启动，这里修改成后台启动
pidfile /var/run/redis.pid # 当redis作为守护进程运行的时候，它会把 pid 默认写到 /var/run/redis.pid 文件里面，但是你可以在这里自己制定它的文件位置
port 6379 # 监听端口号，默认为 6379,可以自定义
tcp-backlog 511          # TCP 监听的最大容纳数量，在高并发的环境下，你需要把这个值调高以避免客户端连接缓慢的问题，Linux 内核会一声不响的把这个值缩小成 /proc/sys/net/core/somaxconn 对应的值，所以你要修改这两个值才能达到你的预期
bind 192.168.1.100 10.0.0.1 127.0.0.1  # 想让它在一个网络接口上监听，那你就绑定一个IP或者多个IP，多个IP用空格隔开
unixsocket /tmp/redis.sock # 指定 unix socket 的路径
timeout 0  # 指定在一个 client 空闲多少秒之后关闭连接（0 就是不管它）
tcp-keepalive 0 # tcp 心跳包，如果设置为非零，则在与客户端缺乏通讯的时候使用 SO_KEEPALIVE 发送 tcpacks 给客户端，推荐一个合理的值就是60秒
loglevel notice #  定义日志级别。 debug (适用于开发或测试阶段)，verbose，notice (适用于生产环境)，warning (仅仅一些重要的消息被记录)
logfile "" # 指定日志文件的位置 比如 /path/logs/
syslog-enabled no # 要想把日志记录到系统日志，就把它改成 yes
syslog-ident redis # 设置 系统日志 的 identity
databases 16 # 设置数据库的数目。默认数据库是 DB 0，你可以在每个连接上使用 select <dbid> 命令选择一个不同的数据库，但是 dbid 必须是一个介于 0 到 databasees - 1 之间的值
###########################################  快照 ################################################
save 900 1
save 300 10
save 60 10000
 
900 秒内如果至少有 1 个 key 的值变化，则保存
300 秒内如果至少有 10 个 key 的值变化，则保存
60 秒内如果至少有 10000 个 key 的值变化，则保存
 
stop-writes-on-bgsave-error yes # redis 最后一次的后台保存失败，redis 将停止接受写操作，你要是安装了靠谱的监控，你可能不希望 redis 这样做，那你就改成 no 好了
rdbcompression yes # dump .rdb 数据库的时候使用 LZF 压缩字符串， 如果你希望保存子进程节省点 cpu ，你就设置它为 no，不过这个数据集可能就会比较大
 
rdbchecksum yes #　是否校验rdb文件
dbfilename dump.rdb # 设置 dump 的文件位置
dir ./ # 工作目录 个配置项一定是个目录，而不能是文件名
############################################# 主从复制 #################################
使用 slaveof 来让一个 redis 实例成为另一个reids 实例的副本
注意这个只需要在 slave 上配置。
# slaveof <masterip> <masterport>
# 如果 master 需要密码认证，就在这里设置
# masterauth <master-password>
 
# 当一个 slave 与 master 失去联系，或者复制正在进行的时候，
# slave 可能会有两种表现：
#
# 1) 如果为 yes ，slave 仍然会应答客户端请求，但返回的数据可能是过时，
#    或者数据可能是空的在第一次同步的时候
#
# 2) 如果为 no ，在你执行除了 info he salveof 之外的其他命令时，
#    slave 都将返回一个 "SYNC with master in progress" 的错误，
#
slave-serve-stale-data yes
 
slave-read-only yes # slaves 都是只读的
repl-ping-slave-period 10 # Slaves 在一个预定义的时间间隔内发送 ping 命令到 server.你可以改变这个时间间隔。默认为 10 秒
repl-timeout 60 # 设置主从复制过期时间 ,这个值一定要比 repl-ping-slave-period 大
 
repl-backlog-size 1mb# 设置主从复制容量大小。这个 backlog 是一个用来在 slaves 被断开连接时存放 slave 数据的 buffer
slave-priority 100 # 当 master 不能正常工作的时候，Redis Sentinel 会从 slaves 中选出一个新的 master，这个值越小，就越会被优先选中，但是如果是 0 ， 那是意味着这个 slave 不可能被选中
 
################################## 安全 ###################################
requirepass liubo1234567890#$#@ # 设置认证密码
rename-command CONFIG "HELLO_LIUBO_CONFIG" # 重命名命令
################################### 限制 ####################################
maxclients 10000 # 一旦达到最大限制，redis 将关闭所有的新连接,并发送一个‘max number of clients reached’的错误
maxmemory <bytes> # 最大使用内存,如果你设置了这个值，当缓存的数据容量达到这个值， redis 将根据你选择的eviction 策略来移除一些 keys
最大内存策略，你有 5 个选择
volatile-lru：从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰
volatile-ttl：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰
volatile-random：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰
allkeys-lru：从数据集（server.db[i].dict）中挑选最近最少使用的数据淘汰
allkeys-random：从数据集（server.db[i].dict）中任意选择数据淘汰
no-enviction（驱逐）：禁止驱逐数据,不让任何 key 过期，只是给写入操作返回一个错误
 
maxmemory-policy noeviction # 默认策略是不让任何key过期
maxmemory-samples 5 # LRU and minimal TTL algorithms 使用策略，10非常耗费cpu，3很快，但是准确性不足，5刚好平衡
 
############################## APPEND ONLY MODE ###############################
appendonly yes # 指定是否在每次更新操作后进行日志记录，Redis在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为 redis本身同步数据文件是按上面save条件来同步的，所以有的数据会在一段时间内只存在于内存中
appendfilename "appendonly.aof" # 指定更新日志文件名，默认为appendonly.aof
 
appendfsync everysec # 指定更新日志条件，共有3个可选值no：表示等操作系统进行数据缓存同步到磁盘（快），always：表示每次更新操作后手动调用fsync()将数据写到磁盘（慢，安全），everysec：表示每秒同步一次（折衷，默认值）
 
################################ REDIS 集群  ###############################
cluster-enabled yes      # 启用或停用集群
cluster-config-file nodes-6379.conf   # 节点配置文件
cluster-node-timeout 15000       # 节点超时毫秒
 
############################### ADVANCED CONFIG ###############################
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-entries 512
list-max-ziplist-value 64
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
```

###  安全相关

>配置bind选项, 限定可以连接Redis服务器的IP, 并修改redis的默认端口6379
配置AUTH, requirepass yourpassword 设置密码, 密码会以明文方式保存在redis配置文件中，Redis的执行效率非常快，外部设备每秒可以测试相当数量的密码，攻击者可能在短期内发送大量猜密码的请求，很容易暴力破解，所以建议密码越长越好，比如20位随机字符串等
从命令表中重命名命令,CONFIG命令被更名为一个更为陌生的名字,配置rename-command CONFIG “HAHA_LIUBO_CONFIG”, 这样即使存在未授权访问, 也能够给攻击者使用config指令加大难度
Redis不需要root权限运行，也不建议以root权限运行







