---
layout: post
title: Java中锁的分类[1]
date: 2018-03-11 00:00:20 +0300
description: 开始学习docker 主要记录 docker pull 查看 删除等等命令
tags: [Docker]
---

- [前言](#前言)
- [获取镜像](#获取镜像)
- [查看镜像](#查看镜像)
- [搜索镜像](#搜索镜像)
- [删除镜像](#删除镜像)


---

#### 前言
最近在学习docker，想记录一下自己的学习进度，以便来督促自己。现在容器技术已经应用很广泛了，感觉和虚拟机有点相似，最大的区别就是docker容器不需要运行一个臃肿的客户机操作系统。

---
#### 获取镜像
因为国外获取镜像比较慢，所以我在[网易云镜像中心](https://c.163yun.com/hub)获取。
docker pull name[:tag]
name:是镜像仓库的名字
tag:镜像的标签

|命令|描述|
|- | - |
|docker pull nginx:1.10|如果不显示指定tag，会默认为latest。此时走的是docker的默认仓库服务器|
|docker pull hub.c.163.com/library/mysql:5.7.18|从网易镜像中心获取公有的景象|

---

#### 查看镜像

docker images
```
# dongbin @ dongbindeMacBook-Pro in ~ [1:26:57]
$ docker images
REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
test                          0.2                 174e81dac9f8        28 minutes ago      112MB
test                          0.1                 ffcb2a6a53fb        31 minutes ago      109MB
nginx                         latest              e548f1a579cf        2 weeks ago         109MB
hub.c.163.com/library/mysql   5.7.18              9e64176cd8a2        10 months ago       407MB
```

|字段|描述|
|-|-|
|REPOSITORY|来源于哪个仓库|
|TAG|镜像的标签信息，如版本等等|
|IMAGE ID|镜像的唯一标志,启动容器的时候可以直接用镜像ID|

docker images -a 列出所有镜像，包括临时文件

** 使用TAG 为镜像添加标签 **
添加标签的主要目的事为了方便使用

docker tag test:0.2 dongbin:0.2
```
# dongbin @ dongbindeMacBook-Pro in ~ [1:35:51]
$ docker tag test:0.2 dongbin:0.2

# dongbin @ dongbindeMacBook-Pro in ~ [1:46:52]
$ docker images
REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
dongbin                       0.2                 174e81dac9f8        39 minutes ago      112MB
test                          0.2                 174e81dac9f8        39 minutes ago      112MB
test                          0.1                 ffcb2a6a53fb        42 minutes ago      109MB
nginx                         latest              e548f1a579cf        2 weeks ago         109MB
hub.c.163.com/library/mysql   5.7.18              9e64176cd8a2        10 months ago       407MB

```
实际上相当于取了个别名，因为它们IMAGE ID 是一样的

**inspect命令**

docker inspect 174e81dac9f8 列出的是所有信息
可以通过 -f 过滤

```
 dongbin @ dongbindeMacBook-Pro in ~ [2:05:27]
$ docker inspect -f '{{.Os}}' 174e81dac9f8
linux
```

运行中的容器也可以通过这个命令查看和筛选信息

---

#### 搜索镜像

docker search TEAM。主要参数，-s,--stars=X 评价星级，默认为0
```
# dongbin @ dongbindeMacBook-Pro in ~ [2:12:28] C:1
$ docker search -s 3 mysql
Flag --stars has been deprecated, use --filter=stars=3 instead
NAME                                                   DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
mysql                                                  MySQL is a widely used, open-source relati...   5757                [OK]
mariadb                                                MariaDB is a community-developed fork of M...   1824                [OK]
mysql/mysql-server                                     Optimized MySQL Server Docker images. Crea...   397                                     [OK]
```

---

#### 删除镜像

docker rmi + 镜像ID或标签


先写到着了，累了



