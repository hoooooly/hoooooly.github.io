---
title: 使用Flask-Bootstrap集成Bootstrap
tags:
  - flask
  - python
comments: true
typora-root-url: 使用Flask-Bootstrap集成Bootstrap
date: 2021-07-30 08:43:53
categories: Flask
index_img:
banner_img:
---

Bootstrap是Twitter开发的一个开源Web框架，它提供的用户界面组件可用于创建整洁且具有吸引力的网页，而且兼容所有现代的桌面和移动平台Web浏览器。

## 安装Flask-Bootstrap扩展

```python
(venv)$pip install flask-bootstrap
```

Flask扩展在创建应用实例时初始化。

```python
from flask_bootstrap import Bootstrap
# ...

app = Flask(__name__)
Bootstrap(app)

# ...
```

## 使用Bootstrap模板

在`index.html`中使用:

```HTML
{% extends 'bootstrap/base.html' %}

{% block title %}主页{% endblock %}

{% block navbar %}
<div class="navbar navbar-inverse" role="navigation">
    <div class="container">
        <div class="navbar-header">
            <button type="button" class="navbar-toggle" data-toggle="collapse" data-target=".navbar-collapse">
                <span class="sr-only">Toggle navigation</span>
                <span class="icon-bar"></span>
                <span class="icon-bar"></span>
                <span class="icon-bar"></span>
            </button>
            <a class="navbar-brand" href="/">主页</a>
        </div>
        <div class="navbar-collapse collapse">
            <ul class="nav navbar-nav">
                <li><a href="/">Home</a></li>
            </ul>
        </div>
    </div>
</div>


{% endblock %}
```

Jinja2中的extends指令从Flask-Bootstrap中导入bootstrap/base.html，从而实现模板继承。Flask-Bootstrap的基模板提供了一个网页骨架，引入了Bootstrap的所有CSS和JavaScript文件。

上面这个index.html模板定义了3个区块，分别名为`title`、`navbar`和`content`。这些区块都是基模板提供的，可在衍生模板中重新定义。`title`区块的作用很明显，其中的内容会出现在渲染后的HTML文档头部，放在`<title>`标签中。`navbar`和`content`这两个区块分别表示页面中的导航栏和主体内容。在这个模板中，navbar区块使用Bootstrap组件定义了一个简单的导航栏。content区块中有个`<div>`容器，其中包含一个页头。之前版本中的欢迎消息，现在就放在这个页头里。

![](image-20210730110812043.png)

## Bootstrap基模板中定义的模块

| 区块名 | 说明 |
| :--- | :---- |
| doc  | 整个HTML文档 |
| html_attribs | <html>标签的属性 |
| html  | <html>标签中的内容 |
| head  | <head>标签中的内容 |
| title | \<title\>标签中的内容 |
| metas  | 一组<meta>标签 |
| styles | CSS声明  |
| body_attribs | <body>标签的属性 |
| body  | <body>标签中的内容  |
| navbar  | 用户定义的导航栏 |
| content | 用户定义的页面内容 |
| scripts | 文档底部的JavaScript声明 |

上面的很多区块都是Flask-Bootstrap自用的，如果直接覆盖可能会导致一些问题。例如，Bootstrap的CSS和JavaScript文件在styles和scripts区块中声明。如果应用需要向已经有内容的块中添加新内容，必须使用Jinja2提供的`super()`函数。例如，如果要在衍生模板中添加新的`JavaScript`文件，需要这么定义scripts区块：

```HTML
{% block scripts %}}
    {{ super() }}
    <script type="text/javascript" src="my.js"></script>
{% endblock %}}
```


[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style> 


