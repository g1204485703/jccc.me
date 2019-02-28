---

title: Java基础-ArrayList
date: 2019-02-28 21:30:00
categories: Java
tags: [Java,Java基础,Java集合类,ArrayList]

---

# ArrayList

​	ArrayList是List接口下的动态数组的一个具体实现，相当于Array的复杂形式，提供所有的可选列表操作，允许存储包括null在内的所有元素。

## ArrayList的类关系总览

​	ArrayList是Collection下的List接口下的一个具体实现类

![ArrayList](/imag/ArrayList.png)ArrayList底层通过数组实现，会随着存入元素的增加而进行动态扩容。

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

​	ArrayList的无参默认构造是

​	

## 扩容机制

​	以add();方法为例，在使用add(E e);方法添加元素时，会调用ensureCapacityInternal(size + 1);方法，确保ArrayList的容量能够完成此次的元素新增。

```java
/** 抽象类AbstractList中的属性，记录List结构修改的次数 */
protected transient int modCount = 0;

/** 当前ArrayList的大小 */
private int size;

/** ArrayList的最大容量 */
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

/** 当前ArrayList存储的元素数组 */
transient Object[] elementData;

/** 添加元素 */
public boolean add(E e) {
	ensureCapacityInternal(size + 1);  // Increments modCount!!
	elementData[size++] = e;
	return true;
}

/** 返回最小需要容量 */
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    // 如果当前为空数组，则返回默认大小和所需大小中的较大值
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    return minCapacity;
}

/** 确保容量 */
private void ensureCapacityInternal(int minCapacity) {
	ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}

/
private void ensureExplicitCapacity(int minCapacity) {
    // ArrayList结构修改数自增
	modCount++;

	// 最小需要容量大于当前数据长度时，对ArrayList进行扩容
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

