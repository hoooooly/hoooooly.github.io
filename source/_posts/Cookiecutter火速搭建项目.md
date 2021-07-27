---
title: Cookiecutter火速搭建项目
tags:
  - cookiecutter
  - python
comments: true
typora-root-url: Cookiecutter火速搭建项目
date: 2021-07-22 15:04:53
categories:
index_img:
banner_img:
---

# `Cookiecutter`安装和使用

`Cookiecutter` 是一个通过项目模板创建项目的命令行工具。比如，通过`Python Package`模板来创建`Python package`项目。（通过`Python`代码调用`Cookiecutter`的`API`可以扩展为自动化创建服务和带有`Web UI`的服务程序）

安装：

```shell
pip3 install cookiecutter
```

使用：

```shell
cookiecutter https://github.com/pydanny/cookiecutter-django.git
```

```python
$ cookiecutter https://github.com/pydanny/cookiecutter-django.git
project_name [My Awesome Project]: zhanhu
project_slug [zhanhu]:
description [Behold My Awesome Project!]: a Q&A website
author_name [Daniel Roy Greenfeld]: __holy__
domain_name [example.com]: holychan.io
email [__holy__@example.com]: espholychan@outlook.com
version [0.1.0]:
Select open_source_license:
1 - MIT
2 - BSD
3 - GPLv3
4 - Apache Software License 2.0
5 - Not open source
Choose from 1, 2, 3, 4, 5 [1]:
timezone [UTC]: Asia/Shanghai
windows [n]: y
use_pycharm [n]: y
use_docker [n]: n
Select postgresql_version:
1 - 13.2
2 - 12.6
3 - 11.11
4 - 10.16
Choose from 1, 2, 3, 4 [1]:
Select js_task_runner:
1 - None
2 - Gulp
Choose from 1, 2 [1]:
Select cloud_provider:
1 - AWS
2 - GCP
3 - None
Choose from 1, 2, 3 [1]:
Select mail_service:
1 - Mailgun
2 - Amazon SES
3 - Mailjet
4 - Mandrill
5 - Postmark
6 - Sendgrid
7 - SendinBlue
8 - SparkPost
9 - Other SMTP
Choose from 1, 2, 3, 4, 5, 6, 7, 8, 9 [1]:
use_async [n]:
use_drf [n]:
custom_bootstrap_compilation [n]:
use_compressor [n]: y
use_celery [n]: y
use_mailhog [n]:
use_sentry [n]:
use_whitenoise [n]:
use_heroku [n]:
Select ci_tool:
1 - None
2 - Travis
3 - Gitlab
4 - Github
Choose from 1, 2, 3, 4 [1]:
keep_local_envs_in_vcs [y]: n
debug [n]: y
 [SUCCESS]: Project initialized, keep up the good work!
```

这样就生成好了项目模板。

## 搭建项目

### 创建数据库

```shell
mysql> create database zanhu charset utf8;
mysql> create database zanhu_test charset utf8;
```

### 创建测试用户

```python
mysql> create user 'zanhutest'@'%' identified by 'test_mysql'
mysql> grant all on zanhu_test.* to 'zanhutest'@'%';
mysql> flush privileges;
```

## 配置远程开发环境

准备一个云主机，或者搭建`linux`环境的虚拟机。

```shell
yum install python-devel mysql-devel mysql-lib
yum install bzip2-devel
```

`pycharm`配置远程开发环境

`tools`下面的`Depolyment`点击`configuration`。

![](image-20210723163026345.png)

![](image-20210723163806895.png)

![](image-20210723164011342.png)





[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


