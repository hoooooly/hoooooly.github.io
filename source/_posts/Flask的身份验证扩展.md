---
title: Flask的身份验证扩展
tags:
  - flask
  - flask-login
comments: true
typora-root-url: Flask的身份验证扩展
date: 2021-08-06 10:26:25
categories: Flask
index_img:
banner_img:
---

## 密码安全性

设计Web应用时，人们往往会忽视数据库中用户信息的安全性。如果攻击者入侵服务器，攫取了数据库，用户的安全就处在风险之中，而且这个风险超乎你的想象。众所周知，多数用户会在不同的网站中使用相同的密码。因此，即便不保存任何敏感信息，攻击者获得存储在数据库中的密码之后，也能访问用户在其他网站中的账户。

若想保证数据库中用户密码的安全，关键在于不存储密码本身，而是存储密码的**散列值**。计算密码散列值的函数接收密码作为输入，添加**随机内容（盐值）**之后，使用多种单向加密算法转换密码，最终得到一个和原始密码没有关系的字符序列，而且无法还原成原始密码。核对密码时，密码散列值可代替原始密码，因为计算散列值的函数是可复现的：只要输入（密码和盐值）一样，结果就一样。

### 使用`Werkzeug`计算密码散列值

`Werkzeug`中的`security`模块实现了密码散列值的计算。这一功能的实现只需要两个函数，分别用在注册和核对两个阶段。

```python
generate_password_hash(password, method='pbkdf2:sha256', salt_length=8)
```

这个函数的输入为原始密码，返回密码散列值的字符串形式，供存入用户数据库。`method`和`salt_length`的默认值就能满足大多数需求。

```python
check_password_hash(hash, password)
```

这个函数的参数是从数据库中取回的密码散列值和用户输入的密码。返回值为`True`时表明用户输入的密码正确。

在创建的User模型的基础上添加密码散列所需的改动，在User模型中加入密码散列。

```python
class User(db.Model):
    # ...
    password_hash = db.Column(db.String(128))

    @property
    def password(self):
        raise AttributeError('密码不可读')

    @password.setter
    def password(self, password):
        self.password_hash = generate_password_hash(password)

    def verify_password(self, password):
        return check_password_hash(self.password_hash, password)
```

计算密码散列值的函数通过名为`password`的只写属性实现。设定这个属性的值时，赋值方法会调用`Werkzeug`提供的`generate_password_hash()`函数，并把得到的结果写入`password_hash`字段。如果试图读取`password`属性的值，则会返回错误，原因很明显，因为生成散列值后就无法还原成原来的密码了。

`verify_password()`方法接受一个参数（即密码），将其传给`Werkzeug`提供的`check_password_hash()`函数，与存储在`User`模型中的密码散列值进行比对。如果这个方法返回`True`，表明密码是正确的。

密码散列功能已经完成，下面在shell中测试一下：

```python
(venv) $ flask shell
>>> u = User()
>>> u.password = 'cat'
>>> u.password
Traceback (most recent call last):
  File "<console>", line 1, in <module>
  File "D:\PycharmProjects\Flask_Projects\flasky\app\models.py", line 17, in password
    raise AttributeError('密码不可读')
AttributeError: 密码不可读
>>> u.password_hash
'pbkdf2:sha256:260000$ylvTADdsUn6mFJv8$824d40ef026f79bf2413ee16b7a9ecacfdac124428ae285574a56136f0bbde60'
>>> u.verify_password('cat')
True
>>> u.verify_password('dog')
False
>>> u2 = User()
>>> u2.password='cat'
>>> u2.password_hash
'pbkdf2:sha256:260000$e3x8WYKw4KVueaUS$e93b63b9cbfb0e719a9b177970cc3815470d2bcbd59079236ef5873a753cc007'
```

注意，尝试访问`password`属性会返回`AttributeError`。另外，即使用户`u`和`u2`使用了相同的密码，它们的密码散列值也完全不一样。为了确保这个功能今后依然能使用，我们可以把上述手动测试的过程写成单元测试，以便重复执行。在`tests`包中新建一个模块，编写3个新测试，测试最近对`User`模型所做的改动。

 `tests/test user model.py：`密码散列测试

```python
import unittest

from app.models import User


class UserModelTestCase(unittest.TestCase):
    def test_password_setter(self):
        u = User(password='cat')
        self.assertTrue(u.password_hash is not None)

    def test_no_password_getter(self):
        u = User(password='cat')
        with self.assertRaises(AttributeError):
            u.password

    def test_password_verification(self):
        u = User(password='cat')
        self.assertTrue(u.verify_password('cat'))
        self.assertFalse(u.verify_password('dog'))
    
    def test_password_salts_are_random(self):
        u = User(password='cat')
        u2 = User(password='cat')
        self.assertTrue(u.password_hash != u2.password_hash)
```

```python
test_app_exists (test_basics.BasicsTestCase) ... <class 'config.TestConfig'>
ok
test_app_is_testing (test_basics.BasicsTestCase) ... <class 'config.TestConfig'>
ok
test_no_password_getter (test_user_model.UserModelTestCase) ... ok
test_password_salts_are_random (test_user_model.UserModelTestCase) ... ok
test_password_setter (test_user_model.UserModelTestCase) ... ok
test_password_verification (test_user_model.UserModelTestCase) ... ok

----------------------------------------------------------------------
Ran 6 tests in 1.092s

OK
```

每次想确认一切功能是否正常时，就可以运行单元测试组件。通过自动化测试验证功能不费吹灰之力，因此应该经常运行，确保以后的改动不会破坏现在可用的功能。

## 创建身份认证蓝本

我们在第7章介绍过蓝本，把创建应用的过程移入工厂函数后，可使用蓝本在全局作用域中定义路由。本节将在一个新蓝本中定义与用户身份验证子系统相关的路由，这个蓝本名为`auth`。把应用的不同子系统放在不同的蓝本中，有利于保持代码整洁有序。

`auth`蓝本保存在同名Python包中。这个蓝本的包构造函数创建蓝本对象，再从`views.py`模块中导入路由。

`app/auth/__init__.py`：创建身份验证蓝本

```python
from flask import Blueprint

auth = Blueprint('auth', __name__)

from . import views
```

`app/auth/views.py`模块导入蓝本，然后使用蓝本的`route`装饰器定义与身份验证相关的路由。这段代码添加了一个`/login`路由，渲染同名占位模板。

`app/auth/views.py`：身份验证蓝本中的路由和视图函数。

```python
from flask import render_template
from . import auth


@auth.route('/login')
def login():
    return render_template('auth/login.html')
```

注意，为`render_template()`指定的模板文件保存在`auth`目录中。这个目录必须在`app/templates`中创建，因为Flask期望模板的路径是相对于应用的模板目录而言的。把蓝本中用到的模板放在单独的子目录中，能避免与`main`蓝本或以后添加的蓝本发生冲突。

> 也可以配置蓝本使用专门的目录保存模板。如果配置了多个模板目录，那么`render_template()`函数会先搜索应用的模板目录，然后再搜索蓝本的模板目录。

`auth`蓝本要在`create_app()`工厂函数中附加到应用上。

`app/__init__.py`：注册身份验证蓝本:

```python

def create_app(config_name):
    # ...
    
    # 注册身份验证蓝本
    from .auth import auth as auth_blueprint
    app.register_blueprint(auth_blueprint, url_prefix='/auth')

    return app
```

注册蓝本时使用的`url_prefix`是可选参数。如果使用了这个参数，注册后蓝本中定义的所有路由都会加上指定的前缀，即这个例子中的`/auth`。例如，`/login`路由会注册成`/auth/login`，在开发Web服务器中，完整的URL就变成了`http://localhost:5000/auth/login`。

## 使用Flask-Login验证用户身份

安装扩展：

```python
(venv) $ pip install flask-login
```

###  准备用于登录的用户模型

`Flask-Login`的运转需要应用中有`User`对象。要想使用`Flask-Login`扩展，应用的`User`模型必须实现几个属性和方法：

| 属性/方法        | 说明                                                         |
| ---------------- | ------------------------------------------------------------ |
| is_authenticated | 如果用户提供的登录凭据有效，必须返回True，否则返回False      |
| is_active        | 如果允许用户登录，必须返回True，否则返回False。如果想禁用账户，可以返回False |
| is_anonymous     | 对普通用户必须返回False，如果是表示匿名用户的特殊用户对象，应该返回True |
| get_id()         | 必须返回用户的唯一标识符，使用Unicode编码字符串              |

这些属性和方法可以直接在模型类中实现，不过还有一种更简单的替代方案。`Flask-Login`提供了一个`UserMixin`类，其中包含默认实现，能满足多数需求。

```python
from flask_login import UserMixin


class User(UserMixin, db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key=True)
    email = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(64), unique=True, index=True)
    role_id = db.Column(db.Integer, db.ForeignKey('roles.id'))
    password_hash = db.Column(db.String(128))
```

注意，这个示例中还添加了`email`字段。在这个应用中，用户使用电子邮件地址登录，因为相对于用户名而言，用户更不容易忘记自己的电子邮件地址。

`Flask-Login`在应用的工厂函数中初始化：

```python
from flask_login import LoginManager
 # ...
 
login_manager = LoginManager()
login_manager.login_view = 'auth.login'

def create_app(config_name):
    # ...
    login_manager.init_app(app)
    # ...
```

`LoginManager`对象的`login_view`属性用于设置登录页面的端点。匿名用户尝试访问受保护的页面时，`Flask-Login`将重定向到登录页面。因为登录路由在蓝本中定义，所以要在前面加上蓝本的名称。

最后，`Flask-Login`要求应用指定一个函数，在扩展需要从数据库中获取指定标识符对应的用户时调用。

```python
from . import login_manager

@login_manager.user_loader
def load_user(user_id):
    return User.query.get(int(user_id))
```

`login_manager.user_loader`装饰器把这个函数注册给Flask-Login，在这个扩展需要获取已登录用户的信息时调用。传入的用户标识符是个字符串，因此这个函数先把标识符转换成整数，然后传给`Flask-SQLAlchemy`查询，加载用户。正常情况下，这个函数的返回值必须是用户对象；如果用户标识符无效，或者出现了其他错误，则返回None。

### 保护路由

为了保护路由，只让通过身份验证的用户访问，Flask-Login提供了一个`loginrequired`装饰器。其用法演示如下

```python
from flask_login import login_required

@app.route('/secert')
@login_required
def secert():
    return 'Only authenticated users are allowed!'
```

多个函数装饰器可以叠加使用。函数上有多个装饰器时，各装饰器只对随后的装饰器和目标函数起作用。在这个示例中，secret()函数受`login_required`装饰器的保护，禁止未授权的用户访问，得到的函数又注册为一个Flask路由。如果调换两个装饰器，得到的结果将是错的，因为原始函数先注册为路由，然后才从`login_required`装饰器接收到额外的属性。

得益于`login_required`装饰器，如果未通过身份验证的用户访问这个路由，Flask-Login将拦截请求，把用户发往登录页面。

### 添加登录表单

呈现给用户的登录表单中包含一个用于输入电子邮件地址的文本字段、一个密码字段、一个“记住我”复选框和一个提交按钮。

`app/auth/forms.py`：登录表单

```python
from flask_wtf import FlaskForm
from wtforms import StringField, PasswordField, BooleanField, SubmitField
from wtforms.validators import DataRequired, Length, Email


class LoginForm(FlaskForm):
    email = StringField('邮箱', validators=[DataRequired(), Length(1, 64), Email()])
    password = PasswordField('密码', validators=[DataRequired()])
    remember_me = BooleanField('保持登录')
    submit = SubmitField('登录')
```

`PasswordField`类表示属性为`type="password"`的`<input>`元素。`BooleanField`类表示复选框。电子邮件字段用到了`WTForms`提供的`Length()`、`Email()`和`DataRequired()`这3个验证函数，不仅确保这个字段有值，而且必须是有效的。提供验证函数列表时，`WTForms`将按照指定的顺序执行各个验证函数。倘若验证失败，显示的错误消息将是首个失败的验证函数的消息。

![](image-20210809134438984.png)

登录页面使用的模板保存在`auth/login.html`文件中。这个模板只需使用Flask-Bootstrap提供的`wtf.quick_form()`宏渲染表单即可。

`base.html`模板中的导航栏可以使用`Jinja2`条件语句判断当前用户的登录状态，分别显示`登录`或`退出`链接。

```html
<ul class="nav navbar-nav navbar-right">
    {% if current_user.is_authenticated %}
        <li><a href="{{ url_for('auth.logout') }}">退出</a></li>
    {% else %}
        <li><a href="{{ url_for('auth.login') }}">登录</a></li>
    {% endif %}
</ul>
```

判断条件中的变量`current_user`由Flask-Login定义，在视图函数和模板中自动可用。这个变量的值是当前登录的用户，如果用户未登录，则是一个匿名用户代理对象。匿名用户对象的`is_authenticated`属性值是False，所以通过`current_user.is_authenticated`表达式就能判断当前用户是否登录。

![](image-20210809135311391.png)

### 登入用户

视图函数`login()`的实现。

```python
@auth.route('/login', methods=['GET', 'POST'])
def login():
    login_form = LoginForm()
    if login_form.validate_on_submit():
        user = User.query.filter_by(email=login_form.email.data).first()
        if user is not None and user.verify_password(login_form.password.data):
            login_user(user, login_form.remember_me.data)
            next = request.args.get('next')
            if next is None or not next.startswith('/'):
                next = url_for('main.index')
            return redirect(next)
        flash('无效的用户名或密码')
    return render_template('/auth/login.html', login_form=login_form)
```

这个视图函数创建了一个`LoginForm`对象，用法和第4章中的那个简单表单一样。当请求类型是GET时，视图函数直接渲染模板，即显示表单。当表单通过POST请求提交时，`Flask-WT`F的`validate_on_submit()`函数会验证表单数据，然后尝试登入用户。

为了登入用户，视图函数首先使用表单中填写的电子邮件地址从数据库中加载用户。如果电子邮件地址对应的用户存在，再调用用户对象的`verify_password()`方法，其参数是表单中填写的密码。如果密码正确，调用Flask-Login的`login_user()`函数，在用户会话中把用户标记为已登录。`login_user()`函数的参数是要登录的用户，以及可选的“记住我”布尔值，“记住我”也在表单中勾选。如果这个字段的值为False，关闭浏览器后用户会话就过期了，所以下次用户访问时要重新登录。如果值为True，那么会在用户浏览器中写入一个长期有效的cookie，使用这个`cookie`可以复现用户会话。`cookie`默认记住一年，可以使用可选的`REMEMBERCOOKIE_DURATION`配置选项更改这个值。

如果用户输入的电子邮件地址或密码不正确，应用会设定一个闪现消息，并再次渲染表单，让用户再次尝试登录。

### 退出用户

`app/auth/views.py`：退出路由

```python
from flask_login import login_required, logout_user

@auth.route('/logout')
@login_required
def logout():
    logout_user()
    flash('您已退出！')
    return redirect(url_for('main.index'))
```

为了登出用户，这个视图函数调用Flask-Login的`logout_user()`函数，删除并重设用户会话。随后会显示一个闪现消息，确认这次操作，然后重定向到首页，这样就成功退出了。

### 登录测试

为验证登录功能可用，可以更新首页，使用已登录用户的名字显示一个欢迎消息。

`app/templates/index.html`：为已登录的用户显示一个欢迎消息。

```python
{% block content %}
    <h1>Hello word,
        {% if current_user.is_authenticated %}
            {{ current_user.username }}
        {% else %}
            Stranger
        {% endif %}
    </h1>
{% endblock %}
```

这个模板再次使用`current_user.username`判断用户是否已经登录。

因为还未实现用户注册功能，所以目前只能在`shell`中注册新用户：

```python
(venv) $ flask shell
>>> u = User(email='espholychan@outlook.com',username='holy',password='123')
>>> db.session.add(u)
>>> db.session.commit()
```

刚刚创建的用户现在可以登录了。用户登录后显示的首页如图所示。

![](image-20210809172042915.png)

## 注册新用户

如果新用户想成为应用的成员，必须在应用中注册，这样应用才能识别并登入用户。应用的登录页面中要显示一个链接，把用户带到注册页面，让用户输入电子邮件地址、用户名和密码。

### 添加用户注册表单

注册页面中的表单要求用户输入电子邮件地址、用户名和密码。

`app/auth/forms.py`：用户注册表单

```python
from flask_wtf import FlaskForm
from wtforms import StringField, PasswordField, BooleanField, SubmitField, ValidationError
from wtforms.validators import DataRequired, Length, Email, Regexp, EqualTo
from ..models import User


class LoginForm(FlaskForm):
    email = StringField('邮箱', validators=[DataRequired(), Length(1, 64), Email()])
    password = PasswordField('密码', validators=[DataRequired()])
    remember_me = BooleanField('保持登录')
    submit = SubmitField('登录')


class RegistrationForm(FlaskForm):
    email = StringField('邮箱', validators=[DataRequired(), Length(1, 64), Email()])
    username = StringField('用户名', validators=[DataRequired(), Length(1, 64),
                                              Regexp('[A-Za-z][A-Za-z0-9.]*$', 0, '用户名必须含有字母数字或下划线')])
    password = PasswordField('密码', validators=[DataRequired(), EqualTo('password2', message='密码必须一致')])
    password2 = PasswordField('确认密码', validators=[DataRequired()])
    submit = SubmitField('注册')

    def validate_email(self, field):
        if User.query.filter_by(email=field.data).first():
            raise ValidationError('邮箱已经被注册。')

    def validate_user(self, field):
        if User.query.filter_by(username=field.data).first():
            raise ValidationError("用户已存在。")
```

这个表单使用`WTForms`提供的`Regexp`验证函数，确保`username`字段的值以字母开头，而且只包含字母、数字、下划线和点号。这个验证函数中正则表达式后面的两个参数分别是正则表达式的标志和验证失败时显示的错误消息。

为了安全起见，密码要输入两次。此时要验证两个密码字段中的值是否一致，这种验证可使用`WTForms`提供的另一验证函数实现，即`EqualTo`。这个验证函数要附属到两个密码字段中的一个上，另一个字段则作为参数传入。

这个表单还有两个自定义的验证函数，以方法的形式实现。如果表单类中定义了以`validate`开头且后面跟着字段名的方法，这个方法就和常规的验证函数一起调用。本例分别为`email`和`username`字段定义了验证函数，确保填写的值在数据库中没出现过。自定义的验证函数要想表示验证失败，可以抛出`ValidationError`异常，其参数就是错误消息。显示这个表单的模板是`/templates/auth/register.html`。与登录模板一样，这个模板也使用`wtf.quick form()`渲染表单。

```python
{% extends 'base.html' %}
{% import 'bootstrap/wtf.html' as wtf %}


{% block title %}注册{% endblock %}

{% block content %}
    <div class="container">
        {% for message in get_flashed_messages() %}
            <div class="alert alert-warning">
                <button type="button" class="close">&times;</button>
                {{ message }}
            </div>
        {% endfor %}
        <div class="col">
            {{ wtf.quick_form(login_form) }}
        </div>
    </div>
{% endblock %}
```

登录页面要显示一个指向注册页面的链接，让没有账户的用户能轻松找到注册页面。

`app/templates/auth/login.html`：链接到注册页面

```html
<p>
    新用户？
    <a href="{{ url_for('auth.register') }}">点击注册用户</a>
</p>
```

### 注册新用户

处理用户注册的过程没有什么难以理解的地方。提交注册表单，通过验证后，系统使用用户填写的信息在数据库中添加一个新用户。

`app/auth/views.py`：用户注册路由。

```python
@auth.route('/register', methods=['GET', 'POST'])
def register():
    register_form = RegistrationForm()
    if register_form.validate_on_submit():
        user = User(email=register_form.email.data,
                    username=register_form.username.data,
                    password=register_form.password.data)
        db.session.add(user)
        db.session.commit()
        flash("注册成功，跳转登录！")
        return redirect(url_for('auth.login'))
    return render_template('/auth/register.html', register_form=register_form
```

![](image-20210809230811876.png)

## 确认账户

对于某些特定类型的应用，有必要确认注册时用户提供的信息是否正确。常见要求是能通过提供的电子邮件地址与用户取得联系。

为了确认电子邮件地址，用户注册后，应用会立即发送一封确认邮件。新账户先被标记成待确认状态，用户按照邮件中的说明操作后，才能证明自己可以收到电子邮件。账户确认过程中，往往会要求用户点击一个包含确认令牌的特殊`URL`链接。

### 使用itsdangerous生成确认令牌

确认邮件中最简单的确认链接是`http://www.example.com/auth/confirm/<id>`这种形式的URL，其中`<id>`是数据库分配给用户的数字id。用户点击链接后，处理这个路由的视图函数将确认收到的用户id，然后将用户状态更新为已确认。

但这种实现方式显然不是很安全，只要用户能判断确认链接的格式，就可以随便指定URL中的数字，从而确认任意账户。解决方法是把URL中的`<id>`换成包含相同信息的令牌，但是只有服务器才能生成有效的确认URL。

Flask使用加密的签名cookie保护用户会话，以防止被篡改。用户会话cookie中有一个由`itsdangerous`包生成的加密签名。如果用户会话的内容被篡改，签名将不再与内容匹配，这样会使Flask销毁会话，然后重建一个。同样的方法也可用在确认令牌上。

下面这个简短的shell会话展示如何使用`itsdangerous`包生成包含用户id的签名令牌：

```python
>>> from itsdangerous import TimedJSONWebSignatureSerializer as Serializer
>>> s = Serializer(app.config['SECRET_KEY'], expires_in=3600)
>>> token = s.dumps({'confirm':23})
>>> token
b'eyJhbGciOiJIUzUxMiIsImlhdCI6MTYyODY0ODMzNywiZXhwIjoxNjI4NjUxOTM3fQ.eyJjb25maXJtIjoyM30.uSG3P9YYEkkwwYF9AYO0vWqyC81uYLu0lLlm-3p6SE4
IyH-oZqg4ygGslzdh0ammAfujT26JLq-TCAw7cVWgzw'
>>> data = s.loads(token)
>>> data
{'confirm': 23}
```

`itsdangerous`提供了多种生成令牌的方法。其中，`TimedJSONWebSignatureSerializer`类生成具有过期时间的`JSON Web`签名（`JWS`）。这个类的构造函数接收的参数是一个密钥，在Flask应用中可使用`SECRET_KEY`设置。

`dumps()`方法为指定的数据生成一个加密签名，然后再对数据和签名进行序列化，生成令牌字符串。`expires_in`参数设置令牌的过期时间，单位为秒。

为了解码令牌，序列化对象提供了`loads()`方法，其唯一的参数是令牌字符串。这个方法会检验签名和过期时间，如果都有效，则返回原始数据。如果提供给`loads()`方法的令牌无效或是过期了，则抛出异常。

我们可以把这种生成和检验令牌的功能添加到User模型中，`app/models.py`：确认用户账户：

```python
from itsdangerous import TimedJSONWebSignatureSerializer as Serializer
from flask import current_app


class User(UserMixin, db.Model):
    # ...
    
    comfirmed = db.Column(db.Boolean, default=False)

    def generate_confitmation_token(self, expiration=3600):
        s = Serializer(current_app.config['SECRET_KEY'], expires_in=expiration)
        return s.dumps({'confirm': self.id}).decode('utf-8')

    def confirm(self, token):
        s = Serializer(current_app.config['SECRET_KEY'])
        try:
            data = s.loads((token.decode('utf-8')))
        except:
            return False
        if data.get('confirm') != self.id:
            return False
        self.comfirmed = True
        db.session.add(self)
        return True
```

`generate_confitmation_token()`方法生成一个令牌，有效期默认为一小时。`confirm()`方法检验令牌，如果检验通过，就把用户模型中新添加的`confirmed`属性设为True。除了检验令牌，`confirm()`方法还检查令牌中的id是否与存储在`current_user`中的已登录用户匹配。这样能确保为一个用户生成的确认令牌无法用于确认其他用户。

### 发送确认邮件

当前的`/register`路由把新用户添加到数据库中之后，会重定向到`/index`。在重定向之前，这个路由现在需要发送确认邮件。

`app/auth/views.py`：能发送确认邮件的注册路由。

```python
@auth.route('/register', methods=['GET', 'POST'])
def register():
    register_form = RegistrationForm()
    if register_form.validate_on_submit():
        user = User(email=register_form.email.data,
                    username=register_form.username.data,
                    password=register_form.password.data)
        db.session.add(user)
        db.session.commit()
        token = user.generate_confitmation_token()
        send_mail(user.email, '确认你的用户信息', 'auth/email/confirm', user=user, token=token)
        flash("确认信息已经发送到你的邮箱，请确认！")
        return redirect(url_for('main.index'))
    return render_template('/auth/register.html', register_form=register_form)
```

注意，在发送确认邮件之前要调用`db.session.commit()`。之所以这么做，是因为提交之后才能赋予新用户id值，而确认令牌需要用到id。

身份验证蓝本使用的电子邮件模板保存在`templates/auth/email`目录中，以便与HTML模板区分开来。

```txt
你好， {{ user.name }}!

欢迎来到Flasky!

点击下面的链接来确认你的账户信息：
{{ url_for('auth.confirm', token=token, external=True) }}

不用回复！
```

默认情况下，`url_for()`生成相对URL，例如`url_for('auth.confirm', token='abc')`返回的字符串是`’/auth/confirm/abc'`。这显然不是能够在电子邮件中发送的正确URL，因为只有URL的路径部分。相对URL在网页的上下文中可以正常使用，因为浏览器会添加当前页面的主机名和端口号，将其转换成绝对URL。但是通过电子邮件发送的URL并没有这种上下文。添加到`url_for()`函数中的`external=True`参数要求应用生成完全限定的URL，包括协议`（http://或https://）`、主机名和端口。

`app/auth/views.py`：确认用户的账户

```python
from flask_login import current_user
@auth.route('/confirm/<token>')
@login_required
def confirm(token):
    if current_user.confirmed:
        return redirect(url_for('main.index'))
    if current_user.confirm(token):
        db.session.commit()
        flash('您的账户已经确认，谢谢！')
    return redirect(url_for('main.index'))
```

Flask-Login提供的`login_required`装饰器会保护这个路由，因此，用户点击确认邮件中的链接后，要先登录，然后才能执行这个视图函数。

这个函数先检查已登录的用户是否已经确认过，如果确认过，则重定向到首页，因为很显然此时不用做什么操作。这样处理可以避免用户不小心多次点击确认令牌带来的额外工作。

由于令牌确认完全在User模型中完成，所以视图函数只需调用`confirm()`方法即可，然后再根据确认结果显示不同的闪现消息。确认成功后，User模型中`confirmed`属性的值会被修改并添加到会话中，然后提交数据库会话。

各个应用可以自行决定用户确认账户之前可以做哪些操作。比如，允许未确认的用户登录，但只显示一个页面，要求用户在获取进一步访问权限之前先确认账户。

这一步可使用Flask提供的`before_request`钩子完成。对蓝本来说，`before_request`钩子只能应用到属于蓝本的请求上。若想在蓝本中使用针对应用全局请求的钩子，必须使用`before_app_request`装饰器。

`app/auth/views.py`：使用`before_app_request`处理程序过滤未确认的账户:

```python
@auth.before_request
def before_request():
    if current_user.is_authenticated \
            and not current_user.confirmed \
            and request.blueprint != 'auth' \
            and request.endpoint != 'static':
        return redirect(url_for('auth.unconfirmed'))


@auth.route('/unconfirmed')
def unconfirmed():
    if current_user.is_anonymous or current_user.confirmed:
        return redirect(url_for('main.index'))
    return render_template('auth/unconfirmed/html')
```

同时满足以下3个条件时，`before_app_request`处理程序会拦截请求。

(1) 用户已登录（`current user.is authenticated`的值为True）。

(2) 用户的账户还未确认。

(3) 请求的URL不在身份验证蓝本中，而且也不是对静态文件的请求。要赋予用户访问身份验证路由的权限，因为这些路由的作用是让用户确认账户或执行其他账户管理操作。如果请求满足以上条件，会被重定向到`/auth/unconfirmed`路由，显示一个确认账户相关信息的页面。

```python
@auth.route('/confirm')
@login_required
def resend_confirmation():
    token = current_user.generate_confirmtion_token()
    send_mail(current_user.email, '确认你的账户', 'auth/email/confirm', user=current_user, token=token)
    flash('新的确认信息已经发送到你的邮箱')
    return redirect(url_for('main.index'))
```

这个路由为`current_user`（即已登录的用户，也是目标用户）重做了一遍注册路由中的操作。这个路由也用`login_required`保护，确保只有通过身份验证的用户才能再次请求发送确认邮件。






[//]:#(设置表格整体居中显示)

<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


