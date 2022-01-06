Mysql Explain

参考官网https://dev.mysql.com/doc/refman/5.7/en/explain-output.html食用

# 一、语法

```sql
EXPLAIN [SQL]
```

```sql
mysql> EXPLAIN SELECT * FROM good;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------+
|  1 | SIMPLE      | good  | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    3 |   100.00 | NULL  |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------+
```



# 二、字段含义

## 2.1 id

SQL的解释顺序从小打大，执行顺序从大到小。

```sql
mysql> EXPLAIN SELECT * FROM good WHERE id IN (SELECT good_id FROM `order`);
+----+--------------+-------------+------------+--------+---------------+---------+---------+---------------------+------+----------+-------------+
| id | select_type  | table       | partitions | type   | possible_keys | key     | key_len | ref                 | rows | filtered | Extra       |
+----+--------------+-------------+------------+--------+---------------+---------+---------+---------------------+------+----------+-------------+
|  1 | SIMPLE       | <subquery2> | NULL       | ALL    | NULL          | NULL    | NULL    | NULL                | NULL |   100.00 | Using where |
|  1 | SIMPLE       | good        | NULL       | eq_ref | PRIMARY       | PRIMARY | 8       | <subquery2>.good_id |    1 |   100.00 | NULL        |
|  2 | MATERIALIZED | order       | NULL       | ALL    | NULL          | NULL    | NULL    | NULL                |    1 |   100.00 | NULL        |
+----+--------------+-------------+------------+--------+---------------+---------+---------+---------------------+------+----------+-------------+
```

## 2.2 select_type

1. SIMPLE：简单的SELECT(不使用UNION或子查询)
2. PRIMARY
   * PRIMARY:查询中最外层的SELECT（如两表做UNION或者存在子查询的外层的表操作为PRIMARY,内层的操作为UNION）
3. UNION/DEPENDENT UNION/UNIOIN RESULT
   * UNION:UNION操作中,查询中处于内层的SELECT（内层的SELECT语句与外层的SELECT语句没有依赖关系）
   * DEPENDENT UNION：UNION操作中,查询中处于内层的SELECT(内层的SELECT语句与外层的SELECT语句有依赖关系)
   * UNION RESULT：UNION的结果，id值通常为NULL
4. SUBQUERY/DEPENDENT SUBQUERY
   * SUBQUERY：子查询中首个SELECT(如果有多个子查询存在):
   * DEPENDENT SUBQUERY：子查询中首个SELECT,但依赖于外层的表(如果有多个子查询存在)
5. DERIVED/MATERIALIZED
   * DERIVED:被驱动的SELECT子查询(子查询位于FROM子句)
   * MATERIALIZED:被物化的子查询
6. UNCACHEABLE SUBQUERY/UNCACHEABLE UNION
   * UNCACHEABLE SUBQUERY:对于外层的主表,子查询不可被物化,每次都需要计算(耗时操作)
   * UNCACHEABLE UNION:UNION操作中,内层的不可被物化的子查询(类似于UNCACHEABLE SUBQUERY)

## 2.3 table

显示这一行的数据是关于哪张表的。有时不是真实的表名字,看到的是derivedx(x是个数字,我的理解是第几步执行的结果)或者subquery。

## 2.4 partitions

## 2.5 type

表示MySQL在表中找到所需行的方式，又称“访问类型”。
常用的类型有： **ALL, index, range, ref, eq_ref, const, system, NULL（从左到右，性能从差到好）**

1. **ALL**：Full Table Scan， 全表扫描
2. **index**：Full Index Scan，索引全扫描，index与ALL区别为index类型只遍历索引树
3. **range**：索引范围扫描，只检索给定范围的行，使用一个索引来选择行，常用语<,<=,>=,between等操作。
4. **ref**：使用非唯一索引扫描或者唯一索引前缀扫描，返回单条记录，常出现在关联查询中
5. **eq_ref**：类似ref，区别就是使用的索引的唯一索引，对于每个索引键值，表中只有一条记录匹配；简单来说，就是夺标连接中使用primary key或者unique key作为关联条件。
6. **const、system**：单挑记录，系统会把匹配行中的其他键作为常数处理，如主键或唯一索引查询
7. **NULL**:Mysql不访问任何表或索引，直接返回结果。

## 2.6 possible_keys

该查询可以利用的索引. 如果没有任何索引可以使用，就会显示成null。

**这一项内容对于优化时候索引的调整非常重要。**

## 2.7 key

MySQL Query Optimizer 从possible_keys 中所选择的实际使用的索引；如果没有选择索引，键是NULL。
要想强制MySQL使用或忽视possible_keys列中的索引，在查询中使用FORCE INDEX、USE INDEX或者IGNORE INDEX。

## 2.8 key_len

表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度（key_len显示的值为索引字段的最大可能长度，并非实际使用长度，即key_len是根据表定义计算而得，不是通过表内检索出的）。
不损失精确性的情况下，长度越短越好。

## 2.9 ref

如果是使用的常数等值查询，这里会显示const，如果是连接查询，被驱动表的执行计划这里会显示驱动表的关联字段，如果是条件使用了表达式或者函数，或者条件列发生了内部隐式转换，这里可能显示为func。

## 2.10 rows

 表示MySQL根据表统计信息及索引选用情况，**估算**的找到所需的记录所需要读取的行数

## 2.11 filtered

Filtered表示返回结果的行数占需读取行数的百分比 Filtered列的值越大越好 Filtered列的值依赖于统计信息。

## 2.12 Extra

该列包含MySQL解决查询的详细信息：

1. Using where

   SQL使用了where条件过滤数据。

2. Using index

   SQL所需要返回的所有列数据均在一棵索引树上，而无需访问实际的行记录。

3. Using index condition

   确实命中了索引，但不是所有的列数据都在索引树上，还需要访问实际的行记录。

4. **Using filesort**

   得到所需结果集，需要对所有记录进行文件排序。

5. **Using temporary**

   需要建立临时表(temporary table)来暂存中间结果。

6. **Using join buffer (Block Nested Loop)**

   进行嵌套循环计算

   

# 三、总结

**explain是SQL优化中最常用的工具，搞定type和Extra，explain也就基本搞定了。**



### 参考链接

1. [Mysql Explain官网](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_rows)
2. [Mysql Explain 详解](http://www.cnitblog.com/aliyiyi08/archive/2008/09/09/48878.html)
3. [MySQL优化常见Extra分析——慢查询优化](https://www.cnblogs.com/linjiqin/p/11254247.html)