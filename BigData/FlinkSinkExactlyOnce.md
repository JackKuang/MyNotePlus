# Flink Sink Exactly Once实现

前面我们提及了Flink内部Exactly-Once精确一次语义，但是无法实现到Sink的精确一次语义。

在Flink 1.5版本之后，正式引入了一个里程碑式的功能：两阶段提交Sink，即TwoPhaseCommitSinkFunction。该SinkFunction提取并封装了两姐u但提交协议中的公共逻辑，自此Flink搭配特定的Source和Sink（特别式0.11版本Kafka）搭建完全的精确一次处理语义（Exactly-Once semantics）应用成为了可能。作为一个抽象类TwoPhaseCommitFunction提供了一个抽象层供用户自行实现特定方法来支持exactly-once semantics

## 一、Flink实现仅一次语义的应用



## 参考：

Flink+Kafka 0.11端到端精确一次处理语义实现：https://blog.csdn.net/rlnLo2pNEfx9c/article/details/81369878