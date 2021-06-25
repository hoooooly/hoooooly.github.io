---
title: Django入门-表单处理
tags:
  - Hexo
  - Fluid
comments: true
date: 2021-06-25 16:36:57
categories:
index_img: /2021/06/25/Django入门-表单处理/1.webp
banner_img:
typora-root-url: Django入门-表单处理
---

`Forms(表单)`用来处理我们的输入。这在任何`web`应用或者网站中都有很常见的任务。标准的做法通过`HTML`表单实现，用户输入一些数据，提交给服务器，然后服务器处理它。

`Django Forms API` 使整个过程变的更加简单，从而实现了大量工作的自动化。而且，最终的结果比大多数程序员自己去实现的代码更加安全。所以，不管 `HTML` 的表单多么简单，总是使用`Form API`。

## 手动实现一个表单

首先，创建一个新的`URL`路由，命名为`new_topic`：

**boards/urls.py**

```pyth
from django.urls import path, re_path
from . import views

urlpatterns = [
    path('', views.home, name='home'),
    re_path(r'^boards/(?P<pk>\d+)/$', views.board_topics, name='board_topics'),
    re_path(r'^boards/(?P<pk>\d+)/new/$', views.new_topic, name='now_topics'),
]
```

创建`new_topic`的视图函数：

**boards/views.py**

```python
from django.shortcuts import render, get_object_or_404
from .models import Board


def home(request):
    boards = Board.objects.all()
    return render(request, 'home.html', {'boards': boards})


def board_topics(request, pk):
    board = Board.objects.get(pk=pk)
    return render(request, 'topics.html', {'boards': board})


def new_topic(request, pk):
    board = get_object_or_404(Board, pk=pk)
    return render(request, 'new_topics.html', {'board': board})
```

接下来命名一个名为`new_topic.html`的模板：

```python
{% extends 'base.html' %}

{% block title %}Start a new Topic{% endblock %}

{% block breadcrumb %}
<li class="breadcrumb-item"><a href="{% url 'home' %}">Boards</a></li>
<li class="breadcrumb-item"><a href="{% url 'board_topics' board.pk %}">{{ board.name }}</a></li>
<li class="breadcrumb-item active">New topic</li>
{% endblock %}

{% block content %}

{% endblock %}
```

接下来，创建表单：

**new_topic.html**

```html
{% extends 'base.html' %}

{% block title %}Start a new Topic{% endblock %}

{% block breadcrumb %}
<li class="breadcrumb-item"><a href="{% url 'home' %}">Boards</a></li>
<li class="breadcrumb-item"><a href="{% url 'board_topics' board.pk %}">{{ board.name }}</a></li>
<li class="breadcrumb-item active">New topic</li>
{% endblock %}

{% block content %}
    <form method="post">
        {% csrf_token %}
        <div class="form-group">
            <label for="id_subject">Subject</label>
            <input type="text" class="form-control" id="id_subject" name="subject">
        </div>
        <div class="form-group">
            <label for="id_message">Message</label>
            <textarea class="form-control" id="id_message" name="message" rows="5"></textarea>
        </div>
        <button type="submit" class="btn btn-success">Post</button>
    </form>
{% endblock %}
```

![image-20210625174259256](image-20210625174259256.png)

在`<form>`标签中，我们定义了`method`属性。它会告诉浏览器我们想如何与浏览器通信。`HTTP`规范定义了几种`request methods`(请求方法)。大部分情况下，只需要使用`GET`和`POST`两种`requests`（请求）类型。

`GET`是最常见的请求类型。它用于从服务器请求数据。每当你点击了一个链接或者直接在浏览器中输入了一个网址，你就创建了一个`GET`请求。

`POST`用于当我们想更改服务器上的数据的时候，一般来说，每次我们发送数据给服务器都会导致资源状态的变化。

`Django`使用`CSPF Token`(Cross-Site Request Forgery Token)`保护所有的`POST`请求。这是避免外部站点或者应用程序向我们的应用程序提交数据的安全措施。应用程序每次接受一个POST时，都先检查`CSPF Token`。如果这个request没有token，或者这个token是无效的，它会抛弃提交的数据。

`CSPF Token`的模板标签：

```python
{% csrf_token %}
```

它是与其他表单数据一起提交的隐藏字段。

另外，我们需要设置HTML输入`name`，`name`将被用来在服务器获取数据。

下面示范我们如何检索数据：

```py
subject = reqeust.POST['subject']
message = reqeust.POST['message']
```

从HTML获取数据并且开始一个型的`topic`视图的简单实现如下：

```pyth
```

这里发生了一些问题，先回头再深入学习下ORM模型。














[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


