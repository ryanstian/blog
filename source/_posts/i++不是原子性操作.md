---
title: i++不是原子性操作
date: 2019-07-02 00:05:00
categories: 
tags: [java]
copyright: true #新增,开启
---

#### 正文


```java
public class CasStudy01 {
    private static int count = 0;

    public static void main(String[] args) {
        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                count++;
            }
        };
        for (int i = 0; i < 10000; i++) {
            new Thread(runnable).start();
        }
        Thread.sleep(1000);//为了等子线程全部运行结束
        System.out.println(count);
    }
}
```
输出:
9945

Process finished with exit code 0

刚才的代码,照我们的设想,他应该是输出10000,然而每次我们run这段demo,输出结果各不相同
这是因为count++这一行代码并不是原子操作,这一行代码实际在运行时,被分为取值,修改,存储三步操作,所以1,2两个线程同时取出值a,并且自增1修改为a+1,再存储的话,两次自增实际上只自增了1