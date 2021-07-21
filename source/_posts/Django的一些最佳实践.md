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

继承`BaseUserManager`和`AbstractBaseUser`，指定`AUTH_USER_MODEL`。

```python
class User(AbstractUser):
    """自定义用户模型"""
    nickname = models.CharField(null=True, blank=True,max_length=255, verbose_name="昵称")
```

如果要求显示用户的昵称：

```python 
class Meta:
    verbose_name = '用户'
    verbose_name_plural = verbose_name
	username_filed = 'nickname'
    require_fields = ['', '']
```

定义用户的权限：

```pytho 
def has_perm(self, perm, obj=None):
	"""定义用户具体的权限"""
	pass
```

自定义用户管理：

```python
class MyUserManager(BaseUserManager):
    def creat_user(self, username, password=None):
        pass
    	return
   	def creat_superuser(self, username, password):
        pass
    	return user
```

`objects = MyUserManager()`放到`User`类的上面。

# 优先使用通用类视图（`Class-based Generic Views`）

## 函数视图（`FBV`）

```python
# 函数视图
def my_view(request):
    if request.method == 'GET':
        return HttpResponse('result')
    elif request.method == 'POST':
        return HttpResponse('result')
    elif request.method == 'DELETE':
        return HttpResponse('result')
```

## 类视图（`CBV`）

```python
# 类视图
from django.views.generic import View


class MyView(View):
    def get(self, request, *args, **kwargs):
        pass

    def post(self, request, *args, **kwargs):
        pass
```

所有基于类的视图都有`as_view`方法，用来作为类调用的入口。

```python
urlpatterns = [
    path('myview', MyView.as_view(a=1), name='detail')
]
```

## 通用类视图（`CBGV`）



# 在系统环境变量中保存敏感信息

项目中的敏感信息和项目分离。

进入项目目录，安装一个包

```shell
 pipenv install django-environ --skip-lock
```

在项目文件夹`.env`文件，定义了项目的绝大部分的项目配置。

# 为不同环境分别配置`settings.py`文件

开发，测试，生产环境的配置文件分离。











[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


