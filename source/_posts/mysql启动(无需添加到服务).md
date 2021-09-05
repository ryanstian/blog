---
title: mysql启动(无需添加到服务)
date: 2019-06-22 16:02:00
categories: 
tags: [mysql]
copyright: true #新增,开启
---

已经安装配置好mysql,无需将mysql添加到服务项中即可启动
<!--more-->
1.打开cmd,通过cd到mysql安装/解压文件夹下

2.调用bin下的mysqld.exe文件(如果是linux则可能是.sh)

3.参数为my.ini/my.cnf

4.具体命令为

&emsp;bin/mysqld --defaults-file=./my.ini

5.输入该命令后cmd应该会挂起,此时mysql已经启动成功。如果关闭cmd命令行那么mysql关闭