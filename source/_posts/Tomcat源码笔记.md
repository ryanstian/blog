---
title: Tomcat源码笔记
date: 2019-08-14 23:24:00
categories: Tomcat
tags: []
copyright: true #新增,开启
---

本章我们不会涉及代码,而是简要的介绍Tomcat的实现原理
<!--more-->
**Tomcat版本9.0.21**

## Tomcat是什么?
在我看来,Tomcat是利用各种模型和设计方式对socket的深度封装,做到适配各种协议同时达到一定性能的代码组  
同时给我们写的各种业务代码(Servlet)提供了容器(也可以理解为tomcat可以将以对象的形式使用我们写的Servlet业务),这是Tomcat的核心。  
当然,Tomcat还实现了一些其他的比如生命周期管理,但是这些都是为了核心而服务的

我们第一次接触Tomcat,相信大多数人都是Hello World。  
想想当时我们是怎么做的:
- 我们建立了个项目
- 按照网上的教程建好了项目里面的文件夹
- 导入servlet包
- 开始编写xml配置文件
- 继承Servlet编写Get,Post代码
- 导出war包放到tomcat下的webapps文件夹
- 启动tomcat
 
so easy,然后我们就可以通过浏览器访问我们之前写好的接口了。

但是,我们有没有想过是为什么,为什么我们GET中的代码会被调用,为什么我们访问一个网址会执行我们的代码,他又是怎么执行的？

这一切,我们将从Tomcat中找到答案

## 消息接收
首先,消息是如何接收的？

这里,我要阐述下自己的理解：  
- 在网络传输的世界里面,一切都是消息。  
  消息是指什么?  
  消息可以理解为一串二进制,一串byte或者字符串（当然,在网络模型的最底层,这些都会被转换为二进制来传输）
- 协议又是什么?  
  协议是一种事先约定的规范。规定了消息格式,消息处理方式等等各种机制  
  例如，我们编写servlet最常用到的http协议,他的可视化表示就如同下面这些内容。  
  其实,每一行后面都跟着\r\n,不过这是换行符,所以在屏幕上展现出来就是一行一行的数据
```
GET / HTTP/1.1
Host: localhost:8080
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Connection: keep-alive
Upgrade-Insecure-Requests: 1
```

了解了这些以后,我们就可以继续进行了  

既然一切都是消息,那么当我们发送一个http请求的时候,Tomcat最开始接收到的也是一串像是上面这种的字符串,java中接收消息用的就是socket,Tomcat也不例外。  

所以我在文章最开始的时候说到：“Tomcat实质就是对socket的深度封装”

在获取到socket套接字以后，Tomcat开始解析  
根据传入内容标明协议的不同,按照在代码中定义好的各种协议模板来解析这个字符串。  
解析完成后封装到Request和Response中交给Servlet执行用户自定义的业务代码,最后再由socket发送响应,这就是Tomcat大致的流程


## 生命周期
Tomcat的各个组件也是有生命周期的,这个生命周期由一种设计模式(状态机)来控制,下面让我们了解一下

首先要介绍的是LifecycleMBeanBase类,下面是这个类的类图
![如果图片失效,请邮件联系作者补图](LifecycleMBeanBase继承图.png)

我们从Lifecycle接口开始了解,Lifecycle定义了一个状态机,下面是Lifecycle的原注释
```
 *            start()
 *  -----------------------------
 *  |                           |
 *  | init()                    |
 * NEW -»-- INITIALIZING        |
 * | |           |              |     ------------------«-----------------------
 * | |           |auto          |     |                                        |
 * | |          \|/    start() \|/   \|/     auto          auto         stop() |
 * | |      INITIALIZED --»-- STARTING_PREP --»- STARTING --»- STARTED --»---  |
 * | |         |                                                            |  |
 * | |destroy()|                                                            |  |
 * | --»-----«--    ------------------------«--------------------------------  ^
 * |     |          |                                                          |
 * |     |         \|/          auto                 auto              start() |
 * |     |     STOPPING_PREP ----»---- STOPPING ------»----- STOPPED -----»-----
 * |    \|/                               ^                     |  ^
 * |     |               stop()           |                     |  |
 * |     |       --------------------------                     |  |
 * |     |       |                                              |  |
 * |     |       |    destroy()                       destroy() |  |
 * |     |    FAILED ----»------ DESTROYING ---«-----------------  |
 * |     |                        ^     |                          |
 * |     |     destroy()          |     |auto                      |
 * |     --------»-----------------    \|/                         |
 * |                                 DESTROYED                     |
 * |                                                               |
 * |                            stop()                             |
 * ----»-----------------------------»------------------------------
```
我们可以看到,这是一种状态机设计模式。规定了组件生命周期的状态转换,可以方便的进行组件生命周期的管理。  

从下图我们可以看到,从Server开始几乎每一个组件间接继承/实现了该状态机
![如果图片失效,请邮件联系作者补图](Lifecycle接口实现列表.png)

接下来让我们看LifecycleBase,这是一个抽象类,实现了fireLifecycleEvent,init,start...等方法

fireLifecycleEvent的设计其实是根据观察者模式

init,start等方法仅仅是用来控制其生命周期的,每个方法例如init,在内部还会调用initInternal(),Tomcat的很多组件的业务代码全部都在xxxInternal()中,由子类负责实现

LifecycleMBeanBase则对LifecycleBase进行了进一步的实现,我们从它的图中可以看到
![如果图片失效,请邮件联系作者补图](LifecycleMBeanBase类图.png)

其中最关键的是initInternal()和destroyInternal()。  
主要实现了委托Register类来将子类(类如StandardServer等)注册到bean容器中。

如果在看源码过程中大家对ManagedBean或者MBeanServer抱有疑惑,可以先去学习一下JMX规范再回来看容器部分代码,不过即使跳过容器部分,也不影响接下来的部分

## Tomcat初始化流程(仅需有个印象即可)
Tomcat工作主要有几个流程:
- init(负责new各级对象,组件依赖关系,加载配置文件)
- start(这一步完成时可以正常接收请求开始处理)
- 处理消息
- 结束

首先放一张tomcat init的流程图(简化版)
![如果图片失效,请邮件联系作者补图](tomcat时序图init.png)

- Bootstrap:是入口,例如命令行输入service tomcat start等操作时,便是由这个类来解析,这个类均通过反射操作来调用Catalina
- Catalina:提供了操控Tomcat启停等行为的方法
- LifecycleBase:状态机设计模式,我们后文会提及,StandardServer等大部分组件都会实现该状态机
- StandardServer:顶级容器,一个Tomcat对应唯一一个Server,负责管理多个service的启停等行为
- StandardService:可以完整执行功能的最小单元容器(如果不明白可以先继续看)  
  下面是一个server.xml文件去掉注释后的内容。根据xml我们可以清楚的看到其构建逻辑  
  Server包含Service,Service包含Connector和Engine,Engine包含host  
  假如我们现在面临一个问题,有两个同名的项目需要发布或者希望不同项目部署在不同的端口,那么我们就可以在后面新增一个service

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Server port="8005" shutdown="SHUTDOWN">
  <Listener className="org.apache.catalina.startup.VersionLoggerListener" />
  <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" />
  <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
  <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
  <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" />
  <GlobalNamingResources>
    <Resource name="UserDatabase" auth="Container"
              type="org.apache.catalina.UserDatabase"
              description="User database that can be updated and saved"
              factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
              pathname="conf/tomcat-users.xml" />
  </GlobalNamingResources>
  <Service name="Catalina">
    <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
    <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
    <Engine name="Catalina" defaultHost="localhost">
      <Realm className="org.apache.catalina.realm.LockOutRealm">
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
               resourceName="UserDatabase"/>
      </Realm>
      <Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />
      </Host>
    </Engine>
  </Service>
</Server>
```
- ScheduledThreadPoolExecutor:线程池,后面我在讲述线程模型的时候会讲到
- Container容器模块,呈现包含关系,之间以责任链形式调用,这里注意一点,虽然方法名是invoke,但实际上并不是通过反射来调用,类似的在tomcat中也有很多继承了Runnable但是有些模块用不到start而是使用run的情况
```
{
  Engine{
    Host{
      Context{
        Wrapper
      }
    }
  }
}
```
- Endpoint:核心部分,后面讲线程模型我会提到
- Tomcat的start流程其实跟init流程类似,在宏观上几乎没有改动,因此省略

## Tomcat接收消息流程

这里我认为一张图足以
![如果图片失效,请邮件联系作者补图](tomcatNIO接收请求逻辑图.png)

## 一些关键的节点

这里将提供一些消息在Tomcat中传递的关键节点,可以帮助大家通过全局搜索快速定位到源码

- init时:  
这里有人会疑惑类的初始化和注入依赖在哪里,答案是digester.parse  
这种感觉就像是我们写springMVC时配置的xml一样,在这里xml就是server.xml。digester会根据这个xml来解析并注入依赖
- NIO接收消息时:  
关键类Acceptor。其run方法是核心,endpoint.setSocketOptions是转折点。随后一系列操作将accept到的封装为PollerEvent加入队列
- 关键类Poller：  
  从队列取处PollerEvent注册到socketChannel的Selector选择器中,并且负责轮询读写事件,将其封装后扔到线程池中
- 关键类SocketProcessor：
  被上文封装的Runnable,负责接下来的读取解析处理返回操作
- 关键类Http11Processor：  
  inputBuffer.parseRequestLine获取并解析请求,如果是文件传输类型,那么不会解析消息体,如果是表格那种文本的,就会一起读取出来
- 具体从socket读取消息的地方:  
  Http11InputBuffer类的socketWrapper.read。  
  NioSocketWrapper类的nRead = fillReadBuffer(block, to)
- 关键类Http11Processor：
  inputBuffer.parseHeaders将读取出的消息解析为消息头

## 这里我们讲Tomcat线程模型

TomcatNIO的线程模型其实非常简单,简单到什么程度?让我们看图
![如果图片失效,请邮件联系作者补图](TomcatNIO线程模型.png)

甚至于io操作和业务操作在同一根线程上进行,没有经过分离,poller的职责仅仅是检测事件,并不负责io操作,
<font color=red>注意这仅仅是NIO模式,不包括另外两种NIO2和APR模式</font>

我们看server.xml配置文档的时候可以看到允许我们设置一些参数,这里之前tomcatNIO接收请求逻辑图已经描述过,不再赘述。

至于Poller,你在看旧文章时有人会说允许最大值不超过2个。没错,在之前的版本是这样,但是在9.0,我们看到注释文档中有句话,在NIO下Poller被改为仅有一个

```
<update>
    Remove <code>pollerThreadCount</code> Connector attribute for NIO,
    one poller thread is sufficient. (remm)
</update>
```

通过这些,我们可以分析到  
Tomcat NIO模式下,是通过粗暴的增加线程来处理请求。  
如果同时请求数过多,会被ServerSocketChannel阻拦掉  
如果交给线程池的read达到线程池上限,那么就会加入队列中进行排队