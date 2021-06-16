---
title: Django入门-Django Admin
tags:
  - django
  - python
comments: true
toc: true
only:
  - home
  - category
  - tag
date: 2021-06-16 12:47:04
categories: Django
pic: Screenshot_8.webp
---

## Django Admin 简介

当我们开始第一个新项目的时候，`Django`已经配置了`Django Admin`，这个应用程序列出的`INSTALLED_APPS`。

使用`Django Admin`的一个很好的列子就是在博客中，它可以用来被作者用来编写和发布文章。另一个列子是电子商务网站，工作人员可以创建，编辑，删除产品。

现在，我们来配置`Django Admin`来维护我们应用程序的板块。

首先，创建一个管理员账户：

```python
>>>python manage.py createsuperuser
```

按说明设置管理员账户名，邮箱和密码：

```
Username (leave blank to use 'q'): Admin
Email address: espholychan@outlook.com
Password:
Password (again):
The password is too similar to the username.
This password is too short. It must contain at least 8 characters.
This password is too common.
Bypass password validation and create user anyway? [y/N]: y
Superuser created successfully.
```

在浏览器访问`http://127.0.0.1/admin`。

![](1.PNG)

输入刚才设置的用户名和密码到管理界面：

![](2.PNG)

它已经配置了一些功能。在这里，我们可以添加用户和组的权限管理，这些概念在后面将会探讨。

添加`Board`模型，在`boards`目录中的`admin.py`文件中添加以下代码：

**boards/admin.py**
```python
from django.contrib import admin
from .models import Board
# Register your models here.

admin.site.register(Board)
```

保存`admin.py`文件，然后刷新页面。

![](3.PNG)

可以新增板块。

![](4.PNG)

查看主页，新的板块已经添加。

![](5.PNG)




[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


