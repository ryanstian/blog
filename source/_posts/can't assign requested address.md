---
title: can't assign requested address,i/o timeout,too many open files in system
date: 2020-07-30 00:05:00
categories: BUG
tags: [go,mac]
copyright: true #新增,开启
---

<!--more-->
#### 状况一:can't assign requested address 
go用了mysql连接池,高并发压测时出现该报错

分析:该报错是由于mac自身fd句柄打开数限制,并且mysql连接池并非一开始就创建出连接,而是随着请求进来才创建连接,大量并发同时进入,占满了mac的fd,导致mysql连接池无法使用fd,无法分配新的连接

结论:修改mac参数限制/上服务器测
临时修改方式,命令行直接输入
永久修改方式,写入/etc/sysctl.conf后重启

参数详情(懂的话建议按需修改,不需要全改):
```
sysctl kern.maxfiles 查看系统所能打开的最大文件数（文件描述符）(全局)
sudo sysctl -w kern.maxfiles 修改
sysctl kern.maxfilesperproc 查看系统所能打开的最大文件数（文件描述符）(一个进程)
sudo sysctl -w kern.maxfilesperproc 修改
ulimit －n 查看shell能打开的最大文件数
ulimit -n 123123 修改(123123为任意值,但不得大于kern.maxfilesperproc)
sysctl net.inet.ip.portrange 查看端口范围
sysctl -w net.inet.ip.portrange.first=32768 修改
```

#### 状况二:dial tcp 127.0.0.1:3306:i/o timeout

分析:待补充

结论:解决方式同状况一

#### 状况三:too many open files in system

分析:原因同状况一

结论:解决方式同状况一
