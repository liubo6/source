---
title: okHttp使用
date: 2016-07-25 10:34:15
categories: HTTP
tags: HTTP
---
http://square.github.io/okhttp/

okhttp是高性能的http库，支持同步、异步，而且实现了spdy、http2、websocket协议，api很简洁易用，和volley一样实现了http协议的缓存
Android4.4的源码中可以看到HttpURLConnection已经替换成OkHttp实现

<!--more-->
## okHttp主要类
1. OkHttpClient.java
1. Request.java
1. Call.java
1. RequestBody.java
1. Response.java

## maven依赖
```
<dependency>
  <groupId>com.squareup.okhttp3</groupId>
  <artifactId>okhttp</artifactId>
  <version>3.4.1</version>
</dependency>
```

## RequestCST
```
public class RequestCST implements Serializable {
    //mediatype 这个需要和服务端保持一致
    private static final MediaType MEDIA_TYPE_FORM = MediaType.parse("application/x-www-form-urlencoded; charset=utf-8");
    private static final MediaType MEDIA_TYPE_JSON = MediaType.parse("application/json; charset=utf-8");
    private static final MediaType MEDIA_TYPE_MARKDOWN = MediaType.parse("text/x-markdown; charset=utf-8");//mdiatype 这个需要和服务端保持一致

    private static final String BASE_URL = "http://liubo.loan/api";//请求接口根地址
}
```
## Requests（请求）
每一个HTTP请求中都应该包含一个URL，一个GET或POST方法以及Header或其他参数，当然还可以含特定内容类型的数据流。

## Responses（响应）
响应则包含一个回复代码（200代表成功，404代表未找到），Header和定制可选的body。


## 基本使用
### HTTP GET
```
#Request是OkHttp中访问的请求，Builder是辅助类。Response即OkHttp中的响应
OkHttpClient client = new OkHttpClient();

String run(String url) throws IOException {
    Request request = new Request.Builder().url(url).build();
    Response response = client.newCall(request).execute();
    if (response.isSuccessful()) {
        return response.body().string();
    } else {
        throw new IOException("Unexpected code " + response);
    }
}

```
Response类：
```
public boolean isSuccessful()
Returns true if the code is in [200..300), which means the request was successfully received, understood, and accepted
```
response.body()返回ResponseBody类
```
public final String string() throws IOException
Returns the response as a string decoded with the charset of the Content-Type header. If that header is either absent or lacks a charset, this will attempt to decode the response body as UTF-8.
Throws:
IOException

```

###  HTTP POST JSON
```
public static final MediaType JSON = MediaType.parse("application/json; charset=utf-8");
OkHttpClient client = new OkHttpClient();
String post(String url, String json) throws IOException {
    RequestBody body = RequestBody.create(JSON, json);
    Request request = new Request.Builder()
      .url(url)
      .post(body)
      .build();
    Response response = client.newCall(request).execute();
    f (response.isSuccessful()) {
        return response.body().string();
    } else {
        throw new IOException("Unexpected code " + response);
    }
}
```


### Form 表单提交 application/x-www-form-urlencoded
```
 OkHttpClient client = new OkHttpClient();
        FormBody formBody = new FormBody.Builder()
                .add("phone", "18012341234")
                .add("password", "123456")
                .build();
        Request request = new Request.Builder()
                .url(URL)
                .post(formBody)
                .build();
        Response response = client.newCall(request).execute();
        if (response.isSuccessful()) {
            System.out.println(response.body().string());
        } else {
            throw new IOException("Unexpected code " + response);
        }
```
###  HTTP POST 键值对
```
OkHttpClient client = new OkHttpClient();
String post(String url, String json) throws IOException {

    RequestBody formBody = new FormEncodingBuilder()
    .add("platform", "android")
    .add("name", "bug")
    .add("subject", "XXXXXXXXXXXXXXX")
    .build();

    Request request = new Request.Builder()
      .url(url)
      .post(body)
      .build();

    Response response = client.newCall(request).execute();
    if (response.isSuccessful()) {
        return response.body().string();
    } else {
        throw new IOException("Unexpected code " + response);
    }
}

```
