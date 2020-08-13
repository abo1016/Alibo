---
title: mysql 分区表
tags: []
id: '199'
categories:
  - - all-blog
    - mysql
date: 2018-06-27 15:13:08
---

> ### mysql 分区表

#### 什么是分区表

**单表数据到了几百万甚至上千万时，会使得数据库的查询效率变得很低，有复杂的联合查询时可能会导致mysql进程挂机或宕机。**此时通过分表或者分区将一张数据表中的数据分割成多份小的数据，本次要讲的是**分区**。

> > 从Mysql5.1之后，分区功能出现了，表分区就像是将一个大表分成了若干个小表，用户在执行查询的时候无需进行全表扫描，只需要对满足要求的表分区中进行查询即可，极大的提高了查询速率，另外，表分区的实现也方便了对数据的管理，比如产品需要删除去年的所有数据，那么只需要将去年数据所在的表分区删除即可。

#### 常见的分区类型

> **RANGE**分区：基于属于一个给定连续区间的列值，把多行分配给分区。 **LIST**分区：类似于按RANGE分区，区别在于LIST分区是基于列值匹配一个离散值集合中的某个值来进行选择。 **HASH**分区：基于用户定义的表达式的返回值来进行选择的分区，该表达式使用将要插入到表中的这些行的列值进行计算。这个函数可以包含MySQL 中有效的、产生非负整数值的任何表达式。 **KEY**分区：类似于按HASH分区，区别在于KEY分区只支持计算一列或多列，且MySQL服务器提供其自身的哈希函数。必须有一列或多列包含整数值。

**注意：**所有的表分区使用的列均需要使用源表中存在的主键或者唯一索引列，否则创建失败，如果源表中本来就不存在任何的主键或者唯一索引列，那么可以在分区的时候根据需要选取任意列。

> > RANGE：顾名思义，通过确定选取列的值的范围的方式进行分区。 如下是创建普通表的语句： 为了实验的方便，此处date 字段使用的时间类型为：DATETIME,而非TIMESTAMP，原因是TIMESTAMP不支持在分区的时候使用YEAR(),MONTH(),TO\_DAYS()等操作，只能使用UNIX\_TIMSTAMP()函数，所以在设计表的时候需要考虑到这点

```sql
CREATE  TABLE t1  ( id INT, date DATETIME DEFAULT CURRENT_TIMESTAMP) ENGINE=Innodb;
```

插入测试数据

```sql
mysql> SELECT * FROM t1;
+------+---------------------+
| id   | date                |
+------+---------------------+
|    1 | 2013-05-23 12:59:39 |
|    2 | 2013-05-23 12:59:43 |
|    3 | 2013-05-23 12:59:44 |
|    4 | 2013-07-04 19:35:45 |
|    5 | 2014-04-04 19:35:45 |
|    6 | 2014-05-04 19:35:45 |
|    7 | 2015-05-04 19:35:45 |
|    8 | 2015-05-05 19:35:45 |
|    9 | 2017-05-05 19:35:45 |
|   10 | 2018-05-05 19:35:45 |
+------+---------------------+
10 rows in set (0.00 sec)
```

查询 结果为全表扫描

```sql
mysql> EXPLAIN SELECT * FROM t1;
+----+-------------+-------+------+---------------+------+---------+------+------+-------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra |
+----+-------------+-------+------+---------------+------+---------+------+------+-------+
|  1 | SIMPLE      | t1    | ALL  | NULL          | NULL | NULL    | NULL |   10 | NULL  |
+----+-------------+-------+------+---------------+------+---------+------+------+-------+
1 row in set (0.00 sec)
mysql> EXPLAIN SELECT * FROM t1 WHERE date >= '2014-03-05 19:00:12'
    -> AND date <= '2016-03-05 18:45:12';
+----+-------------+-------+------+---------------+------+---------+------+------+-------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra       |
+----+-------------+-------+------+---------------+------+---------+------+------+-------------+
|  1 | SIMPLE      | t1    | ALL  | NULL          | NULL | NULL    | NULL |   10 | Using where |
+----+-------------+-------+------+---------------+------+---------+------+------+-------------+
1 row in set (0.00 sec)
```

> **进行RANGE分区**

```sql
mysql> ALTER  TABLE t1  PARTITION BY RANGE (YEAR(date)) (
    ->     PARTITION p2013 VALUES LESS THAN(2014),
    ->     PARTITION p2014 VALUES LESS THAN(2015),
    ->     PARTITION p2015 VALUES LESS THAN(2016),
    ->     PARTITION p2016 VALUES LESS THAN(2017),
    ->     PARTITION p2017 VALUES LESS THAN(2018),
    ->     PARTITION p2099 VALUES LESS THAN MAXVALUE
    -> ) ;
```

查看数据分区状态：

```sql
mysql> SELECT table_name,partition_name,table_rows FROM information_schema.PARTITIONS  WHERE  table_schema=database() AND table_name='t1';
+------------+----------------+------------+
| table_name | partition_name | table_rows |
+------------+----------------+------------+
| t2         | p2013          |          4 |
| t2         | p2014          |          2 |
| t2         | p2015          |          2 |
| t2         | p2016          |          0 |
| t2         | p2017          |          1 |
| t2         | p2099          |          1 |
+------------+----------------+------------+
```

where 查询

```sql
mysql> EXPLAIN PARTITIONS SELECT * FROM t2 WHERE date >= '2014-03-05 19:00:12' AND date <= '2016-03-05 18:45:12';
+----+-------------+-------+-------------------+------+---------------+------+---------+------+------+-------------+
| id | select_type | table | partitions        | type | possible_keys | key  | key_len | ref  | rows | Extra       |
+----+-------------+-------+-------------------+------+---------------+------+---------+------+------+-------------+
|  1 | SIMPLE      | t2    | p2014,p2015,p2016 | ALL  | NULL          | NULL | NULL    | NULL |    5 | Using where |
+----+-------------+-------+-------------------+------+---------------+------+---------+------+------+-------------+
```

> **hash分区** 测试数据量 310万+

未分区前查询： ![PPI4UJ.png](https://s1.ax1x.com/2018/06/27/PPI4UJ.png) **分区** hash规则：通过将整张数据表中的数据分成为7个分区，对于每一天的数据记录分成一个区。在查询数据时 通过取模方法获取所查数据的分区名（part） ，进而指定分区（hash分区后，若不指定分区查询 还是对数据进行全部分区扫描 ，分区的意义也就不在）进行查询。

```sql
alter table camera_alarm partition by hash(to_days(time)) partitions 7;
```

```sql
mysql> SELECT  
    ->   partition_name part,   
    ->   partition_expression expr,   
    ->   table_rows   
    -> FROM  
    ->   INFORMATION_SCHEMA.partitions   
    -> WHERE  
    ->   TABLE_SCHEMA = schema()   
    ->   AND TABLE_NAME='camera_alarm';
+------+---------------+------------+
| part | expr          | table_rows |
+------+---------------+------------+
| p0   | to_days(time) |     528175 |
| p1   | to_days(time) |     549812 |
| p2   | to_days(time) |     144302 |
| p3   | to_days(time) |     493172 |
| p4   | to_days(time) |     493332 |
| p5   | to_days(time) |     494127 |
| p6   | to_days(time) |     512142 |
+------+---------------+------------+
7 rows in set (0.00 sec)
```

**分区后**指定分区查询：

```sql
mysql> select * from camera_alarm PARTITION (p4) where time <= '2018-03-21 23:59:59' and time > '2018-03-21 00:00:00';

mysql> show profiles;
+----------+------------+------------------------------------------------------------------------------------------------------------------------+
| Query_ID | Duration   | Query                                                                                                                  |
+----------+------------+------------------------------------------------------------------------------------------------------------------------+
|        1 |  0.0011805 | explain select * from camera_alarm PARTITION (p4) where time <= '2018-03-21 23:59:59' and time > '2018-03-21 00:00:00' |
|        2 | 1.34151975 | select * from camera_alarm PARTITION (p4) where time <= '2018-03-21 23:59:59' and time > '2018-03-21 00:00:00'         |
+----------+------------+------------------------------------------------------------------------------------------------------------------------+
2 rows in set
```

**分区后指定分区进行查询效率提高超100%**

> > list和key分区类型此处不作研究