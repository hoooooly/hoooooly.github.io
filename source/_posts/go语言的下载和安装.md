---
title: go语言的下载和安装
tags:
  - go
comments: true
toc: true
only:
  - home
  - category
  - tag
date: 2021-05-26 10:37:38
categories: Go
pic:
---


## 下载安装包

访问[https://studygolang.com/dl](https://studygolang.com/dl)下载对应系统的安装包，这里以`windows`平台为例。

![](Screenshot_1.webp)

## 安装

安装下载后的安装文件。

![](Screenshot_2.webp)

使用推荐的默认设置就可以了。

## 增加环境变量

在系统环境变量`PATH`中添加`go`的安装路径，比如`C:\Program Files\Go\bin`。

添加完成后调出`cmd`输入`go version`有下面的显示说明安装成功了。

```cmd
C:\Users\espho>go version
go version go1.16.4 windows/amd64
```

## 设置代理

设置为国内代理，方便下载`go`模块：

```shell
阿里云
https://mirrors.aliyun.com/goproxy/
七牛云
https://goproxy.cn
```

设置方式：

```shell
go env -w GO111MODULE=on
go env -w GOPROXY=https://mirrors.aliyun.com/goproxy/
```

运行`go env`查看设置是否成功。
