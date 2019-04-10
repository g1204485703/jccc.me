---
title: Java基础-HashMap
date: 2019-03-06 15:30:00
updated: 2019-03-06 15:30:00
categories: Java
tags: [Java,Java基础,Java集合类,HashMap]
---

# HashMap



<!--more-->

## 属性

```Java
/** 
 * HashMap中的Node数组
 * Node是链表的节点，存储元素并组成链表
 * 在第一次使用的时候初始化，长度是2的幂 
 */
transient Node<K,V>[] table;

/** 键值对的缓存 */
transient Set<Map.Entry<K,V>> entrySet;

/** HashMap中存储的键值对数量 */
transient int size;

/** 结构变化次数 */
transient int modCount;

/** 扩容阈值，threshold = capacity * loadFactor */
int threshold;

/** 负载因子 */
final float loadFactor;
```

## 成员变量

```Java
/** 默认容量 */
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

/** 最大容量 */
static final int MAXIMUM_CAPACITY = 1 << 30;

/** 默认负载因子 */
static final float DEFAULT_LOAD_FACTOR = 0.75f;

/** 链表转换成红黑树的bin数量阈值 */
static final int TREEIFY_THRESHOLD = 8;

/** 红黑树装换成链表的bin数量阈值 */
static final int UNTREEIFY_THRESHOLD = 6;

/** 
 * 开启链表转换红黑树的最小HashMap容量 
 * 在HashMap的桶满足转红黑树条件，但是当前HashMap容量小于该值时，不会进行转换，而是对HashMap进行扩容
 */
static final int MIN_TREEIFY_CAPACITY = 64;
```

## 内部类

### 基础节点

```Java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;

    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }
}
```

### 红黑树节点

```Java
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    // 父节点
    TreeNode<K,V> parent;  // red-black tree links
    TreeNode<K,V> left;
    TreeNode<K,V> right;
    // 删除后需要取消的节点
    TreeNode<K,V> prev;    // needed to unlink next upon deletion
    boolean red;
    TreeNode(int hash, K key, V val, Node<K,V> next) {
        super(hash, key, val, next);
    }
}
```



## 添加元素

```Java
/** 默认添加元素的方法 */
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // 如果当前table为空或者table长度为0，进行首次扩容
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 将要存入key的hash值与当前table的容量进行与运算，进行桶定位
    // 如果定位的桶是空桶，则直接新建节点并赋值
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    // 如果定位的桶不是空桶
    else {
        Node<K,V> e; K k;
        // 判断要存入的key和当前桶位的key是否相同（==或者equals）
        // 相同的话将当前桶位的Node赋值给e
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        // 不同情况下，判断是否是树节点
        // 是树节点将要存入的元素作为树节点追加
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        // key不相同，当前桶位也不是树节点，将要存入的元素作为链表节点追加
        else {
            // 遍历当前桶位的链表
            for (int binCount = 0; ; ++binCount) {
                // 已到达最末尾节点
                if ((e = p.next) == null) {
                    // 将当前要存入的元素追加到链表尾部
                    p.next = newNode(hash, key, value, null);
                    // 判断当前桶位链表长度是否达到转换红黑树的阈值
                    if (binCount >= TREEIFY_THRESHOLD - 1)
                        // 达到阈值，则进行链表转红黑树
                        treeifyBin(tab, hash);
                    // 完成追加则结束遍历
                    break;
                }
                // 如果已存在当前要存入元素的key，也跳出遍历
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        // 如果当前要存入的key已存在
        if (e != null) {
            V oldValue = e.value;
            // 只有要求覆盖，或者旧值为null时，写入新值
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            // 节点访问后的回调函数
            afterNodeAccess(e);
            return oldValue;
        }
    }
    // 记录结构变化
    ++modCount;
    // 如果当前存入的元素数量大于扩容阈值，进行扩容
    if (++size > threshold)
        resize();
    // 节点插入后的回调函数
    afterNodeInsertion(evict);
    return null;
}

/** 向红黑树中添加元素 */
final TreeNode<K,V> putTreeVal(HashMap<K,V> map, Node<K,V>[] tab,
                               int h, K k, V v) {
    Class<?> kc = null;
    boolean searched = false;
    // 获取红黑树的根节点
    TreeNode<K,V> root = (parent != null) ? root() : this;
    // 从根节点开始循环，寻找位置添加元素
    for (TreeNode<K,V> p = root;;) {
        int dir, ph; K pk;
        // 当前节点hash值大于要插入元素的哈希值，将向左子树查找
        if ((ph = p.hash) > h)
            dir = -1;
        // 当前节点hash值小于要插入元素的哈希值，将向右子树查找
        else if (ph < h)
            dir = 1;
        // 要插入元素的key与当前节点的key相同
        else if ((pk = p.key) == k || (k != null && k.equals(pk)))
            return p;
        else if ((kc == null &&
                  (kc = comparableClassFor(k)) == null) ||
                 (dir = compareComparables(kc, k, pk)) == 0) {
            if (!searched) {
                TreeNode<K,V> q, ch;
                searched = true;
                if (((ch = p.left) != null &&
                     (q = ch.find(h, k, kc)) != null) ||
                    ((ch = p.right) != null &&
                     (q = ch.find(h, k, kc)) != null))
                    return q;
            }
            dir = tieBreakOrder(k, pk);
        }

        TreeNode<K,V> xp = p;
        if ((p = (dir <= 0) ? p.left : p.right) == null) {
            Node<K,V> xpn = xp.next;
            TreeNode<K,V> x = map.newTreeNode(h, k, v, xpn);
            if (dir <= 0)
                xp.left = x;
            else
                xp.right = x;
            xp.next = x;
            x.parent = x.prev = xp;
            if (xpn != null)
                ((TreeNode<K,V>)xpn).prev = x;
            moveRootToFront(tab, balanceInsertion(root, x));
            return null;
        }
    }
}
```

## 扩容

```Java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    // 旧容量
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    // 旧扩容阈值
    int oldThr = threshold;
    int newCap, newThr = 0;
    // 非首次扩容
    if (oldCap > 0) {
        // 如果旧容量
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    // 首次扩容将容量扩大至默认值
    else {
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // 初始化扩容阈值
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    // 新建新容量的newTab数组
    @SuppressWarnings({"rawtypes","unchecked"})
    	Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    // 此处，把新建的newTab数组赋值给了table
    table = newTab;
    // 旧表非空则开始节点转移
    if (oldTab != null) {
        // 遍历桶位
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            // 当前桶位非空
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                // 桶位仅有头节点存在，直接赋值给新表重新定位后的新桶位
                if (e.next == null)
                    // 此步骤未重新计算hash值，只是重新计算所处的桶位
                    newTab[e.hash & (newCap - 1)] = e;
                // 是树节点直接使用树节点的赋值方法
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                // 桶位为链表节点且不仅含有头节点
                else { // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    // 遍历取出旧节点放置新table中合适位置
                    do {
                        next = e.next;
                        // 放回原位置桶位
                        if ((e.hash & oldCap) == 0) {
                            // 首元素插入时放到头部
                            if (loTail == null)
                                loHead = e;
                            // 非首元素，在尾部追加新元素
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        // 放到原位置+oldCap桶位
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    // 新table中还处于j位置桶位转移
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    // 新table中处于（j+oldCap）位置的桶位转移
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

