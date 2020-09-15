# Flink Sink Exactly Once实现

前面我们提及了Flink内部Exactly-Once精确一次语义，但是无法实现到Sink的精确一次语义。

在Flink 1.5版本之后，正式引入了一个里程碑式的功能：两阶段提交Sink，即TwoPhaseCommitSinkFunction。该SinkFunction提取并封装了两姐u但提交协议中的公共逻辑，自此Flink搭配特定的Source和Sink（特别式0.11版本Kafka）搭建完全的精确一次处理语义（Exactly-Once semantics）应用成为了可能。作为一个抽象类TwoPhaseCommitFunction提供了一个抽象层供用户自行实现特定方法来支持exactly-once semantics。

## 一、Flink的仅一次处理

* 仅处理一次，我们理解为每条输入消息只会影响最终结果一次。

  > 影响处理应用处理结果一次，而非被处理一次。
  >
  > 即使Flink机器出现故障或者软件崩溃，Flink也要保证不回右数据被重复处理或者没有被处理从而影响状态。



## 二、Flink实现仅一次语义的应用

结合一个实例来帮助了解两阶段提交以及Flink如何使用它来实现仅一次处理语义。该实例从Kafka中读取数据，经处理之后再写回到Kafka（需要kafka支持事务，支持事务是flink实现端到端仅一次语义的必要条件）。

下面是一个场景：

1. 一个Source，从kakfa中读取数据（即KafkaConsumer）

2. 一个时间窗口化的聚合操作

3. 一个Sink，将结果写回到Kafka（即KafkaProducer）

   ![640?wx_fmt=png](http://img.hurenjieee.com/uPic/640.jpeg)

若要sink支持 exactly-once semantics，它必须以事务的方式写数据到Kafka，这样当提交事务时，两次checkpoint间的写入操作当作为一个事务被提交。这确保了初心啊故障或者崩溃时，这些写入操作能够被回滚。

当然了，在一个分布式且含有多个并发执行sink的应用中，仅仅执行单词提交或回滚是不够的，因为所有组件都必须对这些提交或回滚达成共识，这样才能保证得到一个一致性的结果。Flink使用两阶段提交协议以预提交（pre-commit）阶段来解决这个问题。



### 2.1 pre-commit

Flink checkpoint开始时变进入到pre-commit阶段。具体来说，一旦checkpoint开始，Flink的JobManager向输入流写入一个checkpoint barrier将流中的所有消息分割成属于本次checkpoint的消息以及属于下次checkpoint的。

![640?wx_fmt=png](http://img.hurenjieee.com/uPic/640-20200915195424922.jpeg)

barrier也会在算子中间流转。对于每个operator来说，该barrier会出触发operator状态后端为该operator状态进行打快照。

![640?wx_fmt=png](http://img.hurenjieee.com/uPic/640-20200915195431808.jpeg)

当checkpoint barrier在所有的operator都传递了一遍且对应的快照都成功完成之后，pre-commit阶段才算完成。该过程中所有创建的快照都被视为是checkpoint的一部分。其实，checkpoint时整个应用的全局状态，当然也包含了pre-commit阶段提交的外部状态。当出现崩溃时，我们可以回滚状态到最新已成功完成快照时的时间点。

![640?wx_fmt=png](http://img.hurenjieee.com/uPic/640-20200915195636075.jpeg)

### 2.2 commit

下一个阶段就是是通知所有的operator，告诉他们checkpoint已经成功完成。这便是两阶段提交协议的第二阶段：commit阶段。

该阶段汇总给你的JobManager会为应用中每个operator发起checkpoint已完成的回调逻辑。

本案例中的data source和窗口操作无外部状态，因此在该阶段，这两个operator无序执行任何逻辑，但是data sink是有外部状态的，因此此次我们必须提交外部事务。

![640?wx_fmt=png](http://img.hurenjieee.com/uPic/640-20200915200947036.jpeg)



### 2.3 总结

1. 一旦所有的operator完成各自的pre-commit草足，他们会发起一次commit操作。
2. 倘若一个pre-commit失败，所有其他的pre-commit必须被终止，并且Flink会回滚最精成功完成的checkpoint
3. 一旦pre-commit完成，必须要确保commit也要成功——operator和外部系统都需要对此进行保证。倘若commit失败（比如网络故障等），Flink应用就会崩溃，然后根据用户重启逻辑，之后再次重试commit。这个过程是至关重要的，因为倘若commit无法顺利进行，就可能出现数据丢失的情况。

因此，所有operator必须对checkpoint最终达成共识：即所有operator都必须要保证数据提交要么成功执行，要么被终止然后回滚。

## 三、Flink中实现两阶段提交

这种operator的管理有些复杂，这也就是为什么Flink题去了公共逻辑并封装进TwoPhaseCommitSinkFunction抽象类原因：

这里讨论如何扩展TwoPhaseCommitSinkFunction类来实现一个简单的基于文件的sink。若要实现exactly-once semantics的文件sink，我们需要实现以下4个方法：

1. **beginTransaction**：开启一个事务，在临时目录下创建一个临时文件，之后，写入数据到该文件中。
2. **preCommit**：在pre-commit阶段，flush缓存数据快到资盘，然后关闭该文件，确保再不回写入数据到该文件。同时开启一个新的事务执行下一个checkpoint的写入操作
3. **commit**：在commit阶段，我们以原子性的方式将在上一节顿啊的文件写入正真的文件目录下。注意一点：这会增加输出数据可见性的演示。通俗说就是用户想看到最终数据需要等会，不是实时的。
4. **abort**：一旦终止事务，我们需要自己删除临时文件。



当出现崩溃时，Flink会恢复最新已完成快照中应用状态。需要注意的是在某些偶然的场景下，pre-commit阶段已成功而commit尚未开始（也就是operator尚未来得及告知需要开启commit），此时如果发生崩溃，Flink会将operator状态恢复到已完成pre-commit但撒谎功能为commit的状态。

在一个checkpoint状态中，对于已完成pre-commit的事务状态，我们必须保存足够多的信息，这样才能确保在重启后重新发起commit亦或是终止掉事务。

## 四、总结

1. Flink checkpoint机制是实现两阶段提交协议以及提供仅一次语义的基石
2. 与其他系统持久化传输中的数据不同，flink不需要把计算的每个阶段写入到磁盘中
3. Flink新的TwoPhaseCommitSinkFunction封住两阶段提交协议的公共逻辑使之搭配支持事务的外部系统来共同构建仅一次语义称为可能。
4. Flink 1.4 +Parvega/Kakfa 0.11 producer开始支持仅一次语义
5. Flink Kafka 0.11 producer基于TwoPhaseCommitSinkFunction实现，比起至少一次语义的producer而言开销并未显著增加。



## 参考：

Flink+Kafka 0.11端到端精确一次处理语义实现：https://blog.csdn.net/rlnLo2pNEfx9c/article/details/81369878