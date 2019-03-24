---
title: Java基础-垃圾收集器
date: 2019-03-20 21:00:00
categories: Java基础
tags: [Java,Java基础,垃圾收集器]
---

# 概述

**垃圾收集（Garbage Collection，GC）**最早起源于Lisp语言，是一种自动的存储管理机制。在Java中，垃圾收集是JVM中重要的组成部分。当需要排查各种内存溢出、内存泄漏问题时，以及垃圾收集称为系统达到更高并发量的瓶颈时，我们就需要对这些“自动化”技术实施必要的监控和调节。

在Java堆中，一个接口的多个实现类需要的内存可能不一样，一个方法中的多个分支需要的内存也可能不一样，只有在程序处于运行期间时才能知道创建哪些对象，这部分内存的分配和回收都是动态的，这就是垃圾收集所关注的内存。

<!--more-->

# 对象存活判断算法

## 引用计数法

**引用计数法**是指给对象添加一个引用计数器，每当有一个地方引用它时，计数器就加1；当引用失效时，计数器值就减1；任何时刻计数器都为0的对象就是不可能再被使用的。

引用计数法难以解决对象之间相互循环引用的问题，例如

```Java
public class ReferenceCounting {

    private ReferenceCounting rc;

    public static void main(String[] args) {
        ReferenceCounting rc1 = new ReferenceCounting();
        ReferenceCounting rc2 = new ReferenceCounting();
        rc1.rc = rc2;
        rc2.rc = rc1;

        rc1 = null;
        rc2 = null;
    }
}
```

创建相互引用的`rc1`和`rc2`，随后赋值为`null`。此时因为两对象的相互引用，即使`rc1`,`rc2`都不再指向对象，但是对象的引用计数器都不为0，所以无法被回收。

## 根搜索算法

**根搜索算法**是指通过一系列的名为“GC Roots”的对象作为起始点，从这些节点开始向下搜索，搜索所走过的路径称为*引用链（Reference Chain）*，当一个对象到GC Roots没有任何引用链相连（即从*GC Roots* 到这个对象不可达）时，则证明此对象是不可用的。

如下图中的*Object1*，*Object2*，*Object3*，*Object4*是可以通过*GC Roots* 到达的，即为可用对象；*Object5*，*Object6*，*Object7* 对象之间有关联，但不可通过*GC Roots* 到达，即为不可用对象。

![根搜索算法](E:/jccc.me-old/source/_posts/imag/%E6%A0%B9%E6%90%9C%E7%B4%A2%E7%AE%97%E6%B3%95.png)

在Java中，可作为*GC Roots* 的对象包括下面几种

- 虚拟机栈（栈帧中的本地变量表）中的引用的对象
- 方法区中的静态类属性引用的对象
- 方法区中常量引用的对象
- 本地方法栈中`native`方法的引用的对象

# 附录

## 引用

无论是通过**引用计数法**还是**根搜索算法**判断对象的引用链是否可达，判断对象是否存活都与**引用**有关。

JDK1.2之前，Java对引用的定义很传统：如果reference类型的数据中存储的数值代表的是另外一块内存的起始地址，就称这块内存代表着一个引用。

JDK1.2之后，Java对引用的概念进行了扩充，将引用分为**强引用（Strong Reference）**、**软引用（Soft Reference）**、**弱引用（Weak Reference）**、**虚引用（Phantom Reference）**，这四种引用强度依次逐渐减弱。

### 强引用

**强引用**就是指在程序代码中普遍存在的，类似`Object obj = new() Object()`这类的引用，只要强引用还存在，垃圾收集器就永远不会回收掉被引用的对象。

### 软引用

**软引用**用来描述一些还有用，但是非必需的对象。对于软引用关联的对象，在系统将要发生内存溢出之前，将会把这些对象列进回收范围之中并进行第二次回收。如果这次回收还是没有足够的内存，将会抛出`OutOfMemoryError`异常。JDK1.2之后，提供`java.lang.ref.SoftReference<T>`实现。

### 弱引用

### 虚引用