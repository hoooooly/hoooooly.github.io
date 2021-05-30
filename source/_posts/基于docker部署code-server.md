---
title: 基于docker部署code-server
tags:
  - docker
  - vscode
comments: true
toc: true
only:
  - home
  - category
  - tag
date: 2021-05-25 17:21:25
categories: Docker
pic:
---

## 拉取镜像

```shell
# docker pull codercom/code-server
# docker images
REPOSITORY             TAG       IMAGE ID       CREATED       SIZE
codercom/code-server   latest    f3ac734fcec8   12 days ago   802MB
```

## 创建挂载目录

```shell
# CODE=/home/docker/code
# mkdir $CODE && cd $CODE
```

## 配置文件

启动一个容器：`-u`表示使用`root`用户运行

```shell
# docker run -d -u root -p 8088:8080 --name code-server -v $CODE:/home/code codercom/code-server
# docker ps
CONTAINER ID   IMAGE       COMMAND          CREATED    \ STATUS         PORTS             NAMES
f10030d88e1d  codercom/code-server "/usr/bin/entrypoint鈥 3 minutes ago Up 3 minutes   0.0.0.0:8088->8080/tcp, :::8088->8080/tcp code-server
```

拉出配置文件

```shell
# docker cp code-server:/root/.config/code-server/config.yaml $CODE/
# cat $CODE/config.yaml
bind-addr: 127.0.0.1:8080
auth: password
password: 59bd4df2841fbc77d67f674f
cert: false
```

修改你的密码：

```shell
docker run -d -u root \
  -p 8088:8080 \
  --name code-server \
  -v $CODE/config.yaml:/root/.config/code-server/config.yaml \
  -v $CODE:/home/code \
  codercom/code-server
```

## 启动服务

```shell
# docker stop code-server && docker rm code-server
# docker run -d -u root \
  -p 8088:8080 \
  --name code-server \
  -v $CODE/config.yaml:/root/.config/code-server/config.yaml \
  -v $CODE:/home/code \
  codercom/code-server
```

## 登录

访问`http://holychan.ltd:8088/`，输入密码登录。

![](![](Screenshot_1.webp))