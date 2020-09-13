# Flink Exactly Once实现

## 一、Flink 基础概念

### 1.1 Flink 状态

有状态函数和运算符在各个元素/事件的处理中存储数据。

例如：

* 当应用程序搜索某些事件模式时，状态将会存储到目前为止遇到的时间序列。
* 在每分钟/小时/天聚合事件时，状态保存待处理的聚合。
* 当在数据点流上训练机器学习模型时，状态保持模型参数的当前版本。
* 当需要管理实例数据时，状态允许有效访问过去发生的事件。

### 1.2 什么是状态

* 无状态计算的例子

  > 比如：我们值进行一个字符串拼接，输入a，输出a_666，输入b，输出b_666。输出的结构跟之前的状态没关系，符合幂等性。
  >
  > 幂等性：就是用户对于同一个操作发起的一次请求或者多次请求的结果是一致的，不会因为多次点击而产生副作用。

* 有状态计算的例子

  > 计算pv、uv
  >
  > 数据的结果跟之前的状态有关系，不符合幂等性，访问多次，pv会增加。

### 1.3 Flink CheckPoint过程

我们从Kafka中读取日志记录，从日志中解析出app_id，然后将统计的结果放入内存中的一个Map集合里，app_id作为key，对应的PV作为value，每次只需要将相应的app_id的pv值+1后put到Map中即可。

![640?wx_fmt=png](http://img.hurenjieee.com/uPic/640.png)

1. Flink的Source task记录了当前消费到的kafka topic的所有partation的offet。

   > 例：(0,1000)
   >
   > 表示Source消费了0号partation中offset位1000的数据

2. Flink 的PV task记录了当前计算的各app的PV值。

   > 例：(App01,5000),(App02,4000)
   >
   > 表示App01当前PV值为5000，App02当前PV值为4000
   >
   > 每来一条数据，只要确定响应的app_id，对应的value值+1后put到map中。

3. 此时的CheckPoint记录哪些内容：

   > offset：(0,1000)
   >
   > pv：(App01,5000),(App02,4000)
   >
   > ​		记录的当前checkPoint消费的offset信息以及各app的pv统计信息，记录一下发生CheckPoint当前的状态信息并将该状态信息保存到相应的状态后端。
   >
   > chk-100
   >
   > ​		表示状态信息为第100次CheckPoint。

4. 任务挂了

   > 此时任务的统计信息
   >
   > offset：(0,1100)
   >
   > pv：(App01,5010),(App02,4020)

5. 恢复

   > 从状态中进行维护，也就是从ch-100的地方开始恢复。
   >
   > 包括offset、pv数据。

## 二、Flink Exactly-Once保证

### 2.1 Barrier机制

前面简单理解了CheckPoint机制，有一个问题，所有的任务都在一直运行着，那么如果保证数据的消费准确，如何保证pv统计数据和offset消费位置一致。虽然做了checkPoint，但是source Task做checkPoint和pv task中间的那段数据如何处理。

这里就要引入Flink一个Barrier机制。

![640?wx_fmt=png](http://img.hurenjieee.com/uPic/640-20200913203549506.png)

1. barrier从Source Task处生成，一直流到Sink Task，期间所有的Task只要碰到barrier，就会触发自身进行快照。
2. CheckPoint barrier n-1处做的快照就是指job从开始处理到barrier n-1所有的状态数据。
3. CheckPoint barrier n处做的快照就是指job从开始处理到barrier n所有的状态数据。
4. 对应的pv案例就是，SourceTask接收到JobManager的编号为chk-100的CheckPoint出发请求后，发现自己恰好接收到了kafka offset(0,1000)处的数据，所以会往offset(0,1000)数据之后offset(0,1001)数据之间安插一个barrier，然后自己开始做快照，也就是将offset(0,1000)保存到状态后端chk-100中。然后barrier接着往下游发送，当统计pv的task接收到barrier后，也会暂停处理数据，将自己内存保存的pv信息(App01,5000),(App02,4000)保存到状态后端chk-100中。
5. 统计pv的task接收到barrier，就意味着barrier之前的数据都处理了，所以说，不会出现丢数据的情况。

总结点：

* barrier的作用就是为了把数据区分开，CheckPoint过程中有一个同步做快照的环节，这个时候不能处理barrier后的数据。因为如果做快照的同时也处理数据，那么数据就会修改快照内容，所以先暂停处理数据，把内存中快照保存好后，在处理数据。

### 2.2 流式计算中状态交互

* 流式状态的状态交互

  ![640?wx_fmt=png](http://img.hurenjieee.com/uPic/640-20200913205545435.png)

* 简易场景精确一次

  周期性地对消费offset和统计的统计信息进行快照

  ![640?wx_fmt=png](http://img.hurenjieee.com/uPic/640-20200913205657484.png)

  

### 2.3 多并行度、多Operateor情况下，CheckPoint过程

* 面临的问题

  * 如何确保状态拥有精确一次的容错保证？
  * 如何在分布式场景下替多个拥有本地状态的算子产生一个全局一致的快照？
  * 如何在不中断运算的情况下保产生快照？

* 多并行度下CheckPoint快照

  ![640?wx_fmt=png](http://img.hurenjieee.com/uPic/640-20200913210107118.png)

* 多并行度下CheckPoint恢复

  ![640?wx_fmt=png](http://img.hurenjieee.com/uPic/640-20200913210139396.png)

* 多并行度下CheckPoint过程

  1. 准备CheckPoint

     ![640?wx_fmt=png](https://ss.csdn.net/p?https://mmbiz.qpic.cn/mmbiz_png/8AsYBicEePu40409y3xDnbBjf83TFXbDtvQQu7ftaEDodSjaNDLJRr30P2vYxqbkH1sWhJaUF7IdV3D4eDR3Mtg/640?wx_fmt=png)

   2. Source CheckPoint

      ![640?wx_fmt=png](http://img.hurenjieee.com/uPic/640-20200913210632676.png)

  	3.  继续

       ![640?wx_fmt=png](http://img.hurenjieee.com/uPic/640-20200913210655019.png)

  	4.  Operator#1 CheckPoint

       ![640?wx_fmt=png](http://img.hurenjieee.com/uPic/640-20200913210716974.png)

  	5.  Operator#2 CheckPoint

       ![640?wx_fmt=png](http://img.hurenjieee.com/uPic/640-20200913210751639.png)

  6. CheckPoint完成

CheckPoint执行过程：

1. JobManager端的CheckPointCoordinator向所有SourceTask发送CheckPointTrigger，Source Task会在数据流中安插CheckPoint barrier。
2. 当task收到所有的barrier后，向自己的下游继续传递barrier，然后自身执行快照，并将自己的状态异步写入到持久化存储中。
3. 增量CheckPoint只是把最新的一部分更新到外部存储中。
4. 为了下游尽快做CheckPoint，所以会先发送barrier到下游，自身在同步进行快照。
5. 当task完成备份之后，会将备份数据的地址（state handle）通知给JobManager的CheckPointCoordinator。
6. 如果CheckPoint的持续时长超过了CheckPoint设定的超时时间，CheckPointCoordinator还没有收集完锁哟的State handle，CheckPoint Coordinator就会认为本次CheckPoint失败，会把这次CheckPoint产生的所有状态数据全部删除。
7. 最后CheckPointCoordinator会把这个那个StateHandle封装成completed CheckPoint Meta，写入到HDFS。

### 2.4 Barrier对齐

![640?wx_fmt=png](http://img.hurenjieee.com/uPic/640-20200913211810508.png)

1. 一旦Operator从输入流接收到CheckPointBarrier n，它就不能处理来自该流的任何数据记录，直到它从其他所有的输入接收到barriner n为止。否则，它会混合数据快照n的记录和属于快照n+1的记录。

2. 接收到barrier n的流暂时被搁置。从这些流接收到记录不回被处理，而是放入输入缓冲区。

   上图第2个图中，虽然数字流对应的barrier已经到达，但是barrier之后到1、2、3这些数据只能放到buffer中，等待字母流的barrier到达。

3. 一旦最后所有的输入流都接收到barrier n，Operator就会把缓冲区pending的输出数据发出去，然后把CheckPoint barrier n接着往下游发送。

4. 之后，Operator将继续处理来自所有输入流的记录，在处理来自流的记录之前先处理来自输入缓冲区的记录。



* Barrier不对齐结果

  > Exactly Once时必须保证barrier对齐，入股barrier不对齐就成了At Least Once。
  >
  > CheckPoint的目的时为了保存快照，如果不对齐，那么在chk-100快照之前，已经处理了一些chk-100对对应的offset之后的数据，当程序重chk-100恢复任务，chk-100对应的offset之后的数据还会被处理，所以出现了重复消费。



## 参考

https://blog.csdn.net/weixin_44904816/article/details/102675286

