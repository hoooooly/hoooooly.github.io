---
title: flask数据库
tags:
  - flask
  - SQL
comments: true
typora-root-url: flask数据库
date: 2021-08-02 23:42:21
categories: Flask
index_img:
banner_img:

---

# 关系型数据库

关系型数据库把数据存储在**表**中，表为应用中不同的实体建模。例如，订单管理应用的数据库中可能有`customers`、`products`和`orders`等表。

表中的列数是固定的，行数是可变的。列定义表所表示的实体的数据属性。例如，customers表中可能有name、address、phone等列。表中的行定义部分或所有列对应的真实数据。

表中有个特殊的列，称为**主键**，其值为表中各行的唯一标识符。表中还可以有称为**外键**的列，引用同一个表或不同表中某一行的主键。行之间的这种联系称为关系，这正是关系型数据库模型的基础。

```mermaid
classDiagram
	class roles
	roles: id
	roles: name
	
	class users
	users: id
	users: username
	users: password
	users: role_id
	
	roles <|-- users
	
```

数据库结构的这种图示法称为**实体-关系图**。其中，方框表示数据库表，里面列出表的属性（或列）。`roles`表存储所有可用的用户角色，每个角色都使用一个唯一的`id`值（即表的主键）进行标识。`users`表存储用户，每个用户也有唯一的`id`值。除了`id`主键之外，`roles`表中还有`name`列，`users`表中还有`username`和`password`列。

## Python数据库框架

Flask并不限制你使用何种类型的数据库包，因此你可以根据自己的喜好选择使用`MySQL`、`Postgres`、`SQLite`、`Redis`、`MongoDB`、`CouchDB`或`DynamoDB`。

如果这些都无法满足需求，还有一些数据库抽象层代码包供选择，例如`SQLAlchemy`和`MongoEngine`。你可以使用这些抽象包直接处理高等级的Python对象，而不用处理如表、文档或查询语言之类的数据库实体。

## 使用`Flask-SQLAlchemy`管理数据库

`Flask-SQLAlchemy`是一个Flask扩展，简化了在Flask应用中使用`SQLAlchemy`的操作。`SQLAlchemy`是一个强大的关系型数据库框架，支持多种数据库后台。`SQLAlchemy`提供了高层`ORM`，也提供了使用数据库原生`SQL`的低层功能。

安装：

```shell
(venv) $ pip install flask-sqlalchemy
```

在`Flask-SQLAlchemy`中，数据库使用URL指定。

| 数据库引擎             | URL                                                |
| ---------------------- | -------------------------------------------------- |
| `MySQL`                | `mysql://username:password@hostname/databse`       |
| `Postgres`             | `postgresql://username:password@hostname/database` |
| `SQLite(linux, macos)` | `sqlite:////absolute/path/to/database`             |
| `SQLite(windows)`      | `sqlite:///c:/absolute/path/to/database`           |

在这些URL中，`hostname`表示数据库服务所在的主机，可以是本地主机`（localhost）`，也可以是远程服务器。数据库服务器上可以托管多个数据库，因此`database`表示要使用的数据库名。如果数据库需要验证身份，使用`username`和`password`提供数据库用户的凭据。

> `SQLite`数据库没有服务器，因此不用指定`hostname`、`username`和`password`。URL中的`database`是磁盘中的文件名。

应用使用的数据库URL必须保存到Flask配置对象的`SQLALCHEMY_DATABASEURI`键中。`Flask-SQLAlchemy`文档还建议把`SQLALCHEMY_TRACKMODIFICATIONS`键设为False，以便在不需要跟踪对象变化时降低内存消耗。其他配置选项的作用参阅`Flask-SQLAlchemy`的文档。

配置数据库：

```python
# 数据库
import os
from flask_sqlalchemy import SQLAlchemy

basedir = os.path.abspath(os.path.dirname(__file__))

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///' + os.path.join(basedir, 'data.sqlite')
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

db = SQLAlchemy(app)
```

`db`对象是`SQLAlchemy`类的实例，表示应用使用的数据库，通过它可获得`Flask-SQLAlchemy`提供的所有功能。

## 定义模型

模型这个术语表示应用使用的持久化实体。在`ORM`中，模型一般是一个Python类，类中的属性对应于数据库表中的列。

`Flask-SQLAlchemy`创建的数据库实例为模型提供了一个基类以及一系列辅助类和辅助函数，可用于定义模型的结构。

```python
class Role(db.Model):
    __tablename__ = 'roles'
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(64), unique=True)

    def __repr__(self):
        return '<Role %r>' % self.name


class User(db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(64), unique=True, index=True)

    def __repr__(self):
        return '<User %r>' % self.username
```

类变量`tablename`定义在数据库中使用的表名。如果没有定义`tablename` , `Flask-SQLAlchemy`会使用一个默认名称，但默认的表名没有遵守流行的使用复数命名的约定，所以最好由我们自己来指定表名。其余的类变量都是该模型的属性，定义为`db.Column`类的实例。

`db.Column`类构造函数的第一个参数是数据库列和模型属性的类型。

常用的列类型以及在模型中使用的Python类型。

| 类型名         | Python类型           | 说明                                         |
| -------------- | -------------------- | -------------------------------------------- |
| `Integer`      | `int`                | 普通整数，通常是32位                         |
| `SmallInteger` | `int`                | 取值范围小的整数，通常是16位                 |
| `BigInteger`   | `int或long`          | 不限制精度的整数                             |
| `Float`        | `float`              | 浮点数                                       |
| `Numeric`      | `decimal.Decimal`    | 定点数                                       |
| `String`       | `str`                | 变长字符串                                   |
| Text``         | `str`                | 变长字符串，对较长或不限长度的字符串做了优化 |
| `Unicode`      | `unicode`            | 变长Unicode字符串                            |
| `UnicodeText`  | `unicode`            | 变长Unicode字符串                            |
| `Boolean`      | `bool`               | 布尔值                                       |
| `Date`         | `datatime.date`      | 日期                                         |
| `Time`         | `datatime.time`      | 时间                                         |
| `DateTime`     | `datetime.datetime`  | 时间和日期                                   |
| `Interval`     | `datetime.timedelta` | 时间间隔                                     |
| `Enum`         | `str`                | 一组字符串                                   |
| `PickleType`   | `任何Python对象`     | 自动使用`Pcikle`序列化                       |
| `LargeBinary`  | `str`                | 二进制blob                                   |

`db.Column`的其余参数指定属性的配置选项。

| 选项名      | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| primary_key | 如果设为True，列为表的主键                                   |
| unique      | 如果设为True，列不允许出现重复的值                           |
| index       | 如果设为True，为列创建索引，提升查询效率                     |
| nullable    | 如果设置为True，列允许使用空值，如果设为False，列不允许有空值 |
| default     | 为列定义默认值                                               |

> `Flask-SQLAlchemy`要求每个模型都定义主键，这一列经常命名为`id`。

虽然没有强制要求，但这两个模型都定义了`repr()` 方法，返回一个具有可读性的字符串表示模型，供调试和测试时使用。

## 关系

关系型数据库使用关系把不同表中的行联系起来。关系图表示用户和角色之间的一种简单关系。这是角色到用户的**一对多**关系，因为一个角色可属于多个用户，而每个用户都只能有一个角色。

在数据库模型中定义关系：

```python
class Role(db.Model):
   	# ...
    users = db.relationship('Users', backref='role')

class User(db.Model):
	# ...
    role_id = db.Column(db.Integer, db.ForeignKey('roles.id'))
```

关系使用`users`表中的外键连接两行。添加到`User`模型中的`role_id`列被定义为外键，就是这个外键建立起了关系。传给`db.ForeignKey()`的参数`’roles.id’`表明，这列的值是`roles`表中相应行的`id`值。

`db.relationship()`中的`backref`参数向`User`模型中添加一个`role`属性，从而定义反向关系。通过`User`实例的这个属性可以获取对应的`Role`模型对象，而不用再通过`role_id`外键获取。

### 定义关系中的常用配置选项

| 选项名          | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| `backref`       | 在关系的另一个模型中添加反向引用                             |
| `primaryjoin`   | 明确指定两个模型之间使用的联结条件，只在模棱两可的关系中需要指定 |
| `lazy`          | 指定如何加载相关记录，可选值有select（首次访问时加载）、immediate(源对象加载后就加载)、joined（加载记录，使用联结）、subquery（立即加载，但使用子查询）、noload（永不加载）和dynamic（不加载记录，但提供加载记录的查询） |
| `uselist`       | 如果设置为FALSE，不使用列表，而使用标量值                    |
| `order_by`      | 指定关系中记录的排序方式                                     |
| `secondary`     | 指定多对多关系中关联表的名称                                 |
| `secondaryjoin` | `SQLAlchemy`无法自行决定时，指定多对多关系中的二级联结条件   |

## 数据库操作

设置环境变量`set FLASK_APP=run.py`，进入`flask shell`。

```shell
from run import db
db.create_all()
```

现在查看应用目录，你会发现有个名为`data.sqlite`的文件，文件名与配置中指定的一样。如果数据库表已经存在于数据库中，那么`db.create all()`不会重新创建或者更新相应的表。如果修改模型后要把改动应用到现有的数据库中，这一行为会带来不便。更新现有数据库表的蛮力方式是先删除旧表再重新创建：

```shell
db.drop_all()
db.create_all()
```

![](image-20210804124308055.png)

### 插入行

下面这段代码创建一些角色和用户：

```python
>>> from run import Role,User
>>> admin_role = Role(name='Admin')
>>> mod_role = Role(name='Moderator')
>>> user_role = Role(name='User')
>>> user_john = User(username='john', role=admin_role)
>>> user_susan = User(username='susan', role=user_role)
>>> user_david = User(username='david', role=user_role)
```

模型的构造函数接受的参数是使用关键字参数指定的模型属性初始值。注意，role属性也可使用，虽然它不是真正的数据库列，但却是一对多关系的高级表示。新建对象时没有明确设定id属性，因为在多数数据库中主键由数据库自身管理。现在这些对象只存在于Python中，还未写入数据库。因此，id尚未赋值：

```python
>>> print(admin_role.id)
None
```

对数据库的改动通过数据库会话管理，在`Flask-SQLAlchemy`中，会话由`db.session`表示。准备把对象写入数据库之前，要先将其添加到会话中：

```python
>>> db.session.add(admin_role)
>>> db.session.add(mod_role)
>>> db.session.add(user_role)
>>> db.session.add(user_john)
>>> db.session.add(user_susan)
>>> db.session.add(user_david)
```

或者简写成：

```python
>>> db.session.add_all([admin_role,mod_role,user_role,user_john,user_susan,user_david])
```

为了把对象写入数据库，我们要调用`commit()`方法提交会话：

```python
>>> db.session.commit()
```

提交数据后再查看id属性，现在它们已经赋值了：

```python
>>> print(admin_role.id)
1
>>> print(user_john.id)
1
>>> print(user_role.id)
3
```

> 数据库会话`db.session`和`Flask session`对象没有关系。数据库会话也称为事务。
>
> 数据库会话也可回滚。调用`db.session.rollback()`后，添加到数据库会话中的所有对象都将还原到它们在数据库中的状态。

### 修改行

在数据库会话上调用`add()`方法也能更新模型。我们继续在之前的shell会话中进行操作，下面这个例子把`"Admin"`角色重命名为`"Administrator"`：

```python
>>> admin_role.name = 'Administrator'
>>> db.session.add(admin_role)
>>> db.session.commit()
```

### 删除行

数据库会话还有个`delete()`方法。下面这个例子把`"Moderator"`角色从数据库中删除：

```python
>>> db.session.delete(mod_role)
>>> db.session.commit()
```

注意，删除与插入和更新一样，提交数据库会话后才会执行。

### 查询行

`Flask-SQLAlchemy`为每个模型类都提供了`query`对象。最基本的模型查询是使用`all()`方法取回对应表中的所有记录：

```python
>>> Role.query.all()
[<Role 'Administrator'>, <Role 'Moderator'>, <Role 'User'>]
>>> User.query.all()
[<User 'john'>, <User 'susan'>, <User 'david'>]
```

使用**过滤器**可以配置`query`对象进行更精确的数据库查询。下面这个例子查找角色为"`User`"的所有用户：

```python
>>> User.query.filter_by(role_id='3').all()
[<User 'susan'>, <User 'david'>]
```

若想查看`SQLAlchemy`为查询生成的原生`SQL`查询语句，只需把`query`对象转换成字符串：

```python
>>> str(User.query.filter_by(role_id='3'))
'SELECT users.id AS users_id, users.username AS users_username, users.role_id AS users_role_id \nFROM users \nWHERE users.role_id = ?'
```

如果你退出了shell会话，前面这些例子中创建的对象就不会以Python对象的形式存在，但在数据库表中仍有对应的行。如果打开一个新的shell会话，要从数据库中读取行，重新创建Python对象。下面这个例子发起一个查询，加载名为"User"的用户角色：

```python
>>> user_role = Role.query.filter_by(name='User').first()
>>> user_role
<Role 'User'>
```

这里发起查询的不是`all()`方法，而是`first()`方法。`all()`方法返回所有结果构成的列表，而first()方法只返回第一个结果，如果没有结果的话，则返回`None`。因此，如果知道查询最多返回一个结果，就可以用这个方法。

`filter_by()`等过滤器在`query`对象上调用，返回一个更精确的`query`对象。多个过滤器可以一起调用，直到获得所需结果。

常用的过滤器：

| 过滤器      | 说明                                                 |
| ----------- | ---------------------------------------------------- |
| filter()    | 把过滤器添加到原查询上，返回一个新查询               |
| filter_by() | 把等值过滤器添加到原查询上，返回一个新查询           |
| limit()     | 使用指定的值限制原查询返回的结果数量，返回一个新查询 |
| offset()    | 偏移量查询返回的结果，返回一个新查询                 |
| order_by()  | 根据指定条件对原查询结果进行排序，返回一个新查询     |
| group_by()  | 根据指定条件对原查询结果进行分组，返回一个新查询     |

在查询上应用指定的过滤器后，调用`all()`方法将执行查询，以列表的形式返回结果。除了`all()`方法之外，还有其他方法能触发查询执行。

常用的`SQLAlchemy`查询执行方法：

| 方法           | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| all()          | 以列表形式返回所有查询结果                                   |
| first()        | 返回查询的第一个结果，如果没有结果，则返回None               |
| first_or_404() | 返回查询的第一个结果，如果没有结果，则请求终止，返回404错误码 |
| get()          | 返回指定主键对应的行，如果没有对应的行，则返回None           |
| get_or_404()   | 返回指定主键对应的行，如果没有找到指定的主键，则请求终止，返回404错误码 |
| count()        | 返回查询结果的数量                                           |
| pageinate()    | 返回一个Paginate对象，包含指定范围内的结果                   |

关系与查询的处理方式类似。下面这个例子分别从关系的两端查询角色和用户之间的一对多关系：

```python
>>> users = user_role.users
>>> users
[<User 'susan'>, <User 'david'>]
>>> users[0].role
<Role 'User'>
```

这个例子中的`user_role.users`查询有个小问题。执行`user_role.users`表达式时，隐式的查询会调用`all()`方法，返回一个用户列表。此时，`query`对象是隐藏的，无法指定更精确的查询过滤器。就这个示例而言，返回一个按照字母顺序排列的用户列表可能更好。我们修改下关系的设置，加入`lazy='dynamic’`参数，从而禁止自动执行查询。

```python
class Role(db.Model):
    # ...
    users = db.relationship('User', backref='role', lazy='dynamic')
```

这样配置关系之后，`user_role.users`将返回一个尚未执行的查询，因此可以在其上添加过滤器：

```python
>>> user_role.users.order_by(User.username).all()
[<User 'susan'>, <User 'david'>]
>>> user_role.users.count()
2
```

## 在视图函数中操作数据库

在这个修改后的版本中，提交表单后，应用会使用`filter_by()`查询过滤器在数据库中查找提交的名字。变量`known`被写入用户会话中，因此重定向之后，可以把数据传给模板，用于显示自定义的欢迎消息。注意，为了让应用正常运行，必须按照前面介绍的方法，在`Python shell`中创建数据库表。

```python
@app.route('/', methods=['GET', 'POST'])
def index():
    form = LoginForm()
    if form.validate_on_submit():
        user = User.query.filter_by(username=form.name.data).first()
        if user is None:
            user = User(username=form.name.data)
            db.session.add(user)
            db.session.commit()
            session['known'] = False
        else:
            session['known'] = True
        session['name'] = form.name.data
        form.name.data = ''
        return redirect(url_for('index'))
    return render_template('index.html', form=form, name=session.get('name'), known=session.get('known', False),
                           current_time=datetime.utcnow())
```

这个模板使用`known`参数在欢迎消息中加入了第二行，从而对已知用户和新用户显示不同的内容。

```html
<div class="content">
    {% for message in get_flashed_messages() %}
        <div class="alert alert-warning">
            <button type="button" class="close">&times;</button>
            {{ message }}
        </div>
    {% endfor %}
    <h1>Hello, {% if name %}{{ name }}{% else %}Stranger{% endif %}!</h1>
    {% if not known %}
        <p>Pleased to meet you!</p>
    {% else %}
        <p>Pleased to see you again!</p>
    {% endif %}
    {{ wtf.quick_form(form) }}
</div>
```

## 集成Python shell

若想把对象添加到导入列表中，必须使用`shell_context_processor`装饰器创建并注册一个`shell`上下文处理器。

```python
@app.shell_context_processor
def make_shell_context():
    return dict(db=db, User=User, Role=Role)
```

这个`shell`上下文处理器函数返回一个字典，包含数据库实例和模型。除了默认导入的`app`之外，`flask shell`命令将自动把这些对象导入`shell`。

```python
>>> app
<Flask 'run'>
>>> db
<SQLAlchemy engine=sqlite:///D:\PycharmProjects\Flask_Projects\second_app\data.sqlite>
>>> User
<class 'run.User'>
>>> Role
<class 'run.Role'>
```

## 使用Flask-Migrate实现数据库迁移

在开发应用的过程中，你会发现有时需要修改数据库模型，而且修改之后还要更新数据库。仅当数据库表不存在时，`Flask-SQLAlchemy`才会根据模型创建。因此，更新表的唯一方式就是先删除旧表，但是这样做会丢失数据库中的全部数据。

更新表更好的方法是使**数据库迁移**框架。源码版本控制工具可以跟踪源码文件的变化；类似地，数据库迁移框架能跟踪数据库**模式**的变化，然后以增量的方式把变化应用到数据库中。

`SQLAlchemy`的开发人员编写了一个迁移框架，名为`Alembic`。除了直接使用`Alembic`之外，Flask应用还可使用`Flask-Migrate`扩展。这个扩展是对`Alembic`的轻量级包装，并与`flask`命令做了集成。

### 创建迁移仓库

首先，要在虚拟环境中安装`Flask-Migrate`：

```shell
(venv) $ pip install flask-migrate
```

初始化方法：

```python
from flask_migrate import Migrate

# ...

migrate = Migrate(app, db)
```

了开放数据库迁移相关的命令，`Flask-Migrate`添加了flask_db命令和几个子命令。在新项目中可以使用init子命令添加数据库迁移支持：

```shell
(vnev) $ flask db init
Creating directory D:\PycharmProjects\Flask_Projects\second_app\migrations ...  done
Creating directory D:\PycharmProjects\Flask_Projects\second_app\migrations\versions ...  done
Generating D:\PycharmProjects\Flask_Projects\second_app\migrations\alembic.ini ...  done
Generating D:\PycharmProjects\Flask_Projects\second_app\migrations\env.py ...  done
Generating D:\PycharmProjects\Flask_Projects\second_app\migrations\README ...  done
Generating D:\PycharmProjects\Flask_Projects\second_app\migrations\script.py.mako ...  done
Please edit configuration/connection/logging settings in 'D:\\PycharmProjects\\Flask_Projects\\second_app\\migrations\\alembic.ini' before proceeding.
```

这个命令会创建`migrations`目录，所有迁移脚本都存放在这里。

### 创建迁移脚本

在`Alembic`中，数据库迁移用**迁移脚本**表示。脚本中有两个函数，分别是`upgrade()`和`downgrade()`。`upgrade()`函数把迁移中的改动应用到数据库中，`downgrade()`函数则将改动删除。`Alembic`具有添加和删除改动的能力，意味着数据库可重设到修改历史的任意一点。我们可以使用`revision`命令手动创建`Alembic`迁移，也可使用`migrate`命令自动创建。手动创建的迁移只是一个骨架，`upgrade()`和`downgrade()`函数都是空的，开发者要使用`Alembic`提供的`Operations`对象指令实现具体操作。自动创建的迁移会根据模型定义和数据库当前状态之间的差异尝试生成`upgrade()`和`downgrade()`函数的内容。

> 自动创建的迁移不一定总是正确的，有可能会漏掉一些细节。比如说我们重命名了一列，自动生成的迁移可能会把这当作删除了一列，然后又新增了一列。如果原封不动地使用自动生成的迁移，这一列中的数据就会丢失！鉴于此，自动生成迁移脚本后一定要进行检查，把不准确的部分手动改过来。

使用Flask-Migrate管理数据库模式变化的步骤如下。

1. 对模型类做必要的修改。
2.  执行flask db migrate命令，自动创建一个迁移脚本。
3.  检查自动生成的脚本，根据对模型的实际改动进行调整。
4. 把迁移脚本纳入版本控制。
5.  执行flask db upgrade命令，把迁移应用到数据库中。

`flask db migrate`子命令用于自动创建迁移脚本:

```python
flask db migrate -m "initial migrations"
```

### 更新数据库

检查并修正好迁移脚本之后，执行`flask db upgrade`命令，把迁移应用到数据库中：

```python
(venv) $ flask db upgrade
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
```

> 如果你按照之前的说明操作过，那么已经使用`db.create all()`函数创建了数据库文件。此时，`flask db upgrade`命令将失败，因为它试图创建已经存在的数据库表。一种简单的处理方法是，把`data.sqlite`数据库文件删掉，然后执行`flask dbupgrade`命令，通过迁移框架重新创建数据库。另一种方法是不执行`flask dbupgrade`命令，而是使用`flask db stamp`命令把现有数据库标记为已更新。

### 添加几个迁移

在开发项目的过程中，时常要修改数据库模型。如果使用迁移框架管理数据库，必须在迁移脚本中定义所有改动，否则改动将不可复现。修改数据库的步骤与创建第一个迁移类似。

1. 对数据库模型做必要的修改。
2. 执行flask db migrate命令，生成迁移脚本。
3. 检查自动生成的脚本，改正不准确的地方。
4. 执行flask db upgrade命令，把改动应用到数据库中。

实现一个功能时，可能要多次修改数据库模型才能得到预期结果。如果前一个迁移还未提交到源码控制系统中，可以继续在那个迁移中修改，以免创建大量无意义的小迁移脚本。在前一个迁移脚本的基础上修改的步骤如下。

1. 执行flask db downgrade命令，还原前一个脚本对数据库的改动（注意，这可能导致部分数据丢失）。
2.  删除前一个迁移脚本，因为现在已经没什么用了。
3.  执行flask db migrate命令生成一个新的数据库迁移脚本。这个迁移脚本除了前面删除的那个脚本中的改动之外，还包括这一次对模型的改动。
4. 根据前面的说明，检查并应用迁移脚本。









[//]:#(设置表格整体居中显示)

<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


