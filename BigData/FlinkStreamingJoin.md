# Flink流JOIN

## 一、概述

### 1.1 什么是JOIN

JOIN的本质是分别重N（N>=1）张表中获取不同的字段，进而得到最完整的记录。比如我们有一个查询需求：在学生表（学号、姓名、性别），课程表（课程号、课程名、学分）和成绩表（学号、课程号、分数）中查询所有学生的姓名，课程和考试分数。如下：

![img](http://img.hurenjieee.com/uPic/640.jpg)

### 1.2 为啥需要JOIN

JOIN的本质是数据拼接，那么如果我们将所有的数据都存储在一张大表中，是不是就不需要JOIN了呢？答案确实是不需要JOIN了。但是现实业务中真的能做到所需的数据放到同一张大表里面吗？答案是否定的，核心原因有2个：

* 产生数据的源头坑你不是一个系统；
* 产生数据的源头是同一个系统，但是数据冗余的沉重代价，迫使我们会遵循数据库范式，进行表的设计。简说NF如下：
  * 1NF：列不可拆分
  * 2NF：符合1NF，并且非主键属性全部依赖于主键属性
  * 3NF：符合2NF，并且传递依赖，即：即任何字段不能由其他字段派生出来
  * BCNF：符合3NF，并且主键属性之间无依赖关系

### 1.3 JOIN的种类

* CROSS JOIN：交叉连接，计算笛卡尔积
* INNER JOIN：内连接，返回满足条件的记录
* OUTER JOIN
  * LEFT OUTER JOIN：返回左表所有行，右表不存在补NULL
  * RIGHT OUTER JOIN：返回右边所有行，左表不存在补NULL
  * FULL OUTER JOIN：返回左表和右表的并集，不存在一边补NULL
* SELF JOIN：自连接，将表查询时候明明不同的别名。

## 二、Flink 双流JOIN

Flink对流JOIN支持

* CROSS JOIN：不支持
* INNER JOIN：支持
* OUTER JOIN：支持
* SELF JOIN：支持
* ON（Condition）：必选
* WHERE：可选



Flink目前支持INNER JOIN和LEFT OUTER JOIN（SELF可以转换为普通为INNER和OUTER）。



### 2.1 双流JOIN与传统数据库表JOIN的区别

传统数据库表的JOIN是两张静态表的数据联接，而在流上面是动态表，双流JOIN的数据不断流入。两者的主要区别如下：

|                            | 传统数据库                   | 双流数据库                                                   |
| -------------------------- | ---------------------------- | ------------------------------------------------------------ |
| 左右两边的数据集合是否无穷 | 左右两张表的数据集合是有限的 | 双流JOIN的数据会源源不断地流入                               |
| JOIN的结果不断产生/更新    | 一次执行产生最终结果         | 双流JOIN会持续不断产生新的结果                               |
| 查询计算的双边驱动         | 两边加载完成之后处理         | 由于左右两边的流的 速度不一样，会导致左右两边的数据到达时间不一致。在实现上，需要将左右两边的数据进行保存，以保证JOIN的语义。Flink中，会以State的方式进行行数据的存储。 |

### 2.2 数据Shuffle

分布式流计算所有数据都会进行Shuffle，怎么才能保障左右两边流的要JOIN的数据会在相同的节点进行处理呢？在双流JOIN的场景，我们会利用JOIN中的ON的联接key进行partition，确保两个流相同的联接key会在同一个节点处理。

### 2.3  数据保存

不论是INNER JOIN还是OUTER JOIN都需要对左右两边的流数据进行保存，JOIN算子会开辟左右两个State进行数据存储，左右两边的数据到来时候，进行如下操作：

* LeftEvent到来存储到LState，RightEvent到来的时候存储到RState。
* LeftEvent会去RightState进行JOIN，并发出所有JOIN之后的Event到下游。
* RightEvent会去LeftState进行JOIN，并发出所有JOIN之后的Event到下游。

![img](https://mmbiz.qpic.cn/mmbiz_png/7WvuibJnicwp8XgXibrOYdK1VhibQ3MlGVJ3icS2gE3WJgrFcwI0QsrUpsg1iaxxSnE8uHQBbfEeStAhwv1mM79PJE1A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)





## 参考

Apache Flink 漫谈系列 - 双流JOIN：https://mp.weixin.qq.com/s/BiO4Ba6wRH4tlTdT2w7fzw