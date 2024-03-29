# 缓存设计

随着应用用户数据量的不断提升，服务请求量提升，最后承担压力就变成了数据库、后端服务。如果服务、数据库长期占用计算资源，很容易导致服务波动，甚至影响到业务，造成不可挽回的损失。

![image-20220823200924783](http://img.hurenjieee.com/uPic/image-20220823200924783.png)

在面临大量请求时，可以通过各种能力去降低服务和数据库压力：

1. 数据库层面：分库分表、增加数据库资源
2. 后台服务：扩充多节点、业务处理查询缓存



这里重的三个方案都是通过直接扩展的方式来缓存压力

1. 分库分表
2. 增加数据库资源
3. 扩充多节点

接下来通过目前根据公司的内部业务场景下，通过缓存提高整体的查询性能。

# 一、缓存的设计思路

数据查询逻辑：
缓存的设计无疑会数据的查询性能有很大提升。但是如果缓存设计不当，反而会造成更大的服务器压力，或是业务压力。


## 1.1 缓存设计
缓存中使用最多的中间件就是Redis，顺便复习一下Redis的使用场景。Redis中有五种数据结构：

| 类型   | 简介                 | 应用场景                   |
| ------ | -------------------- | -------------------------- |
| String | 字符串，整数，浮点数 | 字符串，整数，浮点数       |
| List   | 列表                 | 储存一些列表类型的数据结构 |
| Set    | 无序集合             | 交集，并集，差集的操作     |
| ZSet   | 有序集合             | 去重同时也可以排序         |
| Hash   | 键值对的无序列表     | 结构化的数据               |

具体到使用应用场景：

* String
  * 缓存结构体：
    * 保存用户登陆信息
  * 计数：
    * 访问量：用户的访问次数、热点文章的点赞转发数量
    * 限制流量：限制服务访问流量
    * 库存设计：使用计数也可以完成库存容量设计
* List
  * 异步队列
    * 任务放入List中，另外线程取数据并处理。
  * 抢购商品
    * 秒杀场景使用，避免超卖。
* Set
  * 用户唯一验证
    * 抽奖场景：用户仅限一次

* ZSet
  * 热门排序
    * 根据次数进行排序

* Hash
  * 缓存结构体：
    * 保存用户登陆信息，每个字段一个hash保存
    * key作为一个场景，hash作为不同情况存、读取数据



上述中，有部分场景是针对一些特殊场景使用的。这里不做过多的解释。现在考虑的是使用结构体缓存，即String、Hash两种结构。

两者区别如下：

> **String**
>
> * 功能：存储字符串
> * 存储结构：![f75507ad2aad3e71c0e1bd32758ec934.png](http://img.hurenjieee.com/uPic/f75507ad2aad3e71c0e1bd32758ec934.png)
> * 额外功能：
>   * 过期策略单独设置
>   * 自增设置、append设置等。

> **Hash**
>
> * 功能：存储Map，类似Java的HashMap
> * 存储结构：ziplist+hashtable
>   * ziplist：<img src="http://img.hurenjieee.com/uPic/graphviz-d2524c9fe90fb5d91b5875107b257e0053794a2a.png" alt="pic" style="zoom:50%;" /><img src="http://img.hurenjieee.com/uPic/graphviz-7ba8b1f3af17e2e62cdf43608914333bf14d8e91.png" alt="pic" style="zoom:50%;" />
>   * hashtable：![pic](http://img.hurenjieee.com/uPic/graphviz-68cb863d265a1cd1ccfb038d44ce6b856ebbbe3a.png)
>   * 当哈希对象可以同时满足以下两个条件时， 哈希对象使用 `ziplist` 编码：
>     1. 哈希对象保存的所有键值对的键和值的字符串长度都小于(hash-max-ziplist-value) `64` 字节；
>     2. 哈希对象保存的键值对数量小于(hash-max-ziplist-entries) `512` 个；
>   * 不能满足这两个条件的哈希对象需要使用 `hashtable` 编码。
> * 额外功能：
>   * 过期策略所有的key统一设置
>   * 有一定的聚合作用，节省rediskey，减少内存和CPU消耗。

综合以上的特性，我们有一下使用结论：

* 如果程序需要为每个数据项单独设置过期时间，那么使用字符串键。
* 如果程序需要对数据线执行诸如setrange、getrange或者append等操作，那么优先考虑使用字符串键。当然用户也可以选择把数据存储在散列中，然后将类似的操作交给客户端执行。
* 如果程序需要存储的数据项比较多，并且你希望尽可能的减少存储数据所需的内存，就应该优先考虑使用散列键。
* 如果多个数据线在逻辑上属于同一组或者同一类，那么应该优先考虑使用散列键。



> 显然，除了Hash无法完成的内容（过期操作，特殊操作）。都可以使用Hash来操作。



## 1.2 缓存并发问题

混存在设计的时候也要考虑到并发问题，避免因为缓存问题导致中间件异常、甚至服务异常。这里列举常见的三种缓存异常。



### 1.2.1 缓存穿透

> 缓存穿透是指客户端请求的数据在缓存中和数据库中都不存在，这样缓存永远不会生效，这些请求都会被打倒数据库上。
> 即这个数据根本不存在，如果黑客攻击时，启用很多个线程，一直对这个不存在的数据发送请求 ，那么请求就会一直被打到数据库上，很容易将数据库打崩。

**解决方案**：

1. 缓存空值（null）或默认值

> 分析业务请求，如果是正常业务请求时发生缓存穿透现象，可针对相应的业务数据，在数据库查询不存在时，将其缓存为空值（null）或默认值。需要注意的是，针对空值的缓存失效时间不宜过长，一般设置为5分钟之内。当数据库被写入或更新该key的新数据时，缓存必须同时被刷新，避免数据不一致。

2. 业务逻辑前置校验

> 在业务请求的入口处进行数据合法性校验，检查请求参数是否合理、是否包含非法值、是否恶意请求等，提前有效阻断非法请求。

3. 使用布隆过滤器请求白名单

> 在写入数据时，使用布隆过滤器进行标记（相当于设置白名单），业务请求发现缓存中无对应数据时，可先通过查询布隆过滤器判断数据是否在白名单内，如果不在白名单内，则直接返回空或失败。



### 1.2.2 缓存击穿

> 缓存击穿是指热点key在某个时间点过期的时候，而恰好在这个时间点对这个Key有大量的并发请求过来，从而大量的请求打到db，属于常见的“热点”问题

**解决方案**：

1. 使用互斥锁

> 只让一个线程构建缓存，其他线程等待构建缓存执行完毕，重新从缓存中获取数据。单机通过synchronized或lock来处理，分布式环境采用分布式锁。

2. 数据不过期，后台更新数据

> 热点数据不设置过期时间，后台异步更新缓存，适用于不严格要求缓存一致性的场景。



### 1.2.3 缓存雪崩

> 大量的应用请求无法在Redis缓存中进行处理，紧接着应用将大量请求发送到数据库层，导致数据库层的压力激增。
> 击穿与雪崩的区别即在于击穿是对于特定的热点数据来说，而雪崩是全部数据。
> 缓存雪崩的场景通常有两个：
>
> - 大量热点key同时过期；
> - 缓存服务故障；



**解决方案**：

1. 使用【缓存击穿】的两个方案

> 使用互斥锁
> 数据不过期，后台更新数据

2. 过期时间+随机数

> 将key的过期时间后面加上一个`随机数`（比如随机1-5分钟），让key均匀的失效。

3. 双key策略

> 主key设置过期时间，备key不设置过期时间，当主key失效时，直接返回备key值。

4. 缓存高可用

> 构建缓存高可用集群（针对缓存服务故障情况）。

5. 服务高可用

> 当缓存雪崩发生时，服务熔断、限流、降级等措施保障。



## 1.3 其他总结内容

### 1.3.1 key、hashKey的设计

前面我们讲了String和Hash的区别，在使用Hash场景中，key和hashKey的值设计就显得尤为重要。

结合应用场景来设计会更加合理，比如：

HashKey对象很大时，可以转成MD5进行存储。

hashKey数量太多时，可以对haskKey进行取mod，把一个key拆分到多个key上。



## 1.3.2



# 参考文章

1. [节约内存：Instagram的Redis实践](https://searchdatabase.techtarget.com.cn/7-20255/)
2. [Redis中字符串(string)与散列表(hash)比较](https://blog.csdn.net/magi1201/article/details/113864319)
3. [详解缓存穿透、缓存雪崩、缓存击穿](https://blog.csdn.net/qq_45637260/article/details/125866738)
