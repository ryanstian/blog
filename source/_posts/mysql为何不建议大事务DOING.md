---
title: mysql为何不建议大事务
date: 2021-09-05 23:54:00
categories: mysql
tags: [mysql]
copyright: true #新增,开启
---

业务中出现过高并发场景下，大事务无限死锁回滚。本文分析下为何不建议大事务
<!--more-->
DOING
当时出现异常的场景为：上下游通过kafka解耦，我方消费kafka时启动大事务进行处理，消息涉及到对某些条目的增删改查  

排查后异常原因：间隙锁冲突+其他锁冲突



