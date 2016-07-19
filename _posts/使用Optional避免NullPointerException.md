---
title: 使用Optional避免NullPointerException
date: 2016-07-16 10:10:07
categories: java
tags: java
---
```
对于一个Java开发人员来说，最常见的异常估计就是NullPointerException
```
研究Google代码库,我们发现95%的集合不应该包含任何null值,让这些情况快速失败,而不是默默地接受null对开发人员来说更有用。


<!--more-->

Guava库（Java8中也提供了）中提供了Optional接口来使null快速失败，即在可能为null的对象上做了一层封装
Optional的最常用价值在于，例如，假设一个方法返回某一个数据类型，调用这个方法的代码来根据这个方法的返回值来做下一步的动作，若该方法可以返回一个null值表示成功，或者表示失败，在这里看来都是意义含糊的，所以使用Optional作为返回值，则后续代码可以通过```isPresent()```来判断是否返回了期望的值（原本期望返回null或者返回不为null，其意义不清晰），并且可以使用get()来获得实际的返回值

判断当前用户的用户名是为null,如果用户名为null，我们就叫他游客
code1
```
public void sayHello(String name){
    if(name==null){
        name = "游客";
    }
    System.out.println("Hello, "+name);
}
```

code2
```
import com.google.common.base.Optional;
public void sayHello(String name){
    name = Optional.fromNullable(name).or("游客");
    System.out.println("Hello, "+name);
}
```
Guava中Optional类就是用来强制提醒开发者，注意对Null的判断。迫使你积极思考引用缺失的情况

## 常用方法
```
Optional.of(T)
为Optional赋值，当T为Null直接抛NullPointException,建议这个方法在调用的时候直接传常量，不要传变量
 
Optional.fromNullable(T)
 
为Optional赋值，当T为Null则使用默认值。建议与or方法一起用，风骚无比
 
T or(T)
 
当Optional的值为null时，使用or赋予的值返回。与fromNullable是一对好基友
 
Optional.absent()
 
为Optional赋值，采用默认值
 
T get()
 
当Optional的值为null时，抛出IllegalStateException，返回Optional的值
 
boolean isPresent()
 
如果Optional存在值，则返回True
 
T orNull()
 
当Optional的值为null时，则返回Null。否则返回Optional的值
 
Set asSet()
 
将Optional中的值转为一个Set返回，当然只有一个值啦，或者为空，当值为null时。
```