---
title: Flask快速上手
tags:
  - flask
comments: true
toc: true
only:
  - home
  - category
  - tag
date: 2021-05-30 14:56:17
categories: Flask
pic:
---

## 一个最小的应用

一个最小的`Flask`应用如下:

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, World!'
```

那么，这些代码是什么意思呢？

1. 首先我们导入了`Flask`类。 该类的实例将会成为我们的`WSGI`应用。

2. 接着我们创建一个该类的实例。第一个参数是应用模块或者包的名称。如果你使用一个单一模块（就像本例），那么应当使用`__name__`，因为名称会根据这个模块是按应用方式使用还是作为一个模块导入而发生变化（可能是 ‘`__main__`’ ， 也可能是实际导入的名称）。这个参数是必需的，这样`Flask`才能知道在哪里可以找到模板和静态文件等东西。更多内容详见`Flask`文档。

3. 使用`route()`装饰器来告诉`Flask`触发函数的`URL`。

4. 函数名称被用于生成相关联的`URL` 。函数最后返回需要在用户浏览器中显示的信息。

可以使用`flask`命令或者`python`的`-m`开关来运行这个应用。在运行应用之前，需要在终端里导出`FLASK_APP`环境变量:

```shell
$ export FLASK_APP=hello.py
$ flask run
 * Running on http://127.0.0.1:5000/
```

如果是在`Windows`下，那么导出环境变量的语法取决于使用的是哪种命令行解释器。 在`Command Prompt`下:

```shell
C:\path\to\app>set FLASK_APP=hello.py
```

还可以使用`python -m flask`:

```shell
$ export FLASK_APP=hello.py
$ python -m flask run
 * Running on http://127.0.0.1:5000/
```

这样就启动了一个非常简单的内建的服务器。这个服务器用于测试应该是足够了，但是用于生产可能是不够的。

## 路由

现代`web`应用都使用有意义的`URL`，这样有助于用户记忆，网页会更得到用户的青睐， 提高回头率。

使用`route()`装饰器来把函数绑定到`URL`:

```python
@app.route('/')
def index():
    return 'Index Page'

@app.route('/hello')
def hello():
    return 'Hello, World'
```

