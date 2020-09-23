# MyNotePlus
基础知识、Java、大数据笔记

## 起因

* 出身计算机科学，却有愧于计算机科学。工作这几年之后，很多的基础理论只是都已经是忘光了，过去以前的笔记也无从查找。以至虽是计算机专业出身，却无无法证明自己。
* 前段时间出去面试，结果很多面试题都已经忘得差不多了，网上很多面试资料（这里推荐Github上比较火的资料集锦[JavaGuide](https://github.com/Snailclimb/JavaGuide)）。记忆不够深刻，导致很难回答出来问题，很是尴尬。虽然不喜欢背这些面试题，但是现实就是如此，题都背不出来怎么跟别人比。虽然工作用不到这些面试题，但是工作就是有这些原理才能更好的开发。
* 以前整理大部分的资源[笔记MyNote](https://github.com/JackKuang/MyNote)和[大数据BigData](https://github.com/JackKuang/BigData)，大部分是拷贝来自网上的技术总结，虽然是理论知识都是存在的，但是由于是拷贝的，很大一部分知识点记忆不够深刻。
* 所以打算重新创建一个repository，且有以下几个规定：
  * 会从基础只是开始，再到后面的Java开发、大数据开发相关，后面的主要是工作相关的内容。
  * 笔记的形式以全手打为主，拒绝拷贝文字（手打一遍加深理解）。

## 基础知识

| 内容              | 地址                                 | 完成日期   |
| ----------------- | ------------------------------------ | ---------- |
| Linux查看资源占用 | [Linux资源占用](./Linux/Resource.md) | 2020-09-05 |



## Java

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
| 锁                           | [锁](./Java/Lock.md)                                         | 2020-09-10 |
| Netty                        |                                                              |            |
| BitMap实现                   | [BitMap](./Java/BitMap.md)                                   | 2020-09-13 |
| Object有哪些方法             | [Object](./Java/Object.md)                                   | 2020-09-14 |
| HashMap                      | [HashMap](./Java/HashMap.md)                                 | 2020-09-17 |
| Java中如何动态创建接口的实现 |                                                              |            |

## 数据库

| 内容           | 地址                                        | 完成日期   |
| -------------- | ------------------------------------------- | ---------- |
| InnoDB与MyISAM | [InnoDB与MyISAM](./Database/MysqlEngine.md) | 2020-09-11 |
|                |                                             |            |
|                |                                             |            |

## 大数据

| 内容                    | 地址                                                     | 完成日期   |
| ----------------------- | -------------------------------------------------------- | ---------- |
| MapReduce原理+Suffle    | [MapReduce](./BigData/MapReduce.md)                      | 2020-09-22 |
| 数据中台                | [数据中台](./BigData/DataCenter.md)                      | DOING      |
| 大数据层次              | [大数据层次](./BigData/Level.md)                         | 2020-09-13 |
| 大数据建模              | [大数据建模](./BigData/BigDataModeling.md)               | 2020-09-22 |
| Flink流流JOIN           | [Flink Streaming Join](./BigData/FlinkStreamingJoin.md)  | 2020-09-14 |
| Flink精确一次语义       | [Flink精确一次](./BigData/FlinkExactlyOnce.md)           | 2020-09-13 |
| Flink端到端精确一次语义 | [Flink端到端精确一次](./BigData/FlinkSinkExactlyOnce.md) | 2020-09-15 |
| Kylin精确去重与留存分析 | [Kylin精确去重与留存分析](./Bigdata/kylinRetention.md)   | 2020-09-17 |
| ClickHouse为何快        | [ClickHouse为何快](./Bigdata/ClickHouse.md)              | 2020-09-17 |
| Flink Async IO          | [Flink Async IO](./Bigdata/FlinkAsyncIO.md)              |            |
|                         |                                                          |            |



### 数据处理方案

| 内容                  | 简介                                            | 地址 | 完成日期 |
| --------------------- | ----------------------------------------------- | ---- | -------- |
| SpringBoot高并发      | 如何提高Web的高并发请求？                       |      | TBD      |
| Kafka数据延迟处理方案 | 线上事故导致Kafka消费延迟，如何快速消费并恢复？ |      | TBD      |
| Flink CEP             |                                                 |      |          |

