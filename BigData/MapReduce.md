# MapReduce

## 一、介绍

* MapReduc是采用**分而治之**的思想设计出来的分布式计算框架

* 单一复杂的任务，单台服务器无法计算，可以将大任务分别拆分成一个个消任务，小任务分别在不同的服务器上执行，最后再汇总每个小任务的结果。

* MapReduce分为两个阶段：

  * Map（切分成一个个小的任务）
  * Reduce（汇总任务的结果）

  ![mapreduce1](http://img.hurenjieee.com/uPic/mapreduce1.png)

  ![mapreduce2](http://img.hurenjieee.com/uPic/mapreduce2.png)

## 二、Map

* map()函数以kv作为输入，产生一系列新的kv对作为中间输出写入到HDFS。

## 三、Reduce

* reduce()函数通过网络mapreduce的输入(kv)组为输入，产生另外一个kv作为最终结果写入到HDFS。

## 四、Combiner

* Map端本地聚合，相同与执行了一次Reduce操作。
* Combiner再Map之后执行。
* 无论运行多少次Combiner操作，都不应该影响最终的结果。

## 五、Shuffle

过程示意图：

![suffle](http://img.hurenjieee.com/uPic/suffle.png)

![suffle](http://img.hurenjieee.com/uPic/suffle_2.png)

![suffle](http://img.hurenjieee.com/uPic/suffle_3.png)

![suffleExample](http://img.hurenjieee.com/uPic/suffle_example.png)



### 5.1 Shuffle——Map端

1. 分区Partition

   > 自定义Partitioner进行分区操作
   >
   > 默认为HashParititoner

2. 写入环形缓冲区

   > 每个Map任务都会分配一个100M的环形内存缓存区，用于存储map任务输出的键值对以及对应的partition

3. 执行溢出写spill

   > 环形内存缓冲区达到阈值之后（80%），会锁定这部分内存，并再每个分区中对其中的键值进行排序，根据partition和key两个关键字排序。
   >
   > 排序结果为缓冲区的数据按照paritition为单位聚集再一起，同一个partition内的数据按照key有序。
   >
   > 排序完成后会创建一个溢出写文件，然后开启一个后台线程把这部分数据以一个临时文件的方式溢出写到磁盘中。
   >
   > 合并Combiner：
   >
   > * 当作业设置Combiner类后，缓存溢出线程将缓存存放到磁盘，就会调用。
   > * 缓存溢出的数量超过mapreduce.map.combine.minspills(默认为3)是，在缓存溢出文件合并的时候就会调用
   >
   > 合并（Combiner）和归并（Merge）的区别：
   >
   > * 两个键值对<"a",1>和<"a",1>，Combiner会得到<"a",2>，Merge会得到<"a",<1,1>>

4. 归并Merge

   > 当以个Map task处理的数据很大，以至于超过缓冲区内存时，就会产生多个spill文件。此时就需要对同一个map任务产生的多个spill文件进行归并生成最终的一个**已分区且已排序**的大文件。
   >
   > 溢出写文件归并完毕后，Map将删除所有的临时溢出写文件，并告知NodeManager任务已完成，只要其中一个MapTask王城，ReduceTask就开始复制它 的输出。
   >
   > 压缩：写磁盘时压缩map端的输出，因为这样会让写磁盘的速度更快，节约磁盘空间，并减少传给reducer的数量

### 5.2 Shuffle——Reduce端

1. 复制Copy

   > Reduce进程启动一些数据copy线程，通过HTTP方式请求MapTask所在的NodeManager以获取输出文件。
   >
   > 在Map端Partition的时候，相当于制定了Reducer要处理的数据。
   >
   > 任务调度之间，有一个ApplicationMaster，负责联系Map和Reduce的对应关系。

2. 归并Merge

   > Copy的数据先放到内存缓冲区，内存不够用写入到磁盘。
   >
   > 拷贝全部完成之后，回在Reduce上生成多个文件，这个时候执行归并，也就时**磁盘到磁盘Merge**，因为Map那边过来的时候已经有序，这里排序也只是一次归并排序

3. 执行Reduce

   > 经过复制和排序后，就会针对已根据键排好序的key构造对应的Value迭代器。

## 六、示例：

[Example](./MapReduceExample.md)