---
title: ThreadLocal
date: 2016-11-25 11:05:35
categories: java
tags: 多线程
---
ThreadLocal，线程局部变量
在并发编程的时候，成员变量如果不做任何处理其实是线程不安全的，各个线程都在操作同一个变量，显然是不行的，
并且我们也知道volatile这个关键字也是不能保证线程安全的。那么在有一种情况之下，我们需要满足这样一个条件
：变量是同一个，但是每个线程都使用同一个初始值，也就是使用同一个变量的一个新的副本。这种情况之下
ThreadLocal就非常使用，比如说DAO的数据库连接，我们知道DAO是单例的，那么他的属性Connection就不是一个
线程安全的变量。而我们每个线程都需要使用他，并且各自使用各自的。这种情况，ThreadLocal就比较好的解决了这个问题

在 JDK 1.2 的时代，java.lang.ThreadLocal 就诞生了，它是为了解决多线程并发问题而设计的

<!--more-->
# 例子
一个序列号生成器的程序，可能同时会有多个线程并发访问它，要保证每个线程得到的序列号都是自增的，而不能相互干扰。

接口
每次调用 getNumber() 方法可获取一个序列号，下次再调用时，序列号会自增
```
public interface Sequence {
    int getNumber();
}
```
线程类
在线程中连续输出三次线程名与其对应的序列号
```
public class ClientThread extends Thread {
 
    private Sequence sequence;
 
    public ClientThread(Sequence sequence) {
        this.sequence = sequence;
    }
 
    @Override
    public void run() {
        for (int i = 0; i < 3; i++) {
            System.out.println(Thread.currentThread().getName() + " => " + sequence.getNumber());
        }
    }
}
```

普通实现类
序列号初始值是0，在 main() 方法中模拟了三个线程
```
public class SequencesA implements Sequence {
    private static int number = 0;
 
    @Override
    public int getNumber() {
        number = number + 1;
        return number;
    }
 
    public static void main(String[] args) {
        Sequence sequence = new SequencesA();
 
        ClientThread thread1 = new ClientThread(sequence);
        ClientThread thread2 = new ClientThread(sequence);
        ClientThread thread3 = new ClientThread(sequence);
 
        thread1.start();
        thread2.start();
        thread3.start();
    }
}
//运行结果
Thread-0 => 1
Thread-2 => 3
Thread-2 => 5
Thread-1 => 2
Thread-1 => 7
Thread-1 => 8
Thread-2 => 6
Thread-0 => 4
Thread-0 => 9
```
由于线程启动顺序是随机的，所以并不是0、1、2这样的顺序，这个好理解。为什么当 Thread-0 输出了1之后，
而 Thread-2 却输出了3，5呢？线程之间竟然共享了 static 变量！这就是所谓的“非线程安全”问题了。

那么如何来保证“线程安全”呢？对应于这个案例，就是说不同的线程可拥有自己的 static 变量，如何实现呢？
```
public class SequencesB implements Sequence {
 
    private static ThreadLocal<Integer> numberContainer = new ThreadLocal<Integer>() {
        @Override
        protected Integer initialValue() {
            return 0;
        }
    };
 
    public int getNumber() {
        numberContainer.set(numberContainer.get() + 1);
        return numberContainer.get();
    }
 
    public static void main(String[] args) {
        Sequence sequence = new SequencesB();
 
        ClientThread thread1 = new ClientThread(sequence);
        ClientThread thread2 = new ClientThread(sequence);
        ClientThread thread3 = new ClientThread(sequence);
 
        thread1.start();
        thread2.start();
        thread3.start();
    }
}
//运行输出
Thread-1 => 1
Thread-0 => 1
Thread-2 => 1
Thread-0 => 2
Thread-1 => 2
Thread-0 => 3
Thread-2 => 2
Thread-2 => 3
Thread-1 => 3
```
通过 ThreadLocal 封装了一个 Integer 类型的 numberContainer 静态成员变量，并且初始值是0。
再看 getNumber() 方法，首先从 numberContainer 中 get 出当前的值，加1，随后 set 到 
numberContainer 中，最后将 numberContainer 中 get 出当前的值并返回

每个线程相互独立了，同样是 static 变量，对于不同的线程而言，它没有被共享，而是每个线程各一份，
这样也就保证了线程安全。 也就是说，TheadLocal 为每一个线程提供了一个独立的副本！

# ThreadLocal API
```
public void set(T value)：将值放入线程局部变量中
public T get()：从线程局部变量中获取值
public void remove()：从线程局部变量中移除值（有助于 JVM 垃圾回收）since 1.5
protected T initialValue()：返回线程局部变量中的初始值（默认为 null）
public static <S> ThreadLocal<S> withInitial(Supplier<? extends S> supplier) since 1.8
```
为什么 initialValue() 方法是 protected 的呢？就是为了这个方法是要你们来实现的，请给这个线程局部变量一个初始值吧。


自己实现ThreadLocal
ThreadLocal 里面不就是封装了一个 Map
```
public class MyThreadLocal<T> {
 
    private Map<Thread, T> container = Collections.synchronizedMap(new HashMap<Thread, T>());
 
    public void set(T value) {
        container.put(Thread.currentThread(), value);
    }
 
    public T get() {
        Thread thread = Thread.currentThread();
        T value = container.get(thread);
        if (value == null && !container.containsKey(thread)) {
            value = initialValue();
            container.put(thread, value);
        }
        return value;
    }
 
    public void remove() {
        container.remove(Thread.currentThread());
    }
 
    protected T initialValue() {
        return null;
    }
}
```
当您在一个类中使用了 static 成员变量的时候，一定要多问问自己，这个 static 成员变量需要考虑“线程安全”吗？
（也就是说，多个线程需要独享自己的 static 成员变量吗？）如果需要考虑，那就请用 ThreadLocal 吧！

ThreadLocalMap的实现，没有用标准的Map，而是自己实现了一个HashMap（经典hash实现）。这个Map里的Entry类，
采用了弱引用的方式（而不是强引用），这样的好处，就是如果有对象不再使用的时候，就会被系统回收，而不是被这个Map
继续持有引用，而造成系统不可回收，进而使得内存泄露



