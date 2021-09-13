---
title: docker-compose使用
date: 2021-09-12 11:36:00
categories: docker
tags: []
copyright: true #新增,开启
---

docker-compose其实可以理解为一个脚本，代替我们执行创建容器命令。  
如果需要创建多个容器，那么提前编写docker-compse会非常好用
<!--more-->
## 安装docker-compose  
直接`yum -y install docker-compose`即可  
ps：后续版本太高可能需要去github下载特殊版本，并且下载后建立软连接，赋予可执行权限  

安装完后检查安装情况`docker-compose --version`

## 编写docker-compose
docker-compose，本质上就是用yml的形式，把我们平时`docker run xxx`的命令再写一遍。  
基本`docker run xxx`能用的，docker-compose都能用。反之亦然

示例：
```yml
# version不是任意填，要填你docker版本对应的compose版本
version: '3'
# service为容器的根目录，下面my1，my2，my3分别是三个容器，名字可以任意起
services:
    # 创建容器，my1可以任意叫，只是目录结构要求，后续没啥用
    my1:
        # 选用的镜像
        image: mysql
        # 重启策略（可选），如果为no，则运行compose后不会自动启动
        restart: "no"
        # 容器名，相当于docker run --name xxx
        container_name: mysql1
        # 网桥配置（可选），配置好以后同一网桥下可以用别名互访，可配置多个网桥
        networks:
            # 其中一个网桥，名字需要与下方定义一致，具体见代码块最下方网桥定义
            docker-net:
                # 容器在网桥中的别名，用于容器互访
                aliases:
                    # 这里别名叫做mysql1
                    - mysql1
        # 容器相关参数
        deploy:
            # 资源限制
            resources:
                # 最大资源限制
                limits:
                    # 这里1是指的单核cpu限制，多核cpu可以超过1
                    cpus: "1"
                    # 内存限制
                    memory: 4096M
                # 宿主机为容器预留资源
                reservations:
                    cpus: "0.25"
                    memory: 1024M
    # 创建第二个容器
    my2:
        image: mysql
        restart: "no"
        container_name: mysql2
        networks:
            docker-net:
                aliases:
                    - mysql2
        deploy:
            resources:
                limits:
                    cpus: "1"
                    memory: 4096M
                reservations:
                    cpus: "0.25"
                    memory: 1024M
    my3:
        image: mysql
        restart: "no"
        container_name: mysql3
        networks:
            docker-net:
                aliases:
                    - mysql3
        deploy:
            resources:
                limits:
                    cpus: "1"
                    memory: 4096M
                reservations:
                    cpus: "0.25"
                    memory: 1024M
# 定义网桥
networks:
    # 要定义的网桥名，可以定义多个网桥
    docker-net:
```

其他命令还有在my1目录下，诸如environment设置容器环境等（需容器支持），具体可自行查阅

## 运行docker-compose
-f表示指定文件，-d表示后台运行(否则会直接在你当前窗口运行，一关窗口就关闭了)  
`docker-compose -f 自己编写的compose文件名 up -d`

## 扩展 docker容器互访
1、docker安装时会创建默认网桥，容器可通过内网ip互访  
2、自己创建网桥`docker network create testnet`,容器创建时通过--network 网桥名 --network-alias 网络别名 连接到网桥  
然后此网桥下容器间可通过网络别名互访
