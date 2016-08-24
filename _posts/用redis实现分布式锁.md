---
title: 用redis实现分布式锁
date: 2016-08-17 09:18:54
categories: 分布式
tags: redis
---
在*nix系统编程中，遇到多个进程或者线程共享一块资源的时候，通常会使用系统自身提供的锁，譬如一个进程里的多线程，会用互斥锁；多个进程之间，会用信号量等。这个场景中所谓的共享资源仅仅限于本地，倘若共享资源存在于网络上，本地的“锁”就不起作用了。互斥访问某个网络上的资源，需要有一个存在于网络上的锁服务器，负责锁的申请与回收。
<!--more-->

Redis 可以充当锁服务器的角色。首先，Redis 是单进程单线程的工作模式，所有前来申请锁资源的请求都被排队处理，能保证锁资源的同步访问。
可以借助 Redis 管理锁资源，来实现网络资源的互斥。
我们可以在 Redis 服务器设置一个键值对，用以表示一把互斥锁，当申请锁的时候，要求申请方设置（SET）这个键值对，当释放锁的时候，要求释放方删除（DEL）这个键值对
```
lock = redis.get("mutex_lock");
if(!lock)
    error("apply the lock error.");
else
    -- 确定可以申请锁
    redis.set("mutex_lock","locking");
    do_something();
```
这种申请锁的方法，涉及到客户端和 Redis 服务器的多次交互，当客户端确定可以加锁的时候，可能这时候锁已经被其他客户端申请了，最终导致两个客户端同时持有锁，互斥的语意非常容易被打破。在 Redis 官方文档描述了一些方法并且参看了网上的文章，好些方法都提及了这个问题。我们会发现，这些方法的共同特点就是申请锁资源的整个过程分散在客户端和服务端，如此很容易出现数据一致性的问题。
 
因此，最好的办法是将“申请/释放锁”的逻辑操作都放在服务器上，Redis Lua 脚本可以胜任。下面给出申请互斥锁的 Lua 脚本：
```
-- apply for lock
local key = KEYS[1]
local res = redis.call('get', key)
 
-- 锁被占用，申请失败
if res == '0' then
return -1
 
-- 锁可以被申请
else
local setres = redis.call('set', key, 0)
if setres['ok'] == 'OK' then
return 0
end
end
return -1
 
get 命令不成功返回(nil).
实验命令：保存lua 脚本redis-cli script load ”$(cat mutex_lock.lua)”
```
释放锁的操作也可以在 Lua 脚本中实现
```
-- releae lock
local key = KEYS[1]
local setres = redis.call('set', key, 1)
if setres['ok'] == 'OK' then
return 0
return -1
```
如上 Lua 脚本基本的锁管理的问题，将锁的管理逻辑放在服务器端，可见 Lua 能拓展 Redis 服务器的功能。但上面的锁管理方案是有问题的
# 死锁的问题
首先是客户端崩溃导致的死锁。按照上面的方法，当某个客户端申请锁后因崩溃等原因无法释放锁，那么其他客户端无法申请锁，会导致死锁。
 
一般，申请锁是为了让多个访问方对某块数据作互斥访问（修改），而我们应该将访问的时间控制在足够短，如果持有锁的时间过长，系统整体的性能肯定是下降的。可以给定一个足够长的超时时间，当访问方超时后尚未释放锁，可以自动把锁释放。
 
Redis 提供了 TTL 功能，键值对在超时后会自动被剔除，在 Redis 的数据集中有一个哈希表专门用作键值对的超时。所以，我们有下面的 Lua 代码：
```
-- apply for lock
local key = KEYS[1]
local timeout = KEYS[2]
 
local res = redis.call('get', key)
 
-- 锁被占用，申请失败
if res == '0' then
return -1
-- 锁可以被申请
else
local setres = redis.call('set', key, 0)
local exp_res = redis.call('pexpire', key, timeout)
if exp_res == 1 then
return 0
end
end
return -1
```
如此能够解决锁持有者崩溃而锁资源无法释放带来的死锁问题。
 
再者是 Redis 服务器崩溃导致的死锁。当管理锁资源的 Redis 服务器宕机了，客户端既无法申请也无法释放锁，死锁形成了。一种解决的方法是设置一个备份 Redis 服务器，当 Redis 主机宕机后，可以使用备份机，但这需要保证主备的数据是同步的，不允许有延迟。
在同步有延迟的情况下，依旧会出现两个客户端同时持有锁的问题。