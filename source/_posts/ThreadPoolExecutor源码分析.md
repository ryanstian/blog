---
title: ThreadPoolExecutor源码分析
date: 2019-10-05 18:11:00
categories: 
tags: [java]
copyright: true #新增,开启
---

看到网上讲线程池源码的文章一堆大坑,只能自己扒源码扒篇文章出来了,坐标jdk8
<!--more-->
最重要的一个坑放在前面

假如你设置的核心线程数=2,最大线程数=4。很多人都讲向线程池添加任务时会先扩充到最大线程数,多出来的再向队列添加,我只想说,<font color=red>根据源码分析并不是这样</font>

看如下代码,在添加任务时,从源码或从注释中明确表示分了三步

1.如果现存线程数小于核心线程数,则创建线程,添加的任务直接在该线程运行

2.如果核心线程数满了,则add到队列中

3.如果队列满了,则尝试扩充线程,任务在新创建的线程中运行

那我们结合之前提到的错误方式来看,如果这一步理解错了会发生什么。如果我们采用的是无界队列,那会直接导致我们的任务被无限制的添加到队列中而不会主动去扩充线程数,如果核心线程消费速度小于生产速度,会直接导致OOM,这点我们务必要确认清楚

```java
    int c = ctl.get();
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        if (! isRunning(recheck) && remove(command))
            reject(command);
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    else if (!addWorker(command, false))
        reject(command);
```

强调完一个巨坑以后,我们来看一下线程池源码

关于线程池源码的几个特点:

1.之前提到的三步添加任务的坑

2.为了方便读取,采用了位运算将线程池状态和工作线程数整合到了一个AtomicInteger中,高位代表状态,低位代表工作现场数

```java
//这是整合了线程池状态和工作线程数的AtomicInteger
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
//这是解析ctl线程池状态和工作线程数的几个方法
private static int runStateOf(int c)     { return c & ~CAPACITY; }
private static int workerCountOf(int c)  { return c & CAPACITY; }
private static int ctlOf(int rs, int wc) { return rs | wc; }
```

3.执行任务的行为:我们可以通过runWorker方法(该方法被Worker中的run调用,一个Worker即可以当作一个封装好的工作线程)看到,该方法中有一个while循环调用,条件分为两种情况,如果是该线程运行的首个任务,会直接接收任务运行(该任务被封装在Worker中),如果是后续任务,则会从队列中取得任务进行(在getTask方法中会对我们设置的线程超时时间来进行判断,从而起到当非核心线程超时时,返回false使其跳出while循环,从而终止该线程)

4.核心线程和非核心线程其实并无区别,AtomicInteger大家都知道,可以做到原子性操作,当工作线程数大于核心线程数且超时时,会同时请求减少工作线程数,竞争到的则作为非核心线程而关闭,没有竞争到的作为核心线程继续运行。可见,两种线程仅仅是称呼上的不同

5.无处不在的双重检查,即时用了双重检查,但是由于两行代码之间并不是原子性的,我们无法保证双重检查一定可以避免并发操作产生的时间间隙,但确实可以避免一些并发造成的间隙。但是既然线程池是如此操作的,那么很大概率上可能编程人员对此进行过测试,认为双重检查有助于提高效率。另外双重检查也有预防一些因时间间隙产生的BUG的作用,下文会讲到

6.类似beforeExecute这种方法,里面没有做什么事情,这是留给子类来实现的钩子函数

讲完了特点这里列出几个最重要的函数来说一下,其他的自己看就可以了

addWorker函数主要分为两个部分,第一部分是增加工作线程计数,第二部分是创建工作线程,详细可以看代码块中我的注释

```java
private boolean addWorker(Runnable firstTask, boolean core) {
    retry:
    //for循环内的这部分是增加工作线程计数
    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);
        /*
        这里还是双重检测的作用,即时在调用addWorker之前已经检查过一遍了,比较有趣的是if中的逻辑判断
        有点绕这里来解释下,意思为:如果线程池已经SHUTDOWN了,那么它将不再接受新的任务开启工作线程
        (这里我们之前提到过,如果工作线程小于核心线程时,添加的任务会直接在新创建的线程执行)
        但是有一种情况除外,那就是在SHUTDOWN状态下,firstTask等于null且队列中仍有任务(从文章一开始的三步
        添加任务中可以看到,如果第一步判断时工作线程大于等于核心线程数,但是在判断结束之后工作线程全部
        结束了(如果核心线程数设置为0或者线程池被结束了(当线程池被结束时,已经添加进来的任务不会被结束而是
        等待其执行完毕)),那么如果后续没有任务再添加,我们现在队列中的任务将永远不会被执行
        所以在三步添加任务的第二步中的双重检查,有防止这个问题发生的作用,他通过给addWorker传递一个值
        为null的firstTask,促使其创建出一个工作线程继续完成队列中的任务)
        */
        if (rs >= SHUTDOWN &&
            ! (rs == SHUTDOWN &&
                firstTask == null &&
                ! workQueue.isEmpty()))
            return false;

        for (;;) {
            int wc = workerCountOf(c);
            if (wc >= CAPACITY ||
                wc >= (core ? corePoolSize : maximumPoolSize))
                return false;
            if (compareAndIncrementWorkerCount(c))
                break retry;
            c = ctl.get();  // Re-read ctl
            if (runStateOf(c) != rs)
                continue retry;
            // else CAS failed due to workerCount change; retry inner loop
        }
    }
    //接下来的为创建工作线程
    boolean workerStarted = false;
    boolean workerAdded = false;
    Worker w = null;
    try {
        w = new Worker(firstTask);
        final Thread t = w.thread;
        if (t != null) {
            //这里采用锁,个人认为是为了防止设置largestPoolSize时出现并发问题
            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {
                // Recheck while holding lock.
                // Back out on ThreadFactory failure or if
                // shut down before lock acquired.
                int rs = runStateOf(ctl.get());

                if (rs < SHUTDOWN ||
                    (rs == SHUTDOWN && firstTask == null)) {
                    if (t.isAlive()) // precheck that t is startable
                        throw new IllegalThreadStateException();
                    workers.add(w);
                    int s = workers.size();
                    if (s > largestPoolSize)
                        largestPoolSize = s;
                    workerAdded = true;
                }
            } finally {
                mainLock.unlock();
            }
            if (workerAdded) {
                t.start();
                workerStarted = true;
            }
        }
    } finally {
        if (! workerStarted)
            //如果创建失败,会减少工作线程计数
            addWorkerFailed(w);
    }
    return workerStarted;
}
```

运行任务的方法,这个方法有个特点是,如果线程池结束,他仅仅会interrupt标识一下(这里有必要说明,interrupt并不是强制终止线程,而是做标记,具体线程的反应依赖于其代码实现),而不会做出任何反应,其终止仅仅依赖于getTask()的返回值

```java
final void runWorker(Worker w) {
    Thread wt = Thread.currentThread();
    Runnable task = w.firstTask;
    w.firstTask = null;
    w.unlock(); // allow interrupts
    boolean completedAbruptly = true;
    try {
        //根据上文,如果传入的任务为null,或者如果第一个任务执行完了,则会去队列中获取任务
        while (task != null || (task = getTask()) != null) {
            w.lock();
            // If pool is stopping, ensure thread is interrupted;
            // if not, ensure thread is not interrupted.  This
            // requires a recheck in second case to deal with
            // shutdownNow race while clearing interrupt
            if ((runStateAtLeast(ctl.get(), STOP) ||
                    (Thread.interrupted() &&
                    runStateAtLeast(ctl.get(), STOP))) &&
                !wt.isInterrupted())
                //这里仅仅做标记,并无法真正的终止线程
                wt.interrupt();
            try {
                beforeExecute(wt, task);
                Throwable thrown = null;
                try {
                    //真正执行用户定义的任务
                    task.run();
                } catch (RuntimeException x) {
                    //抛出异常并不会造成终止,而是被本方法中外围的try捕获了,这一点可以参考FutureTask类中的操作,会将异常放置到一个对象中
                    thrown = x; throw x;
                } catch (Error x) {
                    thrown = x; throw x;
                } catch (Throwable x) {
                    thrown = x; throw new Error(x);
                } finally {
                    afterExecute(task, thrown);
                }
            } finally {
                task = null;
                w.completedTasks++;
                w.unlock();
            }
        }
        completedAbruptly = false;
    } finally {
        processWorkerExit(w, completedAbruptly);
    }
}
```

getTask方法

```java
private Runnable getTask() {
    boolean timedOut = false; // Did the last poll() time out?

    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);

        // Check if queue empty only if necessary.
        if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
            decrementWorkerCount();
            return null;
        }

        int wc = workerCountOf(c);

        // Are workers subject to culling?
        boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;

        if ((wc > maximumPoolSize || (timed && timedOut))
            && (wc > 1 || workQueue.isEmpty())) {
            if (compareAndDecrementWorkerCount(c))
            /*
            此处就是之前提到的,CAS竞争从而停掉非核心线程,这里要强调一下。
            如果竞争失败,则会超时等待从队列中获取任务；如果超时,则进行下一轮竞争请求返回null从而停掉本线程,直到工作线程等于核心
            线程数，才会一直阻塞等待从队列获取任务(前提是allowCoreThreadTimeOut==false,当然他的默认
            值就是false,除非你主动对其进行了改变)
            */
                return null;
            continue;
        }

        try {
            Runnable r = timed ?
                workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                workQueue.take();
            if (r != null)
                return r;
            timedOut = true;
        } catch (InterruptedException retry) {
            timedOut = false;
        }
    }
}
```