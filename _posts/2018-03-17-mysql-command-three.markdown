---
layout: post
title: Mysql 索引操作[3]
date: 2018-03-17 13:32:20 +0300
description: mysql 的索引操作
img: i-rest.jpg # Add image post (optional)
tags: [Mysql]
---

- [查看索引](#查看索引)
- [创建索引](#创建索引)
- [删除索引](#删除索引)

---

#### 查看索引

show index from tblname;或 show keys from tblname;

![1.png]({{site.baseurl}}/assets/img/mysql-three/1.png)

#### 创建索引

CREATE [UNIQUE|FULLTEXT|SPATIAL] INDEX index_name [USING index_type] ON table_name (index_col_name,...)]
<p style="color:red">不能添加主键索引</p>

或者

ALTER TABLE table_name
ADD [UNIQUE|FULLTEXT|SPATIAL] INDEX index_name (index_col_name,...) [USING index_type]

示例
ALTER TABLE用来创建普通索引、UNIQUE索引或PRIMARY KEY索引。

ALTER TABLE table_name ADD INDEX index_name (column_list)
ALTER TABLE table_name ADD UNIQUE (column_list)
ALTER TABLE table_name ADD PRIMARY KEY (column_list)

UNIQUE(唯一索引):被索引的字段组合，其数据在全表中唯一。
FULLTEXT(全文索引):是全文索引，用于在一篇文章中，检索文本信息的.
SPATIAL(空间索引):通过R树来实现，使得空间搜索变得高效(目前应该不会用到)

index_type 表示索引的具体实现方式，在MySQL中，有两种不同形式的索引——BTREE索引和HASH索引。在存储引擎为MyISAM和InnoDB的表中只能使用BTREE，其默认值就是BTREE；在存储引擎为MEMORY或者HEAP的表中可以使用HASH和BTREE两种类型的索引，其默认值为HASH。



![2.png]({{site.baseurl}}/assets/img/mysql-three/2.png)
![3.png]({{site.baseurl}}/assets/img/mysql-three/3.png)

#### 删除索引

ALTER TABLE table_name DROP INDEX index_name;

![4.png]({{site.baseurl}}/assets/img/mysql-three/4.png)





