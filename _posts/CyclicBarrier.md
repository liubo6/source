---
title: CyclicBarrier
date: 2016-11-20 15:45:35
categories: 多线程
tags: CyclicBarrier
---
CyclicBarrier是一个同步辅助类，它允许一组线程互相等待，直到到达某个公共屏障点 (common barrier point)。
在涉及一组固定大小的线程的程序中，这些线程必须不时地互相等待，此时 CyclicBarrier 很有用。因为该 barrier 
在释放等待线程后可以重用，所以称它为循环 的 barrier

做个小游戏，点击5次返1元钱，点50次赚多少，怎么写？可以考虑用CyclicBarrier，当然性能会比较差，点击线程不足5个时会阻塞
 
# 常用方法
await(); // 在所有参与者都已经在此 barrier 上调用 await 方法之前，将一直等待。

<!--more-->
```
static Integer count=1;
 
    public static void main(String[] args) {
        ExecutorService executor=Executors.newFixedThreadPool(10);
//      final CyclicBarrier cyclicBarrier=new CyclicBarrier(5);
        final CyclicBarrier cyclicBarrier=new CyclicBarrier(5,new Runnable(){
            public void run() {
                System.out.println("已经赚了"+(Test11.count++)+"元");
 
            }
 
        });
 
 
        Thread thread1=new Thread(new Runnable(){
            public void run() {
                System.out.println("点一下");
                try {
                    cyclicBarrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }
        });
 
        for(int i=0;i<50;i++){
            executor.execute(thread1);
        }
    }
```
或者
```
public class CyclicBarrierTest {
 
    static CyclicBarrier c = new CyclicBarrier(2);
 
    public static void main(String[] args) {
        new Thread(new Runnable() {
 
            @Override
            public void run() {
                try {
                    c.await();
                } catch (Exception e) {
 
                }
                System.out.println(1);
            }
        }).start();
 
        try {
            c.await();
        } catch (Exception e) {
 
        }
        System.out.println(2);
    }
}
```
输出
2
1
或者
1
2
如果把new CyclicBarrier(2)修改成new CyclicBarrier(3)则主线程和子线程会永远等待，因为没有第三个线程执行await方法，
即没有第三个线程到达屏障，所以之前到达屏障的两个线程都不会继续执行。


CyclicBarrier还提供一个更高级的构造函数CyclicBarrier(int parties, Runnable barrierAction)，用于在线程到达屏障时，
优先执行barrierAction，方便处理更复杂的业务场景
```
public class CyclicBarrierTest2 {
 
    static CyclicBarrier c = new CyclicBarrier(2, new A());
 
    public static void main(String[] args) {
        new Thread(new Runnable() {
 
            @Override
            public void run() {
                try {
                    c.await();
                } catch (Exception e) {
 
                }
                System.out.println(1);
            }
        }).start();
 
        try {
            c.await();
        } catch (Exception e) {
 
        }
        System.out.println(2);
    }
 
    static class A implements Runnable {
 
        @Override
        public void run() {
            System.out.println(3);
        }
 
    }
 
}
```
输出
3
1
2

# CyclicBarrier的应用场景
CyclicBarrier可以用于多线程计算数据，最后合并计算结果的应用场景。比如我们用一个Excel保存了用户所有银行流水，
每个Sheet保存一个帐户近一年的每笔银行流水，现在需要统计用户的日均银行流水，先用多线程处理每个sheet里的银行流水，
都执行完之后，得到每个sheet的日均银行流水，最后，再用barrierAction用这些线程的计算结果，计算出整个Excel的日均银行流水

# CyclicBarrier和CountDownLatch的区别
CountDownLatch的计数器只能使用一次。而CyclicBarrier的计数器可以使用reset() 方法重置。所以CyclicBarrier能处理
更为复杂的业务场景，比如如果计算发生错误，可以重置计数器，并让线程们重新执行一次。
CyclicBarrier还提供其他有用的方法，比如getNumberWaiting方法可以获得CyclicBarrier阻塞的线程数量。
isBroken方法用来知道阻塞的线程是否被中断



