# ClickHouse集群方案

## 一、配置说明

需要提前安装以下节点：

* Zookeeper*3

  > 存储副本的元数据信息，3.4.5版本+

* ClickHouse*4

  > ClickHouse集群，2*2服务架构，2个分片\*2个副本

* Nginx

  > 把ClickHouse多台服务统一到一个入口。

安装完成后，配置`/etc/metrika.xml`或者`/etc/clickhouse-server/metrika.xml`，具体可参考与config.xml

```
<include_from>/etc/metrica.xml</include_from>
```

配置文件说明

```xml
<yandex>
    <!--ck集群节点-->
    <clickhouse_remote_servers>
      	<!-- 集群名称，clickhouse_cluster_name会用到 -->
        <clickhouse_cluster_name>
            <!-- 分片1 -->
            <shard>
                <internal_replication>true</internal_replication>
                <replica>
                    <host>clickhouse01</host>
                    <port>9000</port>
                    <user>default</user>
                    <password>clickhouse</password>
                </replica>
                <!-- 副本1 -->
                <replica>
                    <host>clickhouse02</host>
                    <port>9000</port>
                    <user>default</user>
                    <password>clickhouse</password>
                </replica>
            </shard>

            <!-- 分片2 -->
            <shard>
                <internal_replication>true</internal_replication>
                <replica>
                    <host>clickhouse03</host>
                    <port>9000</port>
                    <user>default</user>
                    <password>clickhouse</password>
                </replica>
                <!-- 副本2 -->
                <replica>
                    <host>clickhouse04</host>
                    <port>9000</port>
                    <user>default</user>
                    <password>clickhouse</password>
                </replica>
            </shard>
        </clickhouse_cluster_name>
    </clickhouse_remote_servers>
    
    <!--zookeeper相关配置-->
    <zookeeper-servers>
        <node index="1">
            <host>clickhouse02</host>
            <port>2181</port>
        </node>
        <node index="2">
            <host>clickhouse03</host>
            <port>2181</port>
        </node>
        <node index="3">
            <host>clickhouse04</host>
            <port>2181</port>
        </node>
    </zookeeper-servers>
    <!-- 节点配置 -->
    <macros>
        <layer>01</layer>
        <!-- 分片号 -->
        <shard>01</shard>
      	<!-- 副本名称，通常用域名等内容作唯一 -->
        <replica>clickhouse01</replica>
    </macros>
    <networks>
        <ip>::/0</ip>
    </networks>
    <!--压缩相关配置-->
    <clickhouse_compression>
        <case>
            <min_part_size>10000000000</min_part_size>
            <min_part_size_ratio>0.01</min_part_size_ratio>
            <method>lz4</method> <!--压缩算法lz4压缩比zstd快, 更占磁盘-->
        </case>
    </clickhouse_compression>
</yandex>
```

## 二、整体配置

* all_table03：分布式表（Distributed）

* table03：副本表（ReplicatedMergeTree）

  ![ClickHouse2](http://img.hurenjieee.com/uPic/ClickHouse2.png)

* 所有节点基本配置：

* ```xml
  <!--ck集群节点-->
      <clickhouse_remote_servers>
        	<!-- 集群名称，clickhouse_cluster_name会用到 -->
          <clickhouse_cluster_name>
              <!-- 分片1 -->
              <shard>
                  <internal_replication>true</internal_replication>
                  <replica>
                      <host>clickhouse01</host>
                      <port>9000</port>
                      <user>default</user>
                      <password>clickhouse</password>
                  </replica>
                  <!-- 副本1 -->
                  <replica>
                      <host>clickhouse02</host>
                      <port>9000</port>
                      <user>default</user>
                      <password>clickhouse</password>
                  </replica>
              </shard>
  
              <!-- 分片2 -->
              <shard>
                  <internal_replication>true</internal_replication>
                  <replica>
                      <host>clickhouse03</host>
                      <port>9000</port>
                      <user>default</user>
                      <password>clickhouse</password>
                  </replica>
                  <!-- 副本2 -->
                  <replica>
                      <host>clickhouse04</host>
                      <port>9000</port>
                      <user>default</user>
                      <password>clickhouse</password>
                  </replica>
              </shard>
          </clickhouse_cluster_name>
      </clickhouse_remote_servers>
      
      <!--zookeeper相关配置-->
      <zookeeper-servers>
          <node index="1">
              <host>zookeeper01</host>
              <port>2181</port>
          </node>
          <node index="2">
              <host>zookeeper02</host>
              <port>2181</port>
          </node>
          <node index="3">
              <host>zookeeper03</host>
              <port>2181</port>
          </node>
      </zookeeper-servers>
  ```
  
* clickhouse01

  ```xml
  <macros>
    <layer>01</layer>
    <!-- 分片号 -->
    <shard>01</shard>
    <!-- 副本名称，通常用域名等内容作唯一 -->
    <replica>clickhouse01</replica>
  </macros>
  ```

* Clickhouse02

  ```xml
  <macros>
    <layer>01</layer>
    <!-- 分片号 -->
    <shard>01</shard>
    <!-- 副本名称，通常用域名等内容作唯一 -->
    <replica>clickhouse02</replica>
  </macros>
  ```

* Clickhouse03

  ```xml
  <macros>
    <layer>01</layer>
    <!-- 分片号 -->
    <shard>02</shard>
    <!-- 副本名称，通常用域名等内容作唯一 -->
    <replica>clickhouse03</replica>
  </macros>
  ```

* Clickhouse04

  ```xml
  <macros>
    <layer>01</layer>
    <!-- 分片号 -->
    <shard>02</shard>
    <!-- 副本名称，通常用域名等内容作唯一 -->
    <replica>clickhouse04</replica>
  </macros>
  ```

* Nginx配置

  ```properties
  upstream ck {
      server  clickhouse01:8123 weight=25 max_fails=3 fail_timeout=60s;
      server  clickhouse02:8123 weight=25 max_fails=3 fail_timeout=60s;
      server  clickhouse03:8123 weight=25 max_fails=3 fail_timeout=60s;
      server  clickhouse04:8123 weight=25 max_fails=3 fail_timeout=60s;
  }
  
  server {
      listen 18123;
      location / {
          proxy_pass http://ck;
      }
  }
  ```

  

## 三、用户使用

SQL示例：

```sql
-- 创建数据库
CREATE DATABASE cluster ON CLUSTER clickhouse_cluster_name

-- 创建表
CREATE TABLE table03 ON CLUSTER clickhouse_cluster_name
(
    EventDate DateTime,
    CounterID UInt32,
    UserID UInt32
) ENGINE = ReplicatedMergeTree('/clickhouse/tables/{layer}-{shard}/cluster/table03', '{replica}')
PARTITION BY toYYYYMM(EventDate)
ORDER BY (CounterID, EventDate, intHash32(UserID))
SAMPLE BY intHash32(UserID)

-- 创建分布式表
CREATE TABLE IF NOT EXISTS all_table03 ON CLUSTER clickhouse_cluster_name 
(
    EventDate DateTime,
    CounterID UInt32,
    UserID UInt32
)  ENGINE = Distributed(clickhouse_cluster_name, cluster , table03)

-- 插入数据
INSERT INTO cluster.table03 (EventDate,CounterID,UserID) VALUES ('2021-04-26 01:00:00',1,1);
INSERT INTO cluster.table03 (EventDate,CounterID,UserID) VALUES ('2021-04-26 01:00:00',2,2);
INSERT INTO cluster.table03 (EventDate,CounterID,UserID) VALUES ('2021-04-26 01:00:00',3,3);
INSERT INTO cluster.table03 (EventDate,CounterID,UserID) VALUES ('2021-04-26 01:00:00',4,4);
INSERT INTO cluster.table03 (EventDate,CounterID,UserID) VALUES ('2021-04-26 01:00:00',5,5);
INSERT INTO cluster.table03 (EventDate,CounterID,UserID) VALUES ('2021-04-26 01:00:00',6,6);

-- 多次查询结果不一致
select * from cluster.table03;

-- 多次查询结果一致
select * from cluster.all_table03;
```

## 四、故障测试

场景一：分片中一个副本下线 ==> clickhouse03下线

1. 所有节点在线，查询数据：

![image-20210427140556034](http://img.hurenjieee.com/uPic/image-20210427140556034.png)

1. clickhouse03下线，查询数据未丢失数据：

![image-20210427140852412](http://img.hurenjieee.com/uPic/image-20210427140852412.png)

1. clickhouse04下线，服务查询异常：
2. clickhouse04上线，数据查询恢复：

![image-20210427141116911](http://img.hurenjieee.com/uPic/image-20210427141116911.png)

5. 在clickhouse04插入数据：

```sql
INSERT INTO cluster.table03 (EventDate,CounterID,UserID) VALUES ('2021-07-26 01:00:00',7,7);
```

6. 查询数据，数据正确：

![image-20210427141341505](http://img.hurenjieee.com/uPic/image-20210427141341505.png)

* 恢复clickhouse03，查看数据，在clickhouse04上添加数据单独同步：

![image-20210427141519563](http://img.hurenjieee.com/uPic/image-20210427141519563.png)

## 五、监控运维

服务可通过prometheus监听所有ClickHouse服务节点：

```sh
[root@clickhouse01 clickhouse-server]# curl http://clickhouse01:8123
Ok.
[root@clickhouse01 clickhouse-server]# 
```

一旦存在多台ClickHouse节点下线，需要运维处理了。



## 六、其他说明

### 6.1 ReplicatedMergeTree表引擎无法插入数据

* ReplicatedTree无法插入重复数据

> 本身副本表特性就是有重复数据删除的特性的。
>
> 如果有多个相同数据写入（相同数据、相同顺序）的形式写入，那么数据只会被写入一。
>
> 原因在于发生网络错误时，可能是相同的数据块重复发送了，也可能是重复插入请求，节点无法确定数据变化，故设计了数据插入幂等性。
>
> 可通过修改insert_deduplicate(默认1)来设置生效。

参考资料：https://clickhouse.tech/docs/en/operations/settings/settings/#settings-insert-deduplicate