---
title: "Mysql之Explain详解"
date: 2020-07-02T19:18:23+08:00
lastmod: 2022-07-04T19:18:23+08:00
author: ["shaoyi"]
keywords: 
- Mysql
categories: 
- Mysql
tags: 
- Mysql
description: ""
weight:
slug: ""
draft: false # 是否为草稿
comments: true
reward: true # 打赏
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
    image: "" #图片路径例如：posts/tech/123/123.png
    caption: "" #图片底部描述
    alt: ""
    relative: false



---



# Explain详解

explain出来的信息有10列，分别是id、select_type、table、type、possible_keys、key、key_len、ref、rows、Extra

> ## id

selcet查询的序列号，包含一组数字，表示查询中执行select子句或操作表的顺序，大概分为三种情况：

-  id相同时，执行顺序由上至下
-  id不同时，如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行
- id相同又不相同，同时存在
  - id如果相同，可以认为是一组，从上往下顺序执行；在所有组中，id值越大，优先级越高，越先执行

> ## select_type

查询的类型，主要用于区别普通查询、联合查询、子查询等的复杂查询，类型大概为：

- **SIMPLE**   

  简单的select查询语句，查询语句不使用UNION或子查询

- **PRIMARY**	

  查询中若包含任何复杂的子部分,最外层的select被标记为PRIMARY

- **SUBQUERY**  

  在select或where列表中包含的子查询 

- **DERIVED**  

  在from列表中包含的子查询被标记为DERIVED(衍生表)，MySQL会递归执行这些子查询，把结果放在临时表里。

- **UNION**  

  若第二个select出现在UNION之后，会被标记为UNION；若UNION包含在from子句的子查询中，外层select将被标记为：DERIVED

- **UNION RESULT ** 

   UNION的结果

> ## table

显示这一行的数据是关于哪张表的，有时不是真实的表名字,看到的是derivedx(x是个数字,我的理解是第几步执行的结果，也就是id数)

譬如下面的例子：

```sql
mysql> explain select * from (select * from ( select * from t1 where id=2602) a) b;
+----+-------------+------------+--------+-------------------+---------+---------+------+------+-------+
| id | select_type | table      | type   | possible_keys     | key     | key_len | ref  | rows | Extra |
+----+-------------+------------+--------+-------------------+---------+---------+------+------+-------+
|  1 | PRIMARY     | <derived2> | system | NULL              | NULL    | NULL    | NULL |    1 |       |
|  2 | DERIVED     | <derived3> | system | NULL              | NULL    | NULL    | NULL |    1 |       |
|  3 | DERIVED     | t1         | const  | PRIMARY,idx_t1_id | PRIMARY | 4       |      |    1 |       |
+----+-------------+------------+--------+-------------------+---------+---------+------+------+-------+
```

> ## type

表示MySQL在表中找到所需行的方式，又称“访问类型排列”。

常用的类型有： **ALL, index, range, ref, eq_ref, const, system（从左到右，性能从差到好）**

**即：ALL < index < range < ref < eq_ref < const < system**

一般来说，得保证查询至少达到**range**级别，最好能达到**ref**

**ALL：** Full Table Scan， MySQL将遍历全表以找到匹配的行

**index：**  Full Index Scan，index与ALL区别为index类型只遍历索引树。这通常比ALL快，因为索引文件通常比数据文件小。

**range：**  只检索给定范围的行，使用一个索引来选择行。key列显示使用了哪个索引，一般就是在你的where语句中出现了between、<、>、in等的查询，这种范围扫描索引比全表扫描要好，因为它只需要开始于索引的某一点，而结束语另一点，不用扫描全部索引

**ref：**  非唯一性索引扫描，返回匹配某个单独值的所有行，本质上也是一种索引访问，它返回所有匹配某个单独值的行，然而，它可能会找到多个符合条件的行，所以它应该属于查找和扫描的混合体

**eq_ref：**   类似ref，区别就在使用的索引是唯一索引，对于每个索引键值，表中只有一条记录匹配，简单来说，就是多表连接中使用primary key或者 unique key作为关联条件

**const：**  表示通过索引一次就找到了，const用于比较primary key或者unique索引。因为只匹配到一行数据，所以就很快。如将主键置于where列表中，MySQL就能将该查询转换为一个常量

**system：**  是const类型的特例，当查询的表只有一行的情况下，平时不会出现，这个可以忽略不计



> ## possible_keys

显示可能应用在这张表中的索引，一个或多个。查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被查询实际使用

> ## Key

实际使用的索引。如果为NULL，则没有使用索引。查询中若使用了覆盖索引，则该索引和查询的select字段重叠

> ## key_len

表示索引使用的字节数，可通过该列计算查询中使用的索引的长度。在不损失精确性的情况下，长度越短越好。

key_len显示的值为索引字段的最大可能长度，并非实际使用长度，即key_len是根据表定义计算而得，不是通过表内检索出的

> ## ref

显示索引得哪一列被使用了，如果可能的话，是一个常数。哪些列或常量被用于查找索引列上的值

> ## rows

根据表统计信息及索引选用情况，大致估算出找到所需的记录所需要读取的行数

> ## Extra

包含不适合在其他列中显示但十分重要的额外信息

**Using filesort：**  说明mysql会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取。mysql中无法利用索引完成的排序操作称为“文件排序”

**Using temporary：**  使用了临时表保存中间结果，mysql在对查询结果排序时使用临时表。常见于排序order by 和分组查询 group by

**Using index ：**  表示相应的select操作中使用了覆盖索引（Covering Index）,避免访问了表的数据行，效率不错！如果同时出现using where，表明索引被用来执行索引键值的查找；如果没有同时出现using where，表明索引用来读取数据而非执行查找动作。

**Using where：**  表明使用了where过滤

**Using join buffer：**  使用了连接缓存

**impossible where：**  where子句的值总是false，不能用来获取任何元组

**select tables optimized away：**	在没有group by 子句的情况下，基于索引优化min/max操作或者对于MyISAM 存储引擎优化count(*)操作，不必等到执行阶段再进行计算，查询执行计划生产的阶段即完成优化

**distinct：**  优化distinct操作，在找到第一匹配的元组后即停止找同样值得动作



