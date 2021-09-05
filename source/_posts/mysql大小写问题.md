---
title: mysql大小写问题
date: 2019-07-02 12:31:00
categories: 
tags: [mysql]
copyright: true #新增,开启
---

直接上报错,简单来讲就是报错说表没找到
<!--more-->
```
2019-07-02 12:26:48.782  WARN 16022 --- [nio-8080-exec-2] o.h.engine.jdbc.spi.SqlExceptionHelper   : SQL Error: 1146, SQLState: 42S02
2019-07-02 12:26:48.782 ERROR 16022 --- [nio-8080-exec-2] o.h.engine.jdbc.spi.SqlExceptionHelper   : Table 'summertrain.Market_good' doesn't exist
org.springframework.dao.InvalidDataAccessResourceUsageException: could not execute statement; SQL [n/a]; nested exception is org.hibernate.exception.SQLGrammarException: could not execute statement
        ...
Caused by: org.hibernate.exception.SQLGrammarException: could not execute statement
        ... 74 more
Caused by: java.sql.SQLSyntaxErrorException: Table 'summertrain.Market_good' doesn't exist
        ... 92 more

```


由于团队成员都会先从本地进行调试,本地调试成功就会推送到远程服务器让其自动部署
本次情况发生时,成员本地调试通过,远程确报错如上内容

这是由于大小写问题引起的,写这个错误的成员,其本地数据库大小写不敏感,而我们远程服务器使用的mysql大小写敏感,从而summertrain.Market_good,M大写导致出现异常
在更改其所有大写M为小写m后,推送到服务器,测试了可以正常使用

如果不想mysql对大小写敏感的话,可以在my.ini配置文件的字段mysqld下增加：lower_case_table_names=1(0表示大小写敏感,1表示不敏感,2表示存储时按大小写,比较时统一按小写比较)。如果我们设置表名大小写的话,需要操作系统支持,例如有的操作系统对于文件名是不区分大小写的,此时设置0会导致mysql启动异常