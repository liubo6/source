---
title: BTrace介绍
date: 2016-08-14 10:00:04
categories: 工具
tags: 性能
---
# 需求
1. 你是否有时发现线上的代码运行效率不高，但却无法定位到底具体是哪一段代码
1. 你是否有时在测试环境中苦苦测试，最后却发现测试环境和生产环境差距太大而导致测试结果不可信
<!--more-->

# 简介
>BTrace是sun推出的一款java性能监控工具，利用java agent 和 jvm attach技术来实现运行时java程序的跟踪和替换，可以不停机的情况下监控线上情况，并且做到最少的侵入，占用最少的系统资源。BTrace的脚本是用纯java编写的，基于一套官方提供的annotation，使跟踪逻辑实现起来异常简单
[btrace github地址](https://github.com/btraceio/btrace/releases)

# 安装
下载,解压,设置好BTRACE_HOME后将bin目录加入至环境变量PATH中即可

```
vim /etc/profile
BTRACE_HOME=/root/btrace/bin
export PATH=$PATH:$BTRACE_HOME/bin
```
# 使用
## 语法
```
btrace [-I <include-path>] [-p <port>] [-cp <classpath>] <pid> <btrace-script> [<args>]
```
## 说明
```
-I:没有这个表明跳过预编译
include-path:指定用来编译脚本的头文件路径(关于预编译可参考例子ThreadBean.java)
port:btrace agent端口,默认是2020
classpath:编译所需类路径,一般是指btrace-client.jar等类所在路径
pid:java进程id
btrace-script:btrace脚本可以是.java文件,也可以是.class文件
args:传递给btrace脚本的参数, 在脚本中可以通过$(), $length()来获取这些参数(定义在BTraceUtils中)
```
## 方法注解说明
```
@OnMethod:指定使用当前注解的方法应该在什么情况下触发,claszz属性指定要匹配的类的全限定类名,可以用正则表达式:/类名的Pattern/匹配,用”+类名”匹配所有子类,用”@某某注解”匹配用该注解注解过的类method属性指定要匹配的方法名称,可以用正则表达式:/方法名称的Pattern/匹配type属性:void(java.lang.String)可以用于匹配:public void funcName(String param) throws Exception,location属性用@Location来表明,匹配了clazz,method情况,在方法执行的何时去执行脚本(前,后,异常,行,某个方法调用)
@OnTimer:指定一个定时任务
@OnExit:当脚本运行Sys.exit(code)时触发
@OnError:当脚本运行抛出异常时触发
@OnEvent:脚本运行时Ctrl+C可以发送事件
@OnLowMemory:让你指定一个阀值,内存低于阀值触发
@OnProbe:可以用一个xml文件来描述你想在什么时候触发该方法
```
## 方法参数说明
```
@Self:目标对象本身
@Retrun:目标程序方法返回值(Kind.RETURN)
@ProbeClassName:目标类名
@ProbeMethodName:目标方法名
@targetInstance:@Location指定的clazz,method的目标(Kind.CALL)
@targetMethodOrField:@Location指定的clazz,method的目标的方法或字段(Kind.CALL)
@Duration:目标方法执行时间,单位是纳秒,需要与 Kind.RETURN 或者 Kind.ERROR 一起使用
```

## 例子
监控方法耗时
```
 import static com.sun.btrace.BTraceUtils.name; 
 import static com.sun.btrace.BTraceUtils.print; 
 import static com.sun.btrace.BTraceUtils.println; 
 import static com.sun.btrace.BTraceUtils.probeClass; 
 import static com.sun.btrace.BTraceUtils.probeMethod; 
 import static com.sun.btrace.BTraceUtils.str; 
 import static com.sun.btrace.BTraceUtils.strcat; 
 import static com.sun.btrace.BTraceUtils.timeMillis; 
 
 import com.sun.btrace.annotations.BTrace; 
 import com.sun.btrace.annotations.Kind; 
 import com.sun.btrace.annotations.Location; 
 import com.sun.btrace.annotations.OnMethod; 
 import com.sun.btrace.annotations.TLS; 
 
  /**
  * 监控方法耗时
  * 
  * @authorliubo
  */ 
 @BTrace 
 public class PrintTimes { 
 
     /**
      * 开始时间
      */ 
     @TLS 
     private static long startTime = 0; 
 
     /**
      * 方法开始时调用
      */ 
     @OnMethod(clazz = "/com\\.trc\\../", method = "/.+/") 
     public static void startMethod() { 
         startTime = timeMillis(); 
     } 
 
     /**
      * 方法结束时调用<br>
      * Kind.RETURN这个注解很重要
      */ 
     @SuppressWarnings("deprecation") 
     @OnMethod(clazz = "/com\\.trc\\../", method = "/.+/", location = @Location(Kind.RETURN)) 
     public static void endMethod() { 
 
         print(strcat(strcat(name(probeClass()), "."), probeMethod())); 
         print("  ["); 
         print(strcat("Time taken : ", str(timeMillis() - startTime))); 
         println("]"); 
     } 
 } 
 
```

```
btrace <pid> PrintTimes.java >time.log
pid为你想要跟踪的Java程序的pid,PrintTimes.java为BTrace的脚本
```

