---
title: packets.go unexpected EOF
date: 2019-09-08 23:02:00
categories: BUG
tags: []
copyright: true #新增,开启
---

后台mysql报错
[mysql] 2020/09/01 10:30:49 packets.go:36: unexpected EOF

原因：  
mysql服务端和客户端都允许设置idle时间。如果服务端设置1小时，客户端设置2小时。那么当下次访问用到这条链接时，会抛出该错误。  
有的客户端会自动重建链接，有的处理策略则是直接panic

可以试试用心跳解决，如果支持心跳的话