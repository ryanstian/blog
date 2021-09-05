---
title: Servlet源码笔记
date: 2019-06-28 16:16:00
categories: 
tags: [Servlet,java]
copyright: true #新增,开启
---

主要简单介绍下servlet源码结构
<!--more-->
#### 介绍
首先类的主要结构关系需要提及一下

```java
模块一
interface ServletRequest

interface HttpServletRequest extends ServletRequest

class ServletRequestWrapper implements ServletRequest

class HttpServletRequestWrapper extends ServletRequestWrapper implements HttpServletRequest

模块二
interface ServletConfig

interface Servlet

abstract class GenericServlet implements Servlet, ServletConfig, java.io.Serializable

abstract class HttpServlet extends GenericServlet
```

看到上面的类继承关系可能会有点陌生,接下来我给出一段demo

```java
public class HelloWord extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response){
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response){
    }
```
好了,这下应该不陌生了,我们用Servlet写代码一般都是继承HttpServlet来进行,这属于第二模块,而我们代码中操作的request属于第一模块

在具体分析代码之前,有必要先科普下Servlet的生命周期,平时我们在写Servlet服务端的时候,是不需要写main入口类的,仔细想想,没有入口类为何可以启动?
答案来了,是因为tomcat,如果将Servlet看作对象的话,那么tomcat就是Servlet的容器,tomcat负责操控Servlet的生命周期,tomcat从他自己的入口类启动,运行时调用Servlet从而进行一切操作。
我看了很多的博客教程,他们都是这么说的:
```
&emsp;tomcat作为servlet容器,当http请求进来时,发现没有servlet,那么则初始化一个servlet,将http请求封装为Request交给servlet处理,且servlet为单例重复使用,若长时间未调用才会销毁
```
但是在tomcat8.5.28+servlet4.0环境下,在不调整任何参数时(默认),我的测试跟上述操作有点出入,servlet并不是在接到http请求时才初始化,而是在随tomcat启动时便已经初始化,这一点可以根据我对servlet初始化init方法打断点,并且以debug方式启动可以看出,各位尽可以自行尝试,当然这不是重点,大体流程了解即可。

#### 第二模块
+ Servlet这个接口类定义了一系列与tomcat相互交互的一系列接口
+ ServletConfig看名字也知道是提供配置信息的一个接口
+ GenericServlet这个是对Servlet和ServletConfig接口的一些实现,另外增加了一些log方法来传递异常
+ HttpServlet这个类就定义了对GET,PUT,POST,HEAD,DELETE等各种HTTP方法的处理方式
那么问题来了,我们知道之前定义的Servlet接口类提供给tomcat一些交互接口,那么唯一涉及到各种操作的只有service方法,他是如何跟各种HTTP方法的处理结合起来的呢?
在源码面前的朋友可以追着service方法一路下来,最终在HttpServlet中可以看到service方法的实现
```java
protected void service(HttpServletRequest req, HttpServletResponse resp)
        throws ServletException, IOException
    {
        String method = req.getMethod();

        if (method.equals(METHOD_GET)) {
            long lastModified = getLastModified(req);
            if (lastModified == -1) {
                // servlet doesn't support if-modified-since, no reason
                // to go through further expensive logic
                doGet(req, resp);
            } else {
                long ifModifiedSince = req.getDateHeader(HEADER_IFMODSINCE);
                if (ifModifiedSince < lastModified) {
                    // If the servlet mod time is later, call doGet()
                    // Round down to the nearest second for a proper compare
                    // A ifModifiedSince of -1 will always be less
                    maybeSetLastModified(resp, lastModified);
                    doGet(req, resp);
                } else {
                    resp.setStatus(HttpServletResponse.SC_NOT_MODIFIED);
                }
            }

        } else if (method.equals(METHOD_HEAD)) {
            long lastModified = getLastModified(req);
            maybeSetLastModified(resp, lastModified);
            doHead(req, resp);

        } else if (method.equals(METHOD_POST)) {
            doPost(req, resp);
            
        } else if (method.equals(METHOD_PUT)) {
            doPut(req, resp);
            
        } else if (method.equals(METHOD_DELETE)) {
            doDelete(req, resp);
            
        } else if (method.equals(METHOD_OPTIONS)) {
            doOptions(req,resp);
            
        } else if (method.equals(METHOD_TRACE)) {
            doTrace(req,resp);
            
        } else {
            //
            // Note that this means NO servlet supports whatever
            // method was requested, anywhere on this server.
            //

            String errMsg = lStrings.getString("http.method_not_implemented");
            Object[] errArgs = new Object[1];
            errArgs[0] = method;
            errMsg = MessageFormat.format(errMsg, errArgs);
            
            resp.sendError(HttpServletResponse.SC_NOT_IMPLEMENTED, errMsg);
        }
    }
```

从代码中可以看到,他是负责匹配头部信息来进行分发操作,不清楚http报文的朋友可以看下面,这是用firefox浏览器发送的一组请求,第一行的GET即为method.equals(...)中的method内容
```
GET / HTTP/1.1
Host: localhost:8080
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:67.0) Gecko/20100101 Firefox/67.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Pragma: no-cache
Cache-Control: no-cache
```
到此为止第二模块基本结束

#### 接下来分析第一模块
+ ServletRequest主要用来获取被储存信息,例如储存被tomcat封装后的http信息
+ HttpServletRequest特别针对http协议的各种参数在ServletRequest基础上进行了扩展
+ ServletRequestWrapper也是个扩展,不过有个特殊的地方要注意,这个类的构造方法public ServletRequestWrapper(ServletRequest request)接收了一个ServletRequest对象,以后的参数就从这个对象里面拿取
+ HttpServletRequestWrapper就是上面三个类的实现了,没什么意思

#####通过这些介绍,Servlet已经不再神秘,大家可以仔细去看源码,其实Servlet构造十分简单,真正起到关键作用的还是例如Tomcat等Servlet容器  
基础知识:{% post_link Tomcat源码笔记 Tomcat源码笔记 %}
