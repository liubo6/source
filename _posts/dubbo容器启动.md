---
title: dubbo容器启动
date: 2016-11-05 19:22:12
categories: dubbo
tags: java
---
dubbo容器启动，不需要tomcat等web容器
提供了启动方式
```
com.alibaba.dubbo.container
com.alibaba.dubbo.container.spring.SpringContainer
com.alibaba.dubbo.container.jetty.JettyContainer
com.alibaba.dubbo.container.log4j.Log4jContainer
com.alibaba.dubbo.container.logback.LogbackContainer

```
<!--more-->

# 默认启动 只加载spring
```
com.alibaba.dubbo.container.Main
```
# Main 传参启动
```
java com.alibaba.dubbo.container.Main spring jetty log4j 
```
# 通过JVM启动参数传入要加载的容器
java com.alibaba.dubbo.container.Main -Ddubbo.container=spring,jetty,log4j

# 通过classpath下的dubbo.properties配置传入要加载的容器
dubbo.properties
dubbo.container=spring,jetty,log4j 

# 主线程启动
```
/**
* 默认spring容器方式
*/
public class Main {
    public static void main(String[] args) {
        com.alibaba.dubbo.container.Main.main(args);
    }
}
```
# 默认的spring实现类
```
public class SpringContainer implements Container {
 
    private static final Logger logger = LoggerFactory.getLogger(SpringContainer.class);
 
    public static final String SPRING_CONFIG = "dubbo.spring.config";
 
    public static final String DEFAULT_SPRING_CONFIG = "classpath*:META-INF/spring/*.xml";
 
    static ClassPathXmlApplicationContext context;
 
    public static ClassPathXmlApplicationContext getContext() {
        return context;
    }
 
    public void start() {
        String configPath = ConfigUtils.getProperty(SPRING_CONFIG);
        if (configPath == null || configPath.length() == 0) {
            configPath = DEFAULT_SPRING_CONFIG;
        }
        context = new ClassPathXmlApplicationContext(configPath.split("[,\\s]+"));
        context.start();
    }
 
    public void stop() {
        try {
            if (context != null) {
                context.stop();
                context.close();
                context = null;
            }
        } catch (Throwable e) {
            logger.error(e.getMessage(), e);
        }
    }
 
}
```

# 优雅停机
Dubbo是通过JDK的ShutdownHook来完成优雅停机的启动时，需要指定参数 dubbo.shutdown.hook=true
```
if ("true".equals(System.getProperty(SHUTDOWN_HOOK_KEY))) {
                Runtime.getRuntime().addShutdownHook(new Thread() {
                    public void run() {
                        for (Container container : containers) {
                            try {
                                container.stop();
                                logger.info("Dubbo " + container.getClass().getSimpleName() + " stopped!");
                            } catch (Throwable t) {
                                logger.error(t.getMessage(), t);
                            }
                            synchronized (Main.class) {
                                running = false;
                                Main.class.notify();
                            }
                        }
                    }
                });
            }
```
