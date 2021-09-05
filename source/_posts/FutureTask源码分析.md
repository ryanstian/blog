---
title: FutureTask源码分析
date: 2019-10-05 18:11:00
categories: 
tags: [java]
copyright: true #新增,开启
---


FutureTask直接继承了RunnableFuture,间接继承了Future,Runnable。当我们使用Runnable时,是无法获得返回值的,而RunnableFuture则是为了解决这一个问题而存在

在该类中,有一点你会发现,他用到了自定义的链表,并且在get阻塞等待以及任务完成时都会操作该链表,例如awaitDone(boolean timed, long nanos)方法中的这些操作,该链表的作用是,如果多个线程同时get阻塞等待该任务,这些线程的信息分别保存在链表上的一个节点中,那么当任务完成时可以通过该操作来即时唤醒这些线程,超时的被从链表中移除,例如下面代码demo
```java
@Test
    public void test42() throws InterruptedException {
        //构造FutureTask
        FutureTask futureTask = new FutureTask(() -> {
            Thread.sleep(5000);
            System.out.println("callable has run");
            return "123";
        });
        //运行FutureTask中传入的任务
        new Thread(() -> futureTask.run()).start();
        //开启第一个get
        new Thread(() -> {
            try {
                System.out.println("Thread one:"+futureTask.get());
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (ExecutionException e) {
                e.printStackTrace();
            }
        }).start();
        //开启第二个get
        new Thread(() -> {
            try {
                System.out.println("Thread two:"+futureTask.get());
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (ExecutionException e) {
                e.printStackTrace();
            }
        }).start();
        //主线程睡眠等待其他线程结束
        Thread.sleep(10000);
    }
```

在该类中有一处代码,这里保存该任务的状态,这种方式很常见,也可以方便的通过比较大小等方式来判断其状态范围
```java
private volatile int state;
private static final int NEW          = 0;
private static final int COMPLETING   = 1;
private static final int NORMAL       = 2;
private static final int EXCEPTIONAL  = 3;
private static final int CANCELLED    = 4;
private static final int INTERRUPTING = 5;
private static final int INTERRUPTED  = 6;
```

callable是接到的任务,即使构造时采用的Future也会为了统一而被封装为Callable,outcome是返回值,如果任务执行过程中出现异常,那么outcome中保存的则是异常信息,waitNode则是刚刚提到的链表,runner是运行该任务的线程信息,在本类中常被用来做CAS操作,只有当其为null,run()才可被调用,避免连续调用两次run()导致重复运行任务
```java
/** The underlying callable; nulled out after running */
private Callable<V> callable;
/** The result to return or exception to throw from get() */
private Object outcome; // non-volatile, protected by state reads/writes
/** The thread running the callable; CASed during run() */
private volatile Thread runner;
/** Treiber stack of waiting threads */
private volatile WaitNode waiters;
```

finishCompletion()方法就是之前提到的唤醒链表上所有get阻塞的线程
LockSupport.unpark和park可以理解为信号量
runAndReset()这是个protected方法,作用是,通过不改变state的状态使其可以运行多次,普通的run()运行一次以后state状态就改变了

其他的代码没啥难度,看下源码附带的注释就能理解了