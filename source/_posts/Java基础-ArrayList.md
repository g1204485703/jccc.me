---
title: Java基础-ArrayList
date: 2019-02-28 21:30:00
categories: Java
tags: [Java,Java基础,Java集合类,ArrayList]

---

# ArrayList

​	ArrayList是List接口下的动态数组的一个具体实现，相当于Array的复杂形式，提供所有的可选列表操作，允许存储包括null在内的所有元素。底层通过数组实现，随着元素的增加而动态地扩容。

特点：

- 有序数组，可存入重复元素，元素也可为null
- 内部实现为数组，是一段连续的内存
- 容量不固定，随所需容量动态扩容
- 线程不安全，内部所有方法非同步方法
- 相比LinkedList随机插入、随机删除效率较低，顺序插入、读取效率较高。

<!--more-->

## ArrayList的类关系总览

​	ArrayList是Collection下的List接口下的一个具体实现类。

![ArrayList](/imag/ArrayList.png)

## ArrayList构造方法

### 默认构造

​	在默认构造下，默认创建一个元素数组为空的集合，直接将`DEFAULTCAPACITY_EMPTY_ELEMENTDATA`赋值给了`elementData`。

```java
/**
 * Default initial capacity.
 */
private static final int DEFAULT_CAPACITY = 10;

/**
 * Shared empty array instance used for empty instances.
 */
private static final Object[] EMPTY_ELEMENTDATA = {};

/**
 * Shared empty array instance used for default sized empty instances. We
 * distinguish this from EMPTY_ELEMENTDATA to know how much to inflate when
 * first element is added.
 */
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

/**
 * The array buffer into which the elements of the ArrayList are stored.
 * The capacity of the ArrayList is the length of this array buffer. Any
 * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
 * will be expanded to DEFAULT_CAPACITY when the first element is added.
 */
transient Object[] elementData; // non-private to simplify nested class access

/**
 * Constructs an empty list with an initial capacity of ten.
 */
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```

### 有参构造

​	ArrayList提供两种有参构造，分别为自定义初始容量构造和自定义元素构造。

#### 自定义初始容量构造

​	定义初始容量构造，创建ArrayList对象时，提供一个默认的初始容量用来创建相应容量的ArrayList对象。一般在容量确定的情况下使用，可节约ArrayList所需占用的内存。在容量较大的情况下，给定默认大小也可以减少扩容次数，提高效率。

```java
/**
 * Constructs an empty list with the specified initial capacity.
 *
 * @param  initialCapacity  the initial capacity of the list
 * @throws IllegalArgumentException if the specified initial capacity
 *         is negative
 */
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: " + initialCapacity);
    }
}
```

#### 自定义元素构造

​	ArrayList的元素构造是通过传入一个实现Collection接口的容器，构建一个包含该容器所有元素的ArrayList对象。

```Java
/**
 * Constructs a list containing the elements of the specified
 * collection, in the order they are returned by the collection's
 * iterator.
 *
 * @param c the collection whose elements are to be placed into this list
 * @throws NullPointerException if the specified collection is null
 */
public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();
    if ((size = elementData.length) != 0) {
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // replace with empty array.
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```

### 关于`EMPTY_ELEMENTDATA`和`DEFAULTCAPACITY_EMPTY_ELEMENTDATA`

​	在上述构造方法中我们可以看到ArrayList对两个空数组`EMPTY_ELEMENTDATA`和`DEFAULTCAPACITY_EMPTY_ELEMENTDATA`的使用，那么两者有何区别呢？

​	通过阅读构造方法不难发现，`DEFAULTCAPACITY_EMPTY_ELEMENTDATA`是在创建新的ArrayList中使用，表示该ArrayList对象是新建状态，暂时没有存入任何元素，此时ArrayList的容量暂时没有进行初始化。而`EMPTY_ELEMENTDATA`则是在创建新的ArrayList的初始容量为0的时候使用。

​	简单区分就是，在新建初始容量为0的ArrayList对象时，该对象的`elementData`是`EMPTY_ELEMENTDATA`,代表该ArrayList是一个不含任何元素的空数组。在新建未知初始容量或者初始容量不为0的ArrayList时，该对象的`elementData`时`DEFAULTCAPACITY_EMPTY_ELEMENTDATA`，代表当前ArrayList的容量暂未初始化，在新增元素时需进行扩容操作。

​	两者区分使用的实际应用就是在`calculateCapacity(Object[] elementData, int minCapacity)`方法中，如果当前`elementData`是`DEFAULTCAPACITY_EMPTY_ELEMENTDATA`，对当前扩容操作时提供最小为10的默认容量，以减少低容量时的多次扩容操作，提高性能。

## 扩容机制

​	以`add()`方法为例，在使用`add(E e)`方法添加元素时，会调用`ensureCapacityInternal(size + 1)`方法，确保ArrayList的容量能够完成此次的元素新增。

```java
/** 抽象类AbstractList中的属性，记录List结构修改的次数 */
protected transient int modCount = 0;

/** 当前ArrayList的大小 */
private int size;

/** ArrayList的最大容量 */
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

/** 当前ArrayList用来存储元素的数组 */
transient Object[] elementData;

/** 添加元素 */
public boolean add(E e) {
	ensureCapacityInternal(size + 1);  // Increments modCount!!
	elementData[size++] = e;
	return true;
}

/** 容量检测，返回最小需要容量 */
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    // 如果当前是首次扩容，则返回默认大小10和所需大小中的较大值
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    return minCapacity;
}

/** 确保容量 */
private void ensureCapacityInternal(int minCapacity) {
	ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}

/** 为新增元素做结构修改次数的自增，并在必要时扩容 */
private void ensureExplicitCapacity(int minCapacity) {
    // ArrayList结构修改数自增
	modCount++;

	// 最小需要容量大于当前存储数组长度时，对ArrayList进行扩容
	if (minCapacity - elementData.length > 0)
		grow(minCapacity);
}

/** 扩容 */
private void grow(int minCapacity) {
    // 记录就数组的长度
    int oldCapacity = elementData.length;
    // 新数组长度为旧数组的1.5倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    // 扩容1.5倍后还是不足以存储当前元素，直接扩容为当前最小需要容量
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    // 当前需要容量超过ArrayList的最大容量
    // 若newCapacity值溢出，会在此步排除
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}

/** 大额容量处理 */
private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
    MAX_ARRAY_SIZE;
}
```

