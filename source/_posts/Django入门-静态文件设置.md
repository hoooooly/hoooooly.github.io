---
title: Django入门-静态文件设置
tags:
  - django
  - python
comments: true
toc: true
only:
  - home
  - category
  - tag
date: 2021-06-16 00:01:36
categories: Django
index_img: /2021/06/16/Django入门-静态文件设置/Screenshot_8.webp
---

静态文件设置静态文件是指 `CSS`，`JavaScript`，`字体`，`图片`或者是用`来组成用户界面的任何其他资源`。


`Django` 本身是不负责处理这些文件的，但是为了让我们的开发过程更轻松，`Django` 提供了一些功能来帮助我们管理静态文件。这些功能可在 `INSTALLED_APPS` 的` django.contrib.staticfiles` 应用程序中找到。实际上，`Django`为了使得开发方便，也可以处理静态文件，而在生产环境下，静态文件一般直接由 `Nginx` 等反向代理服务器处理，而应用工服务器专心负责处理它擅长的业务逻辑。

市面上很多优秀前端组件框架，我们没有理由继续用简陋的HTML文档来渲染。我们可以轻松地将`Bootstrap 4`添加到我们的项目中。`Bootstrap`是—个用`HTML`，`CSS`和`JavaScript`开发的前端开源工具包。

在项目根目录中，除了 `boards`， `templates` 和`myproject`文件夹外，再创建一个名为`static`的新文件夹，并在`static`文件夹内创建另一个名为`css`的文件夹∶

转到[getbootstrap.com](getbootstrap.com)并下载最新版本：

![](1.png)

下载编译版本的`CSS`和`JS`

在你的计算机中，解压 `bootstrap-5.0.1-dist` 文件，将文件 `css/bootstrap.min.css `复制到我们项目的`css`文件夹中∶


下一步是告诉`Django`在哪里可以找到静态文件。打开`settings.py`，拉到文件的底部，在`STATIC_URL`后面添加以下内容∶

```python
STATIC_URL = '/static/'

STATICFILES_DIRS = [
    os.path.join(BASE_DIR, "static"),
]
```

还记得 `TEMPLATES`目录吗，和这个配置是一样的，现在我们必须在模板中加载静态文件（`Bootstrap CSS`文件）。


**templates/home.html**
```html
{% load static %}
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>Boards</title>
        <link rel="stylesheet" href="{% static 'css/bootstrap.min.css' %}">
    </head>
    <body>
        <h1>Boards</h1>

        <table border="1">
            <thead>
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
                            {{board.name}} <br>
                            <small style="color: #888;">{{board.description}}</small>
                        </td>
                        <td>0</td>
                        <td>0</td>
                        <td></td>
                    </tr>
            {% endfor %}
            </tbody>
        </table>
    </body>
</html>
```

首先，我们在模板的开头使用了 `Static Files App` 模板标签 `{% load static%}`。

模板标签 `{% static %}` 用于构成资源文件完整URL。在这种情况下， `{% static 'css/bootstrap.min.css' %}` 将返回`/static/css/bootstrap.min.css`，它相当于 `http://127.0.0.1:8000/static/css/bootstrap.min.css`。

`{% static %}`模板标签使用 `settings.py` 文件中的 `STATIC_uRL` 配置来组成最终的URL，例如，如果您将静态文件托管在像`https∶//static.example.com/`这样的子域中，那么我们将设置 `STATIC_URL=https∶//static.example.com/` ，然后 `{% static'css/bootstrap.min.css' %}` 返回的是`https://static.example.com/css/bootstrap.min.css`。记得但凡是需要引用`CSS`， `JavaScript`或图片文件的地方就使用 `{% static %}`稍后，当我们开始部署项目到正式环境时，我们将讨论更多。现在都设置好了。

刷新页面 `http∶//127.0.0.1∶8000`，我们可以看到它可以正常运行∶

现在我们可以编辑模板，以利用`Bootstrap CSS`∶

```html
{% load static %}
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>Boards</title>
        <link rel="stylesheet" href="{% static 'css/bootstrap.min.css' %}">
    </head>
    <body>
        <div class="container">
            <ol class="breadcrumb my-4">
                <li class="breadcrumb-item active">Boards</li>
            </ol>


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
                                {{board.name}} <br>
                                <small style="color: #888;">{{board.description}}</small>
                            </td>
                            <td>0</td>
                            <td>0</td>
                            <td></td>
                        </tr>
                {% endfor %}
                </tbody>
            </table>
            </div>
    </body>
</html>
```

显示效果：

![](2.png)

[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


