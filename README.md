# 	MyNotePlus

基础知识、Java、大数据笔记

## 起因

* 出身计算机科学，却有愧于计算机科学。工作这几年之后，很多的基础理论只是都已经是忘光了，过去以前的笔记也无从查找。以至虽是计算机专业出身，却无无法证明自己。
* 前段时间出去面试，结果很多面试题都已经忘得差不多了，网上很多面试资料（这里推荐Github上比较火的资料集锦[JavaGuide](https://github.com/Snailclimb/JavaGuide)）。记忆不够深刻，导致很难回答出来问题，很是尴尬。虽然不喜欢背这些面试题，但是现实就是如此，题都背不出来怎么跟别人比。虽然工作用不到这些面试题，但是工作就是有这些原理才能更好的开发。
* 以前整理大部分的资源[笔记MyNote](https://github.com/JackKuang/MyNote)和[大数据BigData](https://github.com/JackKuang/BigData)，大部分是拷贝来自网上的技术总结，虽然是理论知识都是存在的，但是由于是拷贝的，很大一部分知识点记忆不够深刻。
* 所以打算重新创建一个repository，且有以下几个约定：
  * 会从基础只是开始，再到后面的Java开发、大数据开发相关，后面的主要是工作相关的内容。
  * 笔记的形式以全手打为主，拒绝拷贝文字（手打一遍加深理解）。



## 基础内容

### Java

| 内容         | 地址                                              | 完成日期   |
| ------------ | ------------------------------------------------- | ---------- |
| Netty        | [Netty](./Basic/Java/Netty/README.md)             | PAUSE      |
| SpringClould | [SpringCloud](./Basic/Java/SpringCloud/README.md) | DOING      |
| 基础开发规范 | [开发规范](./Basic/Java/开发规范.md)              | DOING      |
| 设计模式     | 参考OneDrive中Excel和Xmind                        | 2021-10-26 |

### 大数据

| 技术栈     | 用途              | 内容                                                   |
| ---------- | ----------------- | ------------------------------------------------------ |
| Hadoop     | 分布式存储        | [Hadoop入门](./Basic/Bigdata/Hadoop/README.md)         |
| Yarn       | 分布式任务调度    | [Yarn入门](./Basic/Bigdata/Yarn/README.md)             |
| MapReduce  | 分布式计算        | [MapReduce入门](./Basic/Bigdata/MapReduce/README.md)   |
| Hive       | 数据仓库          | [Hive入门](./Basic/Bigdata/Hive/README.md)             |
| HBase      | Key-Value数据库   | [HBase入门](./Basic/Bigdata/HBase/README.md)           |
| Zookeeper  | 分布式协调中心    | [Zookeeper入门](./Basic/Bigdata/Zookeeper/README.md)   |
| Spark      | 分布式计算        | [Spark入门](./Basic/Bigdata/Spark/README.md)           |
| Kafka      | 分布式消息系统    | [Kafka入门](./Basic/Bigdata/Kafka/README.md)           |
| Flink      | 分布式计算        | [Flink入门](./Basic/Bigdata/Flink/README.md)           |
| Presto     | 分布式SQL查询引擎 | [Presto入门](./Basic/Bigdata/Presto/README.md)         |
| ClickHouse | OLAP实时分析      | [ClickHouse入门](./Basic/Bigdata/ClickHouse/README.md) |



## 进阶内容

### Linux

| 内容                         | 地址                                                    | 完成日期   |
| ---------------------------- | ------------------------------------------------------- | ---------- |
| Linux查看资源占用            | [Linux资源占用](./Linux/Resource.md)                    | 2020-09-05 |
| sudo执行命令时环境变量被重置 | [sudo执行命令时环境变量被重置](./Linux/SudoResetEnv.md) | 2021-03-16 |
| kill命令无法删除子进程       | [Linux Kill杀进程的问题](./Linux/Kill.md)               | 2021-03-16 |



### Java

| 内容                         | 地址                                                         | 完成日期   |
| ---------------------------- | ------------------------------------------------------------ | ---------- |
| JVM                          | [JVM](./Java/JVM.md)                                         | 2020-09-03 |
| 线程与进程                   | [线程与进程](./Java/ProcessAndThread.md)                     | 2020-09-03 |
| 多线程                       | [多线程](./Java/MultiThread.md)                              | 2020-09-03 |
| 数据结构【树】               | [树](./Java/Tree.md)                                         | 2020-09-08 |
| 算法【排序】                 | [排序算法](./Java/SortAlgorithm.md)                          | 2020-09-12 |
| SpringBoot高并发(undertow)   |                                                              |            |
| Java并发                     |                                                              |            |
| BIO/NIO/AIO                  | [IO](./Java/IO.md)                                           | 2020-09-10 |
| volatile和synchronized特点   | [volatile和synchronized特点](./Java/VolatileAndSynchronized.md) | 2020-09-03 |
| 锁                           | [锁](./Java/Lock.md)                                         | 202-09-10  |
| BitMap实现                   | [BitMap](./Java/BitMap.md)                                   | 2020-09-13 |
| Object有哪些方法             | [Object](./Java/Object.md)                                   | 2020-09-14 |
| HashMap                      | [HashMap](./Java/HashMap.md)                                 | 2020-09-17 |
| Java中如何动态创建接口的实现 | [Proxy](./Java/Proxy.md)                                     | 2020-10-20 |
| Oracle JDBC连接问题          | [连接异常](./Java/OracleJdbcConnectionError.md)              | 2020-12-21 |
| Arthas                       | [Java调试利器Arthas](./Java/Arthas/Arthas.md)                | 2020-12-24 |

### 数据库

| 内容           | 地址                                        | 完成日期   |
| -------------- | ------------------------------------------- | ---------- |
| InnoDB与MyISAM | [InnoDB与MyISAM](./Database/MysqlEngine.md) | 2020-09-11 |
|                |                                             |            |

### 大数据

| 内容                        | 地址                                                         | 完成日期   |
| --------------------------- | ------------------------------------------------------------ | ---------- |
| MapReduce原理+Suffle        | [MapReduce](./BigData/MapReduce.md)                          | 2020-09-22 |
| 数据中台                    | [数据中台](./BigData/DataCenter.md)(参考OneDrive中的脑图)    | 2021-10-26 |
| 大数据层次                  | [大数据层次](./BigData/Level.md)                             | 2020-09-13 |
| 大数据建模                  | [大数据建模](./BigData/BigDataModeling.md)                   | 2020-09-22 |
| Flink流流JOIN               | [Flink Streaming Join](./BigData/FlinkStreamingJoin.md)      | 2020-09-14 |
| Flink精确一次语义           | [Flink精确一次](./BigData/FlinkExactlyOnce.md)               | 2020-09-13 |
| Flink端到端精确一次语义     | [Flink端到端精确一次](./BigData/FlinkSinkExactlyOnce.md)     | 2020-09-15 |
| Kylin精确去重与留存分析     | [Kylin精确去重与留存分析](./Bigdata/kylinRetention.md)       | 2020-09-17 |
| ElasticSearch基础入门       | [ElasticSearch入门](./BigData/ElasticSearch.md)              | 2021-01-19 |
| ElasticSearch的parent-child | [ElasticSearch的parent-child](./BigData/ElasticSearch-parent-child.md) | 2021-01-19 |
| ClickHouse入门              | [ClickHouse入门](./BigData/ClickHouse.md)                    | 2021-04-26 |
| ClickHouse集群解决方案      | [ClickHouse集群](./BigData/ClickHouse-Cluster.md)            | 2021-04-26 |
|                             |                                                              |            |



### 数据处理方案

| 内容                  | 简介                                            | 地址 | 完成日期 |
| --------------------- | ----------------------------------------------- | ---- | -------- |
| SpringBoot高并发      | 如何提高Web的高并发请求？                       |      |          |
| Kafka数据延迟处理方案 | 线上事故导致Kafka消费延迟，如何快速消费并恢复？ |      | TBD      |
| Flink CEP             |                                                 |      |          |

### 技术文档

| 内容         | 简介 | 地址                               | 完成日期 |
| ------------ | ---- | ---------------------------------- | -------- |
| 领域驱动设计 |      | [领域驱动设计](./Documents/DDD.md) | TBD      |
|              |      |                                    |          |
|              |      |                                    |          |