---
title: Java基础-LinkedList
date: 2019-03-04 09:30:00
categories: Java
tags: [Java,Java基础,Java集合类,LinkedList]
---

# LinkedList

​	LinkedList也是List接口下的一个具体实现，与ArrayList不同的是，ArrayList是以可变数组的形式实现，而LinkedList是以链表的形式实现。同时也实现了Deque-**双端队列**（是一种具有队列和栈性质的抽象数据类型。 双端队列中的元素可以从两端弹出，插入和删除操作限定在队列的两边进行）。

特点：

- 有序链表，可存入重复元素，元素也可为null
- 内部实现为链表，并且是双向链表结构，除头尾节点外，每个元素都含有前后元素的引用
- 线程不安全，内部所有方法非同步方法
- 相比ArrayList随机插入、随机删除效率较高，顺序读取、顺序

<!--more-->

## LinkedList的类关系总览

![LinkedList](/imag/LinkedList.png)

## 构造方法

### 默认构造

​	LinkedList的默认构造无任何操作，这也是其的结构所决定，由节点记录整个链表的相互关系。

```java
/**
 * Constructs an empty list.
 */
public LinkedList() {
}
```

### 有参构造

​	通过传入一个实现Collection接口的容器，构建一个包含该容器所有元素的LinkedList对象。

```Java
/**
 * Constructs a list containing the elements of the specified
 * collection, in the order they are returned by the collection's iterator.
 *
 * @param  c the collection whose elements are to be placed into this list
 * @throws NullPointerException if the specified collection is null
 */
 public LinkedList(Collection<? extends E> c) {
 	this();
 	addAll(c);
 }
```

## 属性

### 节点

​	LinkedList以内部静态类`Node<E>`作为链表的节点。

​	`Node`有三个属性，`E item`用于存储当前节点所需要存储的元素，`Node<E> next`用于存储当前节点的后一个节点，`Node<E> prev`用于存储当前节点的前一个节点。

​	通过`Node`记录当前节点的前后节点，组成双向链表。在`Node`链表中，除首位节点外，每个节点都能被正向或逆向访问到，所以LinkedList需要记录链表的首尾节点。

```Java
private static class Node<E> {
    // 当前节点存储的元素
    E item;
    // 当前节点的下一个节点
    Node<E> next;
    // 当前节点的前一个节点
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

### 首尾节点

​	由于链表结构的特性，LinkedList需要记录链表的首尾节点。

​	记录尾节点可在添加尾部节点时快速添加，而无需从头部开始遍历，极大提高了效率。

```Java
/**
 * Pointer to first node.
 * Invariant: (first == null && last == null) ||
 *            (first.prev == null && first.item != null)
 */
 transient Node<E> first;

/**
 * Pointer to last node.
 * Invariant: (first == null && last == null) ||
 *            (last.next == null && last.item != null)
 */
 transient Node<E> last;
```

## 添加元素

### 直接添加

​	对`List`接口实现的`add(E e)`方法是直接在链表尾部追加元素。

```Java

/**
 * Appends the specified element to the end of this list.
 *
 * <p>This method is equivalent to {@link #addLast}.
 *
 * @param e element to be appended to this list
 * @return {@code true} (as specified by {@link Collection#add})
 */
public boolean add(E e) {
	linkLast(e);
	return true;
}

```

### 尾部添加

```Java
/**
 * Links e as last element.
 */
void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;
    // 判断当前LinkedList是否含有元素
    // 如果没有元素则把当前元素赋值给首节点
    // 如果已有元素则直接追加
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    size++;
    modCount++;
}
```

### 首部添加

```Java
/**
 * Links e as first element.
 */
private void linkFirst(E e) {
	final Node<E> f = first;
	final Node<E> newNode = new Node<>(null, e, f);
	first = newNode;
    // 判断当前LinkedList是否含有元素
    // 如果没有元素则把当前元素赋值给尾节点
    // 如果已有元素则直接追加
	if (f == null)
		last = newNode;
	else
		f.prev = newNode;
	size++;
	modCount++;
}
```

ummmmm，LinkedList好像也没啥写的，就先这样吧。ε=ε=ε=┏(゜ロ゜;)┛ 逃

