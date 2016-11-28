---
title: 不要返回null
date: 2016-11-24 17:05:13
categories: java
tags: null
---

# 问题
经常看到有人为了避免空指针调用，使用这样的语句
```
if(object != null){
    object.doxx();
}
```
或者
```
if(StringUtils.isNotBlank(phone)){
    ...
}
```
<!--more-->
加入一个api，接收id，返回对象信息，id传入为空，不要返回null
直接把错误原因放到assert的参数中，这样不仅能保护你的程序不往下走，而且还能把错误原因返回给调用者
也可以直接抛出空指针异常，此时null是个不合理的参数，有问就应该往外抛

如何避免
# 返回集合类型的
```
返回空集合
Collections.emptyList();
Collections.emptyMap();
Collections.emptySet();
```

# 返回对象的
如果查询不到，对象是空的
```
public class MyParser implements Parser {
  private static Action DO_NOTHING = new Action() {
    public void doSomething() { /* do nothing */ }
  };
 
  public Action findAction(String userInput) {
    // ...
    if ( /* we can't find any actions */ ) {
      return DO_NOTHING;
    }
  }
}


```

# @NotNull注解
```
1.javax.validation.constraints.NotNull
 
运行时进验证，不静态分析
2.edu.umd.cs.findbugs.annotations.NonNull
 
用于finbugs和Sonar静态分析
3.javax.annotation.Nonnull
 
只适用FindBugs,JSR-305不适用
4.org.jetbrains.annotations.NotNull
 
适用用于IntelliJ IDEA静态分析
5.lombok.NonNull
 
适用Lombok项目中代码生成器。不是一个标准的占位符注解.
6.android.support.annotation.NonNull
 
适用于Android项目的标记注解,位于support-annotations包中
```
选择，javax命名空间下的注解，其他的还需要引入jar包依赖，
javax.validation.constraints.NotNull，因为它已经在Java EE 6中定义


