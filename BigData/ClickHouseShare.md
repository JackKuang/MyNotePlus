# ClickHouse入门

## 一、ClickHouse简介

> ClickHouse® is a column-oriented database management system (DBMS) for online analytical processing of queries (OLAP).



### 京东

![image-20220329161157749](http://img.hurenjieee.com/uPic/image-20220329161157749.png)

### B站

![image-20220329161644509](http://img.hurenjieee.com/uPic/image-20220329161644509.png)







## 二、ClickHouse为什么会快



### 2.1 列式数据库

行式数据库

![Row oriented](http://img.hurenjieee.com/uPic/row-oriented.gif)

列式数据库

![Column oriented](http://img.hurenjieee.com/uPic/column-oriented.gif)



1. 针对分析类查询，通常只需要读取表的一小部分列。在列式数据库中你可以只读取你需要的数据。例如，如果只需要读取100列中的5列，这将帮助你最少减少20倍的I/O消耗。
2. 由于数据总是打包成批量读取的，所以压缩是非常容易的。同时数据按列分别存储这也更容易压缩。这进一步降低了I/O的体积。
3. 由于I/O的降低，这将帮助更多的数据被系统缓存。



### 2.2 多核心并行处理





### 三、安装部署

安装过程参考官网，

Docker命令快速安装

```
docker pull yandex/clickhouse-server:21.7.4.18
docker rm -f clickhouse-single
docker run -d --name clickhouse-single --privileged  -p 8123:8123 -p 9000:9000 --ulimit nofile=262144:262144 --volume=$(pwd)/clickhouse-single:/var/lib/clickhouse yandex/clickhouse-server:21.7.4.18
```



## 四、客户端

提供2个端口：8123、9000端口



## 五、表引擎

## 5.1 MergeTree系列

###  5.1.2 MergeTree

Clickhouse 中最强大的表引擎当属 `MergeTree` （合并树）引擎及该系列（`*MergeTree`）中的其他引擎。
当你有大量数据要插入到表中，你要高效地一批批写入数据片段，并希望这些数据片段在后台按照一定规则合并。相比在插入时不断修改（重写）数据进存储，这种策略会高效很多。
主要特点：

* 主键排序
* 数据分区
* 支持副本
* 支持样本



## 5.2 Log



## 5.3 集成表引擎

### 5.3.1 JDBC

### 5.3.2 MySQL

### 5.3.2 Hive

### 5.3.3 HDFS

### 5.3.4 Kafka

