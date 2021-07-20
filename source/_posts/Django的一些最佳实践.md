---
title: Django的一些最佳实践
tags:
  - django
  - python
comments: true
typora-root-url: Django的一些最佳实践
date: 2021-07-21 00:19:30
categories:
index_img:
banner_img:
---

# `Pipenv`项目管理

首先，安装`pipenv`。

```python
pip3 install pipenv
```

创建文件夹`mkdir myproject`。

进入文件下`cd myproject`，创建虚拟环境`pipenv --python 3.7`，虚拟环境的`python`版本必须在系统中安装。

需要在虚拟环境中安装包的话只需要在项目目录中使用`pipenv install 包名`即可。

比如这里安装`pipenv install dajngo`==2.2，这个过程有点慢，因为这里没有更改`pipenv`源。

在创建虚拟环境后，会在当前文件夹下生成一个名为`Pipfile`的文件。

```shell
[[source]]
url = "https://pypi.org/simple"
verify_ssl = true
name = "pypi"

[packages]

[dev-packages]

[requires]
python_version = "3.9"
```

在这里我们可以修改`[source]`中的`url`为其他源。`[dev-packages]`指开发环境中需要用到的包，`[packages]`指生产环境中需要用到的包。

```python
url = "https://mirrors.aliyun.com/pypi/simple"
```

安装包后文件下面会生成`Pipfile.lock`文件，显示包安装的哈希值。

使用`pipenv graph`会显示包与包之间的依赖关系。

```shell
$ pipenv graph
Django==2.2
  - pytz [required: Any, installed: 2021.1]
  - sqlparse [required: Any, installed: 0.4.1]
```

每次使用`pipenv`安装包的时候会更新`Pipfile`和`Pipfile.lock`这两个文件。

可以使用`pipenv install requests --skip-lock`跳过`lock`文件的更新，在安装完所有的包后一次更新。

使用`pipenv`不需要每次都激活虚拟环境。

如果你需要激活虚拟环境使用`pipenv shell`即可，退出使用`exit`命令。

如果你想安装开发环境使用的包，比如测试用的包。

```python
pipenv install --dev pytest --skip-lock
```

这个包就会安装到生产环境。

`pipenv --where`定位项目路径。

`pipenv --venv`定位虚拟环境保持的路径。

`pipenv --py`虚拟解释器的路径。

`pipenv --update`更新所有的包。

`pipenv --rm`删除虚拟环境。

`pipenv --check`检查包的安全漏洞。

# 自定义用户模型





# 优先使用通用类视图（`Class-based Generic Views`）



# 在系统环境变量中保存敏感信息















[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


