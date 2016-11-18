---
title: guava_RateLimiter限流
date: 2016-11-18 13:47:12
categories: guava
tags: 限流
---

# guava_RateLimiter简单实例
google开源工具包guava提供了限流工具类RateLimiter，该类基于“令牌桶算法”
>RateLimiter class was recently added to Guava libraries (since 13.0) and it is already among my
favourite tools. 

```
public class RateLimiterTest {
    public static void main(String[] args) {
        noRateLimiter();
        withRateLimiter();
    }
 
    public static void noRateLimiter() {
        Long start = System.currentTimeMillis();
        for (int i = 0; i < 10; i++) {
            System.out.println("no limit " + i);
        }
        Long end = System.currentTimeMillis();
        System.out.println("总耗时 "+(end - start)+" ms");
 
    }
 
    public static void withRateLimiter() {
        Long start = System.currentTimeMillis();
        RateLimiter limiter = RateLimiter.create(10.0); //guava RateLimiter  每秒不超过10个任务被提交
        for (int i = 0; i < 10; i++) {
            limiter.acquire(); // 请求RateLimiter消费令牌, 超过permits会被阻塞
            System.out.println("limit " + i);
        }
        Long end = System.currentTimeMillis();
        System.out.println("总耗时 "+(end - start)+" ms");
    }
}
```
<!--more-->

## 平滑突发限流(SmoothBursty)
RateLimiter.create(5) 表示桶容量为5且每秒新增5个令牌，即每隔200毫秒新增一个令牌；
imiter.acquire()表示消费一个令牌，如果当前桶中有足够令牌则成功（返回值为0），如果桶中没有令牌则暂停一段时间，
比如发令牌间隔是200毫秒，则等待200毫秒后再去消费令牌（如上测试用例返回的为0.198803，差不多等待了200毫秒桶中
才有令牌可用），这种实现将突发请求速率平均为了固定请求速率。
```
  @org.junit.Test
    public void smooth() throws Exception {
        RateLimiter limiter = RateLimiter.create(5);
        System.out.println(limiter.acquire());
        System.out.println(limiter.acquire());
        System.out.println(limiter.acquire());
        System.out.println(limiter.acquire());
        System.out.println(limiter.acquire());
        System.out.println(limiter.acquire());
    }
//output
0.0
0.198803
0.198495
0.200993
0.200177
0.200141
```
limiter.acquire(5)表示桶的容量为5且每秒新增5个令牌，令牌桶算法允许一定程度的突发，所以可以一次性消费5个令牌，
但接下来的limiter.acquire(1)将等待差不多1秒桶中才能有令牌，且接下来的请求也整形为固定速率了。
```
@org.junit.Test
    public void smooth2() throws Exception {
        RateLimiter limiter = RateLimiter.create(5);
        System.out.println(limiter.acquire(5));
        System.out.println(limiter.acquire(1));
        System.out.println(limiter.acquire(1));
    }
//output
0.0
0.998681
0.196358
```

## 平滑预热限流(SmoothWarmingUp)
RateLimiter.create(doublepermitsPerSecond, long warmupPeriod, TimeUnit unit)
permitsPerSecond表示每秒新增的令牌数，warmupPeriod表示在从冷启动速率过渡到平均速率的时间间隔。
```
  @org.junit.Test
    public void smooth3() throws Exception {
 
        RateLimiter limiter = RateLimiter.create(5, 1000, TimeUnit.MILLISECONDS);
        for(int i = 1; i < 5;i++) {
            System.out.println(limiter.acquire());
        }
        Thread.sleep(1000L);
        for(int i = 1; i < 5;i++) {
            System.out.println(limiter.acquire());
        }
 
    }
//output
0.0
0.518598
0.356984
0.219566
0.0
0.51992
0.35955
0.219104
```
速率是梯形上升速率的，也就是说冷启动时会以一个比较小的速率慢慢到平均速率


# 令牌桶(Token Bucket)和漏桶(leaky bucket)
互联网服务赖以生存的根本是流量, 产品和运营会经常通过各种方式来为应用倒流,比如淘宝的双十一等,如何让系统在处理高并发的同时
还是保证自身系统的稳定,通常在最短时间内提高并发的做法就是加机器,但是如果机器不够怎么办?那就需要做业务降级或系统限流,流量
控制中用的比较多的两个算法就是漏桶和令牌桶
漏桶算法和令牌桶算法最明显的区别是令牌桶算法允许流量一定程度的突发。因为默认的令牌桶算法，取走token是不需要耗费时间的，
也就是说，假设桶内有100个token时，那么可以瞬间允许100个请求通过。
令牌桶算法由于实现简单，且允许某些流量的突发，对用户友好，所以被业界采用地较多。

## 令牌桶算法(Token bucket)
![](http://ww2.sinaimg.cn/mw690/69045600gw1f9w60rdzyzj20dh0csaau.jpg)
令牌桶算法是一个存放固定容量令牌的桶，按照固定速率往桶里添加令牌
### 算法描述
1. 每秒会有 r 个令牌放入桶中，或者说，每过 1/r 秒桶中增加一个令牌
1. 桶中最多存放 b 个令牌，如果桶满了，新放入的令牌会被丢弃
1. 当一个 n 字节的数据包到达时，消耗 n 个令牌，然后发送该数据包
1. 如果桶中可用令牌小于 n，则该数据包将被缓存或丢弃


## 漏桶算法(Leaky bucket)
![](http://ww1.sinaimg.cn/mw690/69045600gw1f9w60qses4j20e50gq74h.jpg)
漏桶作为计量工具（The Leaky Bucket Algorithm as a Meter）时，可以用于流量整形（Traffic Shaping）和流量控制（TrafficPolicing）
### 算法描述
1. 一个固定容量的漏桶，按照常量固定速率流出水滴；
1. 如果桶是空的，则不需流出水滴；
1. 可以以任意速率流入水滴到漏桶；
1. 如果流入水滴超出了桶的容量，则流入的水滴溢出了（被丢弃），而漏桶容量是不变的。

## 令牌桶和漏桶对比
令牌桶是按照固定速率往桶中添加令牌，请求是否被处理需要看桶中令牌是否足够，当令牌数减为零时则拒绝新的请求；
漏桶则是按照常量固定速率流出请求，流入请求速率任意，当流入的请求数累积到漏桶容量时，则新流入的请求被拒绝；
令牌桶限制的是平均流入速率（允许突发请求，只要有令牌就可以处理，支持一次拿3个令牌，4个令牌），并允许一定程度突发流量；
漏桶限制的是常量流出速率（即流出速率是一个固定常量值，比如都是1的速率流出，而不能一次是1，下次又是2），从而平滑突发流入速率；
令牌桶允许一定程度的突发，而漏桶主要目的是平滑流入速率；
两个算法实现可以一样，但是方向是相反的，对于相同的参数得到的限流效果是一样的。

## 计数器
使用计数器来进行限流，主要用来限制总并发数，比如数据库连接池、线程池、秒杀的并发数；只要全局总请求数或者一定时间段的总
请求数设定的阀值则进行限流，是简单粗暴的总数量限流，而不是平均速率限流

# 应用级限流
## 限流总并发/连接/请求数
Tomcat，Connector 配置
acceptCount：如果Tomcat的线程都忙于响应，新来的连接会进入队列排队，如果超出排队大小，则拒绝连接
maxConnections： 瞬时最大连接数，超出的会排队等待；
maxThreads：Tomcat能启动用来处理请求的最大线程数，如果请求处理量一直远远大于最大线程数则可能会僵死。
## 限流某个接口的总并发/请求数
如果接口可能会有突发访问情况，但又担心访问量太大造成崩溃，如抢购业务；这个时候就需要限制这个接口的总并发/请求数总请求数了；
因为粒度比较细，可以为每个接口都设置相应的阀值。可以使用Java中的AtomicLong进行限流
```
try {
  if(atomic.incrementAndGet() > 限流数) {
      //拒绝请求
  }
  //处理请求
} finally {
  atomic.decrementAndGet();
}
```
## 限流某个接口的时间窗请求数
如想限制某个接口/服务每秒/每分钟/每天的请求数/调用量。
```
oadingCache<Long, AtomicLong> counter =
        CacheBuilder.newBuilder()
                .expireAfterWrite(2, TimeUnit.SECONDS)
                .build(new CacheLoader<Long, AtomicLong>() {
                    @Override
                    public AtomicLong load(Long seconds) throws Exception {
                        return new AtomicLong(0);
                    }
                });
long limit = 1000;
while(true) {
    //得到当前秒
    long currentSeconds = System.currentTimeMillis() / 1000;
    if(counter.get(currentSeconds).incrementAndGet() > limit) {
        System.out.println("限流了:" + currentSeconds);
        continue;
    }
    //业务处理
}
使用Guava的Cache来存储计数器，过期时间设置为2秒（保证1秒内的计数器是有的），然后我们获取
当前时间戳然后取秒数来作为KEY进行计数统计和限流
```

# 分布式限流
分布式限流最关键的是要将限流服务做成原子化，而解决方案可以使使用redis+lua或者nginx+lua技术进行实现，
通过这两种技术可以实现的高并发和高性能
## redis+lua
```
local key = KEYS[1] --限流KEY（一秒一个）
local limit = tonumber(ARGV[1])        --限流大小
local current = tonumber(redis.call('get', key) or "0")
if current + 1 > limit then --如果超出限流大小
return 0
else  --请求数+1，并设置2秒过期
redis.call("INCRBY", key,"1")
redis.call("expire", key,"2")
return 1
end
```
java
```
public static boolean acquire() throws Exception {
    String luaScript = Files.toString(new File("limit.lua"), Charset.defaultCharset());
    Jedis jedis = new Jedis("192.168.147.52", 6379);
    String key = "ip:" + System.currentTimeMillis()/ 1000; //此处将当前时间戳取秒数
    Stringlimit = "3"; //限流大小
    return (Long)jedis.eval(luaScript,Lists.newArrayList(key), Lists.newArrayList(limit)) == 1;
}

```

## Nginx+Lua实现
```
local locks = require "resty.lock"
local function acquire()
    local lock =locks:new("locks")
    local elapsed, err =lock:lock("limit_key") --互斥锁
    local limit_counter =ngx.shared.limit_counter --计数器
 
    local key = "ip:" ..os.time()
    local limit = 5 --限流大小
    local current =limit_counter:get(key)
 
    if current ~= nil and current + 1> limit then --如果超出限流大小
       lock:unlock()
       return 0
    end
    if current == nil then
       limit_counter:set(key, 1, 1) --第一次需要设置过期时间，设置key的值为1，过期时间为1秒
    else
        limit_counter:incr(key, 1) --第二次开始加1即可
    end
    lock:unlock()
    return 1
end
ngx.print(acquire())
```
实现中我们需要使用lua-resty-lock互斥锁模块来解决原子性问题(在实际工程中使用时请考虑获取锁的超时问题)，并使用
ngx.shared.DICT共享字典来实现计数器。如果需要限流则返回0，否则返回1。使用时需要先定义两个共享字典（分别用来存放锁和计数器数据）
```
http {
  ……
  lua_shared_dict locks 10m;
  lua_shared_dict limit_counter 10m;
}
```

http://jinnianshilongnian.iteye.com/blog/2305117
https://en.wikipedia.org/wiki/Token_bucket
https://en.wikipedia.org/wiki/Leaky_bucket

