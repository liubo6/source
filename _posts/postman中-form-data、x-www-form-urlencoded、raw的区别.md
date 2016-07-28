---
title: postman中 form-data、x-www-form-urlencoded、raw的区别
date: 2016-07-23 17:46:56
categories: 工具
tags: postman
---

例如消息
```
appId=001
orderId=9f74a51eb71e4d48895cb447f9751a01
```

<!--more-->
## form-data
http请求中的multipart/form-data,它会将表单的数据处理为一条消息，以标签为单元，用分隔符分开。既可以上传键值对，也可以上传文件。当上传的字段是文件时，会有Content-Type来表名文件类型；content-disposition，用来说明字段的一些信息；
由于有boundary隔离，所以multipart/form-data既可以上传文件，也可以上传键值对，它采用了键值对的方式，所以可以上传多个文件
```
POST /orders/app/pay HTTP/1.1
Host: localhost:8088
Content-Type: application/json
Cache-Control: no-cache
 
----WebKitFormBoundaryE19zNvXGzXaLvS5C
Content-Disposition: form-data; name="appId"
 
001
----WebKitFormBoundaryE19zNvXGzXaLvS5C
Content-Disposition: form-data; name="orderId"
 
9f74a51eb71e4d48895cb447f9751a01
----WebKitFormBoundaryE19zNvXGzXaLvS5C
```

## x-www-form-urlencoded
application/x-www-from-urlencoded,会将表单内的数据转换为键值对，比如
```
POST /orders/app/pay HTTP/1.1
Host: localhost:8088
Content-Type: application/json
Cache-Control: no-cache
Content-Type: application/x-www-form-urlencoded
 
appId=001&orderId=9f74a51eb71e4d48895cb447f9751a01
```


## raw
可以上传任意格式的文本，可以上传text、json、xml、html等
```
POST /orders/app/pay HTTP/1.1
Host: localhost:8088
Content-Type: application/json
Cache-Control: no-cache
 
{"appId":"001", "orderId":"9f74a51eb71e4d48895cb447f9751a01" }
```
## 区别
multipart/form-data：既可以上传文件等二进制数据，也可以上传表单键值对，最后会转化多条信息
x-www-form-urlencoded：只能上传键值对，并且键值对都是&符号间隔分开的,最后转化为一条消息