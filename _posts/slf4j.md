---
title: slf4j
date: 2016-10-24 14:17:14
categories: 日志
tags: java
---

# 介绍
Simple Logging Facade for Java (SLF4J)
slf4j 日志接口,通过这个接口，我们可以方便的切换日志的实现框架，比如log4j,commons-logging,logback,jdk-log 等等
slf4j的API在 slf4j-api.jar中，核心接口与类
```
org.slf4j.Logger 
org.slf4j.LoggerFactory  
```
<!--more-->

# 动态绑定
slf4j的实际调用日志在运行时才会动态绑定，基本原理是查找classpath下面的jar包，如果存在slf4j的实现框架，就采用该实现框架
![](http://ww2.sinaimg.cn/mw690/69045600gw1f934kwvnxqj20gs0gktaf.jpg)

# 各种jar
## slf4j-nop.jar


# 新旧迁移
整理一下日志框架迁移的流程
老日志框架：类比与上例的common-logging代码
新日志框架：类比与上例的log4j
适配包：类比与上例的jcl-over-slf4j.jar
对接包：类比与上上例的slf4j-log4j12-xxx.jar
 
 
老日志框架  ---(适配包)--->  SLF4J API ---(对接包)---> 新日志框架

# 使用建议
## 占位符
```
logger.debug("The entry is " + i );
```
改成提高性能
（为了提高运行效率，log4j中往往在输出信息之前，还要进行级别判断，以避免无效的字符串连接操作，slf4j巧妙的解决了这个问题：先传入带有占位符的字符串，同时把其他参数传入，在slf4j的内容部实现中，如果级别合适再去用传入的参数去替换字符串中的占位符，否则不用执行）
```
logger.debug("The entry is {}", entry);
```

## 推荐logback实现类
```
logback的理由：
1. logback比log4j要快大约10倍，而且消耗更少的内存。
2. logback-classic模块直接实现了SLF4J的接口，所以我们迁移到logback几乎是零开销的。
3. logback不仅支持xml格式的配置文件，还支持groovy格式的配置文件。相比之下，Groovy风格的配置文件更加直观，简洁。
4. logback-classic能够检测到配置文件的更新，并且自动重新加载配置文件。
5. logback能够优雅的从I/O异常中恢复，从而我们不用重新启动应用程序来恢复logger。
6. logback能够根据配置文件中设置的上限值，自动删除旧的日志文件。
7. logback能够自动压缩日志文件。
8. logback能够在配置文件中加入条件判断（if-then-else)。可以避免不同的开发环境（dev、test、uat…）的配置文件的重复。
9. logback带来更多的filter。
10. logback的stack trace中会包含详细的包信息。
11. logback-access和Jetty、Tomcat集成提供了功能强大的HTTP-access日志。
配置文件：需要在项目的src目录下建立一个logback.xml。
注：（1）logback首先会试着查找logback.groovy文件；
（2）当没有找到时，继续试着查找logback-test.xml文件；
（3）当没有找到时，继续试着查找logback.xml文件；
（4）如果仍然没有找到，则使用默认配置（打印到控制台）
```

## 常见问题
```
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
```
经常可以看到这个，是因为上下文路径中没有日志实现类
当你添加
```
slf4j-api-1.7.21.jar
slf4j-simple-1.7.21.jar
```
不再显示警告信息

## 可以绑定的日志实现框架
### slf4j-log4j12-1.7.21.jar
Binding for log4j version 1.2, a widely used logging framework. You also need to place log4j.jar on your class path.
### slf4j-jdk14-1.7.21.jar
Binding for java.util.logging, also referred to as JDK 1.4 logging

### slf4j-nop-1.7.21.jar
Binding for NOP, silently discarding all logging.
丢弃日志
### slf4j-simple-1.7.21.jar
Binding for Simple implementation, which outputs all events to System.err. Only messages of level INFO and higher are printed. This binding may be useful in the context of small applications.

### slf4j-jcl-1.7.21.jar
Binding for Jakarta Commons Logging. This binding will delegate all SLF4J logging to JCL
### logback-classic-1.0.13.jar (requires logback-core-1.0.13.jar)
There are also SLF4J bindings external to the SLF4J project, e.g. logback which implements SLF4J natively. Logback's ch.qos.logback.classic.Logger class is a direct implementation of SLF4J's org.slf4j.Logger interface. Thus, using SLF4J in conjunction with logback involves strictly zero memory and computational overhead

## 切换日志框架
To switch logging frameworks, just replace slf4j bindings on your class path. For example, to switch from java.util.logging to log4j, just replace slf4j-jdk14-1.7.21.jar with slf4j-log4j12-1.7.21.jar.
![](http://ww2.sinaimg.cn/mw690/69045600gw1f93bsmzlhej20w00homz4.jpg)