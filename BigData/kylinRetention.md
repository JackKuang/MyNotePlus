# Kylin留存率

## 一、留存率

公司为了拓展用户，结合各个场景下设置了推广页面，然后交由不同的渠道商去做广告投放。

然后根据广告投放的结果、转换率、留存率等数据进行渠道流量结算，一旦结算机制不够完善，就很可能被渠道商褥羊毛，这个时候，就需要对各种数据进行完整的判断。

1. 渠道流量引入 -> 下载 -> 注册
2. 注册 -> 下单 -> 支付

其中，路径1是用户手机号去重，路径2时用户ID去重复。

那么问题来了，如何做到精确分析，因为前后关联的数据时关联在uid上。也就时说，如果单纯在sql表上做留存分析，那么sql会是这样子的

```sql
select a.*
from (
	select * from log_table where log_type = '下载' and log_type = '注册'
) a
inner join (
	select uid from log_table where log_type = '渠道流量引入')
) b on a.uid = b.uid
```

这个sql在某种程度上是可行的，但是一个很大的问题，就是当a,b两张表都特别大的时候，也就是大表join大表的时候，查询性能会很差。可以优化sql进行分析。

```sql
select * from log_table where log_type = '下载' and log_type = '注册' and uid in 
 ( select uid from log_table where log_type = '渠道流量引入')
```

当然，对于Hive Sql来说，第二个sql执行效率上会更好。

## 二、Kylin如何实现精准留存分析

显然，直接通过sql计算的方式并不适合处理数据。那么就需要一个更好的处理方案。

下面我门介绍Kylin是如何处理这个问题的。

首先，Kylin社区有一篇技术博文：https://kylin.apache.org/blog/2016/11/28/intersect-count/，这篇文章详解介绍了Kylin的留存、转化计算。此类运算基于Bitmap和UDAF的intersect_count函数。

* Bitmap

  > Kylin本身的精确去虫就是基于Bitmap算法的

* inertsect_count函数

  > Intersect_count(columnToCount,columnToFilter,filterValueList)
  >
  > columnToCount:去重统计字段
  >
  > columnToFilter:去重过滤字段
  >
  > filterValueList:去重过滤字段对应的值
  >
  > * Intersect_count(uid,dt,array['20161014','20161015']):统计20161014和20161015同时存在的uid精确去重
  > * Intersect_count(uid,dt,array['20161014']):统计20161014存在的uid精确去重