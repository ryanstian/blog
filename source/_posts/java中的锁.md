---
title: java中的锁
date: 2019-08-28 22:57:00
categories: 
tags: [java]
copyright: true #新增,开启
---

### 锁的几种用法

####synchronized
e.g:1
用synchronized对一个代码块加锁,object可以是任意的对象,任何其他synchronized(该对象)的代码块均需要获取到锁以后才可以执行,如果object=this,那么就是锁住的整个对象
```java
synchronized(object) {
    //代码块
}
```

e.g:2
下方代码锁住的是此方法所在的对象,也就是如果该对象中两个不同的方法前面均有synchronized时,在多个线程操作同一个对象时,同一时间只有一个方法可以被调用
```java
public synchronized void work(){
        System.out.println(123);
}
```
e.g:3
如果是对一个class或者static类型的对象加锁,那么因为class和static类型的对象只会在jvm虚拟机保存一份,所以加锁要额外注意

e.g:4
特别强调的一点是,synchronized是针对对象加锁,Class和static也可以看作一个对象,假如说出现下面代码,此时你的锁是无效的,虽然引用没变,但是引用指向的对象已经改变
```java
Object obj = new Object();
synchronized(obj){
    obj = new Object();
    //代码块
}
````
待续

拓展:{% post_link 分布式锁 分布式锁 %}