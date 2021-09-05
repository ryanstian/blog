---
title: IDEA修改java版本号
date: 2019-09-15 18:56:00
categories: IDEA
tags: [IDEA]
copyright: true #新增,开启
---


总共有4处需要修改,直接上图(在后面),如果懒得每次改版本号,也可以利用maven插件
<!--more-->
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>自行找个版本</version>
    <configuration>
        <source>1.8</source>
        <target>1.8</target>
    </configuration>
</plugin>
```
![如果图片失效,请邮件联系作者补图](修改jdk版本01.png)
![如果图片失效,请邮件联系作者补图](修改jdk版本02.png)
![如果图片失效,请邮件联系作者补图](修改jdk版本03.png)
![如果图片失效,请邮件联系作者补图](修改jdk版本04.png)

