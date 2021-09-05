---
title: Exception thrown when sending a message with key='null'
date: 2019-08-25 22:16:00
categories: BUG
tags: [kafka]
copyright: true #新增,开启
---

#### 报错
```
2019-08-20 18:45:09 [nioEventLoopGroup-3-15] ERROR o.s.k.s.LoggingProducerListener - Exception thrown when sending a message with key='null' and payload='xxxxxxxxxxxxxx' to topic abc-event:
org.apache.kafka.common.errors.TimeoutException: Failed to update metadata after 60000 ms.
```

#### 原因
&emsp;连接的远程kafka,服务器防火墙没开