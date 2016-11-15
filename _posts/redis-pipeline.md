---
title: redis.pipeline
date: 2016-11-15 10:47:04
categories: 缓存
tags: redis
---
一般情况下，Redis Client端发出一个请求后，通常会阻塞并等待Redis服务端处理，Redis服务端处理完后请求命令后会将结果通过响应报文返回给Client。
通过pipeline方式当有大批量的操作时候，我们可以节省很多原来浪费在网络延迟的时间，需要注意到是用pipeline方式打包命令发送，
redis必须在处理完所有命令前先缓存起所有命令的处理结果。打包的命令越多，缓存消耗内存也越多。所以并不是打包的命令越多越好。
Redis Pipelining可以一次发送多个命令，并按顺序执行、返回结果，节省RTT(Round Trip Time)。

<!--more-->
使用Pipeline在对Redis批量读写的时候，性能上有非常大的提升

# redis 批量get提高性能-mget
```
List<String> tt = jedis.mget("name","age","address"); 
```
# 测试
```
    @Test
    public void pipelineTest() throws Exception {
        long st = System.currentTimeMillis();
        Pipeline p = jedis.pipelined();
        for (int i = 0; i < 1000000; i++) {
            p.hset("hash", "K_"+i, "V_" + i);
        }
        p.sync();
        System.out.println("dbsize:[" + jedis.dbSize() + "] .. ");
        System.out.println("hashsize:[" + jedis.hlen("hash") + "] .. ");
        System.out.println(System.currentTimeMillis() - st +" ms");
    }
    @Test
    public void withoutPipelineTest() throws Exception {
        long st = System.currentTimeMillis();
        for (int i = 0; i < 1000000; i++) {
            jedis.hset("hash","K_"+i, "V_" + i);
        }
        System.out.println("dbsize:[" + jedis.dbSize() + "] .. ");
        System.out.println("hashsize:[" + jedis.hlen("hash") + "] .. ");
        System.out.println(System.currentTimeMillis() - st +" ms");
 
    }
```
# 结果
```
hash表插入一万条,十万条，一百万条数据
withoutPipelineTest
 
dbsize:[1] ..
hashsize:[10000] ..
419 ms
 
dbsize:[1] ..
hashsize:[100000] ..
4046 ms
 
dbsize:[1] ..
hashsize:[1000000] ..
35352 ms
 
 
pipelineTest
 
dbsize:[1] ..
hashsize:[10000] ..
57 ms
 
dbsize:[1] ..
hashsize:[100000] ..
234 ms
 
dbsize:[1] ..
hashsize:[1000000] ..
2714 ms
 
本地环回接口(loopback interface)的原因RTT会非常短，真实环境下的差距会更大。
结果相差十倍多
```


