---
title: AtomicLong
date: 2019-07-02 00:02:00
categories: 
tags: [java]
copyright: true #新增,开启
---


#### 正文
一.AtomicLong是做什么用的
首先我们可以先看一下我的另一篇文章{% post_link i++不是原子性操作 i++不是原子性操作 %}

此时,我们通常选择会是进行这样的操作

```java
public class CasStudy01 {
    private static int count = 0;

    private synchronized static void add(){
            count++;
    }
    public static void main(String[] args) throws InterruptedException {
        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                add();
            }
        };
        for (int i = 0; i < 10000; i++) {
            new Thread(runnable).start();
        }

        Thread.sleep(1000);
        System.out.println(count);
    }
}
```
我们将count++操作放在了一个带锁的方法里面,来保证其线程安全性。然而,我们知道,加锁解锁操作会造成性能的消耗,在并发量不算太高的情况下,我们可以考虑采用AtomicLong(无锁的方式,采用{% post_path CAS机制 %})来保证线程安全性

二.AtomicLong的实现
AtomicLong在源码中持有Unsafe类的实例,其大部分操作都是交付给Unsafe类来完成的(Unsafe中大多是本地方法,虽然我们可以通过反射来调用,但是官方强烈不建议我们这么做)

AtomicLong里面持有一个long类型的valueOffset变量,这个变量表示的是其value值的内存偏移量(详见JVM内存模型),当我们调用incrementAndGet时,会交付Unsafe类来进行操作

```java
    public final long incrementAndGet() {
        return unsafe.getAndAddLong(this, valueOffset, 1L) + 1L;
    }
```
我们传入本类的实例,value的偏移量,以及增加量

```java
    public final long getAndAddLong(Object var1, long var2, long var4) {
        long var6;
        do {
            var6 = this.getLongVolatile(var1, var2);
        } while(!this.compareAndSwapLong(var1, var2, var6, var6 + var4));

        return var6;
    }
```
在Unsafe中就会进行CAS操作,使得value增加1,这是线程安全的

三.AtomicLong的缺点
当并发量极大的时候,由于CAS机制本身的原因,导致CAS失败率极高,从而拖慢性能。此时,我们可以考虑使用LongAdder(待补充)