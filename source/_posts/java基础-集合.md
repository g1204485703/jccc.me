---

title: Java基础-集合类
date: 2019-02-28 09:30:00
categories: Java
tags: [Java,Java基础,Java集合类]

---

# Java集合类

## Java中的集合类框架

**	集合类**其实是Java中集合的广义说法，在《Java编程思想》的11章提到

> ​	Java实用类库还提供了一套相当完善的容器类来解决这个问题，其中基本的数据类型是**List**、**Set**、**Queue**和**Map**。这些对象类型也称为集合类，但由于Java的类库中使用了Collection这个名字来指代该类库的一个特殊子集，所以我是用了范围更广的术语“容器”称呼他们。

故，本文中以“集合类”称呼这些类型对象，以“集合”称呼Collection的实现类。

​	集合类中基本数据类型有四种，List、Set、Queue和Map，在Java对集合类的设计时，分为了两大类型的接口Collection和Map。Controller即为集合，存储一个元素的集合，具有List、Set和Queue三种结构数据。Map为图，存储键/值对映射，仅有Map一种结构的数据。

![img](/imag/b171c70578a6d6f369066d6dfbc45555.jpg)

<!--more-->

## Collection

### List接口

​	List是有序可重复元素的集合，同时也称为序列，可对其中的每个元素的插入位置进行精准地控制，可以通过索引来访问元素，遍历元素。

​	List接口主要实现类uml图：![LinkedList](/imag/LinkedList-1551342550459.png)

#### ArrayList

​	ArrayList底层通过数组实现，会随着存入元素的增加而进行动态扩容。

##### ArrayList的默认容量是10

​	在默认构造下，默认创建容量为10的ArrayList对象。

```java
/**
 * Default initial capacity.
 */
private static final int DEFAULT_CAPACITY = 10;
```

##### 扩容机制

​	在使用add(E e);方法添加元素时，会调用ensureCapacityInternal(size + 1);方法，确保ArrayList的容量能够完成此次的元素新增。

```java
/**抽象类AbstractList中的属性，记录List结构修改的次数*/
protected transient int modCount = 0;

/**当前ArrayList的大小*/
private int size;

/**ArrayList的最大容量*/
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

/**当前ArrayList存储的元素数组*/
transient Object[] elementData;

public boolean add(E e) {
	ensureCapacityInternal(size + 1);  // Increments modCount!!
	elementData[size++] = e;
	return true;
}

/**返回最小需要容量*/
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    //如果当前为空数组，则返回默认大小和所需大小中的较大值
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    return minCapacity;
}

/**确保容量*/
private void ensureCapacityInternal(int minCapacity) {
	ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}

//
private void ensureExplicitCapacity(int minCapacity) {
	modCount++;

	// overflow-conscious code
	if (minCapacity - elementData.length > 0)
		grow(minCapacity);
}

private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```



#### LinkedList

​	LinkedList底层通过链表实现，随着元素的增加不断向链表尾部追加节点。

### Set

### Queue