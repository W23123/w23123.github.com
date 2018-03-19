---
layout: post
title: Mysql 执行计划[4]
date: 2018-03-18 00:00:20 +0300
description: mysql 执行计划
img: i-rest.jpg # Add image post (optional)
tags: [Mysql]
---

### Mysql explain

查看mysql的执行计划
示例
![1.png]({{site.baseurl}}/assets/img/mysql-explain/1.png)

<span style="color:red">select_type</span>
simple:
进行不需要Union操作或不含子查询的简单select查询时，响应查询语句的select_type即为simple（查询中包含连接的情形也一样）。无论查询语句是多么复杂，执行计划中select_type为simple的单位查询一定只有一个。最外侧的select查询的select_type通常为simple

primary
一个需要Union操作或含子查询的select查询执行计划中，位于最外层的select_type即为primary。与simple一样，select_type为primary的单位select查询也只存在1个，位于查询最外侧的select单位查询的select_type为primary

union
由union操作联合而成的单位select查询中，除第一个外，第二个以后的所有单位select查询的select_type都为union。union的第一个单位select的select_type不是union，而是DERIVED。它是一个临时表，用于存储联合（Union）后的查询结果。
![2.png]({{site.baseurl}}/assets/img/mysql-explain/2.png)

DEPENDENT UNION , union result :union的结果集

![3.png]({{site.baseurl}}/assets/img/mysql-explain/3.png)

<span style="color:red">table</span>:显示这一行数据关于哪个表的

<span style="color:red">type</span>:显示连接使用类型，system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL
system：表只有一行：system表。这是const连接类型的特殊情况。
const ：表中的一个记录的最大值能够匹配这个查询（索引可以是主键或惟一索引）。因为只有一行，这个值实际就是常数，因为MYSQL先读这个值然后把它当做常数来对待。
eq_ref：在连接中，MYSQL在查询时，从前面的表中，对每一个记录的联合都从表中读取一个记录，它在查询使用了索引为主键或惟一键的全部时使用。
ref：这个连接类型只有在查询使用了不是惟一或主键的键或者是这些类型的部分（比如，利用最左边前缀）时发生。对于之前的表的每一个行联合，全部记录都将从表中读出。这个类型严重依赖于根据索引匹配的记录多少—越少越好。
range：这个连接类型使用索引返回一个范围中的行，比如使用>或<查找东西时发生的情况。
index：这个连接类型对前面的表中的每一个记录联合进行完全扫描（比ALL更好，因为索引一般小于表数据）。
ALL：这个连接类型对于前面的每一个记录联合进行完全扫描，这一般比较糟糕，应该尽量避免。

<span style="color:red">possible_keys</span>:查询涉及到的字段上存在索引，则该索引将被列出，但不一定被查询实际使用

<span style="color:red">key</span>:实际使用的索引，如果为NULL，则没有使用索引。

<span style="color:red">key_len</span>:使用的索引的长度。在不损失精确性的情况下，长度越短越好

<span style="color:red">ref</span>:显示索引的哪一列被使用了，如果可能的话，是一个常数

<span style="color:red">rows</span>:MYSQL认为必须检查的用来返回请求数据的行数

<span style="color:red">Extra</span>:额外的信息，很重要
Distinct ：一旦mysql找到了与行相联合匹配的行，就不再搜索了。
Not exists ：mysql优化了LEFT JOIN，一旦它找到了匹配LEFT JOIN标准的行，就不再搜索了。
Range checked for each Record（index map:#） ：没有找到理想的索引，因此对从前面表中来的每一个行组合，mysql检查使用哪个索引，并用它来从表中返回行。这是使用索引的最慢的连接之一。
Using filesort ：看到这个的时候，查询就需要优化了。mysql需要进行额外的步骤来发现如何对返回的行排序。它根据连接类型以及存储排序键值和匹配条件的全部行的行指针来排序全部行。
Using index ：列数据是从仅仅使用了索引中的信息而没有读取实际的行动的表返回的，这发生在对表的全部的请求列都是同一个索引的部分的时候。
Using temporary ：看到这个的时候，查询需要优化了。这里，mysql需要创建一个临时表来存储结果，这通常发生在对不同的列集进行ORDER BY上，而不是GROUP BY上。
Where used ：使用了WHERE从句来限制哪些行将与下一张表匹配或者是返回给用户。如果不想返回表中的全部行，并且连接类型ALL或index，这就会发生，或者是查询有问题。

索引失效

1.索引不存储空值
单列索引不存储null值，复合索引不存储全为null的值。
![4.png]({{site.baseurl}}/assets/img/mysql-explain/4.png)

2.键值较少的列不适合索引，比如性别等

3.like '%XX',不能使用索引，因为%开始模糊匹配，只能全局搜索。

4.条件查询里or,其实中有条件带有索引，也不会使用，只有全部带有索引(innodb是不行的)。

5.多列索引，最左匹配，如果匹配不上，则不使用索引。

6.列名如果是字符串，条件数据要引号括起来，否则不使用索引。

mysql 2种索引算法 B-Tree索引 Hash索引（后面在具体分析）

B-Tree 具有范围查找和前缀查找，对于有N节点的B树，检索一条记录的复杂度为O(LogN)
Hash 只做查找，查找复杂度O(1)