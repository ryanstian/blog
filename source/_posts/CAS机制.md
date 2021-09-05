---
title: CAS机制
date: 2019-07-02 00:02:00
categories: 
tags: []
copyright: true #新增,开启
---

一.什么是CAS机制
CAS机制的全名叫做compare and swap
让我们来看一行代码

```java
public final native boolean compareAndSwapLong(Object var1, long var2, long var4, long var6);
```
这行代码源于Unsafe类(待补充),参数var1和var2我们先不考虑,var4表示旧值,var6表示新值  
这行代码的作用是,如果var4的值等于内存中的现有值,那么将内存中的值替换为var6同时返回true,否则返回false。这就是CAS机制,同时也是其在Java中的体现

二.为什么要使用CAS/有哪些好处
一般情况下,当我们并发访问同一个int变量时,我们往往需要加锁操作,但每次加锁会造成大量的开销,影响性能  
所以就有了CAS机制,可以让我们在不加锁的情况下做到线程安全

三.CAS机制存在哪些问题
1.ABA问题
先看一段代码,代码源于Unsafe类
①这一步的意义是得到内存中的现有值(参数可忽略)

```java
public final long getAndAddLong(Object var1, long var2, long var4) {
        long var6;
        do {
            var6 = this.getLongVolatile(var1, var2);①
        } while(!this.compareAndSwapLong(var1, var2, var6, var6 + var4));

        return var6;
    }
```

ABA问题简述:如代码所示  
1、假设存在线程1,2
2、线程1运行了①之后等待
3、线程2开始运行，线程2将A改变为B,再将B改变为A,线程2结束
4、线程1继续运行。此时,线程1会认为A依旧是原来他读取到的A,期间并没有改变,并且将他按照正常流程改变为B。

当然,在正常情况下,变量加减方面这并不会造成什么影响。但是若将CAS用在堆栈或者链表上(网上搜索一下有很多这种ABA问题的例子),或由于业务错误,同时发出了两次修改金钱100为50的操作,但是此时又加入了一个修改金钱50为100的操作(参考自漫画：[什么是CAS机制？（进阶篇）](https://mp.weixin.qq.com/s?__biz=MzIxMjE5MTE1Nw==&mid=2653192736&idx=1&sn=24d4054b062e28db9e54c735aafe2407&chksm=8c99f0fabbee79ecfd9198aa89bc78084e9b7db056078982975d8910c12b5d3dd1d16c2509c3&mpshare=1&scene=23&srcid=020903SQqmtv0yiUHSI1DuGd#rd)),那么就会出现严重的问题

解决方案:
最常见的ABA问题的解决方案就是诸如java并发包中的AtomicStampedReference类,其内部实现类似于{% post_link AtomicLong使用 AtomicLong使用 %}。  
不同点是,其内部维护了一个内部类Pair,采用记录版本号的方式来避免ABA问题,不过每次在更改时都会new一个新的Pair来进行CAS,如果对性能有极高的要求,那么需要谨慎选择

2.由于在使用CAS时,往往使用的是重试机制,即在while循环中一直重试CAS直到成功为止。  
所以在极高并发情况下,CAS的失败率将增大,会导致严重的性能问题。  
对于这个问题,很多时候解决方案是在一般程度并发时采取CAS,极高并发时进行排队。  
LongAddr钟采用了分治的办法,拥有多个计数器,在高并发时将计数任务分配到多个计数器中,分担压力