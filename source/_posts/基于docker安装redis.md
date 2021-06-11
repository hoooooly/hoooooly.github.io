---
title: 基于docker安装redis
tags:
  - docker
  - redis
comments: true
toc: true
only:
  - home
  - category
  - tag
date: 2021-06-04 19:11:27
categories: Docker
pic:
---

## 获取redis镜像

安装环境，腾讯云。

```shell
docker pull redis
```

不加版本号默认获取最新版本，也可以使用 `docker search redis` 查看镜像来源。

## 查看本地镜像

```shell
[root@VM-0-15-centos ~]# docker images
REPOSITORY                TAG       IMAGE ID       CREATED        SIZE
redis                     latest    fad0ee7e917a   2 days ago     105MB
```

## 配置

官网获取[redis](http://download.redis.io/redis-stable/redis.conf)配置文件:

- 修改默认的配置文件

```shell
# bind 127.0.0.1 -::1   #注释这部分，去掉限制只能本地访问

protected-mode no   # 默认yes,开启保护模式，限制为本地访问

daemonize no    # 默认no，改为yes意为以守护进程方式启动，可后台运行，除非kill进程（可选），改为yes会使配置文件方式启动redis失败

dir ./root/redis/data #输入本地redis数据库存放文件夹（可选）

appendonly yes #redis持久化（可选）
```

使用xftp登录服务器，在`/root/redis/data `创建好文件夹用于存放`redis`数据，这个文件夹位置可自己设定。然后将配置好的`redis.conf`文件复制入`/root/redis/`文件夹下。

## 运行

```python
docker run -p 6379:6379 --name redis -v /root/redis/redis.conf:/etc/redis/redis.conf  -v /root/redis/data:/data -d redis:latest redis-server /etc/redis/redis.conf --appendonly yes
```

启动命令解释如下：

```shell
-p 6379:6379:把容器内的6379端口映射到宿主机6379端口
-v /root/redis/redis.conf:/etc/redis/redis.conf：把宿主机配置好的redis.conf放到容器内的这个位置中
-v /root/redis/data:/data：把redis持久化的数据在宿主机内显示，做数据备份
redis-server /etc/redis/redis.conf：这个是关键配置，让redis不是无配置启动，而是按照这个redis.conf的配置启动
–appendonly yes：redis启动后数据持久化
```
