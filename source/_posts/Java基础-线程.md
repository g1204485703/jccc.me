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

## 实现Runnable接口