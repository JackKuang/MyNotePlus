

# Flink

[Flink DataStream.md](./FlinkDataStream.md)

[Flink DataSet.md](./FlinkDataSet.md)

[FlinkState案例.md](./FlinkState案例.md)

[Flink Window/WaterMark](./FlinkWindow+WaterMark.md)

[Flink Window](./FlinkWindow.md)

## 一、Flink简介

**Apache Flink® — Stateful Computations over Data Streams**

![flink](http://img.hurenjieee.com/uPic/flink.png)

Apache Flink 是一个框架和分布式处理引擎，用于在*无边界和有边界*数据流上进行有状态的计算。Flink 能在所有常见集群环境中运行，并能以内存速度和任意规模进行计算。

### 1.1 处理无解数据和有界数据

任何类型的数据都可以形成一种事件流。信用卡交易、传感器测量、机器日志、网站或移动应用程序上的用户交互记录，所有这些数据都形成一种流。

数据可以被作为 *无界* 或者 *有界* 流来处理。

1. **无界流** 有定义流的开始，但没有定义流的结束。它们会无休止地产生数据。无界流的数据必须持续处理，即数据被摄取后需要立刻处理。我们不能等到所有数据都到达再处理，因为输入是无限的，在任何时候输入都不会完成。处理无界数据通常要求以特定顺序摄取事件，例如事件发生的顺序，以便能够推断结果的完整性。

2. **有界流** 有定义流的开始，也有定义流的结束。有界流可以在摄取所有数据后再进行计算。有界流所有数据可以被排序，所以并不需要有序摄取。有界流处理通常被称为批处理

   ![bounded-unbounded](http://img.hurenjieee.com/uPic/bounded-unbounded.png)

**Apache Flink 擅长处理无界和有界数据集** 精确的时间控制和状态化使得 Flink 的运行时(runtime)能够运行任何处理无界流的应用。有界流则由一些专为固定大小数据集特殊设计的算法和数据结构进行内部处理，产生了出色的性能。

### 1.2 部署应用到任意地方

Apache Flink 是一个分布式系统，它需要计算资源来执行应用程序。Flink 集成了所有常见的集群资源管理器，例如 [Hadoop YARN](https://hadoop.apache.org/docs/stable/hadoop-yarn/hadoop-yarn-site/YARN.html)、 [Apache Mesos](https://mesos.apache.org/) 和 [Kubernetes](https://kubernetes.io/)，但同时也可以作为独立集群运行。
Flink 被设计为能够很好地工作在上述每个资源管理器中，这是通过资源管理器特定(resource-manager-specific)的部署模式实现的。Flink 可以采用与当前资源管理器相适应的方式进行交互。
部署 Flink 应用程序时，Flink 会根据应用程序配置的并行性自动标识所需的资源，并从资源管理器请求这些资源。在发生故障的情况下，Flink 通过请求新资源来替换发生故障的容器。提交或控制应用程序的所有通信都是通过 REST 调用进行的，这可以简化 Flink 与各种环境中的集成

### 1.3 运行任意规模应用

Flink 旨在任意规模上运行有状态流式应用。因此，应用程序被并行化为可能数千个任务，这些任务分布在集群中并发执行。所以应用程序能够充分利用无尽的 CPU、内存、磁盘和网络 IO。而且 Flink 很容易维护非常大的应用程序状态。其异步和增量的检查点算法对处理延迟产生最小的影响，同时保证精确一次状态的一致性。
Flink 用户报告了其生产环境中一些令人印象深刻的扩展性数字：

* 每天处理数万亿的事件
* 可以维护几TB大小的状态
* 可以部署上千个节点的集群（可伸缩）

### 1.4 利用内存性能

有状态的 Flink 程序针对本地状态访问进行了优化。任务的状态始终保留在内存中，如果状态大小超过可用内存，则会保存在能高效访问的磁盘数据结构中。任务通过访问本地（通常在内存中）状态来进行所有的计算，从而产生非常低的处理延迟。Flink 通过定期和异步地对本地状态进行持久化存储来保证故障场景下精确一次的状态一致性。

![local-state](http://img.hurenjieee.com/uPic/local-state-1573651201478.png)

## 二、Flink Stream

[Flink Stream.md](./FlinkSteam,md)

## 三、Flink Set

[Flink Set.md](./FlinkSet.md)

## 四、Flink 广播变量

* 广播变量允许编程人员在每台机器上保持1个只读的缓存变量，而不是传送变量的副本给tasks
* 广播变量创建后，它可以运行在集群中的任何function上，而不需要多次传递给集群节点。另外需要记住，不应该修改广播变量，这样才能确保每个节点获取到的值都是一致的。
* 一句话解释，可以理解为是一个公共的共享变量，我们可以把一个dataset 数据集广播出去，然后不同的task在节点上都能够获取到，这个数据在每个节点上只会存在一份。如果不使用broadcast，则在每个节点中的每个task中都需要拷贝一份dataset数据集，比较浪费内存(也就是一个节点中可能会存在多份dataset数据)。
* 用法
  1. 初始化数据
     DataSet<Integer> toBroadcast = env.fromElements(1, 2, 3)
  2. 广播数据
     .withBroadcastSet(toBroadcast, "broadcastSetName");
  3. 获取数据
     Collection<Integer> broadcastSet = getRuntimeContext().getBroadcastVariable("broadcastSetName");
* 注意：
  1. 广播出去的变量存在于每个节点的内存中，所以这个数据集不能太大。因为广播出去的数据，会常驻内存，除非程序执行结束。
  2. 广播变量在初始化广播出去以后不支持修改，这样才能保证每个节点的数据都是一致的。

```java

/**
 * broadcast广播变量
 * 需求：
 *  flink会从数据源中获取到用户的姓名
 *  最终需要把用户的姓名和年龄信息打印出来
 *  分析：
 *  所以就需要在中间的map处理的时候获取用户的年龄信息
 *  建议吧用户的关系数据集使用广播变量进行处理
 *
 */
public class BroadCastDemo {
    public static void main(String[] args) throws Exception{

        //获取运行环境
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        //1：准备需要广播的数据
        ArrayList<Tuple2<String, Integer>> broadData = new ArrayList<>();
        broadData.add(new Tuple2<>("zs",18));
        broadData.add(new Tuple2<>("ls",20));
        broadData.add(new Tuple2<>("ww",17));
        DataSet<Tuple2<String, Integer>> tupleData = env.fromCollection(broadData);

        //1.1:处理需要广播的数据,把数据集转换成map类型，map中的key就是用户姓名，value就是用户年龄
        DataSet<HashMap<String, Integer>> toBroadcast = tupleData.map(new MapFunction<Tuple2<String, Integer>, HashMap<String, Integer>>() {
            @Override
            public HashMap<String, Integer> map(Tuple2<String, Integer> value) throws Exception {
                HashMap<String, Integer> res = new HashMap<>();
                res.put(value.f0, value.f1);
                return res;
            }
        });
        //源数据
        DataSource<String> data = env.fromElements("zs", "ls", "ww");
        //注意：在这里需要使用到RichMapFunction获取广播变量
        DataSet<String> result = data.map(new RichMapFunction<String, String>() {
            List<HashMap<String, Integer>> broadCastMap = new ArrayList<HashMap<String, Integer>>();
            HashMap<String, Integer> allMap = new HashMap<String, Integer>();

            /**
             * 这个方法只会执行一次
             * 可以在这里实现一些初始化的功能
             * 所以，就可以在open方法中获取广播变量数据
             */
            @Override
            public void open(Configuration parameters) throws Exception {
                super.open(parameters);
                //3:获取广播数据
                this.broadCastMap = getRuntimeContext().getBroadcastVariable("broadCastMapName");
                for (HashMap map : broadCastMap) {
                    allMap.putAll(map);
                }
            }
            @Override
            public String map(String value) throws Exception {
                Integer age = allMap.get(value);
                return value + "," + age;
            }
        }).withBroadcastSet(toBroadcast, "broadCastMapName");//2：执行广播数据的操作
        result.print();
    }

}
```

## 五、Flink Counter（计数器）

* Accumulator即累加器，与Mapreduce counter的应用场景差不多，都能很好地观察task在运行期间的数据变化
* 可以在Flink job任务中的算子函数中操作累加器，但是只能在任务执行结束之后才能获得累加器的最终结果。
* Counter是一个具体的累加器(Accumulator)实现：IntCounter, LongCounter 和 DoubleCounter
* 用法：
  1. 创建累加器
     private IntCounter numLines = new IntCounter(); 
  2. 注册累加器
     getRuntimeContext().addAccumulator("num-lines", this.numLines);
  3. 使用累加器
     this.numLines.add(1); 
  4. 获取累加器的结果
     myJobExecutionResult.getAccumulatorResult("num-lines")

```java
/**
 * 计数器
 */
public class CounterDemo {
    public static void main(String[] args) throws Exception{
        //获取运行环境
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        DataSource<String> data = env.fromElements("a", "b", "c", "d");
        DataSet<String> result = data.map(new RichMapFunction<String, String>() {

            //1:创建累加器
            private IntCounter numLines = new IntCounter();
            @Override
            public void open(Configuration parameters) throws Exception {
                super.open(parameters);
                //2:注册累加器
                getRuntimeContext().addAccumulator("num-lines",this.numLines);

            }
            //int sum = 0;
            @Override
            public String map(String value) throws Exception {
                //如果并行度为1，使用普通的累加求和即可，但是设置多个并行度，则普通的累加求和结果就不准了
                //sum++;
                //System.out.println("sum："+sum);
                this.numLines.add(1);
                return value;
            }
        }).setParallelism(8);
        //如果要获取counter的值，只能是任务
        //result.print();
        result.writeAsText("d:\\data\\mycounter");
        JobExecutionResult jobResult = env.execute("counter");
        //3：获取累加器
        int num = jobResult.getAccumulatorResult("num-lines");
        System.out.println("num:"+num);

    }
}
```

## 六、Flink State

### 6.1 State概述

**Apache Flink® — Stateful Computations over Data Streams**

```java
/**
 * 单词计数
 */
public class WordCount {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        DataStreamSource<String> data = env.socketTextStream("localhost", 8888);
        SingleOutputStreamOperator<Tuple2<String, Integer>> result = data.flatMap(new FlatMapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public void flatMap(String line, Collector<Tuple2<String, Integer>> collector) throws Exception {
                String[] fields = line.split(",");
                for (String word : fields) {
                    collector.collect(new Tuple2<>(word, 1));
                }
            }
        }).keyBy("0")
                .sum(1);

        result.print();

        env.execute("WordCount");
    }
}
```

运行程序，我们会发现单词出现的次数时有累计的效果的。如果没有状态的管理，是不会有累积的效果的（这里像Spark Streaming[checkpoint/updateStateByKey/mapWithState]）。所以Flink里面，还有State的概念。

![State](http://img.hurenjieee.com/uPic/State.png)

**State**：一般之一个具体的task/operate的状态。State可以被记录，在失败的情况下数据还可以恢复，Flink中有两种基本类型的State：Keyed State，Operator State，他们两个都还可以一两种形式存在：原始状态(raw state)和托管状态(managed state)

**托管状态**：由Flink框架管理的状态，我们通常使用的就是这种。

**原始状态**：由用户自行管理状态具体的数据结构，框架在做checkpoint的时候，使用byte[]来读写状态内容，对其内部数据结构一无所知。通常在DataStream上的状态推荐使用托管的状态，当实现一个用户自定义的operator时，**会使用到原始状态**。但是我们工作中一般不常用，所以我们不考虑他。

### 6.2 State类型

#### 6.2.1 Operator State

没发生Suffle

![Operator State](http://img.hurenjieee.com/uPic/Operator%20State.png)

* Operaor States是Task级別的State，也就是说，每个Task都对应着一个State。
* Kafka Connector source中的每个分区（task）都需要记录消费的topic的partition和offset等信息。
* operator state 只有一种托管状态：`ValueState`

#### 6.2.2 Keyed State

发生Suffle

![Keyed State](http://img.hurenjieee.com/uPic/Keyed%20State.png)

1. keyed state 记录的是每个key的状态
2. Keyed state托管状态有六种类型：
   1. ValueState
   2. ListState
   3. MapState
   4. ReducingState
   5. AggregatingState
   6. FoldingState

### 6.3 State案例

[FlinkState案例.md](./FlinkState.md)

## 七、Flink State Backend

### 7.1 概述

Flink支持的StateBackend:

- *MemoryStateBackend*
- *FsStateBackend*
- *RocksDBStateBackend*

### 7.2 MemoryStateBackend

![MemoryStateBackend](http://img.hurenjieee.com/uPic/MemoryStateBackend-1574599431047.png)

* 默认情况下，状态信息是存储在 TaskManager 的堆内存中的，c heckpoint 的时候将状态保存到 JobManager 的堆内存中。
* 缺点：
  * 只能保存数据量小的状态
  * 状态数据有可能会丢失
* 优点：
  * 开发测试很方便

### 7.3 FSStateBackend

![FSStateBackend](http://img.hurenjieee.com/uPic/FSStateBackend-1574599431048.png)

* 状态信息存储在 TaskManager 的堆内存中的，checkpoint 的时候将状态保存到指定的文件中 (HDFS 等文件系统)
* 缺点：
  * 状态大小受TaskManager内存限制(默认支持5M)
* 优点：
  * 状态访问速度很快
  * 状态信息不会丢失
  * 用于： 生产，也可存储状态数据量大的情况

### 7.4 RocksDBStateBackend

![RocksDBStateBackend](http://img.hurenjieee.com/uPic/RocksDBStateBackend-1574599431048.png)

*  RocksDB 数据库 (key-value 的数据存储服务)， 最终保存在本地文件中
  checkpoint 的时候将状态保存到指定的文件中 (HDFS 等文件系统)
* 缺点：
  * 状态访问速度有所下降
* 优点：
  * 可以存储超大量的状态信息
  * 状态信息不会丢失
  * 用于： 生产，可以存储超大量的状态信息

### 7.5 StateBackend配置方式

（1）单任务调整

```java
修改当前任务代码
env.setStateBackend(new FsStateBackend("hdfs://namenode:9000/flink/checkpoints"));
或者new MemoryStateBackend()
或者new RocksDBStateBackend(filebackend, true);【需要添加第三方依赖】

```

（2）全局调整

```java
修改flink-conf.yaml
state.backend: filesystem
state.checkpoints.dir: hdfs://namenode:9000/flink/checkpoints
注意：state.backend的值可以是下面几种：jobmanager(MemoryStateBackend), filesystem(FsStateBackend), rocksdb(RocksDBStateBackend)

```

## 八、checkpoint

### 8.1 checkpoint概述

1. 为了保证state的容错性，Flink需要对state进行checkpoint。

2. Checkpoint是Flink实现容错机制最核心的功能，它能够根据配置周期性地基于Stream中各个Operator/task的状态来生成快照，从而将这些状态数据定期持久化存储下来，当Flink程序一旦意外崩溃时，重新运行程序时可以有选择地从这些快照进行恢复，从而修正因为故障带来的程序数据异常。

3. Flink的checkpoint机制可以与(stream和state)的持久化存储交互的前提：
   持久化的source，它需要支持在一定时间内重放事件。这种sources的典型例子是持久化的消息队列（比如Apache Kafka，RabbitMQ等）或文件系统（比如HDFS，S3，GFS等）
   用于state的持久化存储，例如分布式文件系统（比如HDFS，S3，GFS等）

   

* 生成快照

![1569326195474](http://img.hurenjieee.com/uPic/1569326195474-1574600020491.png)

* 恢复快照

![1569326229867](http://img.hurenjieee.com/uPic/1569326229867-1574600020491.png)

### 8.2 checkpoint配置

默认checkpoint功能是disabled的，想要使用的时候需要先启用，checkpoint开启之后，checkPointMode有两种，Exactly-once和At-least-once，默认的checkPointMode是Exactly-once，Exactly-once对于大多数应用来说是最合适的。At-least-once可能用在某些延迟超低的应用程序（始终延迟为几毫秒）。

```java
默认checkpoint功能是disabled的，想要使用的时候需要先启用
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
// 每隔1000 ms进行启动一个检查点【设置checkpoint的周期】
env.enableCheckpointing(1000);
// 高级选项：
// 设置模式为exactly-once （这是默认值）
env.getCheckpointConfig().setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE);
// 确保检查点之间有至少500 ms的间隔【checkpoint最小间隔】
env.getCheckpointConfig().setMinPauseBetweenCheckpoints(500);
// 检查点必须在一分钟内完成，或者被丢弃【checkpoint的超时时间】
env.getCheckpointConfig().setCheckpointTimeout(60000);
// 同一时间只允许进行一个检查点
env.getCheckpointConfig().setMaxConcurrentCheckpoints(1);
// 表示一旦Flink处理程序被cancel后，会保留Checkpoint数据，以便根据实际需要恢复到指定的Checkpoint【详细解释见备注】
env.getCheckpointConfig().enableExternalizedCheckpoints(ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION);
```

## 九、恢复数据

### 9.1 重启策略概述

* Flink支持不同的重启策略，以在故障发生时控制作业如何重启，集群在启动时会伴随一个默认的重启策略，在没有定义具体重启策略时会使用该默认策略。 如果在工作提交时指定了一个重启策略，该策略会覆盖集群的默认策略，默认的重启策略可以通过 Flink 的配置文件 flink-conf.yaml 指定。配置参数 restart-strategy 定义了哪个策略被使用。
* 常用的重启策略
  1. 固定间隔 (Fixed delay)
  2. 失败率 (Failure rate)
  3. 无重启 (No restart)
* 如果没有启用 checkpointing，则使用无重启 (no restart) 策略。 
  如果启用了 checkpointing，但没有配置重启策略，则使用固定间隔 (fixed-delay) 策略， 尝试重启次数默认值是：Integer.MAX_VALUE，重启策略可以在flink-conf.yaml中配置，表示全局的配置。也可以在应用代码中动态指定，会覆盖全局配置。

### 9.2 重启策略

固定间隔 (Fixed delay)

```java
第一种：全局配置 flink-conf.yaml
restart-strategy: fixed-delay
restart-strategy.fixed-delay.attempts: 3
restart-strategy.fixed-delay.delay: 10 s
第二种：应用代码设置
env.setRestartStrategy(RestartStrategies.fixedDelayRestart(
  3, // 尝试重启的次数
  Time.of(10, TimeUnit.SECONDS) // 间隔
));

```

失败率 (Failure rate)

```java
第一种：全局配置 flink-conf.yaml
restart-strategy: failure-rate
restart-strategy.failure-rate.max-failures-per-interval: 3
restart-strategy.failure-rate.failure-rate-interval: 5 min
restart-strategy.failure-rate.delay: 10 s
第二种：应用代码设置
env.setRestartStrategy(RestartStrategies.failureRateRestart(
  3, // 一个时间段内的最大失败次数
  Time.of(5, TimeUnit.MINUTES), // 衡量失败次数的是时间段
  Time.of(10, TimeUnit.SECONDS) // 间隔
));

```

无重启 (No restart)

```java
第一种：全局配置 flink-conf.yaml
restart-strategy: none
第二种：应用代码设置
env.setRestartStrategy(RestartStrategies.noRestart());
```

### 9.3 多checkpoint

* 默认情况下，如果设置了Checkpoint选项，则Flink只保留最近成功生成的1个Checkpoint，而当Flink程序失败时，可以从最近的这个Checkpoint来进行恢复。但是，如果我们希望保留多个Checkpoint，并能够根据实际需要选择其中一个进行恢复，这样会更加灵活，比如，我们发现最近4个小时数据记录处理有问题，希望将整个状态还原到4小时之前Flink可以支持保留多个Checkpoint，需要在Flink的配置文件conf/flink-conf.yaml中，添加如下配置，指定最多需要保存Checkpoint的个数：

```java
state.checkpoints.num-retained: 20
```

* 这样设置以后就查看对应的Checkpoint在HDFS上存储的文件目录
  hdfs dfs -ls hdfs://namenode:9000/flink/checkpoints
  如果希望回退到某个Checkpoint点，只需要指定对应的某个Checkpoint路径即可实现

### 9.4 从checkpoint恢复数据

* 如果Flink程序异常失败，或者最近一段时间内数据处理错误，我们可以将程序从某一个Checkpoint点进行恢复

```java
bin/flink run -s hdfs://namenode:9000/flink/checkpoints/467e17d2cc343e6c56255d222bae3421/chk-56/_metadata flink-job.jar
```

* 程序正常运行后，还会按照Checkpoint配置进行运行，继续生成Checkpoint数据。
* 当然恢复数据的方式还可以在自己的代码里面指定checkpoint目录，这样下一次启动的时候即使代码发生了改变就自动恢复数据了。

### 9.5 savepoint

* Flink通过Savepoint功能可以做到程序升级后，继续从升级前的那个点开始执行计算，保证数据不中断
  全局，一致性快照。可以保存数据源offset，operator操作状态等信息，可以从应用在过去任意做了savepoint的时刻开始继续消费

checkPoint vs savePoint

**checkPoint**
应用定时触发，用于保存状态，会过期，内部应用失败重启的时候使用。
**savePoint**
用户手动执行，是指向Checkpoint的指针，不会过期，在升级的情况下使用。
注意：为了能够在作业的不同版本之间以及 Flink 的不同版本之间顺利升级，强烈推荐程序员通过 uid(String) 方法手动的给算子赋予 ID，这些 ID 将用于确定每一个算子的状态范围。如果不手动给各算子指定 ID，则会由 Flink 自动给每个算子生成一个 ID。只要这些 ID 没有改变就能从保存点（savepoint）将程序恢复回来。而这些自动生成的 ID 依赖于程序的结构，并且对代码的更改是很敏感的。因此，强烈建议用户手动的设置 ID。

```
.uid("xxxx");
算子后面指定算子的方法。
```

savepoint的使用

```java
1：在flink-conf.yaml中配置Savepoint存储位置
不是必须设置，但是设置后，后面创建指定Job的Savepoint时，可以不用在手动执行命令时指定Savepoint的位置
state.savepoints.dir: hdfs://namenode:9000/flink/savepoints

2：触发一个savepoint【直接触发或者在cancel的时候触发】
bin/flink savepoint jobId [targetDirectory] [-yid yarnAppId]【针对on yarn模式需要指定-yid参数】
bin/flink cancel -s [targetDirectory] jobId [-yid yarnAppId]【针对on yarn模式需要指定-yid参数】

3：从指定的savepoint启动job
bin/flink run -s savepointPath [runArgs]

```

## 十、Flink Window

[Flink Window](./FlinkWindow.md)

[Flink Window+WaterMark](./FlinkWindow+WaterMark.md)