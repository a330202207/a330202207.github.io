---
title: 关于SELECT COUNT一些问题
date: 2019-11-20 17:58:09
categories:
- MySQL
---
## COUNT函数

关于COUNT函数:

 - COUNT(expr) ，返回SELECT语句检索的行中expr的值不为NULL的数量。结果是一个BIGINT值
 - 如果查询结果没有命中任何记录，则返回0

**注**：
COUNT(*)的统计结果会包含值为NULL的行数
假如有张表如下

![](https://ned.oss-cn-beijing.aliyuncs.com/image/Xnip2019-11-20_18-17-03.jpg)

使用语句`count(*)`, `count(name)`, `count(age)`, `count(class)`, `count(1)`查询结果如下
![](https://ned.oss-cn-beijing.aliyuncs.com/image/Xnip2019-11-21_14-25-35.jpg)



## COUNT(字段)、COUNT(常量) 和 COUNT(*) 之间的区别
`COUNT(常量)` 和 `COUNT(*)`表示的是直接查询**符合条件**的数据库表的行数。而COUNT(字段)表示的是查询符合条件的列的值**不为NULL**的行数

除了查询得到结果集有区别之外，`COUNT(*)`相比 `COUNT(常量)` 和 `COUNT(字段)`来讲，**COUNT(*)是SQL92定义的标准统计行数的语法，因为他是标准语法，所以MySQL数据库对他进行过很多优化**

> SQL92，是数据库的一个ANSI/ISO标准。它定义了一种语言（SQL）以及数据库的行为（事务、隔离级别等）。

## COUNT(*) 和 COUNT(1) 区别
[MySQL官方文档][1]：

> InnoDB handles SELECT COUNT(*) and SELECT COUNT(1) operations in the same way. There is no performance difference

![](https://ned.oss-cn-beijing.aliyuncs.com/image/Xnip2019-11-21_14-30-36.jpg)
所以，对于**COUNT(1)和COUNT(*)，性能是一样的**，完全不存在谁比谁快，建议使用`COUNT(*)`！因为这个是SQL92定义的标准统计行数的语法

## InnoDB 与 MyISAM 引擎中 COUNT(*)
 - 在 `MyISAM` 存储引擎中， `COUNT(*)` 函数是直接读取数据表保存的行记录数并返回
 - 在 `InnoDB` 存储引擎中， `COUNT(*)` 函数是先从内存中读取表中的数据到内存缓冲区，然后扫描全表获得行记录数的。


在使用 `COUNT` 函数中加上 `WHERE` 条件时，在两个存储引擎中的效果是一样的，都会**扫描全表**计算某字段有值项的次数


## 关于聚簇索引与辅助索引(二级索引)的 COUNT(字段)

Innodb索引分类：

 - 聚簇索引(clustered index)
    - 有主键时，根据主键创建聚簇索引
    - 没有主键时，会用一个唯一且不为空的索引列做为主键，成为此表的聚簇索引
    - 如果以上两个都不满足那innodb自己创建一个虚拟的聚集索引
 - 辅助索引(secondary index)
    - 非聚簇索引都是辅助索引，像复合索引、前缀索引、唯一索引

在InnoDB中**默认会使用可用辅助索引来统计行数，如果不存在辅助索引，则通过扫描聚簇索引来统计行数**

**原理**：
查询优化器是选择能够让**IO次数最少的索引**，也就是基于**占用空间最小**的字段所建的索引（每次IO读取的数据量是固定的，索引占用的空间越小所需的IO次数也就越少）

> 目前基于磁盘的数据库或者搜索引擎（比如Lucene）的性能瓶颈主要都是在IO阶段，相比于CPU和RAM，IO操作实在太慢了，所以这类系统的优化方向也都都是类似的——尽一切可能减少IO的次数（所以很多用ES的程序在性能优化到极限的时候选择直接上SSD）

Innodb中**聚簇索引**的叶子节点中保存的是整行记录，而**辅助索引**的叶子节点中保存的是该行记录的主键的值，相比之下，辅助索引要比聚簇索引小很多，所以MySQL会优先选择**最小的辅助索引**来扫表，当存在多个辅助索引情况下会**选择占用空间最小的**。针对于主键字段比较大，可以先建一个小字段并创建一个辅助索引专门用来统计行数

## 总结

 - COUNT(1)和COUNT(\*)，使用COUNT(\*)
 - COUNT(字段)辅助索引    **>**  COUNT(字段)聚簇索引
 - COUNT(字段)辅助索引(空间小)   **>** COUNT(字段)辅助索引


  [1]: https://dev.mysql.com/doc/refman/8.0/en/group-by-functions.html