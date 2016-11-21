---
title: Java-静态代码块，构造代码块，构造函数
date: 2016-11-21 10:38:22
categories: java
tags: java
---
静态代码块：用staitc声明，jvm加载类时执行，仅执行一次
构造代码块：类中直接用{}定义，每一次创建对象时执行。
执行顺序优先级：静态块, main()，函数，构造块，构造方法

<!--more-->
# 静态代码块
```
static {//静态代码块   
}
```
1. 它是随着类的加载而执行，只执行一次，并优先于主函数。具体说，静态代码块是由类调用的。类调用时，先执行静态代码块，
然后才执行主函数的
1. 静态代码块其实就是给类初始化的，而构造代码块是给对象初始化的
1. 静态代码块中的变量是局部变量，与普通函数中的局部变量性质没有区别
1. 一个类中可以有多个静态代码块

静态常量的初始化和静态代码块的执行级别是一样的，所以根据他们在代码中的顺序执行
```
public class Test{
staitc int cnt=6;//第一步
static{//第二步
      cnt+=9;
}
public static void main(String[] args) {
//第四步
      System.out.println(cnt);
}
static{//第三步
      cnt/=3;
}
}
运行结果：
5
```

···
```
public class Test{
public Test(){//构造函数
     System.out.println("test的构造函数");   
}
{//构造代码块
     System.out.println("test的构造代码块");   
}
static {//静态代码块
     System.out.println("test的静态代码块");       
}
public static void main(String[] args) {
}
}
运行结果：
test的静态代码块

```
···

```
public class Test{
public Test(){//构造函数
     System.out.println("test的构造函数");   
}
{//构造代码块
     System.out.println("test的构造代码块");   
}
static {//静态代码块
     System.out.println("test的静态代码块");       
}
public static void main(String[] args) {
     Test=new Test();   
}
}
运行结果：
test的静态代码块
test的构造代码块
test的构造函数
```
···
```
public cltestss Test {
public Test(){//构造函数
     System.out.println("test的构造函数");   
}
{//构造代码块
     System.out.println("test的构造代码块");   
}
sttesttic {//静态代码块
     System.out.println("test的静态代码块");       
}
public sttesttic void mtestin(String[] testrgs) {
     Test test=new Test();
     Test b=new Test();
}
}
运行结果：
test的静态代码块
test的构造代码块
test的构造函数
test的构造代码块
test的构造函数
```
# 构造代码块
```
{//构造代码块   
}
```
构造代码块的作用是给对象进行初始化
对象一建立就运行构造代码块了，而且优先于构造函数执行。这里要强调一下，有对象建立，才会运行构造代码块，
类不能调用构造代码块的，而且构造代码块与构造函数的执行顺序是前者先于后者执行。
构造代码块与构造函数的区别是：构造代码块是给所有对象进行统一初始化，而构造函数是给对应的对象初始化，
因为构造函数是可以多个的，运行哪个构造函数就会建立什么样的对象，但无论建立哪个对象，都会先执行相同的构造代码块。
也就是说，构造代码块中定义的是不同对象共性的初始化内容。

# 构造函数
```
public Test(){//构造函数
    }
```
对象一建立，就会调用与之相应的构造函数，不建立对象，构造函数是不会运行的。
构造函数的作用是用于给对象进行初始化
一个对象建立，构造函数只运行一次，而一般方法可以被该对象调用多次

对于一个类而言，按照如下顺序执行：
执行静态代码块
执行构造代码块
执行构造函数

---

```
public class HelloA {
    public HelloA(){//构造函数
        System.out.println("A的构造函数");   
    }
    {//构造代码块
        System.out.println("A的构造代码块");   
    }
    static {//静态代码块
        System.out.println("A的静态代码块");       
    }
}
public class HelloB extends HelloA{
    public HelloB(){//构造函数
        System.out.println("B的构造函数");   
    }
    {//构造代码块
        System.out.println("B的构造代码块");   
    }
    static {//静态代码块
        System.out.println("B的静态代码块");       
    }
    public static void main(String[] args) {
        HelloB b=new HelloB();       
    }
}
运行结果：
A的静态代码块
B的静态代码块
A的构造代码块
A的构造函数
B的构造代码块
B的构造函数
```
当涉及到继承时，按照如下顺序执行：
 
执行父类的静态代码块，并初始化父类静态成员变量
执行子类的静态代码块，并初始化子类静态成员变量
执行父类的构造代码块，执行父类的构造函数，并初始化父类普通成员变量
执行子类的构造代码块， 执行子类的构造函数，并初始化子类普通成员变量


