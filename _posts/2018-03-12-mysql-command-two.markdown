---
layout: post
title: Mysql 授权操作[2]
date: 2018-03-12 13:32:20 +0300
description: mysql 的授权操作
img: i-rest.jpg # Add image post (optional)
tags: [Mysql]
---

- [创建用户](#创建用户)
- [授权](#授权)
- [修改用户密码](#修改用户密码)
- [撤销用户权限](#撤销用户权限)
- [删除用户](#删除用户)
- [其他](#其他)

---

#### 创建用户

 CREATE USER 'username'@'host' IDENTIFIED BY 'password';

  username:用户名
  host:允许登录主机地址,如果允许所有地址登录，则用 %
  password:登录密码

  示例：
  ```
mysql> CREATE USER 'dongbin'@'localhost' IDENTIFIED BY 'dongbin';
Query OK, 0 rows affected (0.02 sec)

mysql> select Host,User,authentication_string from user ;
+-----------+-----------+-------------------------------------------+
| Host      | User      | authentication_string                     |
+-----------+-----------+-------------------------------------------+
| localhost | root      | *F31999B09228C0153A42DD7D608F0BED45CC9AD4 |
| localhost | mysql.sys | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE |
| %         | root      | *F31999B09228C0153A42DD7D608F0BED45CC9AD4 |
| localhost | dongbin   | *35BD47D734545D0B838770AA261283DF32D25B1E |
+-----------+-----------+-------------------------------------------+
4 rows in set (0.00 sec)

  ```

刷新 flush privileges; 就可以登录了。

<p style="color:red">注意Ubuntu服务器下，MySQL默认是只允许本地登录,#bind-address       = 127.0.0.1     #注释掉这一行就可以远程登录了 </p>

---

#### 授权
命令:GRANT privileges ON databasename.tablename TO 'username'@'host'

privileges:为用户的权限，如select，update等等，如果要授予所有权限为ALL
databasename.tablename:数据库和表，授予哪个数据库的哪张表，若要全部，用*代替
username:用户名
  host:允许登录主机地址,如果允许所有地址登录，则用 %

```
mysql> grant insert on test.* to 'dongbin'@'localhost';
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql>
```
登录后,使用select
```
mysql> select * from user;
ERROR 1142 (42000): SELECT command denied to user 'dongbin'@'localhost' for table 'user'
mysql>
```

<p style="color:red">GRANT privileges ON databasename.tablename TO 'username'@'host' WITH GRANT OPTION --- username 用户可以给其他用户授权，上面的写法是不能的</p>

---

#### 修改用户密码

SET PASSWORD FOR 'username'@'host' = PASSWORD('newpassword');

```
mysql> SET PASSWORD FOR 'dongbin'@'localhost' = PASSWORD('113689');
Query OK, 0 rows affected, 1 warning (0.01 sec)

mysql>
```
---

#### 撤销用户权限

查看用户权限
```
mysql> show grants for 'dongbin'@'localhost'
    -> ;
+---------------------------------------------------+
| Grants for dongbin@localhost                      |
+---------------------------------------------------+
| GRANT USAGE ON *.* TO 'dongbin'@'localhost'       |
| GRANT INSERT ON `test`.* TO 'dongbin'@'localhost' |
+---------------------------------------------------+
2 rows in set (0.01 sec)

mysql>
```
撤销权限
REVOKE privilege ON databasename.tablename FROM 'username'@'host';
```
mysql> revoke INSERT ON `test`.* from 'dongbin'@'localhost';
Query OK, 0 rows affected, 1 warning (0.01 sec)

mysql>

mysql> show grants for 'dongbin'@'localhost';
+---------------------------------------------+
| Grants for dongbin@localhost                |
+---------------------------------------------+
| GRANT USAGE ON *.* TO 'dongbin'@'localhost' |
+---------------------------------------------+
1 row in set (0.00 sec)

mysql>
```
---
#### 删除用户

DROP USER 'username'@'host';

#### 其他

root开启远程访问

1. update user set host = '%' where user = 'root';flush privileges;
2. GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'WITH GRANT OPTION

