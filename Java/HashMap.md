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

1. get(key)方法时获取key的hash值。
2. 计算hash&(n-1)得到在链表数组中国中的位置first=tab[hash&(n-1)]。
3. 先判断first的key是否与参数key相等，如果相等说明值找到了。
4. 判断first是否是TreeNode，如果是则根据树查找
5. 否则遍历链表，遍历查找key。

```java

    public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }

    /**
     * Implements Map.get and related methods.
     *
     * @param hash hash for key
     * @param key the key
     * @return the node, or null if none
     */
    final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
          	// 检查第一个
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            if ((e = first.next) != null) {
              // 检查树
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
              // 检查链表
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }

```

#### 1.4.5 RESIZE机制

构造Hash表时，如果不指明初始大小，默认大小为16（即Node数组大小为16），如果Node[]数组中的元素达到(填充比*Node.length)，则会进行扩容，把HashMap的大小变为原来2倍大小，整个扩容时间很耗时。

```java
// 初始化或者扩容时都会调用
    /**
     * Initializes or doubles table size.  If null, allocates in
     * accord with initial capacity target held in field threshold.
     * Otherwise, because we are using power-of-two expansion, the
     * elements from each bin must either stay at same index, or move
     * with a power of two offset in the new table.
     *
     * @return the table
     */
    final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
      	// newCap 新的数量大小
      	// newThr 新的发起扩容数量
        int newCap, newThr = 0;
      	// 旧表长度大于0
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
              // 新表长度扩容为2倍
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
          // 扩容大小已经在 oldThr里面了
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
          // 大小为0，那么就需要初始化
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
      	// 定义新的数组
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
          	// 扩容，迁移数据
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    // 空值存放
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                  	// 树存放
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                   // 链表存放
                    else { // preserve order
                        // 存放在j
                        Node<K,V> loHead = null, loTail = null;
                        // 存放在j + oldCap
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                          	// 偶数数位置
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                          	// 奇数位置
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
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





## 二、对比

### 2.1 HashTable

HashTable底层也同样使用数组+链表实现，无论是key还是value都不能为null，线程安全。

HashTable实现线程安全的方式在与修改数据时会通过sychronized会锁住整个HashTable，效率低下，查看源码的时候，就可以看到有大量的方法使用了sychronized。

* HashMap和HashTable都实现了Map接口，但是HashTable的实现是基于Dictionary抽象类（定义了一些键值对数据）的。
* HashMap的迭代器Iterator是fail-fast迭代器，而HashTable的迭代器Iterator是fail-safe。fail-safe是基于容器的一个克隆，所以允许在遍历时对容器中的数据进行修改，但是fail-fast不允许，一旦发现容器中的数据倍修改了，就会抛出ConcurrentModificationException异常（ArrayList也同样会有）。
* 计算index方法不一样：hashtable：index = (hash & 0x7FFFFFFF) % tab.length； hashmap：index = hash & (tab.length – 1)。（这也导致了两个集合初始大小不一样）
* HashTable初始大小为11（底层的Hash算法不一致），扩容为原来的2倍+1，而HashMap初始大小为16，扩容为原来的2倍。

### 2.2 ConcurrentHashMap

ConcurrentHashMap底层采用分段的数组+链表，线程安全。

通过把整个Map分为N个Segment，可以提供相同的线程安全，但是效率提升了N倍，默认提升了16倍。（读操作不加锁，由于HashEntry的value变量时volatile的，也能保证读取到最新的值）。

有些方法需要跨段，比如size()和containsValue()，他们kennel需要锁整个表而不仅仅是某个段，这需要按顺序锁定所有段，操作完毕后，有按照顺序释放所有段的锁。

扩容：段内扩容（段内元素超过该段对应的Entry数组长度的75%触发扩容，不会对整个Map进行扩容），插入前检测需不需要扩容，有效避免了无法扩容。s