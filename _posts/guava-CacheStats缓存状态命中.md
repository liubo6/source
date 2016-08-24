---
title: guava_CacheStats缓存状态命中
date: 2016-08-19 09:03:11
categories: guava
tags: 缓存
---
Guava Cache提供了一种非常简便的方式，用于收集缓存执行的统计信息，需要注意的是，跟踪缓存操作将会带来性能的损失，想要收集缓存的信息，我们只需要在使用CacheBuilder的时候声明我们想要收集统计信息即可
<!--more-->
```
LoadingCache<String,Goods> goodsCache =  CacheBuilder.newBuilder().`recordStats()`
```

想要启用缓存信息的统计，我们唯一要做的就是在builder里面通过recordStats()注册，而想要获取统计的信息，我们只需要通过Cache或LoadingCache调用stats()方法
```
CacheStats cacheStats = cache.stats();
```
可以通过CacheStats获取的一些信息

- 加载缓存条目值所耗费的平均时间；
- 请求的缓存条目的命中率；
- 请求的缓存条目的未命中率；
- 缓存条数被移除的数量；

常用方法
```
requestCount()：返回Cache的lookup方法查找缓存的次数，不论查找的值是否被缓存。
hitCount()：返回Cache的lookup方法命中缓存的次数。
hitRate()：返回缓存请求的命中率，命中次数除以请求次数。
missCount()：返回缓存请求的未命中的次数。
missRate()：返回缓存请求未命中的比率，未命中次数除以请求次数。
loadCount()：返回缓存调用load方法加载新值的次数。
loadSuccessCount()：返回缓存加载新值的成功次数。
loadExceptionCount()：返回缓存加载新值出现异常的次数。
loadExceptionRate()：返回缓存加载新值出现异常的比率。
totalLoadTime()：返回缓存加载新值所耗费的总时间。
averageLoadPenalty()：缓存加载新值的耗费的平均时间，加载的次数除以加载的总时间。
evictionCount()：返回缓存中条目被移除的次数。
minus(CacheStats other)：返回一个新的表示当前CacheStats与传入CacheStats之间差异的CacheStats实例。
plus(CacheStats other)：返回一个新的表示当前CacheStats与传入CacheStats之间总计的CacheStats实例。
```

