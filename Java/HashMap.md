HashMap

## 一、HashMap底层原理

### 1.1 概述

HashMap基于Map接口实现，元素以兼键值对的方式存储，并且允许使用null健和null值，因为key不允许重复，因此只能有一个键为null。

另外HashMap不能保证放入元素的顺序，它是无序的，和放入的顺序不能相同。HashMap是线程不安全的。

HashMap的扩容操作是一项很耗时的任务，所以如果能估算Map的容量，最好给他一个默认初始值，避免进行多次扩容。

### 1.2 继承关系

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {

```

### 1.3 基本属性

```java

public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
    
    // 序列化
    private static final long serialVersionUID = 362498820763181265L;
    // 默认大小
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
    // 最大容量
    static final int MAXIMUM_CAPACITY = 1 << 30;
    // 默认负载因子
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    // 链表转树阈值
    static final int TREEIFY_THRESHOLD = 8;
    // 树转链表阈值
    static final int UNTREEIFY_THRESHOLD = 6;
    // 最小树性化容量阈值；即当哈希表中的容量 > MIN_TREEIFY_CAPACITY时，才会有链表转树
    static final int MIN_TREEIFY_CAPACITY = 64

        
        
    
    // 存放元素的数组
    transient Node<K,V>[] table;
    // 存放entry对象
    transient Set<Map.Entry<K,V>> entrySet;
    // 存放元素的个数
    transient int size;
    // 记录了HashMap的修改次数
    transient int modCount;
    // 临界值，一旦实际数量超过临界值(负载因子*容量)，会进行扩容
    int threshold;
    // 负载因子
    final float loadFactor;
```

### 1.4 数据存储结构

HashMap采用Entry数组来存储key-value对，每一个键值对组成了一个Entry实体，Entry类实际上是一个单向的链表结构，它具有Next指针，可以连接下一个Entry实体，以此来解决Hash冲突的问题。

* 数组存储区间是连续的，占用内存严重，故空间复杂度很大。但是数组的二分查找复杂度小，为O(1)；数组的特点是：寻址容易，插入和删除困难。
* 链表存储区间离散，占用内存比较宽松，故空间复杂度很小，但是时间复杂度很大，达到O(N)；链表的特特点，寻址困难，插入和删除容易。

![img](http://img.hurenjieee.com/uPic/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMzQ1Nzcz,size_16,color_FFFFFF,t_70-20200917223145829.png)

从上图中，我们可以发现数据结构是有数组+链表组成，一个长度为16的数组中，每个元素存储的是一个链表的头节点。那么这些元素是按照什么样的规则存储到数组中的，一般情况是通过hash(key.hashCode)%len获得，也就是元素的key的哈希值对应数组的长度取模得到。



#### 1.4.1 数据结构

HashMap里面实现了一个静态内部类Entry，其重要的属性有hash、key、value、next。

```java
// 数组元素
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

```java

// 红黑树
    static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
        TreeNode<K,V> parent;  // red-black tree links
        TreeNode<K,V> left;
        TreeNode<K,V> right;
        TreeNode<K,V> prev;    // needed to unlink next upon deletion
        boolean red;
        TreeNode(int hash, K key, V val, Node<K,V> next) {
            super(hash, key, val, next);
        }
    }
```

#### 1.4.2 构造函数

```java
// 定义构造初始化大小和负载因子
    public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        this.threshold = tableSizeFor(initialCapacity);
    }
    
// 定义构造初始化大小
    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }
    
// 默认参数封装HashMap
    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }
    
// 用一个已有的数据集构建
    public HashMap(Map<? extends K, ? extends V> m) {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        putMapEntries(m, false);
    }
```

#### 1.4.3 PUT机制

1. 判断键值对数组tab[]是否空或者为null，否则默认大小resize()；
2. 根据键值key计算hash值得到插入的数组索引i。
3. 如果tab[i]==null，直接新建节点添加，否则转入4
4. 判断当前数组存储的第一个元素时链表还是红黑树，分别进行处理

```java
// jdk1.8
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        // 判断是否需要初始化
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        //  如果(n - 1) & hash位置数据为空，则在该节点加入数据，p已经时第一个元素了
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        // 节点已经有数据了，开始处理
        else {
            Node<K,V> e; K k;
            // 检查第一个值和输入值是否等等，
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
          	// 如果是树，放入树中
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            // 链表处理数据
          	else {
                for (int binCount = 0; ; ++binCount) {
                    // 如果到达了链表末尾
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        // 链表转树
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    // 判断是key相同，结束遍历
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            // 如果链表中有相同的key，进行数据替换
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        // 扩容
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

#### 1.4.4 GET机制

