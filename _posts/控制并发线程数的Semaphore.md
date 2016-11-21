---
title: 控制并发线程数的Semaphore
date: 2016-11-20 17:46:38
categories: 多线程
tags: Semaphore
---

Semaphore（信号量）是用来控制同时访问特定资源的线程数量，它通过协调各个线程，以保证合理的使用公共资源
把它比作是控制流量的红绿灯，比如XX马路要限制流量，只允许同时有一百辆车在这条路上行使，其他的都必须在路口等待，
所以前一百辆车会看到绿灯，可以开进这条马路，后面的车会看到红灯，不能驶入XX马路，但是如果前一百辆中有五辆车
已经离开了XX马路，那么后面就允许有5辆车驶入马路，这个例子里说的车就是线程，驶入马路就表示线程在执行，离开
马路就表示线程执行完成，看见红灯就表示线程被阻塞，不能执行。

信号量可以用来限制对某个共享资源进行访问的线程的数量。在对资源进行访问之前，线程必须从得到信号量的许可（调用
Semaphore对象的acquire()方法）；在完成对资源的访问后，线程必须向信号量归还许可（调用Semaphore对象的release()方法）

<!--more-->
# 方法
```
acquire() 获取许可
release()向信号量归还许可
int availablePermits() ：返回此信号量中当前可用的许可证数。
int getQueueLength()：返回正在等待获取许可证的线程数。
boolean hasQueuedThreads() ：是否有线程正在等待获取许可证。
void reducePermits(int reduction) ：减少reduction个许可证。是个protected方法。
Collection getQueuedThreads() ：返回所有等待获取许可证的线程集合。是个protected方法。
```

# 应用场景
Semaphore可以用于做流量控制，特别公用资源有限的应用场景，比如数据库连接。假如有一个需求，要读取几万个文件的数据，
因为都是IO密集型任务，我们可以启动几十个线程并发的读取，但是如果读到内存后，还需要存储到数据库中，而数据库的连接
数只有10个，这时我们必须控制只有十个线程同时获取数据库连接保存数据，否则会报错无法获取数据库连接。这个时候，我们
就可以使用Semaphore来做流控

一个计数信号量。从概念上讲，信号量维护了一个许可集。如有必要，在许可可用前会阻塞每一个 acquire()，然后再获取该许可。
每个 release() 添加一个许可，从而可能释放一个正在阻塞的获取者。但是，不使用实际的许可对象，Semaphore 只对可用许可
的号码进行计数，并采取相应的行动。拿到信号量的线程可以进入代码，否则就等待。通过acquire()和release()获取和释放访问许可。

```
public static void main(String[] args) {
        // 线程池
        ExecutorService exec = Executors.newCachedThreadPool();
        // 只能5个线程同时访问
        final Semaphore semp = new Semaphore(5);
        // 模拟10个客户端访问
        for (int index = 0; index < 10; index++) {
            final int NO = index;
            Runnable run = new Runnable() {
                public void run() {
                    try {
                        // 获取许可
                        semp.acquire();
                        System.out.println("Accessing: " + NO);
                        Thread.sleep((long) (Math.random() * 6000));
                        // 访问完后，释放
                         System.out.println("release: " + NO);
                        semp.release();
                        //availablePermits()指的是当前信号灯库中有多少个可以被使用
                        //System.out.println("-----------------" + semp.availablePermits());
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            };
            exec.execute(run);
        }
        // 退出线程池
        exec.shutdown();
    }
```
//输出
```
Accessing: 0
Accessing: 1
Accessing: 2
Accessing: 3
Accessing: 4
release: 2
release: 1
Accessing: 5
Accessing: 6
release: 0
Accessing: 7
release: 3
release: 4
Accessing: 8
Accessing: 9
release: 6
release: 5
release: 7
release: 8
release: 9
```

