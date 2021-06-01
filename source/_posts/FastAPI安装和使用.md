---
title: FastAPI安装和使用
tags:
  - fastapi
  - python
comments: true
toc: true
only:
  - home
  - category
  - tag
date: 2021-05-30 15:51:17
categories: FastAPI
pic:
---

## 简介

`FastAPI` 是一个用于构建 API 的现代、快速（高性能）的 web 框架，使用 Python 3.6+ 并基于标准的 Python 类型提示。

关键特性:

- 快速：可与 NodeJS 和 Go 比肩的极高性能（归功于 Starlette 和 Pydantic）。最快的 Python web 框架之一。

- 高效编码：提高功能开发速度约 200％ 至 300％。*

- 更少 bug：减少约 40％ 的人为（开发者）导致错误。*
- 智能：极佳的编辑器支持。处处皆可自动补全，减少调试时间。
- 简单：设计的易于使用和学习，阅读文档的时间更短。
- 简短：使代码重复最小化。通过不同的参数声明实现丰富功能。bug 更少。
- 健壮：生产可用级别的代码。还有自动生成的交互式文档。
- 标准化：基于（并完全兼容）API 的相关开放标准：OpenAPI (以前被称为 Swagger) 和 JSON Schema。

## 依赖

`Python 3.6`及更高版本

`FastAPI`站在以下巨人的肩膀之上：

- `Starlette` 负责 `web` 部分。
- `Pydantic` 负责数据部分。

## 安装

```python
pip install fastapi
```

你还会需要一个 `ASGI` 服务器，生产环境可以使用 `Uvicorn` 或者 `Hypercorn`。

```python
pip install uvicorn[standard]
```

## 示例

### 创建

创建一个`main.py`文件并写入以下内容:

```python
from typing import Optional

from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def read_root():
    return {"Hello": "World"}


@app.get("/items/{item_id}")
def read_item(item_id: int, q: Optional[str] = None):
    return {"item_id": item_id, "q": q}
```

{% collapse 或者使用 async def... %}

如果你的代码里会出现 async / await，请使用 async def：

```python
from typing import Optional

from fastapi import FastAPI

app = FastAPI()


@app.get("/")
async def read_root():
    return {"Hello": "World"}


@app.get("/items/{item_id}")
async def read_item(item_id: int, q: Optional[str] = None):
    return {"item_id": item_id, "q": q}
```

{% endcollapse %}

## 运行

通过以下命令运行服务器：

```python
uvicorn main:app --reload
```

## 检查

使用浏览器访问[http://127.0.0.1:8000/items/5?q=somequery](http://127.0.0.1:8000/items/5?q=somequery)。

你将会看到如下`JSON`响应：

```python
{"item_id": 5, "q": "somequery"}
```

你已经创建了一个具有以下功能的 `API`：

- 通过 *路径* `/` 和 `/items/{item_id}` 接受 `HTTP` 请求。
- 以上 *路径* 都接受 `GET` 操作（也被称为 `HTTP` 方法）。
- `/items/{item_id}` *路径* 有一个路径参数 `item_id` 并且应该为 `int` 类型。
- `/items/{item_id}` *路径* 有一个可选的 `str` 类型的 查询参数 `q`。

## 交互式 API 文档

访问[http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)，你会看到自动生成的交互式 `API` 文档（由 Swagger UI生成）：

![](Screenshot_1.webp)

## 示例升级

现在修改`main.py`文件来从 `PUT` 请求中接收请求体。

我们借助 `Pydantic` 来使用标准的 `Python` 类型声明请求体。

```python
from typing import Optional

from fastapi import FastAPI

from pydantic import BaseModel


app = FastAPI()



class Item(BaseModel):

    name: str

    price: float

    is_offer: Optional[bool] = None



@app.get("/")
def read_root():
    return {"Hello": "World"}


@app.get("/items/{item_id}")
def read_item(item_id: int, q: Optional[str] = None):
    return {"item_id": item_id, "q": q}



@app.put("/items/{item_id}")

def update_item(item_id: int, item: Item):

    return {"item_name": item.name, "item_id": item_id}
```

## 交互式 API 文档升级

访问[http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)。

- 交互式 API 文档将会自动更新，并加入新的请求体：

- 点击「Try it out」按钮，之后你可以填写参数并直接调用 API：

- 然后点击「Execute」按钮，用户界面将会和 API 进行通信，发送参数，获取结果并在屏幕上展示：

![](Screenshot_2.webp)