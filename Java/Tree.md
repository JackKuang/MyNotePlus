# Tree

## 一、二叉树

* 定义

> 二叉树的每个节点至多只有两颗子树（不存在度大于2的节点），二叉树的子树有左右之分，次序不能颠倒。二叉树的第i层之多有 $2^i-1$个节点；深度为k的二叉树之多有$2^k-1$个节点；对于任意一个儿二叉树T，其终端节点个数为$n_0$，度为2的结点树为$n_2$，则$n_0$=$n_2$+1。

* 示例

  ![img](http://img.hurenjieee.com/uPic/2009050402.jpg)

  ![img](http://img.hurenjieee.com/uPic/141749056837546.png)

* 应用：

  > 完全二叉树是效率很高的数据结构，堆是一种完全二叉树或者近似完全二叉树，所以效率极高。像十分常用的排序算法、Dijkstra算法、Prim算法等都需呀用堆才能优化，二叉排序树的效率也要借助平衡性来提高，二平衡性基于完全二叉树。

## 二、二叉查找树

* 定义

> 在二叉树基础上，具有额外特性：
>
> 1. 若左子树不为空，则左子树上所有节点的值均小于它的根节点的值；
> 2. 若右节点不为空，则右子树上所有节点的值均大于或者等于他的根节点的值；
> 3. 左、右子树叶分别是二叉查找树。
> 4. 没有健值相等的节点。

* 示例

  ![image-20200906224607235](http://img.hurenjieee.com/uPic/image-20200906224607235.png)

* 应用：

> 暂无

* 特性

> * 二叉查找树进行中序遍历，即可得到有序的数列。
> * 在时间复杂度上，它和二分查找一样，插入和查找的时间复杂度为O(logN)，但是最坏的情况下仍然会有O(n)的时间复杂度。
> * 二叉查找树的高度决定了二叉查找树的查找效率。

## 三、平衡二叉树

* 对于一般的二叉搜索树，其期望高度（即为一颗平衡树时）为$log_2$n，其各操作的时间复杂度O($log_2n$) 同时也由此而决定。但是，在某些极端情况下（如在插入的序列是有序的时候），二叉搜索树将退化成近似链或，此时，其操作的时间复杂度将退化成线性的，即O(n)。我们可以通过随机化建立二叉搜索树来精良避免这种情况，但是在进行了多次的操作之后，由于在删除时，我们总是选择将待删除的节点的后继代替它本身，这样就会造成总是右边的节点数目减少，以至于树向左边偏沉。这同时也会造成树的平衡性受到破坏，提高它的操作的时间复杂度。于是，就有了平衡二叉树。
* **平衡二叉树**：
  * 它是一颗空树或者它的左右两个子树的高度差的绝对值不超过1。
  * 左右两个子树都是平衡二叉树。

### 3.1 平衡查找树之AVL树

* **特点**：

> * 在AVL树中任意节点的两个儿子子树的高度最大差别为1，所以它也被称为高度平衡树。
> * N个节点的AVL树的最大深度为1.44$log_2$N。
> * 查找、插入和删除在平均和最坏的情况下都是O(logN)。
> * 增加和删除可能需要一次或多次树旋转来重新评很这个树。

* AVL树很好的解决了二叉查找树退化成链表的问题，把插入、查找、删除的时间复杂度最好和最坏情况都维持在O(logN)。但是频繁旋转会使插入和删除牺牲掉O(logN)左右的时间，不过相对二叉查找树来说，时间上稳定了很多。
* **旋转：**
  * 单旋转
  * ![img](http://img.hurenjieee.com/uPic/avltree35.jpg)
  * 双旋转
  * ![img](http://img.hurenjieee.com/uPic/2012082016534455.jpg)

### 3.2 平衡查找树之红黑树

* **特点**：

> * 它是复杂的，但是它的操作有着良好的最坏情况运行时间，并且在实践中是高效的。
> * 它可以在O(logN)时间内做查找、插入和删除。

* **性质**：

> 1. 节点是红色或黑色。
> 2. 根是黑色。
> 3. 所有的叶子都是黑色（也子是NIL节点）
> 4. 每个红色节点必须有两个黑色的子节点。（从每个叶子到根的所有路径上不能有两个连续的红色节点。）
> 5. 从任意节点到每个叶子的所有简单路径都包含相同数据的黑色节点。

* 图例：

  ![An example of a red-black tree](http://img.hurenjieee.com/uPic/450px-Red-black_tree_example.svg.png)

## 四、B树

* B树是一种树状数据结构，能够用来存储排序后的数据。这种数据结构能让查找数据、循序存取、插入数据及删除数据的操作，都在对数时间内完成。B树，概括来是一个一般化的二叉查找树查找树，可以拥有多于2个子节点。与自平衡二叉查找树不同，B树为系统最优化大块数据的读和写操作。

* 图例：

  ![img](http://img.hurenjieee.com/uPic/4.JPG)

## 五、B+树

* B+树是B树的变体，在B+树，至于到达叶子节点才会命中（B树可以在非叶子节点中命中）。

* 所有的叶子节点增加一个链指针。

* 图例：

  ![img](http://img.hurenjieee.com/uPic/5.JPG)

## 六、B*树

* B+树的遍体变体，在B+树的非根和非叶子节点再增加指向兄弟的指针。

* 图例：

  ![img](http://img.hurenjieee.com/uPic/6.JPG)

## 七、总结

| 树类型     | 特点                                                         | 应用场景              |
| ---------- | ------------------------------------------------------------ | --------------------- |
| 二叉树     | 1. 效率高                                                    |                       |
| 二叉查找树 | 1. 左边小，右边大                                            | 1. 排序算法，中序遍历 |
| 平衡二叉树 | 1. 自平衡<br />2. 查询速度快                                 | 1. HashMap红黑树      |
| B树        | 1. 只出现一次<br />2. 非叶子节点可以命中，访问更快           | 1. 数据库、文件系统   |
| B+树       | 1. 非叶子节点是索引，叶子节点是存储<br />2. 叶子节点有一个链指针<br />3. 查询稳定<br />4. 遍历速度快 | 1. Mysql索引          |
| B*树       | 1. 非叶子节点也存在链指针                                    |                       |

