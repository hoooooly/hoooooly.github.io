---
title: FastAPI教程-入门介绍
tags:
  - fastapi
  - python
comments: true
toc: true
only:
  - home
  - category
  - tag
date: 2021-05-30 19:57:07
categories: FastAPI
pic:
---


## 第一步

最简单的 `FastAPI` 文件可能像下面这样：

最简单的 FastAPI 文件可能像下面这样：

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
async def root():
    return {"message": "Hello World"}
```

将其复制到 `main.py` 文件中。

运行实时服务器：

```shell
uvicorn main:app --reload
```

{% colorpanel info Note %}

uvicorn main:app 命令含义如下:

- main：main.py 文件（一个 Python「模块」）。
- app：在 main.py 文件中通过 app = FastAPI() 创建的对象。
- \-\-reload：让服务器在更新代码后重新启动。仅在开发时使用该选项。

{% endcolorpanel %}

## 分步概括

### 步骤1：导入FastAPI

```python
from fastapi import FastAPI
```

`FastAPI` 是一个为 `API` 提供了所有功能的 `Python` 类。

{% colorpanel info 技术细节 %}

FastAPI 是直接从 Starlette 继承的类。
你可以通过 FastAPI 使用所有的 Starlette 的功能。

{% endcolorpanel %}

### 步骤 2：创建一个 FastAPI「实例」

```python
app = FastAPI()
```

这里的变量 `app` 会是 `FastAPI` 类的一个「实例」。

这个实例将是创建你所有 `API` 的主要交互对象。

这个 `app` 同样在如下命令中被 `uvicorn` 所引用：

```python
uvicorn main:app --reload
```

如果你像下面这样创建应用：

```python
from fastapi import FastAPI

my_awesome_api = FastAPI()

@my_awesome_api.get("/")
async def root():
    return {"message": "Hello World"}
```

将代码放入 `main.py` 文件中，然后你可以像下面这样运行 `uvicorn`：

```python
uvicorn main:my_awesome_api --reload
```

### 步骤 3：创建一个路径操作

#### 路径

这里的「路径」指的是 `URL` 中从第一个 `/` 起的后半部分。

所以，在一个这样的 `URL` 中：

```python
https://example.com/items/foo
```

...路径会是：

```python
/items/foo
```

{% colorpanel info info %}

「路径」也通常被称为「端点」或「路由」。

{% endcolorpanel %}

开发 `API` 时，「路径」是用来分离「关注点」和「资源」的主要手段。

#### 操作

这里的「操作」指的是一种 `HTTP「方法」`。

下列之一：

- POST
- GET
- PUT
- DELETE

...以及更少见的几种：

- OPTIONS
- HEAD
- PATCH
- TRACE

在 `HTTP` 协议中，你可以使用以上的其中一种（或多种）「方法」与每个路径进行通信。

在开发 `API` 时，通常使用特定的 `HTTP` 方法去执行特定的行为。

通常使用：

- POST：创建数据。
- GET：读取数据。
- PUT：更新数据。
- DELETE：删除数据。

因此，在 `OpenAPI` 中，每一个 `HTTP` 方法都被称为`「操作」`。

#### 定义一个路径操作装饰器

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def root():
    return {"message": "Hello World"}
```

`@app.get("/")`告诉 `FastAPI` 在它下方的函数负责处理如下访问请求：

- 请求路径为 `/`
- 使用 `get` 操作

{% colorpanel info "@decorator Info" %}

`@something`语法在 `Python` 中被称为「装饰器」。

装饰器接收位于其下方的函数并且用它完成一些工作。

在上面的例子中，这个装饰器告诉 `FastAPI` 位于其下方的函数对应着路径 `/` 加上 `get` 操作。

它是一个「路径操作装饰器」。

{% endcolorpanel %}

### 步骤 4：定义路径操作函数

这是「路径操作函数」：

- 路径：是 `/`。
- 操作：是 `get`。
- 函数：是位于「装饰器」下方的函数（位于` @app.get("/") `下方）。

```python
from fastapi import FastAPI
app = FastAPI()

@app.get("/")
async def root():
    return {"message": "Hello World"}
```

这是一个 `Python` 函数。

每当 `FastAPI` 接收一个使用 `GET` 方法访问 `URL「/」`的请求时这个函数会被调用。

在这个例子中，它是一个 `async` 函数。当然也可以定义为常规函数。

### 步骤 5：返回内容

```python
return {"message": "Hello World"}
```

你可以返回一个 `dict`、`list`，像 `str`、`int` 一样的单个值，等等。

你还可以返回 `Pydantic` 模型（稍后你将了解更多）。

还有许多其他将会自动转换为 `JSON` 的对象和模型（包括 `ORM` 对象等）。


