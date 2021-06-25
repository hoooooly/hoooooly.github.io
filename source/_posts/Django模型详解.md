---
title: Django模型详解
tags:
  - django
  - model
  - orm
  - python
comments: true
date: 2021-06-25 22:59:37
categories: Django
index_img:
banner_img:
---

首先，快速创建一个`Django`工程。

```python
django-admin startproject learn_test
```

然后在工程里面创建一个叫做`things`的应用。

```pytho
cd learn_test
python manage.py startapp things
```

# 模型

模型准确且唯一的描述了数据。它包含您储存的数据的重要字段和行为。一般来说，每一个模型都映射一张数据库表。

基础：

- 每个模型都是一个 Python 的类，这些类继承 [`django.db.models.Model`](https://docs.djangoproject.com/zh-hans/3.2/ref/models/instances/#django.db.models.Model)
- 模型类的每个属性都相当于一个数据库的字段。
- 利用这些，`Django` 提供了一个自动生成访问数据库的 `API`；请参阅 [执行查询](https://docs.djangoproject.com/zh-hans/3.2/topics/db/queries/)。

## 快速上手

在`things/modles.py`中定义了一个 `Person` 模型，拥有 `first_name` 和 `last_name`:

```python 
from django.db import models


# Create your models here.
class Person(models.Model):
    first_name = models.CharField(max_length=50)
    last_name = models.CharField(max_length=30)
```

`first_name` 和 `last_name` 是模型的 [字段](https://docs.djangoproject.com/zh-hans/3.2/topics/db/models/#fields)。每个字段都被指定为一个类属性，并且每个属性映射为一个数据库列。

上面的 `Person` 模型会创建一个如下的数据库表：

```
CREATE TABLE myapp_person (
    "id" serial NOT NULL PRIMARY KEY,
    "first_name" varchar(30) NOT NULL,
    "last_name" varchar(30) NOT NULL
);
```

一些技术上的说明：

- 该表的名称 `myapp_person` 是自动从某些模型元数据中派生出来，但可以被改写。参阅 [表名称](https://docs.djangoproject.com/zh-hans/3.2/ref/models/options/#table-names) 获取更多信息。
- 一个 `id` 字段会被自动添加，但是这种行为可以被改写。请参阅 [自动设置主键](https://docs.djangoproject.com/zh-hans/3.2/topics/db/models/#automatic-primary-key-fields)。
- 本例子中 `创建数据表` 的语法是 `PostgreSQL` 格式的。值得注意的是，`Django` 依据你在 [配置文件](https://docs.djangoproject.com/zh-hans/3.2/topics/settings/) 中指定的数据库后端生成对应的 `SQL` 语句。

## 使用模型

一旦你定义了你的模型，你需要告诉 `Django` 你准备 *使用* 这些模型。你需要修改设置文件中的 `INSTALLED_APPS` ，在这个设置中添加包含 `models.py` 文件的模块名称。

例如，若模型位于项目中的 `things.models` 模块（ 此包结构由 [`manage.py startapp`](https://docs.djangoproject.com/zh-hans/3.2/ref/django-admin/#django-admin-startapp) 命令创建）， `INSTALLED_APPS` 应设置如下：

```pytho
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'things',
]
```

当你向 `INSTALLED_APPS` 添加新的应用的时候，请务必运行 `manage.py migrate`，此外你也可以先使用以下命令进行迁移 `manage.py makemigrations`。

## 字段

模型中最重要且唯一必要的是数据库的字段定义。字段在类属性中定义。定义字段名时应小心避免使用与 [`模型 API`](https://docs.djangoproject.com/zh-hans/3.2/ref/models/instances/) 冲突的名称， 如 `clean`, `save`, or `delete` 等.














[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


