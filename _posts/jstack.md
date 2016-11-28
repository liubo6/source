---
title: jstack
date: 2016-11-26 16:10:45
categories: java
tags: jstack
---
jstack - Stack Trace
jstack用于打印出给定的java进程ID或core file或远程调试服务的Java堆栈信息

# 语法
```
jstack [ option ] pid
jstack [ option ] executable core
jstack [ option ] [server-id@]remote-hostname-or-IP
```

<!--more-->
# 参数
```
pid
process id for which the stack trace is to be printed. The process must be a Java process. 
To get a list of Java processes running on a machine, jps may be used.
pid 需要被打印配置信息的java进程id,可以用jps查询

executable
Java executable from which the core dump was produced.
产生core dump的java可执行程序

core
core file for which the stack trace is to be printed.
将被打印信息的core dump文件

remote-hostname-or-IP
remote debug server's (see jsadebugd) hostname or IP address.
远程debug服务的主机名或ip
server-id
optional unique id, if multiple debug servers are running on the same remote host.
server-id 唯一id,假如一台主机上多个远程debug服务
```
基本参数

-F 当’jstack [-l] pid’没有相应的时候强制打印栈信息
-l 长列表. 打印关于锁的附加信息,例如属于java.util.concurrent的ownable synchronizers列表.
-m 打印java和native c/c++框架的所有栈信息.
-h | -help 打印帮助信息
pid 需要被打印配置信息的java进程id,可以用jps查询

# 线程分析
一般情况下，通过jstack输出的线程信息主要包括：jvm自身线程、用户线程等。其中jvm线程会在jvm启动时就会存在。
对于用户线程则是在用户访问时才会生成
## JVM线程
在线程中，有一些 JVM内部的后台线程，来执行譬如垃圾回收，或者低内存的检测等等任务，这些线程往往在JVM初始化的时候就存在
```
"Service Thread" #7 daemon prio=9 os_prio=0 tid=0x00007fa59c0bd000 nid=0x50bb runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE
 
"C1 CompilerThread1" #6 daemon prio=9 os_prio=0 tid=0x00007fa59c0b8000 nid=0x50b7 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE
 
"C2 CompilerThread0" #5 daemon prio=9 os_prio=0 tid=0x00007fa59c0b5000 nid=0x50b5 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE
 
"Signal Dispatcher" #4 daemon prio=9 os_prio=0 tid=0x00007fa59c0b3800 nid=0x50b4 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE
```

## 用户线程
会根据用户请求的不同而发生变化。该类线程的运行情况往往是我们所关注的重点。而且这一部分也是最容易产生死锁的地方
```
"pool-3-thread-1" #64 prio=5 os_prio=0 tid=0x00007fa58804b800 nid=0x56ee waiting on condition [0x00007fa547af9000]
   java.lang.Thread.State: TIMED_WAITING (parking)
    at sun.misc.Unsafe.park(Native Method)
    - parking to wait for  <0x00000000f1d24d88> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
    at java.util.concurrent.locks.LockSupport.parkNanos(LockSupport.java:215)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.awaitNanos(AbstractQueuedSynchronizer.java:2078)
    at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(ScheduledThreadPoolExecutor.java:1093)
    at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(ScheduledThreadPoolExecutor.java:809)
    at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1067)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1127)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
    at java.lang.Thread.run(Thread.java:745)
 
"AMQP Connection 121.43.116.29:5672" #63 prio=5 os_prio=0 tid=0x00007fa58804a800 nid=0x56ed runnable [0x00007fa547bfa000]
   java.lang.Thread.State: RUNNABLE
    at java.net.SocketInputStream.socketRead0(Native Method)
    at java.net.SocketInputStream.socketRead(SocketInputStream.java:116)
    at java.net.SocketInputStream.read(SocketInputStream.java:170)
    at java.net.SocketInputStream.read(SocketInputStream.java:141)
    at java.io.BufferedInputStream.fill(BufferedInputStream.java:246)
    at java.io.BufferedInputStream.read(BufferedInputStream.java:265)
    - locked <0x00000000f1d19130> (a java.io.BufferedInputStream)
    at java.io.DataInputStream.readUnsignedByte(DataInputStream.java:288)
    at com.rabbitmq.client.impl.Frame.readFrom(Frame.java:95)
    at com.rabbitmq.client.impl.SocketFrameHandler.readFrame(SocketFrameHandler.java:131)
    - locked <0x00000000f1d19110> (a java.io.DataInputStream)
    at com.rabbitmq.client.impl.AMQConnection$MainLoop.run(AMQConnection.java:508)
```
从上述的代码示例中我们可以看到该用户线程的以下几类信息
线程的状态：waiting on condition(等待条件发生),runnable（正在运行）
线程的调用情况
线程对资源的锁定情况

线程状态
Runnable
该状态表示线程具备所有运行条件，在运行队列中准备操作系统的调度，或者正在运行
Waiton condition
该状态出现在线程等待某个条件的发生。具体是什么原因，可以结合stacktrace来分析。最常见的情况是线程在等待网络的读写，比如当网络数据
没有准备好读时，线程处于这种等待状态，而一旦有数据准备好读之后，线程会重新激活，读取并处理数据。在 Java引入 NIO之前，对于每个网络
连接，都有一个对应的线程来处理网络的读写操作，即使没有可读写的数据，线程仍然阻塞在读写操作上，这样有可能造成资源浪费，而且给操作系统
的线程调度也带来压力。在 NIO里采用了新的机制，编写的服务器程序的性能和可扩展性都得到提高



参考 [oracle 文档](http://docs.oracle.com/javase/7/docs/technotes/tools/share/jstack.html)

