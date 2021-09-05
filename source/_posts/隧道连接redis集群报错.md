---
title: 隧道连接redis集群报错
date: 2019-09-15 14:56:00
categories: BUG
tags: [redis]
copyright: true #新增,开启
---

#### 问题描述
&emsp;java springboot程序访问redis,由于redis集群分布于多个目标服务器上,且均有防火墙阻拦,平时调试都是通过tunnel建立隧道来访问。单个redis通过隧道访问成功,但是redis集群通过隧道访问失败。
<!--more-->
使用的jar包如下
```
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-data-redis</artifactId>
```

单个redis连接时配置如下(已经提前建立好隧道)
```
spring.redis.host: 单个ip
spring.redis.port: 单个端口
```

redis集群连接时配置如下(已经提前建立好隧道)
```
spring.redis.cluster.nodes: localhost:6379
```

按照单个redis连接配置时正常连接,按照集群配置时报错
```
org.springframework.data.redis.ClusterStateFailureException: Could not retrieve cluster information. CLUSTER NODES returned with error.
	- ipA:端口A failed: Could not get a resource from the pool
	- ipB:端口B failed: Could not get a resource from the pool
	- ipC:端口C failed: Could not get a resource from the pool
	- ipD:端口D failed: Could not get a resource from the pool
	- ipE:端口E failed: Could not get a resource from the pool
	- ipF:端口F failed: Could not get a resource from the pool
```

#### 问题排查
因为配置文件配置了生成环境开发环境测试环境等一系列的环境,分别对应不同的集群配置,因为当时只配置了一个ip端口,以为是读取配置文件出现错乱,所以在这方面排查了一段时间

后来通过分析源码发现了问题,我们在配置文件中配置的服务器ip和端口,在集群模式下仅仅相当于入口的作用。即,先与配置文件中的ip端口建立socket链接,然后发送cluster获取集群信息命令,然后断开该socket链接,转而直接跟各个集群服务器建立socket链接,从而导致后续请求不会走我们配置的隧道。也就出现了之前我们只配置了一个ip和端口缺出现了和6个redis均连接失败的情况,误以为时配置读取问题