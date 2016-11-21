---
title: 等待多线程完成的CountDownLatch
date: 2016-11-20 13:54:18
categories: 多线程
tags: CountDownLatch
---
# 简介
CountDownLatch在完成一组正在其他线程中执行的操作之前，允许一个或多个线程等待其他线程完成操作
CountDownLatch类是一个同步计数器,构造时传入int参数,该参数就是计数器的初始值，每调用一次countDown()方法，
计数器减1,计数器大于0 时，await()方法会阻塞程序继续执行，如果计数到达零，则释放所有等待的线程，利用这种特性，
可以让主线程等待子线程的结束
CountDownLatch类位于java.util.concurrent包下，利用它可以实现类似计数器的功能。比如有一个任务A，它要等待其他
4个任务执行完毕之后才能执行，此时就可以利用CountDownLatch来实现这种功能了

# 构造函数
```
public CountDownLatch(int count);//参数count为计数值
```

<!--more-->
# 主要方法
```
public void await() throws InterruptedException
//调用await()方法的线程会被挂起，它会等待直到count值为0才继续执行

public boolean await( longtimeout, TimeUnit unit)throws InterruptedException
//和await()类似，只不过等待一定的时间后count值还没变为0的话就会继续执行

public void countDown();
//将count值减1
```

# 例子
```
public class Test {
    public static void main(String[] args) {
        final CountDownLatch latch = new CountDownLatch(2);
        new Thread() {
            public void run() {
                try {
                    System.out.println("子线程" + Thread.currentThread().getName() + "正在执行");
                    Thread.sleep(3000);
                    System.out.println("子线程" + Thread.currentThread().getName() + "执行完毕");
                    latch.countDown();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            };
        }.start();
 
        new Thread() {
            public void run() {
                try {
                    System.out.println("子线程" + Thread.currentThread().getName() + "正在执行");
                    Thread.sleep(3000);
                    System.out.println("子线程" + Thread.currentThread().getName() + "执行完毕");
                    latch.countDown();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            };
        }.start();
        try {
            System.out.println("等待2个子线程执行完毕...");
            latch.await();
            System.out.println("2个子线程已经执行完毕");
            System.out.println("继续执行主线程");
        } catch (InterruptedException e) {            e.printStackTrace();        }    }}
```

//输入
线程Thread-0正在执行
线程Thread-1正在执行
等待2个子线程执行完毕...
线程Thread-0执行完毕
线程Thread-1执行完毕
2个子线程已经执行完毕
继续执行主线程

# 例子
```
public class CountDownLatchTest {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newFixedThreadPool(5);
        final CountDownLatch countDownLatch = new CountDownLatch(3);
 
        executor.execute(new Runnable() {
            public void run() {
                try {
                    System.out.println("订机票");
                } finally {
                    countDownLatch.countDown();
                }
            }
        });
        executor.execute(new Runnable() {
            public void run() {
                try {
                    System.out.println("订巴士");
                } finally {
                    countDownLatch.countDown();
                }
            }
        });
        executor.execute(new Runnable() {
            public void run() {
                try {
                    System.out.println("订酒店");
                } finally {
                    countDownLatch.countDown();
                }
            }
        });
 
        try {
            countDownLatch.await();
            System.out.println("可以出发了");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
 
}
```
//输出
订机票
订巴士
订酒店
可以出发了
