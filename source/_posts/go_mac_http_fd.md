---
title: go_mac_http_fd
date: 2020-07-30 00:06:00
categories: BUG
tags: [go,mac]
copyright: true #新增,开启
---

<!--more-->
用jmeter压测go web项目时突然新的请求被阻塞无法被执行,通过命令lsof -nP -i -l查看,发现大量类似这样信息,此时jmeter已经stop,客户端与服务端之间并不存在存活的请求
#### 状况一: TCP[IP]PORT->[IP]PORT (CLOSED)
链接断开
我们代码客户端和mysql服务端的连接时存在超时时间的，如果一定时间内没有使用，则会断开。  
当时用到mysql客户端包，并不会在断开时帮我们自动重建连接

#### 状况二:TCP[IP]PORT->[IP]PORT (ESTABLISHED)
结论:正常现象,表示连接状态的socket