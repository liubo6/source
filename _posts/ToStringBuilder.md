---
title: ToStringBuilder
date: 2016-10-21 15:20:24
categories: 工具类
tags: 工具类
---
# 简介
org.apache.commons.lang3.builder.ToStringBuilder
ToStringBuilder是用于构建一个类的toString字符串的工具类，提供了多种不同的格式
<!--more-->

# 常用模式
## ToStringBuilder.reflectionToString
```
Person person = new Person("100", "hello", "18267290340", "hangzhou");
System.out.println(ToStringBuilder.reflectionToString(person));//默认模式
System.out.println(ToStringBuilder.reflectionToString(person, ToStringStyle.SHORT_PREFIX_STYLE));//短前缀模式
System.out.println(ToStringBuilder.reflectionToString(person, ToStringStyle.SIMPLE_STYLE));//只是打印值,简单模式
System.out.println(ToStringBuilder.reflectionToString(person, ToStringStyle.NO_FIELD_NAMES_STYLE));//只是打印值
System.out.println(ToStringBuilder.reflectionToString(person, ToStringStyle.MULTI_LINE_STYLE));//多行模式

//output
com.trc.demo.shop.Person@30dae81[userId=100,userName=hello,phone=18267290340,address=hangzhou]

Person[userId=100,userName=hello,phone=18267290340,address=hangzhou]

100,hello,18267290340,hangzhou

com.trc.demo.shop.Person@30dae81[100,hello,18267290340,hangzhou]

com.trc.demo.shop.Person@30dae81[
  userId=100
  userName=hello
  phone=18267290340
  address=hangzhou
]
```
## append
重写toString 方法,只添加需要的参数
```
@Override
    public String toString() {
        return new ToStringBuilder(this)
                .append("userId", userId)
                .append("userName", userName)
                .toString();
    }

Person person = new Person("100", "hello", "18267290340", "hangzhou");
System.out.println(person);
//output
com.trc.demo.shop.Person@30dae81[userId=100,userName=hello]

```
