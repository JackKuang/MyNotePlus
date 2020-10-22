# BigData

## 文档说明

* 这个项目为大数据相关的基础知识内容，原来在我的另外一个项目[MyNote](https://github.com/JackKuang/MyNote/tree/master/大数据)，记录了我所那有的学习笔记。但是项目较乱，体验不佳：
  * 图片上传至Github，没使用图床等工具，导致项目较大，clone较慢。
  * 目录结构不够清晰，需要手动查看各个文档，切换不方便。
* 所以，在大数据这个模块下，重新进行了创建，并把之前整理的思维导图也上传（思维导图文件比较大，但暂时没啥办法优化）。

## 技术栈入门

| 技术栈    | 用途           | 内容 |
| --------- | -------------- | ---- |
| Hadoop | 分布式存储 | [Hadoop入门](./Hadoop/README.md) |
| Yarn     | 分布式任务调度 | [Yarn入门](./Yarn/README.md) |
|  MapReduce          | 分布式计算 | [MapReduce入门](./MapReduce/README.md) |
| Hive | 数据仓库 | [Hive入门](./Hive/README.md) |
| HBase | Key-Value数据库 | [HBase入门](./HBase/README.md) |
| Zookeeper | 分布式协调中心 | [Zookeeper入门](./Zookeeper/README.md) |
| Spark | 分布式计算 | [Spark入门](./Spark/README.md) |
| Kafka | 分布式消息系统 | [Kafka入门](./Kafka/README.md) |
| Flink | 分布式计算 | [Flink入门](./Flink/README.md) |
| Presto | 分布式SQL查询引擎 | [Presto入门](./Presto/README.md) |

## 技术工具入门

| 工具名称 | 用途         | 功能                                        |
| -------- | ------------ | ------------------------------------------- |
| azkaban  | 分布式脚本   | 分布式调度脚本                              |
| XXL-JOB  | 分布式脚本   | 分布式调度Python、Java、Shell脚本           |
| Flume    | 日志收集     | 收集日志到HDFS、Kafka等数据                 |
| Sqoop    | 数据同步     | 离线同步数据源到Hadoop的HDFS、HIVE、HBASE等 |
| DataX    | 数据同步     | 离线同步数据源到HDFS、RDBMS等               |
| Debezium | 数据日志采集 | 实时采集数据库binlog到kafka                 |
| maxwell  | 数据日志采集 | 实时采集数据库binlog到kafka（使用简单）     |

## 技术集群

