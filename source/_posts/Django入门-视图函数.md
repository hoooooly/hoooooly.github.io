---
title: Django入门-视图函数
tags:
  - django
comments: true
toc: true
only: 
  - home
  - category
  - tag
date: 2021-06-11 13:45:32
categories: Django
index_img: /2021/06/11/Django入门-视图函数/Screenshot_8.webp
---

## 第一个视图函数

第一个视图函数目前我们已经有一个视图函数叫 `home` ，这个视图在我们的应用程序主页上显示为`"Hello， World!"` 。

**boards/views.py**
```python
from django.http import HttpResponse

def home(request):
    return HttpResponse('hello World')
```

首先要做的是导入Board模型并列出所有的板块。

**boards/views.py**
```python
from django.http import HttpResponse
from .models import Board

def home(request):
    boards = Board.objects.all()
    boards_names = list()

    for board in boards:
        boards_names.append(board.name)
    
    response_html = '<br>'.join(boards_names)
    return HttpResponse(response_html)
```

访问[http://127.0.0.1:8000/](http://127.0.0.1:8000/)如下：

![](Screenshot_1.webp)

当然，真正的项目里面不会这样去渲染HTML，渲染的部分会交给`Django模板引擎`来实现。

## 模板引擎

在`manage.py`所在的目录创建一个名为`templates`的新文件夹：

在`templates`文件夹中，创建一个名为`home.html`的`HTML`文件。

**templates/home.html**
```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>Boards</title>
    </head>
    <body>
        <h1>Boards</h1>

        {% for board in boards %}
            {{board.name}} <br>
        {% endfor %}
    </body>
</html>
```

上面的例子中，混入了原始的`HTML`和一些特殊的标签`{% for ... in ... %}`和`{{variable}}`。它们是Django模板语法的一部分。上面的列子展示了如何使用`for`遍历列表对象。`{{board.name}}`会在HTMl模板中会被渲染成板块的名称，最后生成动态HTML文档。

在我们可以使用这个HTML页面之前，我们必须告诉Django在哪里可以找到我们应用程序的模板。

打开`myproject目录`下面的`settings.py`文件，搜索 `TEMPLATES` 变量，并设置 `DIRS` 的值为 `os.path.join（BASE_DIR，'templates'）`∶

本质上，刚添加的这一行所做的事情就是找到项目的完整路径并在后面附加`"templates"`。

我们可以使用`Python shell`进行调试∶

```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```

打开`python shell`进行调试。

```python
python manage.py shell
>>> from django.conf import settings
>>> settings.BASE_DIR 
WindowsPath('C:/Users/espho/PycharmProjects/PythonProjs/django/myproject/myproject')
>>> import os
>>> os.path.join(settings.BASE_DIR, 'templates') 
'C:\\Users\\espho\\PycharmProjects\\PythonProjs\\django\\myproject\\myproject\\templates'
```

可以看到，它只是指向了我们在前面步骤中创建的`templates`文件夹。

现在我们可以更新视图：

**boards/views.py**
```python
from django.shortcuts import render
from .models import Board

def home(request):
    boards = Board.objects.all()
    return render(request, 'home.html', {'boards': boards})
```

重新运行，访问[http://127.0.0.1:8000/](http://127.0.0.1:8000/)。

![](Screenshot_2.webp)


可以修改HTML文件，使其更加美观。

**templates/home.html**
```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>Boards</title>
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

不用重启，刷新页面：

![](Screenshot_3.webp)










[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


