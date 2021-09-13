---
title: dockerfile笔记
date: 2021-09-12 13:38:00
categories: docker
tags: []
copyright: true #新增,开启
---

编写自己的dockerfile
<!--more-->
## 编写镜像

示例1：
```
//你是基于哪个已存在的镜像搭建的
FROM centos
//制作者名字(可选)
MAINTAINER ryans
//RUN 后面跟着的是创建镜像时，容器内要使用的命令
RUN yum update -y
RUN yum upgrade -y
RUN yum intall inetutils-ping -y
RUN yum install cat -y
RUN yum install vim -y
```

示例2：
```
FROM centos:7
MAINTAINER ryans

RUN yum install openssh-server -y
RUN yum install openssh-clients -y

RUN ssh-keygen -f /etc/ssh/ssh_host_rsa_key
//声明22端口，在docker run时会随机映射Expose端口
EXPOSE 22
//RUN 表示创建容器时执行的命令， CMD表示运行容器时执行的命令
//CMD 命令最多仅能存在一个，多余的会被最后一条覆盖
//docker容器遵循一个容器做一件事的原则，所以CMD内应该为该容器所负责的服务启动命令
CMD ["/usr/sbin/sshd", "-D"]
```

示例3：
```
FROM centos7-ssh
//将宿主机文件拷贝到容器内
COPY jdk-8u301-linux-x64.tar.gz /usr/local
RUN tar -zxvf /usr/local/jdk-8u301-linux-x64.tar.gz -C /usr/local
//设置容器内环境变量
ENV JAVA_HOME /usr/local/jdk1.8.0_301
ENV PATH $JAVA_HOME/bin:$PATH

COPY hadoop-3.3.1.tar.gz /usr/local
RUN tar -zxvf /usr/local/hadoop-3.3.1.tar.gz -C /usr/local
ENV HADOOP_HOME /usr/local/hadoop-3.3.1
ENV PATH $HADOOP_HOME/bin:$PATH
ENV PATH $HADOOP_HOME/sbin:$PATH
```

ps:dockerfile建议多个命令之间尽量用 && 链接。由于每个命令均会建立一层，导致镜像体积膨胀

## 运行镜像
-f后面指定dockerfile文件， -t后面指定制造出的镜像叫啥， 别忘记最后还有一个`.`  
`docker build -f Dockerfile.test -t image-train-test .`