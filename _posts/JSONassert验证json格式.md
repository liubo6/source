---
title: JSONassert验证json格式
date: 2016-11-17 14:10:41
categories: java
tags: json
---
# 需求
越来越多的项目将信息传递格式定义为JSON。在Java中为JSON写测试是很耗费时间的。
[JSONassert](http://jsonassert.skyscreamer.org/)是一个很小的库，减少了处理JSON断言的代码，并提供了更友好的错误提示

# maven依赖
```
<dependency>
  <groupId>org.skyscreamer</groupId>
  <artifactId>jsonassert</artifactId>
  <version>1.4.0</version>
</dependency>
```


<!--more-->

# 使用
语法和junit类似
## 语法
```
JSONAssert.assertEquals(expectedJSON, actualJSON, strictMode);
```
正确
```
 @Test
    public void test() throws Exception {
        //get json from server
        String result = userService.get("/user/liubo.json");
 
        JSONAssert.assertEquals("{id:1,name:\"liubo\"}", result, false);
    }
```

错误
```
 @Test
    public void test() throws Exception {
        //get json from server
        String result = userService.get("/user/liubo.json");
 
        JSONAssert.assertEquals("{id:1,name1:\"liubo\"}", result, false);
    }
```
报错
```
java.lang.AssertionError:
Expected: name
     but none found
```

# 严格模式
严格模式下，字段名称必须匹配存在，顺序无关
```
 @Test
    public void test() throws Exception {
        String result = "{id:1,name:\"liubo\"}";
        JSONAssert.assertEquals("{id:1}", result, false); // Pass
        JSONAssert.assertEquals("{id:1}", result, true); // Fail java.lang.AssertionError:Unexpected: name
    }
```
