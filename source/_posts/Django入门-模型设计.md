---
title: Django入门-模型设计
tags:
  - django
  - python
comments: true
toc: true
only:
  - home
  - category
  - tag
date: 2021-06-09 21:29:50
categories: Django
index_img: /2021/06/09/Django入门-模型设计/Screenshot_8.webp
---

## 模型

这些模型基本上代表了应用程序的数据库设计。在本节中要做的是创建 `Django` 所表示的类，这些类就是在上一节中建模的类∶ `Board`，`Topic`和 `Post`。`User` 模型被命名为内置应用叫 `auth`，它以命名空间 `django.contrib.auth` 的形式出现在 `INSTALLED_APPS`配置中。

我们要做的工作都在 `boards/models.py 文`件中。以下是我们在`Django应用程序`中如何表示类图的代码∶

```python
from django.db import models
from django.contrib.auth.models import User

# Create your models here.

class Board(models.Model):
    name = models.CharField(max_length=30, unique=True)
    description = models.CharField(max_length=100)

class Topic(models.Model):
    subject = models.CharField(max_length=255)
    last_updated = models.DateTimeField(auto_now_add=True)
    board = models.ForeignKey(Board, related_name='topics', on_delete=models.CASCADE)
    starter = models.ForeignKey(User, related_name='topics', on_delete=models.CASCADE)

class Post(models.Model):
    message = models.TextField(max_length=4000)
    topic = models.ForeignKey(Topic, related_name='posts', on_delete=models.CASCADE)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now_add=True)
    create_by = models.ForeignKey(User, related_name='posts', on_delete=models.CASCADE)
    updated_by = models.ForeignKey(User, null=True, related_name='+', on_delete=models.CASCADE)
```

所有模型都是`django.db.models.Model`类的子类。每个类将被转换为**数据库表**。每个字段由 `django.db.models.Field`子类（内置在`Django core`）的实例表示，它们并将被转换为数据库的列。

`字段 CharField`，`DateTimeField`等等，都是 `django.db.models.Field` 的子类，包含在`Django`的核心里面-随时可以使用。

在这里，我们仅使用 `CharField``，TextField` ，`DateTimeField`，和`ForeignKey`字段来定义我们的模型。不过在`Django`提供了更广泛的选择来代表不同类型的数据，例如 `IntegerField`，`BooleanField`， `DecimalField`和其它一些字段。

我们会在需要的时候提及它们。

有些字段需要参数，例如`CharField`。我们应该始终设定一个`max_length` 。这些信息将用于创建数据库列。`Django`需要知道数据库列需要多大。该 `max_length` 参数也将被`Django Forms API`用来验证用户输入。

在 `Board` 模型定义中，更具体地说，在 `name` 字段中，我们设置了参数 `unique=True` ，顾名思义，它将强制数据库级别字段的唯一性。

在 `Post` 模型中， `created_at` 字段有一个可选参数， `auto_now_add` 设置为 `True` 。这将告诉Django创建 `Post` 对象时为当前日期和时间。

模型之间的关系使用 `Foreignkey` 字段。它将在模型之间创建一个连接，并在数据库级别创建适当的关系 （注∶ 外键关联）。该 `Foreignkey` 字段需要一个位置参数 `related_name` ，用于引用它关联的模型。（注∶ 例如 `created_by` 是外键字段，关联的`User`模型，表明这个帖子是谁创建的， `related name=posts` 表示在 `User` 那边可以使用 `user.posts` 来查看这个用户创建了哪些帖子）

例如，在 `Topic` 模型中， `board` 字段是 `Board` 模型的 `ForeignKey` 。它告诉`Django`，一个 `Topic` 实例只涉及一个`Board`实例。 `related_name` 参数将用于创建反向关系， `Board` 实例通过属性 `topics` 访问属于这个版块下的 `Topic` 列表。

`Django`自动创建这种反向关系， `related_name` 是可选项。但是，如果我们不为它设置一个名称，`Django`会自动生成它∶`（class_name）_set` 。例如，在 `Board` 模型中，所有 `Topic` 列表将用 `topic_set` 属性表示。而这里我们将其重新命名为了 `topics` ，以使其感觉更自然。

在 `Post` 模型中，该 `updated_by` 字段设置 `related_name='+'`。这指示 `Django`我们不需要这种反向关系，所以它会被忽略（注∶也就是说我们不需要关系用户修改过哪些帖子）。

下面您可以看到类图和Django模型的源代码之间的比较，绿线表示我们如何处理反向关系。

![](Screenshot_1.webp)

## 迁移模型

下一步是告诉`Django`创建数据库，以便我们可以开始使用它。打开终端，激活虚拟环境，转到` manage.py`文件所在的文件夹，然后运行以下命令∶

```python
python manage. py makemigrations
```

你会看到输出的内容是∶

```python
Migrations for 'boards':
  boards\migrations\0001_initial.py
    - Create model Board
    - Create model Topic
    - Create model Post
```

此时， `Django` 在 `boards/migrations` 目录创建了一个名为 `0001_initial.py` 的文件。它代表了应用程序模型的当前状态。在下一步，`Django`将使用该文件创建表和列。

迁移文件将被翻译成SQL语句。如果您熟悉SQL，则可以运行以下命令来检验将是要被数据库执行的SQL指令

```python
python manage.py sqlmigrate boards 0001
```

如果你不熟悉SQL，也不要担心。所有的工作都将使用`Django ORM`来完成，它是一个与数据库进行通信的抽象层。下一步是将我们生成的迁移文件应用到数据库∶

```python
python manage.py migrate
```

输出结果如下：

```python
Operations to perform:
  Apply all migrations: admin, auth, boards, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying admin.0003_logentry_add_action_flag_choices... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying auth.0009_alter_user_last_name_max_length... OK
  Applying auth.0010_alter_group_name_max_length... OK
  Applying auth.0011_update_proxy_permissions... OK
  Applying auth.0012_alter_user_first_name_max_length... OK
  Applying boards.0001_initial... OK
  Applying sessions.0001_initial... OK
```

好了数据库就迁移成功了。

## 试验 Models API

使用Python进行开发的一个重要优点是`交互式shell`。我一直在使用它。这是一种快速尝试和试验API的方法。

可以使用`manage.py`工具加载我们的项目来启动 `Python shell` ∶

```python
python manage. py shell
```

这与直接输入 `python` 指令来调用交互式控制台是非常相似的，除此之外，项目将被添加到 `sys.path` 并加载Django。这意味着我们可以在项目中导入我们的模型和其他资源并使用它。

让我们从导入`Board类`开始∶

```python
from boards.models import Board
```


要创建新的 `board` 对象，我们可以执行以下操作∶

```python
board = Board(name='Django',description='This is a board abo ut Django.')
```

为了将这个对象保存在数据库中，我们必须调用`save`方法∶

```python
board.save()
```

`save` 方法用于创建和更新对象。这里Django创建了一个新对象，因为这时 `Board` 实例没有`id`。第一次保存后，Django会自动设置`ID`∶

```python
>>> board.id
1
```

您可以将其余的字段当做Python属性访问∶

```python
>>> board.name
'Django'
>>> board.description
'This is a board about django'
```

要更新一个值，我们可以这样做∶

```python
board.description ='Django discussion board'
board.save()
>>> board.description 
'this is new description'
```
每个Django模型都带有一个特殊的属性；我们称之为`模型管理器（Model Manager）`。你可以通过属性 `objects` 来访问这个管理器，它主要用于数据库操作。例如，我们可以使用它来直接创建一个新的`Board`对象∶

```python
>>> board = Board.objects.create(name='Python', description='id 2 for django model')
>>> board.save()
>>> board.id
2
```

所以，现在我们有两个版块了。我们可以使用 `objects` 列出数据库中所有现有的版块

```python
>>> Board.objects.all()
<QuerySet [<Board: Board object (1)>, <Board: Board object (2)>]>
```

结果是一个`QuerySet`。稍后我们会进一步了解。基本上，它是从数据库中查询的对象列表。我们看到有两个对象，但显示的名称是 `Board object`。这是因为我们尚未实现 `Board` 的 `__str___` 方法。

`_str__ `方法是对象的字符串表示形式。我们可以使用版块的名称来表示它。

首先，退出交互式控制台∶

```python
exit()
```

现在编辑`boards app`中的 `models.py` 文件:

```python
class Board(models.Model):
    name = models.CharField(max_length=30, unique=True)
    description = models.CharField(max_length=100)

    def __str__(self):
        return self.name

```

再进入交互台，查询：

```python
<class 'boards.models.Board'>
>>> Board.objects.all()
<QuerySet [<Board: Django>, <Board: Python>]>
```

这样是不是好多了呢。

我们可以将这个`QuerySet`看作一个列表。假设我们想遍历它并打印每个版块的描述∶

```python
boards_list = Board.objects.all() 
>>> board_list = Board.objects.all()
>>> for board in board_list:
...     print(board.description)
... 
this is new description
id 2 for django model
```

同样，我们可以使用模型的 `管理器（Manager）`来查询数据库并返回单个对象。为此，我们要使用 `get` 方法∶

```python
>>> django_board = Board.objects.get(id='1')
>>> django_board
<Board: Django>
>>> django_board.name
'Django'
>>> django_board.description
'this is new description'
```

但我们必须小心这种操作。如果我们试图查找一个不存在的对象，例如，查找`id=3`的版块，它会引发一个异常∶

```python
>>> django_board = Board.objects.get(id='3') 
boards.models.Board.DoesNotExist: Board matching query does not exist.
```

`get` 方法的参数可以是模型的任何字段，但最好使用可唯一标识对象的字段来查询。否则，查询可能会返回多个对象，这也会导致异常。

```python
Board.objects.get(name='Django')
<Board: Django>
```

请注意，查询区分大小写，小写"django"不匹配∶

```Python
Board.objects.get (name='django')
boards.models.DoesNotExist: Board matching query does not exist.
```








[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


