---
title: redis-bitmaps
date: 2016-08-03 11:26:31
categories: 分布式
tags: redis
---
# 介绍
bitmaps并非一种数据类型，而是支持对string类型的value进行二进制置位运算

# 主要使用命令
- setbit
>设置某个键的某位的值
用法: setbit key offset value
例子: setbit logins 2 1

<!--more-->
- getbit
>获取某个键的某位的值
用法: getbit key offset
例子: getbit logins 2

- bitop
>对多个键进行位运算
用法: bitop operation destkey key key1 [key2]
参数说明:operation表示位算符，有AND,OR,NOT,XOR;destkey 表示最终保存结果的键; key key1 key2等表示用于运算的键
例子: setbit login:1 2 1
setbit login:2 3 1
bitop AND login-2 login:1 login:2


- bitcount
>统计某个键的有多少位上的值1
用法: bitcount key [start end]
例子: bitcount login:2


- bitpos
>获取某个键的位第一个是1或者0的位的位置
用法: bitpos key bit [start end]
例子: 查看位的值是1的最开始的位数,bitpos login:1 1;查看位的值是0的最开始的位数,bitpos login:1 0;

# 实际使用
## 日活跃用户
很多网站的活跃用户是指登录用户或者付费用户
为了统计今日登录的用户数，我们建立了一个bitmap,每一位标识一个用户ID。当某个用户访问我们的网页或执行了某个操作，就在bitmap中把标识此用户的位置为1
![](http://ww4.sinaimg.cn/mw690/69045600gw1f6gdttsb21j20aa04i3ye.jpg)
这个简单的例子中，每次用户登录时会执行一次redis.setbit(daily_active_users, user_id, 1)。将bitmap中对应位置的位置为1，时间复杂度是O(1)。统计bitmap结果显示有今天有9个用户登录。Bitmap的key是daily_active_users，它的值是1011110100100101。

因为日活跃用户每天都变化，所以需要每天创建一个新的bitmap。我们简单地把日期添加到key后面，实现了这个功能。例如，要统计某一天有多少个用户至少听了一个音乐app中的一首歌曲，可以把这个bitmap的redis key设计为play:yyyy-mm-dd-hh。当用户听了一首歌曲，我们只是简单地在bitmap中把标识这个用户的位置为1，时间复杂度是O(1)。
```
setbit play:yyyy-mm-dd user_id 1  
```
今天听过歌曲的用户就是key是play:yyyy-mm-dd的bitmap的位图计数。如果要按周或月统计，只要对这周或这个月的所有bitmap求并集，得出新的bitmap，在对它做位图计数
利用这些bitmap做其它复杂的统计也非常容易。例如，统计11月听过歌曲的高级用户(premium user),只需通过bitop命令进行AND,OR,XOR,NOT操作即可，如下伪代码：
```
(play:2016-08-01 ∪ play:2016-08-02 ∪...∪play:2016-08-30) ∩ premium:2016-08
```
下面的表格显示了在1亿2千8百万用户上完成的时间粒度为1天，一周，一个月的用户统计的时间消耗比较
|周期|耗时(ms)
---|---|---
日|50.2
周|392.0
月|1624.8

## 存储布尔性质的值，节约内存
最常见的存储用户
```
set user:1:name "liubo"
set user:1:age 20
set user:1:set 1 # 性别 1表示男，0表示女
```
如果要节省内存空间
可以用每个userId 对应每个二进制位,
比如我们总共有1亿个用户，userID从1到一亿，当然数字不一定是连贯性的，那么userID为0则表示offset(偏移量)为0， userID为15则表示offset为15,(如果你的uid不是从1开始的，比如从100000开始，实际上你也可以相应的用uid减去初始值来表示其位数，比如1000000用户对应到bitmap的第一位)如下命令我们创建了该键值
```
SETBIT users:sex 100000000 0
```
redis的bitmap只支持2^32大小,即使用512M内存

初始化userID用户性别都是0，这里看下userID为0和15和1亿的性别
```
127.0.0.1:6379> GETBIT users:sex 0
(integer) 0
127.0.0.1:6379> GETBIT users:sex 15
(integer) 0
127.0.0.1:6379> GETBIT users:sex 100000000
(integer) 0
```
UserID为15的设置了性别为男
```
127.0.0.1:6379> setbit users:sex 15 1
(integer) 0
127.0.0.1:6379> GETBIT users:sex 15
(integer) 1
```
统计男女比例，则可以
```
127.0.0.1:6379> BITCOUNT users:sex
(integer) 1
```
一亿里才有1个男人,1亿多数据，其实整个操作耗时基本是毫秒级别的，时间复杂度为O(1).
还可以用来 用户标识一条消息的已读未读,只要涉及布尔形式的都可以用bitmaps


