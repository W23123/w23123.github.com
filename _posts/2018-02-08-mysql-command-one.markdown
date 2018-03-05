---
layout: post
title: Mysql 常用命令[1]
date: 2018-02-08 13:32:20 +0300
description: 主要列举了Mysql的连接，database的创建，数据库备份以及还原，table操作
img: i-rest.jpg # Add image post (optional)
tags: [Mysql]
---

- [数据库连接](#数据库连接)
- [数据库备份及还原](#数据库备份及还原)
- [操作database命令](#操作database命令)
- [操作table命令](#操作table命令)


#### 数据库连接
数据库连接命令格式：mysql -h主机地址 -P端口 -u用户名 －p用户密码（可以跟上要访问的数据库名称）

|示例|描述|
|---|---|
|mysql -uroot -pabc123|本地连接,也可以 mysql -h localhost -P3306 -uroot -pabc123|
|mysql -h 10.1.10.11 -P 63306 -u root -p abc123|远程连接|

#### 数据库备份及还原
数据库备份命令格式 mysqldump -h主机地址 -P端口 -u 用户名 -p密码 备份数据库名 > 存储文件地址

|示例|描述|
|---|---|
|mysqldump -h10.1.10.11 -P63306 -uroot -p abcc123  test > /opt/test.sql|远程备份到本地,如果不远程备份去掉 -h参数 或者 -h localhost|

数据库还原命令格式 source 存储sql文件的地址

|示例|描述|
|---|---|
|source /opt/test.sql| 前提是先进入 要导入的数据库(use database)|
|mysqldump -h host -u root -p dbname < dbname_backup.sql |其他格式|

#### 操作database命令

|命令|描述|
|---|---|
|show databases|显示所有的database|
|use database  |选择使用database  |
|create database name| 创建名为name的database|
|drop database name|删除database,不提示|
|alter database test character set utf8;|修改数据库编码|

查看数据库编码
`use test;` --> `show variables like 'character_set_database';`-->`alter database xxx CHARACTER SET utf-8;`
    一般修改mysql的参数到my.conf里去修改(后续会详细分析my.conf文件);
    show variables;显示当前所有变量的值
    查看当前连接数 show processlist;
    或者 mysqladmin -h10.1.10.11 -P53306 -uroot -pabc123 processlist;
    杀死连接数 kill ID；

修改mysql的root密码（use mysql）
update user set password=password("abc123") where user='root';
//刷新数据库
flush privileges

#### 操作table命令

|命令|描述|
|---|---|
|show tables|显示所有表|
|describe tablename|显示表结构|
|create table <表名> (<字段名1> <类型1> [,..<字段名n> <类型n>]);|create table test(dong int(4) not null default 0,bin varchar(50) not null default "" primary key(`dong`))DEFAULT CHARSET=utf-8 TYPE=INNODB|
|alter table test change bin bins int(10) not null default 0;|修改table的属性|
|alter table test add add_test varchar(50) not null |添加表属性|
|alter table test character set utf8;|修改表编码|
|alter table test modify col_name varchar(50) CHARACTER SET utf8;|修改字段编码|
|alter table test drop add_test|删除字段|
|ALTER TABLE 表名 ADD 索引类型 （unique,primary key,fulltext,index）[索引名]（字段名)|alter table test add index index_bin using btree(`bin`) comment ''|
|alter table table_name drop index index_name 或 alter table table_name drop primary key|删除索引|
|alter table table_test drop primary key;|删除主键|
|alter table table_test add primary key(id);|添加主键|