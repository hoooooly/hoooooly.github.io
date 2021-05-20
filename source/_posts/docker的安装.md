---
title: docker的安装
tags:
  - linux
  - docker
comments: true
toc: true
only:
  - home
  - category
  - tag
date: 2021-05-20 16:45:41
categories: Linux
pic:
---


## 安装环境

安装环境是Centos环境，要求版本是` CentOS 7 or 8`以上。

## 卸载旧版本的docker

```shell
 sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

## 使用yum仓库安装

安装方式有很多种，可以参考官方文档。

### 设置仓库源

下载`yum-utils`工具包和稳固的安装源。

```shell
sudo yum install -y yum-utils

sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

### 安装docker引擎

- 1.安装最新版本

```shell
sudo yum install docker-ce docker-ce-cli containerd.io
```

- 2.安装特定版本

列出可安装的版本。

```python
yum list docker-ce --showduplicates | sort -r
```

选择安装版本。

```python
sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
```

### 启动docker

运行下列命令启动`docker`。

```shell
sudo systemctl start docker
```

测试`docker`安装正确。

```python
sudo docker run hello-world
```

## 卸载docker

卸载docker和下载的容器及包：

```shell
sudo yum remove docker-ce docker-ce-cli containerd.io

 ```

镜像等配置文件不会自动移除，需要手动删除。

```shell
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
```
