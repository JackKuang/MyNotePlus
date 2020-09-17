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
		// 初始化大小 16
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;

		// 负载因子 0.75
    static final int MAXIMUM_CAPACITY = 1 << 30;

		// 初始化的默认数组
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

		// HashMap中元素的数量
    static final int TREEIFY_THRESHOLD = 8;

		// 判断是否需要调整HashMap的容量
    static final int MIN_TREEIFY_CAPACITY = 64;
```

### 1.4 数据存储结构

HashMap采用Entry数组来存储key-value对，每一个键值对组成了一个Entry实体，Entry类实际上是一个单向的链表结构，它具有Next指针，可以连接下一个Entry实体，以此来解决Hash冲突的问题。

* 数组存储区间是连续的，占用内存严重，故空间复杂度很大。但是数组的二分查找复杂度小，为O(1)；数组的特点是：寻址容易，插入和删除困难。
* 链表存储区间离散，占用内存比较宽松，故空间复杂度很小，但是时间复杂度很大，达到O(N)；链表的特特点，寻址困难，插入和删除容易。

![img](http://img.hurenjieee.com/uPic/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMzQ1Nzcz,size_16,color_FFFFFF,t_70-20200917223145829.png)

从上图中，我们可以发现数据结构是有数组+链表组成，一个长度为16的数组中，每个元素存储的是一个链表的头节点。那么这些元素是按照什么样的规则存储到数组中的，一般情况是通过hash(key.hashCode)%len获得，也就是元素的key的哈希值对应数组的长度取模得到。

HashMap里面实现了一个静态内部类Entry，其重要的属性有hash、key、value、next。

```java
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
// jdk1.8
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
          	// 如果是树，放入树中
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            // 
          	else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

