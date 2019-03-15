---
title: Java基础-线程
date: 2019-03-12 09:30:00
categories: Java
tags: [Java,Java基础,线程]
---

# 定义

线程（thread）是操作系统能够进行运算调度的最小单位。它被包含在进程之中，是进程的实际运作单位。一条线程指的是进程中的一个单一顺序的控制流，一个进程中可以并发多个线程，每条线程并行执行不同的任务。

线程是独立调度和分派的基本单位。

同一进程中的多条线程将共享该进程中的全部系统资源。

<!--more-->

# 状态

`Thread.State`中定义了6种线程状态，任一时间，任一线程当且只能处于一种状态。

## 新建（New）

```Java
/**
 * Thread state for a thread which has not yet started.
 */
 NEW
```

线程对象创建完成即进入新建状态。

## 就绪（Runnable）

```Java
/**
 * Thread state for a runnable thread.  A thread in the runnable
 * state is executing in the Java virtual machine but it may
 * be waiting for other resources from the operating system
 * such as processor.
 */
 RUNNABLE
```

调用线程对象的`start()`方法，线程即进入就绪状态。

线程正在Jvm中运行的状态，但是在操作系统中可能处于Ready或者Running状态。

## 阻塞（Blocked）

```Java
/**
 * Thread state for a thread blocked waiting for a monitor lock.
 * A thread in the blocked state is waiting for a monitor lock
 * to enter a synchronized block/method or
 * reenter a synchronized block/method after calling
 * {@link Object#wait() Object.wait}.
 */
 BLOCKED
```

处于运行状态中的线程获取`synchronized`同步锁失败，线程进入阻塞状态。

## 等待（Waiting）

```java
/**
 * Thread state for a waiting thread.
 * A thread is in the waiting state due to calling one of the
 * following methods:
 * <ul>
 *   <li>{@link Object#wait() Object.wait} with no timeout</li>
 *   <li>{@link #join() Thread.join} with no timeout</li>
 *   <li>{@link LockSupport#park() LockSupport.park}</li>
 * </ul>
 */
 WAITING
```

线程由于调用了`Object.wait()`且未设置超时时间，`Thread.join()`且未设置超时时间，`LockSupport.park()`方法，线程进入等待状态。

因为`Object.wait()`等待的线程可由`Object.notify()`或者`Object.notifyAll()`唤醒。

因为`Thread.join()`等待的线程需等待调用线程执行完毕进入终结状态方可唤醒。

## 定时等待（Timed_waiting）

```Java
/**
 * Thread state for a waiting thread with a specified waiting time.
 * A thread is in the timed waiting state due to calling one of
 * the following methods with a specified positive waiting time:
 * <ul>
 *   <li>{@link #sleep Thread.sleep}</li>
 *   <li>{@link Object#wait(long) Object.wait} with timeout</li>
 *   <li>{@link #join(long) Thread.join} with timeout</li>
 *   <li>{@link LockSupport#parkNanos LockSupport.parkNanos}</li>
 *   <li>{@link LockSupport#parkUntil LockSupport.parkUntil}</li>
 * </ul>
 */
 TIMED_WAITING
```

线程由于调用`Thread.sleep(long millis)`,`Object.wait(long timeout)`，`Thread.join(long millis)`，`LockSupport.parkNanos(Object blocker, long nanos)`，`LockSupport.parkUntil(Object blocker, long deadline)`进入定时等待状态，当调用进程终结后，或者达到指定时间结束后，线程进入就绪状态。

## 终止（Terminated）

```Java
/**
 * Thread state for a terminated thread.
 * The thread has completed execution.
 */
 TERMINATED
```

已终止线程的线程状态，线程已经结束运行。

# 新建线程

## 继承Thread类

继承`Thread`类，重写`run()`方法

创建线程的时候直接new一个对象，创建线程对象

调用`start()`方法启动线程

```Java
public class ExThread extends Thread {
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + " is running");
    }

    public static void main(String[] args) {
        for (int i = 0; i < 8; i++) {
            ExThread exThread = new ExThread();
            exThread.start();
        }
    }
}
```

输出

```
Thread-0 is running
Thread-3 is running
Thread-7 is running
Thread-4 is running
Thread-2 is running
Thread-1 is running
Thread-6 is running
Thread-5 is running
```



## 实现Runnable接口

实现`Runnable`接口,实现`run()`方法

新建`RunnableThread`对象，使用`new Thread(Runnable target)`创建线程

调用`start()`方法启动线程

```Java
public class RunnableThread implements Runnable {

    private int power;

    public RunnableThread(int power) {
        this.power = power;
    }

    @Override
    public void run() {
        if (power > 0) {
            System.out.println(
                Thread.currentThread().getName() + " -- power " + this.power--);
        }else {
            System.out.println(
                Thread.currentThread().getName() + " -- no power ");
        }
    }

    public static void main(String[] args) {
        RunnableThread thread = new RunnableThread(5);
        for (int i = 0; i < 8; i++)
            new Thread(thread, "1").start();
    }
}
```

输出

```
Thread-0 --power 5
Thread-1 --power 5
Thread-3 --power 3
Thread-2 --power 4
Thread-6 --power 2
Thread-5 --power 1
Thread-7 --no power 
Thread-4 --no power 
```

此处特意采用特殊输出，不是手误打错，是当前非线程安全的实际体现。

## 实现Callable接口

实现`callable<V>`接口，实现`call()`方法，返回相应的值

新建`CallableThread`对象，使用`new FutureTask(Callable<V> callable)`创建`FutureTask`对象

使用`new Thread(Runnable target)`创建线程

调用`start()`方法启动线程

```Java
public class CallableThread implements Callable<Integer> {
    private int power;

    public CallableThread(int power) {
        this.power = power;
    }

    @Override
    public Integer call() throws Exception {
        if (this.power > 0) {
            System.out.println(Thread.currentThread().getName() + " -- power " + this.power--);
        } else {
            System.out.println(Thread.currentThread().getName() + " -- no power ");
        }
        return this.power;
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        CallableThread callableThread = new CallableThread(5);
        for (int i = 0; i < 8; i++) {
            FutureTask<Integer> futureTask = new FutureTask(callableThread);
            new Thread(futureTask).start();
            System.out.println("futureTask get " + futureTask.get());
        }
    }
}
```

输出

```
Thread-0 -- power 5
futureTask get 4
Thread-1 -- power 4
futureTask get 3
Thread-2 -- power 3
futureTask get 2
Thread-3 -- power 2
futureTask get 1
Thread-4 -- power 1
futureTask get 0
Thread-5 -- no power 
futureTask get 0
Thread-6 -- no power 
futureTask get 0
Thread-7 -- no power 
futureTask get 0
```

## 对比

- 实现`Runnable`，`Callable`接口形式比继承`Thread`形式更加灵活，不受Java单继承的限制。
- 实现`Runnable`，`Callable`接口新建的多个线程可以共享同一个`target`对象，适合多个相同线程来 处理同一份资源的情况。

# 线程池

## `Executors`提供的静态方法

### CachedThreadPool

可缓存的线程池，如果已有线程可复用，则复用已有线程，无可复用则创建线程

可无限扩大，最大线程数量为`Integer.MAX_VALUE`

线程空闲时间超过`60`s就会被终结

适合处理执行时间比较小的任务

```Java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

### FixedThreadPool

固定大小的线程池

根据给定的`nThread`创建该数量的线程

线程即使空闲，也不会被终结

```Java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
```

### ScheduledThreadPool

延时任务线程池

```Java
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}
```

### SingleThreadExecutor

单个线程的线程任务

```Java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

### SingleThreadScheduledExecutor

单线程的延时线程任务

```Java
public static ScheduledExecutorService newSingleThreadScheduledExecutor() {
    return new DelegatedScheduledExecutorService
        (new ScheduledThreadPoolExecutor(1));
}
```

### WorkStealingPool

支持并行的线程池，不给定并行数量时默认创建当前虚拟机可用的CPU数量的并行数

```Java
public static ExecutorService newWorkStealingPool(int parallelism) {
    return new ForkJoinPool
        (parallelism,
         ForkJoinPool.defaultForkJoinWorkerThreadFactory,
         null, true);
}

public static ExecutorService newWorkStealingPool() {
    return new ForkJoinPool
        (Runtime.getRuntime().availableProcessors(),
         ForkJoinPool.defaultForkJoinWorkerThreadFactory,
         null, true);
}
```

## ThreadPoolExecutor

