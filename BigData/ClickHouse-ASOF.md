# ClickHouse-ASOF

## 一、需求概述

假设我们有两张表：诊疗(treament)和医生表(doctor)表。treament是固定增量的表，而doctor是一张会周期更新的维表。treament中有个字段是doctor_id关联doctor表主键。

这意味着如doctor里面一条数据发生变更之后，那么会导致join数据出错，比方说某一个科室医生从科室1到科室2，如果仅仅通过id处理，那么可能导致诊疗记录A从科室1变为了科室2。

因此把doctor设计成了一张拉链表，通过时间区间控制数据连接。

表结构和数据如下：

```shell
CREATE TABLE `doctor` (
  `id` int(11) NOT NULL,
  `start_date` varchar(20) COLLATE utf8_bin DEFAULT NULL,
  `end_date` varchar(20) COLLATE utf8_bin DEFAULT NULL,
  `dept` varchar(255) COLLATE utf8_bin DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin;

INSERT INTO `hbos_data_dus`.`doctor`(`id`, `start_date`, `end_date`, `dept`) VALUES (1, '2021-10-01', '2021-10-30', '科室A');
INSERT INTO `hbos_data_dus`.`doctor`(`id`, `start_date`, `end_date`, `dept`) VALUES (1, '2021-11-01', '2021-11-30', '科室B');
INSERT INTO `hbos_data_dus`.`doctor`(`id`, `start_date`, `end_date`, `dept`) VALUES (1, '2021-12-01', '2021-12-30', '科室C');
INSERT INTO `hbos_data_dus`.`doctor`(`id`, `start_date`, `end_date`, `dept`) VALUES (2, '2021-10-01', '2021-10-30', '科室BB');
INSERT INTO `hbos_data_dus`.`doctor`(`id`, `start_date`, `end_date`, `dept`) VALUES (2, '2021-11-01', '2021-12-30', '科室CC');

CREATE TABLE `treament` (
  `id` int(11) NOT NULL,
  `doctor_id` int(11) DEFAULT NULL,
  `treament` varchar(255) COLLATE utf8_bin DEFAULT NULL,
  `date` varchar(20) COLLATE utf8_bin DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin;

INSERT INTO `hbos_data_dus`.`treament`(`id`, `doctor_id`, `treament`, `date`) VALUES (1, 1, '哈哈哈', '2021-10-1');
INSERT INTO `hbos_data_dus`.`treament`(`id`, `doctor_id`, `treament`, `date`) VALUES (2, 1, '哈哈哈2', '2021-11-12');
INSERT INTO `hbos_data_dus`.`treament`(`id`, `doctor_id`, `treament`, `date`) VALUES (3, 1, '大大', '2021-12-20');
INSERT INTO `hbos_data_dus`.`treament`(`id`, `doctor_id`, `treament`, `date`) VALUES (4, 2, '123', '2021-10-1');
INSERT INTO `hbos_data_dus`.`treament`(`id`, `doctor_id`, `treament`, `date`) VALUES (5, 2, '哈哈3', '2021-12-20');
```

## 二、MySQL处理方案

JOIN连接表的时候，除了id关联之外，增加时间的连接条件。

```sql
SELECT
	doctor.dept,
	count( treament.id ) AS _cnt 
FROM
	treament
	LEFT JOIN doctor ON treament.doctor_id = doctor.id 
	AND treament.date >= doctor.start_date 
	AND treament.date <= doctor.end_date 
GROUP BY
	doctor.dept
ORDER BY
	_cnt DESC;

  
SELECT
	doctor.dept,
	count( treament.id ) AS _cnt 
FROM
	treament
	LEFT  JOIN doctor ON treament.doctor_id = doctor.id 
	WHERE treament.date >= doctor.start_date 
	AND treament.date <= doctor.end_date 
GROUP BY
	doctor.dept
ORDER BY
	_cnt DESC;
  
-- 两种方法理论上都可以，目测前面这个性能高点。
```

数据结果：

![img](http://img.hurenjieee.com/uPic/1637289480359-96d8d11b-41ac-4023-b648-3135140d05be.png)

## 三、ClickHouse处理方案

CK先建表：

```sql
CREATE TABLE doctor
(
    `id` Int32,
    `start_date` String,
    `end_date` String,
    `dept` String
)
ENGINE = MergeTree
ORDER BY id
SETTINGS index_granularity = 8192;

CREATE TABLE treament
(
    `id` Int32,
    `doctor_id` Int32,
    `treament` String,
    `date` String
)
ENGINE = MergeTree
ORDER BY id
SETTINGS index_granularity = 8192;
```

考虑不同数据对SQL的兼容性，同样的SQL在CK也试下：

提示：

```shell
SQL 错误 [48]: ClickHouse exception, code: 48, host: 127.0.0.1, port: 8123; Code: 48, e.displayText() = DB::Exception: JOIN ON inequalities are not supported. Unexpected 'date >= start_date': While processing date >= start_date (version 21.6.4.26 (official build))

第二个可以执行。
```

可以看到关键字【JOIN ON inequalities】->非相等JOIN不能连接；

转到CK官方文档搜索一下相关内容

![img](http://img.hurenjieee.com/uPic/1637289817574-78f9331d-da86-4ce1-b707-ff6c10f919bf.png)

分别看下两个内容：

![img](http://img.hurenjieee.com/uPic/1637289948851-28f4d797-e30f-40c9-bf10-ddf39110ab86.png)

![img](http://img.hurenjieee.com/uPic/1637290072775-ac9c397b-29a9-43a1-8247-e2853a80ca51.png)

看到有一个ASOF JOIN连接操作：



![img](http://img.hurenjieee.com/uPic/1637293036356-bdea7038-7f7a-4aa9-a889-19718435dbad.png)

这里我们得到一个新的Sql

```sql
SELECT
	doctor.dept ,
	count(treament.id) as _cnt
FROM
	treament ASOF
LEFT JOIN doctor ON
	treament.doctor_id = doctor.id
AND toUInt64(treament.date) <= toUInt64(doctor.end_date)
GROUP BY doctor.dept;
```



## 四、性能测试

经过上述测试，我们可以通过两种方法处理：

Mysql：

- JOIN表条件增加区间判断
- WHERE条件额外过滤

ClickHouse（2C4U）：

- 使用 ASOF JOIN表条件增加区间判断
- WHERE条件额外过滤



下面进行相关的CK相关的性能测试：

```sql
-- 删除数据
ALTER table doctor delete  where 1 = 1;
ALTER table treament delete  where 1 = 1;

-- 插入模拟数据
INSERT INTO `doctor`(`id`, `start_date`, `end_date`, `dept`) VALUES (1, '20211001', '20211030', '科室A');
INSERT INTO `doctor`(`id`, `start_date`, `end_date`, `dept`) VALUES (1, '20211101', '20211130', '科室B');
INSERT INTO `doctor`(`id`, `start_date`, `end_date`, `dept`) VALUES (1, '20211201', '20211230', '科室C');
INSERT INTO `doctor`(`id`, `start_date`, `end_date`, `dept`) VALUES (2, '20211001', '20211030', '科室BB');
INSERT INTO `doctor`(`id`, `start_date`, `end_date`, `dept`) VALUES (2, '20211101', '20211230', '科室CC');

INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) VALUES (1, 1, '哈哈哈', '20211001');
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) VALUES (2, 1, '哈哈哈2', '20211112');
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) VALUES (3, 1, '大大', '20211220');
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) VALUES (4, 2, '123', '20211001');
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) VALUES (5, 2, '哈哈3', '20211220');

-- 数据复制，保证id可关联
INSERT INTO `doctor`(`id`, `start_date`, `end_date`, `dept`) select id+2, start_date ,end_date ,randomPrintableASCII(1) from doctor;
INSERT INTO `doctor`(`id`, `start_date`, `end_date`, `dept`) select id+4, start_date ,end_date ,randomPrintableASCII(1) from doctor;
INSERT INTO `doctor`(`id`, `start_date`, `end_date`, `dept`) select id+8, start_date ,end_date ,randomPrintableASCII(1) from doctor;
INSERT INTO `doctor`(`id`, `start_date`, `end_date`, `dept`) select id+16, start_date ,end_date ,randomPrintableASCII(1) from doctor;
INSERT INTO `doctor`(`id`, `start_date`, `end_date`, `dept`) select id+32, start_date ,end_date ,randomPrintableASCII(1) from doctor;
INSERT INTO `doctor`(`id`, `start_date`, `end_date`, `dept`) select id+64, start_date ,end_date ,randomPrintableASCII(1) from doctor;
INSERT INTO `doctor`(`id`, `start_date`, `end_date`, `dept`) select id+128, start_date ,end_date ,randomPrintableASCII(1) from doctor;
INSERT INTO `doctor`(`id`, `start_date`, `end_date`, `dept`) select id+256, start_date ,end_date ,randomPrintableASCII(1) from doctor;
INSERT INTO `doctor`(`id`, `start_date`, `end_date`, `dept`) select id+512, start_date ,end_date ,randomPrintableASCII(1) from doctor;
INSERT INTO `doctor`(`id`, `start_date`, `end_date`, `dept`) select id+1024, start_date ,end_date ,randomPrintableASCII(1) from doctor;
INSERT INTO `doctor`(`id`, `start_date`, `end_date`, `dept`) select id+2048, start_date ,end_date ,randomPrintableASCII(1) from doctor;
INSERT INTO `doctor`(`id`, `start_date`, `end_date`, `dept`) select id+4096, start_date ,end_date ,randomPrintableASCII(1) from doctor;

INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) select rand(), doctor_id+2 ,randomPrintableASCII(2) ,date from treament;
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) select rand(), doctor_id+4 ,randomPrintableASCII(2) ,date from treament;
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) select rand(), doctor_id+8 ,randomPrintableASCII(2) ,date from treament;
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) select rand(), doctor_id+16 ,randomPrintableASCII(2) ,date from treament;
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) select rand(), doctor_id+32,randomPrintableASCII(2) ,date from treament;
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) select rand(), doctor_id+64 ,randomPrintableASCII(2) ,date from treament;
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) select rand(), doctor_id+128,randomPrintableASCII(2) ,date from treament;
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) select rand(), doctor_id+256,randomPrintableASCII(2) ,date from treament;
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) select rand(), doctor_id+512,randomPrintableASCII(2) ,date from treament;
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) select rand(), doctor_id+1024,randomPrintableASCII(2) ,date from treament;
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) select rand(), doctor_id+2048,randomPrintableASCII(2) ,date from treament;
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) select rand(), doctor_id+4096,randomPrintableASCII(2) ,date from treament;

INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) select rand(), doctor_id ,randomPrintableASCII(2) ,date from treament;
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) select rand(), doctor_id ,randomPrintableASCII(2) ,date from treament;
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) select rand(), doctor_id ,randomPrintableASCII(2) ,date from treament;
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) select rand(), doctor_id ,randomPrintableASCII(2) ,date from treament;
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) select rand(), doctor_id ,randomPrintableASCII(2) ,date from treament;
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) select rand(), doctor_id ,randomPrintableASCII(2) ,date from treament;
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) select rand(), doctor_id ,randomPrintableASCII(2) ,date from treament;


select count(*) from doctor;
-- 20480
select count(*) from treament;
-- 2621440

SELECT
	doctor.dept ,
	count(treament.id) as _cnt
FROM
	treament ASOF
LEFT JOIN doctor ON
	treament.doctor_id = doctor.id
AND toUInt64(treament.date) <= toUInt64(doctor.end_date)
GROUP BY doctor.dept;

-- 600-700ms


SELECT
	doctor.dept,
	count( treament.id ) AS _cnt 
FROM
	treament
	LEFT  JOIN doctor ON treament.doctor_id = doctor.id 
	WHERE treament.date >= doctor.start_date 
	AND treament.date <= doctor.end_date 
GROUP BY
	doctor.dept
ORDER BY
	_cnt DESC;
  
 -- 1100-1400ms
```



总体对比下来，使用 ASOF LEFT JOIN在对查询性能上会有比较大的提升。



另外，对比了 ASOF LEFT JOIN和LEFT JOIN的性能

```sql
SELECT
	doctor.dept ,
	count(treament.id) as _cnt
FROM
	treament
LEFT JOIN doctor ON
	treament.doctor_id = doctor.id
-- AND toUInt64(treament.date) <= toUInt64(doctor.end_date)
GROUP BY doctor.dept;
-- 300-400ms
```

对比下来，因为加了非相等的条件匹配，必然会增加计算量，总体数据延迟在25-50%之间。



同样的，Mysql也测试下，因为两个数据库不同，因此我们需要调整一下插入语句：

```sql
INSERT INTO `doctor`(`id`, `start_date`, `end_date`, `dept`) VALUES (1, '20211001', '20211030', '科室A');
INSERT INTO `doctor`(`id`, `start_date`, `end_date`, `dept`) VALUES (1, '20211101', '20211130', '科室B');
INSERT INTO `doctor`(`id`, `start_date`, `end_date`, `dept`) VALUES (1, '20211201', '20211230', '科室C');
INSERT INTO `doctor`(`id`, `start_date`, `end_date`, `dept`) VALUES (2, '20211001', '20211030', '科室BB');
INSERT INTO `doctor`(`id`, `start_date`, `end_date`, `dept`) VALUES (2, '20211101', '20211230', '科室CC');

INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) VALUES (1, 1, '哈哈哈', '20211001');
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) VALUES (2, 1, '哈哈哈2', '20211112');
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) VALUES (3, 1, '大大', '20211220');
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) VALUES (4, 2, '123', '20211001');
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) VALUES (5, 2, '哈哈3', '20211220');

INSERT INTO `doctor`(`id`, `start_date`, `end_date`, `dept`) select id+2, start_date ,end_date ,'科室'+FLOOR(id+5 * 100) from doctor;
INSERT INTO `doctor`(`id`, `start_date`, `end_date`, `dept`) select id+4, start_date ,end_date ,'科室'+FLOOR(id+5 * 100) from doctor;
INSERT INTO `doctor`(`id`, `start_date`, `end_date`, `dept`) select id+8, start_date ,end_date ,'科室'+FLOOR(id+5 * 100) from doctor;
INSERT INTO `doctor`(`id`, `start_date`, `end_date`, `dept`) select id+16, start_date ,end_date ,'科室'+FLOOR(id+5 * 100) from doctor;
INSERT INTO `doctor`(`id`, `start_date`, `end_date`, `dept`) select id+32, start_date ,end_date ,'科室'+FLOOR(id+5 * 100) from doctor;
INSERT INTO `doctor`(`id`, `start_date`, `end_date`, `dept`) select id+64, start_date ,end_date ,'科室'+FLOOR(id+5 * 100) from doctor;
INSERT INTO `doctor`(`id`, `start_date`, `end_date`, `dept`) select id+128, start_date ,end_date ,'科室'+FLOOR(id+5 * 100) from doctor;
INSERT INTO `doctor`(`id`, `start_date`, `end_date`, `dept`) select id+256, start_date ,end_date ,'科室'+FLOOR(id+5 * 100) from doctor;
INSERT INTO `doctor`(`id`, `start_date`, `end_date`, `dept`) select id+512, start_date ,end_date ,'科室'+FLOOR(id+5 * 100) from doctor;
INSERT INTO `doctor`(`id`, `start_date`, `end_date`, `dept`) select id+1024, start_date ,end_date ,'科室'+FLOOR(id+5 * 100) from doctor;
INSERT INTO `doctor`(`id`, `start_date`, `end_date`, `dept`) select id+2048, start_date ,end_date ,'科室'+FLOOR(id+5 * 100) from doctor;
INSERT INTO `doctor`(`id`, `start_date`, `end_date`, `dept`) select id+4096, start_date ,end_date ,'科室'+FLOOR(id+5 * 100) from doctor;


INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) select id+5, doctor_id+2 ,'科室'+FLOOR(id+5 * 100) ,date from treament;
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) select id+10, doctor_id+4 ,'科室'+FLOOR(id+5 * 100) ,date from treament;
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) select id+20, doctor_id+8 ,'科室'+FLOOR(id+5 * 100) ,date from treament;
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) select id+40, doctor_id+16 ,'科室'+FLOOR(id+5 * 100) ,date from treament;
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) select id+80, doctor_id+32,'科室'+FLOOR(id+5 * 100) ,date from treament;
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) select id+160, doctor_id+64 ,'科室'+FLOOR(id+5 * 100) ,date from treament;
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) select id+320, doctor_id+128,'科室'+FLOOR(id+5 * 100) ,date from treament;
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) select id+640, doctor_id+256,'科室'+FLOOR(id+5 * 100) ,date from treament;
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) select id+1280, doctor_id+512,'科室'+FLOOR(id+5 * 100) ,date from treament;
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) select id+2560, doctor_id+1024,'科室'+FLOOR(id+5 * 100) ,date from treament;
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) select id+5120, doctor_id+2048,'科室'+FLOOR(id+5 * 100) ,date from treament;
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) select id+10240, doctor_id+4096,'科室'+FLOOR(id+5 * 100) ,date from treament;
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) select id+10240*2, doctor_id ,'科室'+FLOOR(id+5 * 100) ,date from treament;
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) select id+10240*4, doctor_id ,'科室'+FLOOR(id+5 * 100) ,date from treament;
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) select id+10240*8, doctor_id ,'科室'+FLOOR(id+5 * 100) ,date from treament;
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) select id+10240*16, doctor_id ,'科室'+FLOOR(id+5 * 100) ,date from treament;
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) select id+10240*32, doctor_id ,'科室'+FLOOR(id+5 * 100) ,date from treament;
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) select id+10240*64, doctor_id ,'科室'+FLOOR(id+5 * 100) ,date from treament;
INSERT INTO `treament`(`id`, `doctor_id`, `treament`, `date`) select id+10240*128, doctor_id ,'科室'+FLOOR(id+5 * 100) ,date from treament;




select count(*) from doctor;
-- 20480
select count(*) from treament;
-- 2621440

SELECT
	doctor.dept,
	count( treament.id ) AS _cnt 
FROM
	treament
	LEFT JOIN doctor ON treament.doctor_id = doctor.id 
	AND treament.date >= doctor.start_date 
	AND treament.date <= doctor.end_date 
GROUP BY
	doctor.dept
ORDER BY
	_cnt DESC;
-- 200s无结果
  
SELECT
	doctor.dept,
	count( treament.id ) AS _cnt 
FROM
	treament
	LEFT  JOIN doctor ON treament.doctor_id = doctor.id 
	WHERE treament.date >= doctor.start_date 
	AND treament.date <= doctor.end_date 
GROUP BY
	doctor.dept
ORDER BY
	_cnt DESC;
-- 100s无结果
```

事实证明，这种大数据量的JOIN mysql很难支