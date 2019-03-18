---
title: Java基础-类加载器
date: 2019-03-18 15:00:00
categories: Java
tags: [Java,Java基础,类加载器]
---

# 定义

Java类加载器是Java运行时环境的一部分，负责动态加载Java类到Java虚拟机的内存空间中，类通常是按需加载，即第一次使用该类时才加载。由于有了类加载器，Java运行时系统不需要知道文件与文件系统。

Java源文件（.java）在经过Java编译器编译之后就被转换成Java自己代码（.class）。类加载器负责读取Java字节代码，并转换成`java.lang.Class`类的一个实例，每个这样的实例用来表示一个Java类。通过此实例的`newInstance()`方法，就可以创建出该类的一个对象。

<!--more-->

# JVM中默认的类加载器

![img](/imag/v2-4b379b6a26ae8567dedae02295805cf3_hd.png)

## 启动（Bootstrap）类加载器

由原生代码（如C++语言）编写，是虚拟机的一部分，不继承自`java.lang.ClassLoader`，也不对应Java代码。负责将`<JAVA_HOME>/lib`下的核心Java库或`-Xbootclasspath`参数指定的路径下的jar包加载到内存中。

## 扩展（Extension）类加载器

是位于`sun.misc.Launcher$ExtClassLoader`的类，是`Launcher`类的内部类。负责加载`<JAVA_HOME>/lib/ext`目录下或`-Djava.ext.dir`指定的路径下的类库。父类加载器是启动类加载器，但在JVM环境中，因为启动类加载器并不对应Java代码，故对父类加载器取值时为null。

## 系统（System）类加载器

是位于`sun.misc.Launcher$AppClassLoader`的类，是`Laucher`类的内部类。负责加载`java -classpath`目录下或`-D java.class.path`指定的路径下的类库。一般情况下系统类加载器是程序中默认的类加载器。父类加载器为扩展类加载器。

## 查看类加载器

新建对象，先获取`class`，再通过`getclassLoader()`方法即可查看加载该类的类加载器。

```Java
public class Test {
    public static void main(String[] args) {

        System.out.println(new ArrayList().getClass().getClassLoader());
        System.out.println(new Test().getClass().getClassLoader());
    }
}
```

输出可见，`lib`目录下的类`ArrayList`由启动类加载器加载，且启动类加载器并不存在于Java代码，故输出为null

`classpath`下的类`Test`是由系统类加载器加载的

```Java
null
sun.misc.Launcher$AppClassLoader@18b4aac2
```

# 双亲委派模型

抽象类`ClassLoader`中定义的类加载方法`loadClass(String name, boolean resolve)`在类加载过程中，明确先调用了父加载器加载Class，只有在父加载器加载失败后，才会使用启动类加载器进行加载。

类加载器调用父加载器进行类加载，即称作双亲委派加载。

```Java
public abstract class ClassLoader {
    // The parent class loader for delegation
    // Note: VM hardcoded the offset of this field, thus all new fields
    // must be added *after* it.
    private final ClassLoader parent;
    
    protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        // 获取该类的类加载锁
        synchronized (getClassLoadingLock(name)) {
            // 检查该类是否已被加载
            Class<?> c = findLoadedClass(name);
            // 未被加载时即开始加载
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    // 如果当前类加载器的父加载器不为空，则交给父加载器加载
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    // 没有父加载器，则由启动类加载器加载    
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // 如果还是没有加载成功，则使用findClass方法加载类
                    long t1 = System.nanoTime();
                    c = findClass(name);

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            // 判断是否需要链接
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
}
```

# 类加载过程

类加载过程中包括了**加载**、**验证**、**准备**、**解析**、**初始化**五个阶段。在这五个阶段中，`加载`、`验证`、`准备`和`初始化`这四个阶段发生的顺序时确定的，而`解析`阶段则不一定，它在某些情况下可以在`初始化`阶段之后开始，这是为了支持Java语言的运行时绑定（动态绑定）。并且，这几个阶段是按顺序开始，而不是按顺序进行或完成，因为这些阶段通常是相互交叉混合进行，通常在一个阶段执行的过程中调用或激活另一个阶段。

![类加载过程](/imag/331425-20160621125942490-979409578.png)

## 加载

主要负责查找并加载类的二进制数据

虚拟机完成如下任务

1. 通过一个类的全限定名来获取其定义的二进制字节流
2. 将这个字节流所代表的静态存储结构转换为方法区的运行时数据结构。
3. 在Java堆中生成一个代表这个类的`java.lang.Class`对象，作为对方法区中这些数据的访问入口。

加载过程由类加载器完成，同时缓存所有已加载的类，加双亲委派机制，控制值单类只加载一次。

## 连接

### 验证

### 准备

### 解析

## 初始化

