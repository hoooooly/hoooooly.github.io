---
title: Django入门-复用模板
tags:
  - django
  - python
comments: true
date: 2021-06-25 15:04:36
categories: Django
index_img: /2021/06/25/Django入门-复用模板/1.webp
banner_img:
typora-root-url: Django入门-复用模板
---

在`templates`文件夹中创建一个名为`base.html`的文件：

```html
{% load static %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{% block title %}Django Boards{% endblock %}</title>
    <link rel="stylesheet" href="{% static 'css/bootstrap.min.css' %}">
    </head>
    <body>
        <div class="container">
            <ol class="breadcrumb my-4">
                {% block breadcrumb %}
                {% endblock %}
            </ol>
            {% block content %}
            {% endblock %}
            </div>
    </body>
</html>
```

把这个作为母版页，我们每个创建的模板都`extend`（继承）这个特殊的模板。`{% block %}`标签用于在模板中间保留一个空间，一个"字"模板（继承这个母版页的模板）可以在这个空间中插入代码和HTML。

在`{% block title %}`中设置了一个默认值"`Django Boards`"，如果在其他模板中未设置`{% block title %}`的值就会使用这个默认值。

接下来重写下这两个模板：`home.html`和`topics.html`。

**home.html**

```html
{% extends 'base.html' %}
{% block breadcrumb %}
    <li class="breadcrumb-item active">Boards</li>
{% endblock %}

{% block content %}
<table class="table">
    <thead class="thead-inverse">
        <tr>
            <th>Board</th>
            <th>Posts</th>
            <th>Topics</th>
            <th>Last Post</th>
        </tr>
    </thead>
    <tbody>
        {% for board in boards %}
            <tr>
                <td>
                    <a href="{%url 'board_topics' board.pk%}">{{ board.name }}</a><br/>
                    <small style="text-muted d-block">{{board.description}}</small>
                </td>
                <td>0</td>
                <td>0</td>
                <td></td>
            </tr>
    {% endfor %}
    </tbody>
</table>
{% endblock %}
```

**topics.html**

```html
{% extends 'base.html' %}
{% block title %}
    {{ board.name }} - {{ block.super }}
{% endblock %}
{% block breadcrumb %}
    <li class="breadcrumb-item"><a href="% url 'home' &"></a></li>
    <li class="breadcrumb-item active">{{ boards.name }}</li>
{% endblock %}

{% block content %}

{% endblock %}
```

有了`base.html`模板，现在可以很方便的添加一个菜单快：

**templates/base.html**

```html
{% load static %}<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{% block title %}Django Boards{% endblock %}</title>
    <link rel="stylesheet" href="{% static 'css/bootstrap.min.css' %}">
    </head>
    <body>
        <nav class="navbar navbar-expand navbar-dark bg-dark">
            <div class="container">
                <a class="navbar-brand" href="{% url 'home' %}">Django Borads</a>
            </div>
        </nav>
        <div class="container">
            <ol class="breadcrumb my-4">
                {% block breadcrumb %}
                {% endblock %}
            </ol>
            {% block content %}
            {% endblock %}
        </div>
    </body>
</html>
```

效果如下：

![image-20210625160119695](image-20210625160119695.png)












[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


