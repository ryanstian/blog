---
title: linux安装jdk(非openjdk)
date: 2019-07-01 08:56:00
categories: 
tags: [linux,jdk]
copyright: true #新增,开启
---

1.官网下载压缩包,这里我下载的是解压版不是rpm版本,现在可能需要你登陆才可以下载,自己去注册个账户吧,或者用其他方式得到压缩包
[oracle下载jdk8的网址](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

2.解压压缩包
tar -zxvf 你压缩包的名字.tar.gz

3.安装vim,这是个文本编辑器,你可以把它理解为记事本这种东西,至少我的ubuntu18.04是不自带vim的你可以使用
sudo apt install vim这条命令安装,也可以在命令行输入vim按照他的提示安装

3.修改配置文件,原理跟window一样,只要将路径添加到配置文件中,操作系统就可以检测到我们想要安装的东西
vim /etc/profile(这里需要注意了,要用root权限进行,前面加sudo)

4.打开配置文件后,我们在尾部追加如下内容,vim的操作方式请自行搜索
```
export JAVA_HOME=你的jdk目录,注意是根目录,不是bin目录
export CLASSPATH=$JAVA_HOME/lib/
export PATH=$JAVA_HOME/bin:$PATH
```

5.使操作系统重新加载配置文件,注意需要root权限
source /etc/profile

6.输入java -version出现java版本信息即我们配置成功了