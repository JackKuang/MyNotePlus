# Spark

## 一、Spark简介

* Apache Spark是针对大规模数据处理的统一分析引擎



## 二、Spark四大特性

### 2.1 速度快

* 运行速度提高了100倍

  Apache Spark使用最先进的DAG调度程序，查询优化程序和物理执行引擎，实现批量和流式数据的高性能。

* Spark比MapReduce快的2个原因

  * 基于内存

    > mapreduce任务后期再计算的时候，每一个job的输出结果会落地到磁盘，后续有其他的job需要依赖于前面job的输出结果，这个时候就需要进行大量的磁盘io操作。性能就比较低。
    >
    > spark任务后期再计算的时候，job的输出结果可以保存在内存中，后续有其他的job需要依赖于前面job的输出结果，这个时候就直接从内存中获取得到，避免了磁盘io操作，性能比较高

  * 进程与线程

    > （1）mapreduce任务以进程的方式运行在yarn集群中，比如程序中有100个MapTask，一个task就需要一个进程，这些task要运行就需要开启100个进程。
    >
    > （2）spark任务以线程的方式运行在进程中，比如程序中有100个MapTask，后期一个task就对应一个线程，这里就不在是进程，这些task需要运行，这里可以极端一点：
    > 只需要开启1个进程，在这个进程中启动100个线程就可以了。
    > 进程中可以启动很多个线程，而开启一个进程与开启一个线程需要的时间和调度代价是不一样。 开启一个进程需要的时间远远大于开启一个线程。

### 2.2 易用性

* 可以快速去编写spark程序通过 java/scala/python/R/SQL等不同语言

### 2.3 通用型

* spark框架不在是一个简单的框架，可以把spark理解成一个**生态系统**，它内部是包含了很多模块，基于不同的应用场景可以选择对应的模块去使用
  - sparksql
    - 通过sql去开发spark程序做一些离线分析
  - sparkStreaming
    - 主要是用来解决公司有实时计算的这种场景
  - Mlib
    - 它封装了一些机器学习的算法库
  - Graphx
    - 图计算

### 2.4 兼容性

* spark程序就是一个计算逻辑程序，这个任务要运行就需要计算资源（内存、cpu、磁盘），哪里可以给当前这个任务提供计算资源，就可以把spark程序提交到哪里去运行
  - standAlone
    - 它是spark自带的集群模式，整个任务的资源分配由spark集群的老大Master负责
  - yarn
    - 可以把spark程序提交到yarn中运行，整个任务的资源分配由yarn中的老大ResourceManager负责
  - mesos
    - 它也是apache开源的一个类似于yarn的资源调度平台。

## 三、Spark集群架构

![image-20200921210504996](http://img.hurenjieee.com/uPic/image-20200921210504996.png)

* **Driver**

  > * 它会执行客户端写好的main方法，它会构建一个名为SparkContext对象
  > * 该对象是所有Spark程序的执行入口

* **Application**

  > * Spark的应用程序，它包含了客户端的代码和任务运行的资源

* **ClusterManager**

  > * 为程序提供计算资源的外部服务
  >   * StandAlone
  >   * yarn
  >   * mesos

* **Master**

  > Spark集群的老大，负责任务资源的分配

* **Worker**

  > Spark集群的小弟，负责任务计算的执行

* **Executor**

  > 一个进程，它会在work节点启动该进程（计算资源）

* **Task**

  > Spark任务是以task线程的方式运行在worker对应的executor进行中的

## 四、Spark集群安装与部署

1. 下载，解压

2. 修改配置文件 spark-env.sh

   ```bash
   # mv spark-env.sh.template spark-env.sh
   # 配置java的环境变量
   export JAVA_HOME=/opt/bigdata/jdk
   # 配置zk相关信息
   # 集群配置，依赖于Zookeeper
   export SPARK_DAEMON_JAVA_OPTS="-Dspark.deploy.recoveryMode=ZOOKEEPER  -Dspark.deploy.zookeeper.url=node1:2181,node2:2181,node3:2181  -Dspark.deploy.zookeeper.dir=/spark"
   ```

3. 修改配置文件 salves

   ```bash
   # mv slaves.template salves
   # 指定spark集群的worker节点
   node2
   node3
   ```

4. 启动集群

   ```sh
   $SPARK_HOME/sbin/start-all.sh
   # 在当前节点启动一个Master+Salves
   ```

5. 单独启动Master

   ```sh
   $SPARK_HOME/sbin/start-master.sh/
   ```
   
6. 停止集群

   ```sh
   $SPARK_HOME/sbin/stop-all.sh
   $SPARK_HOME/sbin/stop-master.sh
   ```

   


> 1. 如何恢复到上一次活着Master挂掉之前的状态？
>
>    > 在高可用模式下，整个Spark集群就有很多个Master，其中一个master被zk选举成活的Master，其他的多个Master都处于StandBy，同时把整个Spark集群的元数据信息通过zk中节点进行保存。
>    >
>    > 如果活着的Master挂掉。首先zk会感知到活着的Master挂掉，下面开始在多个处于standby中的Master进行选举，再次产生一个活着的Master，这个活着的Master会读取保存在zk节点中的Spark集群元数据信息，恢复到上一次Master的状态。
>    >
>    > 整个过程在恢复的时候经历了很多个不同的阶段，每个阶段都需要一定时间，最终恢复到上一个活着的Master状态，整个恢复过程一般需要1-2分钟。
>
> 2. 在Master的恢复阶段对任务的影响？
>
>    > * 对已经运行的任务是没有任何影响
>    >
>    >   由于该任务正在运行，说明它已经拿了计算资源，这个时候已经不需要Master。
>    >
>    > * 对即将要提交的任务有影响
>    >
>    >   由于该任务需要计算资源，这个时候会找活着的Master去申请计算资源，由于没有个活着的master，该任务是获取不到计算资源，也就是任务无法运行。

## 五、Spark任务提交

### 5.1 普通模式提交

```sh
bin/spark-submit \
# 指定运行主类
--class org.apache.spark.examples.SparkPi \
# 指定Master地址
--master spark://node1:7077 \
# 指定每个executor内存
--executor-memory 1G \
# 指定运行任务需要的CPU数量
--total-executor-cores 2 \
# 运行jar包
examples/jars/spark-examples_2.11-2.3.3.jar \
# main 方法参数
10
```

### 5.2 高可用提交

```sh
bin/spark-submit \
# 指定运行主类
--class org.apache.spark.examples.SparkPi \
# 指定Master地址,程序会轮训找出活着的节点
--master park://node1:7077,node2:7077,node3:7077 \
# 指定每个executor内存
--executor-memory 1G \
# 指定运行任务需要的CPU数量
--total-executor-cores 2 \
# 运行jar包
examples/jars/spark-examples_2.11-2.3.3.jar \
# main 方法参数
10
```

## 六、Spark Shell

### 6.1 本地运行+本地统计

```scala
// shell
// spark-shell master local[2]

sc.textFile("file:///home/hadoop/words.txt").flatMap(x=>x.split(" ")).map(x=>(x,1)).reduceByKey((x,y)=>x+y).collect

sc.textFile("file:///home/hadoop/words.txt").flatMap(_.split(" ")).map((_,1)).reduceByKey(_+_).collect
```

### 6.2 本地运行+HDFS统计

```sh
# 指定HDFS配置
# vim spark-env.sh（所有的节点）
export HADOOP_CONF_DIR=/opt/bigdata/hadoop/etc/hadoop
```

```scala
// shell
// spark-shell master local[2]

// 指定本机hdfs
sc.textFile("/words.txt").flatMap(_.split(" ")).map((_,1)).reduceByKey(_+_).collect

// 指定某个HDFS
sc.textFile("hdfs://node1:9000/words.txt").flatMap(_.split(" ")).map((_,1)).reduceByKey(_+_).collect
```

### 6.3 集群运行+HDFS统计

```scala
// shell
// spark-shell --master spark://node1:7077 --executor-memory 1g --total-executor-cores 4

sc.textFile("hdfs://node1:9000/words.txt").flatMap(_.split(" ")).map((_,1)).reduceByKey(_+_).collect

// 实现读取hdfs上文件之后，需要把计算的结果保存到hdfs上
sc.textFile("/words.txt").flatMap(_.split(" ")).map((_,1)).reduceByKey(_+_).saveAsTextFile("/out")
```

