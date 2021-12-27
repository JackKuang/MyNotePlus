# ClickHouse 中处理实时更新

## **ClickHouse 更新的简短历史**

* 2016

> https://clickhouse.yandex/blog/en/how-to-update-data-in-clickhouse
>
> ClickHouse 并不支持数据修改，只能使用特殊的插入结构来模拟更新，并且数据必须按分区丢弃。

* 2018

> https://altinity.com/blog/2018/10/16/updates-in-clickhouse
>
> 这种异步、非原子性的更新以 ALTER TABLE UPDATE 语句的形式实现，并且可能会打乱大量数据。
> 这对于批量操作和不频繁的更新是很有用的，因为它们不需要即时的结果。

## 用例

考虑一个生成各种报警的系统。
用户或机器学习算法会不时查询数据库，以查看新的报警并进行确认。确认操作需要修改数据库中的报警记录。一旦得到确认，报警将从用户的视图中消失。这看起来像是一个 OLTP 操作，与 ClickHouse 格格不入。



## ClickHouse单节点启动

```sh
docker pull yandex/clickhouse-server
docker rm -f clickhouse-single
docker run -d --name clickhouse-single --privileged  -p 8123:8123 -p 9000:9000 --ulimit nofile=262144:262144 --volume=$(pwd)/clickhouse-single:/var/lib/clickhouse yandex/clickhouse-server
```

## 方案一、 ReplacingMergeTree

我们创建一张存储报警信息的表

```sql
CREATE TABLE alerts(
  tenant_id     UInt32,
  alert_id      String,
  timestamp     DateTime Codec(Delta, LZ4),
  alert_data    String,
  acked         UInt8 DEFAULT 0,
  ack_time      DateTime DEFAULT toDateTime(0),
  ack_user      LowCardinality(String) DEFAULT ''
)
ENGINE = ReplacingMergeTree(ack_time)
PARTITION BY tuple()
ORDER BY (tenant_id, timestamp, alert_id);
```

其中几个字段如下：

> * tenant_id	渠道id
> * alert_id	告警id
> * alert_data	告警内容（实际上，这里可能是很多列，但是为了简单，这里默认一列）
> * acked	0是未告警，1是告警
> * ack_time	告警信息
> * ack_user	告警用户

其中ReplacingMergeTree特性如下：

> https://clickhouse.com/docs/en/engines/table-engines/mergetree-family/replacingmergetree/
>
> ReplacingMergeTee 是一个特殊的表引擎，它借助 ORDER BY  语句按主键替换数据——具有相同键值的新版本行将替换旧版本行。上面的时间即以(tenant_id, timestamp, alert_id)作为主键，并以ack_time作为版本号。
> 替换是在后台合并操作中进行的，它不会立即发生，也不能保证会发生，很容易发生数据一致性问题。不过ClickHouse有特殊语法处理该问题。

我们先导入数据，为 1000 个租户生成 500 万个报警（由于是docker启动，内存给低了，可执行多次）：

```sql
INSERT
	INTO
	alerts(tenant_id, alert_id, timestamp, alert_data)
SELECT
	toUInt32(rand(1)%1000 + 1) AS tenant_id,
	randomPrintableASCII(64) as alert_id,
	toDateTime('2020-01-01 00:00:00') + rand(2)%(3600 * 24 * 30) as timestamp,
	randomPrintableASCII(1024) as alert_data
FROM
	numbers(5000000);
```

确认 99% 的报警，为“acked”、“ack_user”和“ack_time”列提供新值。我们只是插入一个新行，而不是更新。

```sql
INSERT
	INTO
	alerts (tenant_id, alert_id, timestamp, alert_data, acked, ack_user, ack_time)
SELECT
	tenant_id,
	alert_id,
	timestamp,
	alert_data,
	1 as acked,
	concat('user',
	toString(rand()%1000)) as ack_user,
	now() as ack_time
FROM
	alerts
WHERE
	cityHash64(alert_id) % 99 != 0;
```

查看一下数据总量。

```sh
314d64507090 :) SELECT count() FROM alerts;
┌─count()─┐
│ 9949425 │
└─────────┘
1 rows in set. Elapsed: 0.003 sec. 
```

很显然，数据还没有发生替换，表里包含了所有的数据，我们需要查询一下真正的数据

```sh
314d64507090 :) SELECT count() FROM alerts FINAL;
Query id: aca5535f-3d2c-4673-b6ea-43558108b3ab
┌─count()─┐
│ 5000000 │
└─────────┘
1 rows in set. Elapsed: 1.463 sec. Processed 9.95 million rows, 855.65 MB (6.80 million rows/s., 584.81 MB/s.)
```

对比之后，我们发现使用了FINAL关键字之后，ClickHouse花费了大量的时间。主要是ClickHouse在执行查询之后必须扫描所有的行，并根据主键进行合并操作。

同样的，我们查询一下未确认的数据。

```sql
314d64507090 :) SELECT count() FROM alerts FINAL where not acked;
┌─count()─┐
│   50575 │
└─────────┘
1 rows in set. Elapsed: 1.286 sec. Processed 9.95 million rows, 855.65 MB (7.73 million rows/s., 665.11 MB/s.)
```

很显然，虽然数据的计算量减少，但是查询时间和处理的数据量还是一样。这种情况下，筛选并不会加快查询速度。同时，随着表的增大，成本可能会更大，扩展很不方便。

整表查询在实时更新的场景下没什么帮助，那如果是精确查询场景下呢？随机查询某个tenant_id下的数据。

```sh
314d64507090 :) SELECT
                  count(),
                  sum(cityHash64(*)) AS data
                FROM alerts FINAL
                WHERE (tenant_id = 455) AND (NOT acked);
┌─count()─┬─────────────data─┐
│      55 │ 2951804477181383 │
└─────────┴──────────────────┘
1 rows in set. Elapsed: 0.404 sec. Processed 32.77 thousand rows, 36.77 MB (81.04 thousand rows/s., 90.95 MB/s.)
```

这里只用了0.4秒就查询出了结果，对比之前的全表查询，确实快了很多。原因这里加上了主键查询，CK在FINAL之前已经筛选了数据。也就是是说，只有在主键查询的情况下，ReplacingMergeTree才会快。根据这个道理，如果我们根据ack_user去查询，因为没有使用索引，所以查询会很慢。

```sh
314d64507090 :) SELECT count() FROM alerts FINAL
                WHERE (ack_user = 'user455') AND acked
┌─count()─┐
│    4818 │
└─────────┘
1 rows in set. Elapsed: 1.675 sec. Processed 9.95 million rows, 880.17 MB (5.94 million rows/s., 525.42 MB/s.)
```

我们不能将ack_user添加到索引，因为它将破坏 ReplacingMergeTree 语义。不过，我们可以用 PREWHERE 进行一个巧妙的处理：

```sh
314d64507090 :) SELECT count() FROM alerts FINAL
                                PREWHERE (ack_user = 'user455') AND acked;
┌─count()─┐
│    4818 │
└─────────┘
1 rows in set. Elapsed: 0.465 sec. Processed 9.95 million rows, 452.80 MB (21.40 million rows/s., 973.88 MB/s.)
```

PREWHERE 是一个特别的妙招，能让 ClickHouse 以不同方式应用筛选器。通常情况下 ClickHouse 是足够智能的，可以自动将条件移动到 PREWHERE ，因此用户不必在意。

> https://clickhouse.com/docs/en/sql-reference/statements/select/prewhere
>
> prewhere是更有效地进行过滤的优化。 默认情况下，即使在 PREWHERE 子句未显式指定。 它也会自动移动 WHERE条件到prewhere阶段。 
> 使用prewhere优化，首先只读取执行prewhere表达式所需的列。 如果prewhere表达式所在的那些块 “true” ，CK才会读取运行其余查询所需的其他列。如果有很多块，prewhere表达式是 “false” 的情况下，prewhere比where查询读取更少的列，这通常允许从磁盘读取更少的数据以执行查询。
> 该子句具有与 WHERE 相同的含义，区别在于从表中读取数据。 当手动控制PREWHERE对于查询中的少数列使用的过滤条件，但这些过滤条件提供了强大的数据过滤。 这减少了要读取的数据量。

## 方案二、聚合函数

ClickHouse 支持各种聚合函数：https://clickhouse.com/docs/en/sql-reference/aggregate-functions/

依旧是上面的数据，我们使用“argMax”聚合函数执行针对"user455"的相同查询：

```sql
314d64507090 :)
SELECT count(), sum(cityHash64(*)) data FROM (
  SELECT tenant_id, alert_id, timestamp, 
         argMax(alert_data, ack_time) alert_data, 
         argMax(acked, ack_time) acked,
         max(ack_time) ack_time_,
         argMax(ack_user, ack_time) ack_user
  FROM alerts 
  GROUP BY tenant_id, alert_id, timestamp
)
WHERE tenant_id=455 AND NOT acked;

┌─count()─┬─────────────data─┐
│      55 │ 2951804477181383 │
└─────────┴──────────────────┘
1 rows in set. Elapsed: 0.095 sec. Processed 32.77 thousand rows, 36.77 MB (345.76 thousand rows/s., 388.02 MB/s.)
```

同样的结果，同样的行数，但性能是之前的 4 倍，这就是 ClickHouse 聚合的效率。

缺点在于，查询变得更加复杂。但是我们可以让它变得更简单。

请注意，当确认报警时，我们只更新以下 3 列：

> acked: 0 => 1
> ack_time: 0 => now()
> ack_user: ‘’ => ‘user1’

因此，我可以可以吧argMax修改为max函数，alert_data可以使用any函数。因此我可以改造为如下：

```sh
314d64507090 :) SELECT count(), sum(cityHash64(*)) data FROM (
                  SELECT tenant_id, alert_id, timestamp,
                    any(alert_data) alert_data,
                    max(acked) acked,
                    max(ack_time) ack_time,
                    max(ack_user) ack_user
                  FROM alerts
                  GROUP BY tenant_id, alert_id, timestamp
                )
                WHERE tenant_id=455 AND NOT acked;


┌─count()─┬─────────────data─┐
│      55 │ 2951804477181383 │
└─────────┴──────────────────┘

1 rows in set. Elapsed: 0.121 sec. Processed 32.77 thousand rows, 36.77 MB (269.79 thousand rows/s., 302.76 MB/s.)
```

因此查询变简单了，而且更快了一点！原因就在于使用“any”函数后，ClickHouse 不需要对“alert_data”列计算“max”！

## 三、AggregatingMergeTree

AggregatingMergeTree 是 ClickHouse 最强大的功能之一。

> AggregatingMergeTree：https://clickhouse.com/docs/en/engines/table-engines/mergetree-family/aggregatingmergetree/

与物化视图结合使用时，它可以实现实时数据聚合。既然我们在之前的方法中使用了聚合函数，那么能否用 AggregatingMergeTree 使其更加完善呢？实际上，这并没有什么改善。

我们一次只更新一行，所以一组只有两个要聚合。对于这种情况，AggregatingMergeTree不是最好的选择。

不过我们有个小技巧。报警总是先以非确认状态，然后再变成确认状态。用户确认报警之后，只有3列需要修改。如果我们不重复其他列的数据，就可以节省磁盘空间并提高性能吗？

我们创建一张新表来处理：

```sql
DROP TABLE alerts_amt_max;

CREATE TABLE alerts_amt_max (
  tenant_id     UInt32,
  alert_id      String,
  timestamp     DateTime Codec(Delta, LZ4),
  alert_data    SimpleAggregateFunction(max, String),
  acked         SimpleAggregateFunction(max, UInt8),
  ack_time      SimpleAggregateFunction(max, DateTime),
  ack_user      SimpleAggregateFunction(max, LowCardinality(String))
)
Engine = AggregatingMergeTree()
ORDER BY (tenant_id, timestamp, alert_id);

INSERT INTO alerts_amt_max SELECT * FROM alerts WHERE NOT acked;

INSERT INTO alerts_amt_max 
SELECT tenant_id, alert_id, timestamp,
  '' as alert_data, 
  acked, ack_time, ack_user 
FROM alerts WHERE acked;
```

对于已确认的事件，我们会插入一个空字符串，而不是“alert_data”。我们知道数据不会改变，我们只能存储一次！聚合函数将填补空白。在实际应用中，我们可以跳过所有不变的列，让它们获得默认值。

有了数据之后，我们对比一下数据大小：

```sql
314d64507090 :) SELECT
                    table,
                    sum(rows) AS r,
                    sum(data_compressed_bytes) AS c,
                    sum(data_uncompressed_bytes) AS uc,
                    uc / c AS ratio
                FROM system.parts
                WHERE active AND (database = 'test_01')
                GROUP BY table;

┌─table──────────┬───────r─┬───────────c─┬──────────uc─┬──────────────ratio─┐
│ alerts         │ 9949425 │ 10935788564 │ 10999130997 │  1.005792214491831 │
│ alerts_amt_max │ 9949425 │  5837490113 │  5925962534 │ 1.0151559007873903 │
└────────────────┴─────────┴─────────────┴─────────────┴────────────────────┘
```

对之后，我们聚合之后，我们的数据量大小可以减少一半。

再次进行数据查询：

```sql
314d64507090 :) SELECT count(), sum(cityHash64(*)) data FROM (
                   SELECT tenant_id, alert_id, timestamp,
                          max(alert_data) alert_data,
                          max(acked) acked,
                          max(ack_time) ack_time,
                          max(ack_user) ack_user
                     FROM alerts_amt_max
                   GROUP BY tenant_id, alert_id, timestamp
                )
                WHERE tenant_id=455 AND NOT acked;

┌─count()─┬─────────────data─┐
│      55 │ 2951804477181383 │
└─────────┴──────────────────┘

1 rows in set. Elapsed: 0.204 sec. Processed 81.92 thousand rows, 41.56 MB (402.04 thousand rows/s., 203.98 MB/s.)
```

按照逻辑，这里扫描的数据量会更少，但是实际处理的数据和之前差不多。

## 实时更新

ClickHouse 会尽最大努力在后台合并数据，从而删除重复的行并执行聚合。然而，有时强制合并是有意义的，例如为了释放磁盘空间。这可以通过 OPTIMIZE FINAL 语句来实现。OPTIMIZE 操作速度慢、代价高，因此不能频繁执行。让我们看看它对查询性能有什么影响。

```sql
314d64507090 :) OPTIMIZE TABLE alerts FINAL;
0 rows in set. Elapsed: 170.404 sec. 
314d64507090 :) OPTIMIZE TABLE alerts_amt_max FINAL
0 rows in set. Elapsed: 137.213 sec. 
314d64507090 :) SELECT
                                    table,
                                    sum(rows) AS r,
                                    sum(data_compressed_bytes) AS c,
                                    sum(data_uncompressed_bytes) AS uc,
                                    uc / c AS ratio
                                FROM system.parts
                                WHERE active AND (database = 'test_01')
                                GROUP BY table;
┌─table──────────┬───────r─┬──────────c─┬─────────uc─┬─────────────ratio─┐
│ alerts         │ 5000000 │ 5500565158 │ 5530017683 │ 1.005354454343144 │
│ alerts_amt_max │ 5000000 │ 5500565158 │ 5530017683 │ 1.005354454343144 │
└────────────────┴─────────┴────────────┴────────────┴───────────────────┘
```

执行 OPTIMIZE FINAL 后，两个表的行数相同，数据也相同。

| 插入后                   | 插入后 | 执行 OPTIMIZE FINAL 后 |
| ------------------------ | ------ | ---------------------- |
| ReplacingMergeTree FINAL | 0.187  | 0.070                  |
| argMax                   | 0.156  | 0.091                  |
| any/max                  | 0.141  | 0.072                  |
| AggregatingMergeTree     | 0.237  | 0.071                  |

## 结论

ClickHouse 提供了丰富的工具集来处理实时更新，如 ReplacingMergeTree、CollapsingMergeTree（本文未提及）、AggregatingMergeTree 和聚合函数。所有这些方法都具有以下三个共性：

* 通过插入新版本来“修改”数据。ClickHouse 中的插入速度非常快。
* 有一些有效的方法来模拟类似于 OLTP 数据库的更新语义。
* 然而，实际的修改并不会立即发生。

具体方法的选择取决于应用程序的用例。

* ReplacingMergeTree 是直截了当的，也是最方便的方法，但只适用于中小型的表，或者数据总是按主键查询的情况。
* 使用聚合函数可以提供更高的灵活性和性能，但需要大量的查询重写。
* AggregatingMergeTree 可以节约存储空间，只保留修改过的列。



## 参考

[【ClickHouse 技术系列】- 在 ClickHouse 中处理实时更新](https://mp.weixin.qq.com/s/MDu0cf8wu5NZwK9MRx3quw)

测试机器：

> Mac-Docker
> 2 GHz 四核Intel Core i5
> 16 GB
