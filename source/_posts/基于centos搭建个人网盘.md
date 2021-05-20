---
title: 基于centos搭建个人网盘
tags:
  - seafile
comments: true
toc: true
only:
  - home
  - category
  - tag
date: 2021-05-20 16:16:41
categories: Python
pic:
---

## Nextcloud

`Nextcloud`是一款开源免费的私有云存储网盘项目，可以让你快速便捷地搭建一套属于自己或团队的云同步网盘，从而实现跨平台跨设备文件同步、共享、版本控制、团队协作等功能。它的客户端覆盖了`Windows`、`Mac`、`Android`、`iOS`、`Linux`等各种平台，也提供了网页端以及`WebDAV`接口，所以你几乎可以在各种设备上方便地访问你的云盘。

## 安装 Nextcloud

## 安装Mysql

```shell
docker pull mysql # 拉取镜像

docker images # 查看名称/镜像id

docker run -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=******** -d c0cdc95609f1 #c0cdc95609f1（镜像id）
```

运行`mysql`并且设置访问端口：`3306`，容器名称：`mysql` ,管理员密码：`********`

进入容器`bash`。

```shell
docker exec -it 容器名称/容器id bash
```

登陆`mysql`

```shell
mysql -uroot -p
```

接着输入管理员密码：`admin`,回车。
创建一个数据库

```mysql
CREATE DATABASE nextcloud;
```

创建一个用户

```mysql
CREATE USER 'nextcloud'@'%' IDENTIFIED BY 'admin';
```

创建一个用户名称为：`nextcloud；‘%’`：代表不限`ip`登陆，远程登陆; 密码为：`admin`。

授权。

```mysql
GRANT ALL PRIVILEGES ON nextcloud.* TO nextcloud@'%'  WITH GRANT OPTION;
```

给这个用户`nextcloud`授予 这个数据库`nextcloud.*`所有的权限，远程登陆，密码为`admin`；

### 基于Docker部署Nextcloud服务端

选择以`Docker`的方式来部署`nextcloud`是因为`Docker`可以跨平台上运行，可以确保执行环境的一致性，有利于应用的迁移和管理。

服务端部署的基本流程是：**安装`Docker`并启动** --> **运行`Nextcloud`容器** --> **访问`Web`端初始化**。

安装的话不做过多说明，参考官方文档或本博客其他博文。

下载`Nextcloud`镜像

```shell
docker pull nextcloud
```

运行`Nextcloud`

{% colorpanel info "-d #容器后台运行

--name nextcloud #容器名

-v /data/nextcloud:/var/www/html #将宿主机的目录/data/nextcloud挂载到容器的/var/www/html

-p 8000:80 #将宿主机的端口（此处以8000为例）映射到容器的80端口" %}

```shell
docker run -d \
--name nextcloud \
-p 8000:80 \
-v /data/nextcloud:/var/www/html \
nextcloud
```

查看运行的容器。

```shell
docker ps -a
```

停止运行的容器。

```shell
docker stop id
```