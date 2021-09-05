---
title: docker for mac网络不通
date: 2020-08-08 21:15:00
categories: docker
tags: [mac,docker]
copyright: true #新增,开启
---

现象:docker安装在linux上一切正常,安装在mac上,宿主机无法ping通容器ip
经常出现在宿主机连接容器kafka集群的情况下,无法连通

<!--more-->

## docker与宿主机之间网络模型

这是由于docker是借助Linux内核的Namespace技术来实现的,而在mac上无法这样。所以在mac上安装docker实际上是先安装了一个linux虚拟机,然后在这台linux虚拟机内部跑docker
那么mac上的docker和原生linux上的docker网络结构便不一致,如下图所示

1.linux docker网络结构
![如果图片失效,请邮件联系作者补图](linux docker网络结构.png)

2.Docker Toolbox网络结构(已逐渐被第三种取代)
![如果图片失效,请邮件联系作者补图](Docker Toolbox网络结构.png)

3.HyperKit (Docker Desktop for Mac)网络结构
![如果图片失效,请邮件联系作者补图](HyperKit (Docker Desktop for Mac)网络结.png)

从图中我们可以看到
如果是linux机器上安装docker,可以通过网桥直接使用容器id来访问容器
而在mac机器上,这两种结构均无法直接通过容器ip访问容器(因为隔了两层网络)


## 宿主机与容器访问的解决方案
#### 对于图2的结构
我们即使在docker run时指定了-p,也无法在宿主机通过端口访问到。

因为此时的映射如下图,仅仅将docker容器端口映射到了linux虚拟机上

所以我们如果想要从宿主机通过localhost:post访问到对应容器,还需要手动将linux虚拟机的对应端口映射到宿主机上(当然,在docker中的几个容器如果没有设置防火墙可以互相通过各自内网ip访问,因为他们在同一网域)


![如果图片失效,请邮件联系作者补图](图2-1.png)
<center>docker端口映射</center>

![如果图片失效,请邮件联系作者补图](图2-2.png)
<center>docker+虚拟机端口映射</center>

刚才我们说了宿主机主动访问容器的方法,那么接下来说容器中访问宿主机的方法。

方法一:如果宿主机连接了外网/局域网,可以通过宿主机ip来访问,具体数据流向如下图

![如果图片失效,请邮件联系作者补图](图2-3.png)
<center>网络结构2,容器中访问宿主机方法一数据流向</center>

方法二:由于linux虚拟机既可以通过网桥一访问宿主机,又可以通过网桥二访问容器,那么我们在linux虚拟机中做vpn,将容器的流量传输到宿主机中

![如果图片失效,请邮件联系作者补图](图2-4.png)
<center>网络结构2,容器中访问宿主机方法二数据流向</center>

方法三(并不确定Docker Toolbox是否提供该功能):通过docker提供的域名来访问宿主机

#### 对于图3的结构
其实该结构本质上与结构2本质并没有什么区别,只不过宿主机与linux虚拟机之间的数据交互由原来的网桥改为了sock文件(可能位于/var/run/docker.sock)

但是HyperKit (Docker Desktop for Mac)相比于Docker Toolbox,提供了自动的端口映射,也就是说,我们在docker run -p指定端口映射以后,会自动帮我们把这个端口映射到宿主机,无需我们手动操作虚拟机

另外如果容器想要访问宿主机,可以通过docker for mac提供的本地域名kubernetes.docker.internal来访问(无法保证每个版本的域名都是这个,如不一致,请使用官方提供的最新域名)。该域名仅最大作用于宿主机范围,对外部机器不适用

很遗憾我并未找到kubernetes.docker.internal的原理。虽然你可以从宿主机的hosts中找到127.0.0.1 kubernetes.docker.internal这一条,但我肯定这并不是使他起作用的根本,原因如下:
+ 如果仅仅根据dns查找原理,子dns通过kubernetes.docker.internal收到的ip仅仅是127.0.0.1
+ 在容器中ping该域名,得到了一个内网ip,然而从宿主机通过ifconfig,并无法找到该对应的该内网ip(这是当然,因为他们是通过sock文件进行网络通信)
+ 在宿主机hosts添加一条127.0.0.1 kubernetes.docker.test后,从容器中ping kubernetes.docker.test,得到的ip为127.0.0.1

由此可见,必然docker for mac在linux虚拟机上帮我们做了一些工作,他可能通过类似转发流量的行为来达到这一目的

## 关于连接docker中kafka集群失败的原因及解决方法

之前提到的网络结构仅仅是一部分原因,另一部分则是由于kafka等组件的配置问题

有一点我们要知道,当我们从宿主机连接kafka集群时,会首先访问zookeeper获取kafka挂载到zookeeper的host和port。

在kafka的server.properties配置文件中,我们可以看到,kafka默认的寻找zookeeper的ip为localhost。  
显而易见,通过容器的localhost是无法找到zookeeper的，只能找到容器自己。  
所以这里我们可以通过填写zookeeper容器在docker的内部ip,或者通过域名来访问zookeeper(实际数据流向为kafka容器->虚拟机->宿主机->经过zookeeper映射出来的端口->虚拟机->zookeeper容器),这样kafka才能连通zookeeper将自身信息挂载上

另外还有一个问题我们需要解决。  
在默认情况下,kafka挂载到zookeeper的host为自身的容器id,显然我们的宿主机是无法将容器id当作host找到该容器的。那么挂载为自己容器id的原因是什么呢?  
答案：  
kafka会先尝试获取advertised.host.name,如果未设置则尝试获取host.name,如果均未设置则会调用java.net.InetAddress.getCanonicalHostName()方法获取hostname(该参数可以通过/etc/hostname来找到其配置)  
在容器中hostname默认为容器id,所以我们通过添加advertised.host.name的参数为kubernetes.docker.internal域名,即可使宿主机得到的host为kubernetes.docker.internal,从而通过映射出去的端口找到kafka容器

注:不同版本获取的参数或者获取参数的顺序或位置可能不同,但基本原理一致

