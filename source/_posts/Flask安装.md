---
title: Flask安装
tags:
  - flask
  - python
comments: true
toc: true
only:
  - home
  - category
  - tag
date: 2021-05-30 14:24:17
categories: Flask
pic:
---

# 安装

## Python版本

推荐使用最新版本的`Python3` 。 `Flask`支持`Python 3.5`及更高版本的`Python 3` 、`Python 2.7`和`PyPy`。

## 依赖

当安装`Flask`时，以下配套软件会被自动安装。

- `Werkzeug`用于实现`WSGI`，应用和服务之间的标准`Python`接口。

- `Jinja`用于渲染页面的模板语言。

- `MarkupSafe`与`Jinja`共用，在渲染页面时用于避免不可信的输入，防止注入攻击。

- `ItsDangerous`保证数据完整性的安全标志数据，用于保护`Flask`的`session cookie`.

- `Click`是一个命令行应用的框架。用于提供`flask`命令，并允许添加自定义管理命令。

## 可选依赖

以下配套软件不会被自动安装。如果安装了，那么`Flask`会检测到这些软件。

- `Blinker`为信号提供支持。

- `SimpleJSON`是一个快速的`JSON`实现，兼容`Python’s json`模块。如果安装了这个软件，那么会优先使用这个软件来进行`JSON`操作。

- `python-dotenv`当运行`flask`命令时为通过`dotenv`设置环境变量 提供支持。

- `Watchdog`为开发服务器提供快速高效的重载。

## 虚拟环境

建议在开发环境和生产环境下都使用虚拟环境来管理项目的依赖。

为什么要使用虚拟环境？随着你的Python项目越来越多，你会发现不同的项目会需要不同的版本的Python库。同一个Python库的不同版本可能不兼容。

虚拟环境可以为每一个项目安装独立的Python库，这样就可以隔离不同项目之间的Python库，也可以隔离项目与操作系统之间的Python库。

Python3内置了用于创建虚拟环境的`venv`模块。如果你使用的是较新的Python版本，那么可以按照下面的步骤创建虚拟环境。

## 创建一个虚拟环境

创建一个项目文件夹，然后创建一个虚拟环境。创建完成后项目文件夹中会有一个`venv`文件夹：

```shell
mkdir myproject
cd myproject
python3 -m venv venv
```

在`Windows`下：

```shell
py -3 -m venv venv
```

在老版本的`Python`中要使用下面的命令创建虚拟环境：

```shell
python2 -m virtualenv venv
```

在`Windows`下：

```shell
\Python27\Scripts\virtualenv.exe venv
```

### 激活python环境

在开始工作前，先要激活相应的虚拟环境：

```shell
. venv/bin/activate
```

在`Windows`下：

```shell
venv\Scripts\activate
```

激活后，你的终端提示符会显示虚拟环境的名称。

{% alertbox info "如果你使用的VS code，要在cmd命令行下激活虚拟环境，如果是power shell则不可以。" %}

## 安装Flask

在已激活的虚拟环境中可以使用如下命令安装`Flask`：

```python
pip install Flask
```

`Flask`现在已经安装完毕。
