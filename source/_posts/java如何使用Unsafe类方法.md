---
title: 如何使用Unsafe类方法
date: 2019-10-05 18:11:00
categories: 
tags: [java]
copyright: true #新增,开启
---

版本：JDK8
首先Unsafe类是不建议被使用的,因为他面向底层,可能在每一代jdk版本中发生变化,除非你有把握在在每一次升级jdk时维护你的项目
<!--more-->
Unsafe是作为单例而存在的,当我们尝试调用getUnsafe方法时,会报安全错误,这是由于双亲加载机制导致的。通常我们可以通过反射来绕过这些检测

在如下代码中,我们通过反射获取到了Unsafe类的实例,Unsafe类中的方法往往都是通过偏移量来操作对象的,我们可以看到,我们定义了Thread对象,并且通过objectFieldOffset获取其偏移量,在test()方法中,通过CAS来将其置换,成功的使用了Unsafe中的方法
```java
public class UnsafeTest {
    private volatile Thread runner;

    private static final long runnerOffset;

    private static final Unsafe UNSAFE;

    static {
        try {
            Field f = Unsafe.class.getDeclaredField("theUnsafe");
            f.setAccessible(true);
            UNSAFE = (Unsafe) f.get(null);
            //UNSAFE = sun.misc.Unsafe.getUnsafe();
            Class<?> k = UnsafeTest.class;
            runnerOffset = UNSAFE.objectFieldOffset
                    (k.getDeclaredField("runner"));

        } catch (Exception e) {
            throw new Error(e);
        }
    }
    public void test(){
        UNSAFE.compareAndSwapObject(this, runnerOffset,
                null, Thread.currentThread());
    }
    public void print(){
        System.out.println(runner);
    }

    public static void main(String[] args) {
        UnsafeTest unsafeTest = new UnsafeTest();
        unsafeTest.test();
        unsafeTest.print();
    }
}

```