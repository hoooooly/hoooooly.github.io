---
title: Metasploit渗透测试框架
tags:
  - metaploit
comments: true
typora-root-url: Metasploit渗透测试框架
date: 2021-10-01 16:00:21
categories: 安全
index_img:
banner_img:
---

# 环境搭建

## 概述

Metasploit是一个免费的、可下载的框架，通过它可以很容易地获取、开发并对计算机软件漏洞实施攻击。它本身附带数百个已知软件漏洞的专业级漏洞攻击工具。而且它还提供了许多个接口，其最受欢迎的是由Rapid 7和Strategic Cyber LLC公司维护的。由Rapid 7和Strategic Cyber LLC公司维护的接口包括MetasploitFramework Edition、Metasploit Community Edition、Metasploit Express、Metasploit Pro、Armitage和Cobalt strike。其中，Metasploit Framework（命令行接口）和Armitage（图形界面接口）是较常用的两种，并且是免费的。

## 安装要求

1. 禁用杀毒软件
2. 禁用防火墙
3. 使用管理员权限安装

## 安装Metasploit Framework

Metasploit Framework的下载地址为https://www.rapid7.com/products/metasploit/download/editions/。

选择版本安装即可。

## 安装及连接PostgreSQL数据库服务

PostgreSQL是一个免费的对象-关系数据库服务，用来存储一些数据记录。Metasploit启动后，会选择连接PostgreSQL数据库服务，然后就可以完整地利用MSF数据库查询exploit和记录了。在Kali Linux系统中，启动Metasploit后，将自动连接到PostgreSQL服务的Postgres数据库。但是，在其他系统中，需要手动连接该数据库。

### 安装PostgreSQL数据库服务

在大部分系统中，默认并不会安装PostgreSQL数据库服务。所以，如果要使用该数据库，需要先在系统中进行安装。PostgreSQL数据库服务的下载地址为http://www.postgresql.org/download/。

> 安装PostgreSQL的分区最好是NTFS格式的。PostgreSQL的首要任务是保证数据的完整性，而FAT和FAT32文件系统不能提供这样的可靠性保障，而且FAT文件系统本身缺乏安全性保障，无法保证原始数据在未经授权的情况下被更改。

记住安装的密码和端口号。

### 初始化PostgreSQL数据库

为了便于存储数据，Metasploit会使用特定的用户名来访问PostgresSQL数据库，并将数据保存在自有的数据库中。所以，在启动Metasploit之前，需要先初始化数据库。在初始化过程中，Metasploit会在PostgreSQL中建立一个专用账户msf，并创建对应的数据库。操作过程如下：

1．启动PostgreSQL数据库服务

为了方便渗透测试人员操作，Metasploit提供了专门的数据库服务管理命令。下面依次讲解数据库服务的启动和关闭方法。

（1）启动PostgreSQL数据库服务，执行的命令如下：

```shell
msfdb start
```

输出信息表示数据库服务已经成功启动了。

（2）当不使用Metasploit时，为了节省系统资源，可以停止PostgreSQL数据库服务，执行的命令如下：

```shell
msfdb stop
```

成功启动PostgreSQL数据库服务，PostgeSQL会监听5432端口。查看该端口是否被监听，执行命令如下：

```shell
ss -ant
```

2．初始化数据库

初始化数据库，执行命令如下：

```shell
msfdb init
```





[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


