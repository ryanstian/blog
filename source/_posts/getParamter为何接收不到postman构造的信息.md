---
title: getParamter为何接收不到postman构造的信息
date: 2019-06-28 16:16:00
categories: http
tags: [Tomcat,http]
copyright: true #新增,开启
---

之前发生了这样一件事,由于是用的postman发送的消息,消息体有几种常用构造方式:none,form-data,x-www-form-urlencoded
有一些构造方式通过getParameter方法是获取不到数据的,接下来让我们一起看一下这个问题
<!--more-->

首先我对两种构造方式进行了抓包,看到他们发出去的请求
首先时form-data格式下的Get,Post方式
```
GET http://localhost:8080/TestHttp/HelloWord HTTP/1.1
Content-Type: multipart/form-data; boundary=--------------------------130695699130126180335395
User-Agent: PostmanRuntime/7.15.0
Accept: */*
Cache-Control: no-cache
Postman-Token: 255c0f6d-0296-4095-af08-3a0bc1e1f756
Host: localhost:8080
accept-encoding: gzip, deflate
content-length: 281
Connection: keep-alive

----------------------------130695699130126180335395
Content-Disposition: form-data; name="username"

admin
----------------------------130695699130126180335395
Content-Disposition: form-data; name="password"

123456
----------------------------130695699130126180335395--
```

```
POST http://localhost:8080/TestHttp/HelloWord HTTP/1.1
Content-Type: multipart/form-data; boundary=--------------------------666026373795318990654180
User-Agent: PostmanRuntime/7.15.0
Accept: */*
Cache-Control: no-cache
Postman-Token: f7e5f00a-65d9-4aba-961d-34762e8c410c
Host: localhost:8080
accept-encoding: gzip, deflate
content-length: 281
Connection: keep-alive

----------------------------666026373795318990654180
Content-Disposition: form-data; name="username"

admin
----------------------------666026373795318990654180
Content-Disposition: form-data; name="password"

123456
----------------------------666026373795318990654180--
````

接下来是x-www-form-urlencoded格式下的Get,Post方式
```
GET http://localhost:8080/TestHttp/HelloWord HTTP/1.1
Content-Type: application/x-www-form-urlencoded
User-Agent: PostmanRuntime/7.15.0
Accept: */*
Cache-Control: no-cache
Postman-Token: 7e03a95e-5539-4a2a-b0e7-0fe399a4af28
Host: localhost:8080
accept-encoding: gzip, deflate
content-length: 30
Connection: keep-alive

username=admin&password=123456
```
```
POST http://localhost:8080/TestHttp/HelloWord HTTP/1.1
Content-Type: application/x-www-form-urlencoded
User-Agent: PostmanRuntime/7.15.0
Accept: */*
Cache-Control: no-cache
Postman-Token: 13ed95bb-803d-451c-b046-db4e5eb142b2
Host: localhost:8080
accept-encoding: gzip, deflate
content-length: 30
Connection: keep-alive

username=admin&password=123456
```

根据实验结果,在POST模式下x-www-form-urlencoded才可以通过getParameter获取数据,那么导致这一问题的原因是什么呢?
通过代码,锁定了Tomcat Request类的parseParameters方法
```java
//这里对消息格式进行判断
if ("multipart/form-data".equals(contentType)) {
    parseParts(false);
    success = true;
    return;
}
//在这一行对method进行了判断,如果是POST,则进行接下来的解析,如果是其他的,那么直接返回
if( !getConnector().isParseBodyMethod(getMethod()) ) {
    success = true;
    return;
}
//这里对消息格式进行判断
if (!("application/x-www-form-urlencoded".equals(contentType))) {
    success = true;
    return;
}
```