---
title: Web表单
tags:
  - flask
  - python
comments: true
typora-root-url: Web表单
date: 2021-07-30 11:35:19
categories: Flask
index_img:
banner_img:
---

使用HTML可以创建Web表单，供用户填写信息。表单数据由Web浏览器提交给服务器，这一过程通常使用POST请求。Flask请求对象包含客户端在请求中发送的全部信息，对包含表单数据的POST请求来说，用户填写的信息通过request.form访问。尽管Flask的请求对象提供的信息足以处理Web表单，但有些任务很单调，而且要重复操作。比如，生成表单的HTML代码和验证提交的表单数据。

Flask-WTF扩展可以把处理Web表单的过程变成一种愉悦的体验。这个扩展对独立的WTForms包进行了包装，方便集成到Flask应用中。Flask-WTF及其依赖可使用pip安装：

```shell
(venv) $ pip install flask-wtf
```

## 配置

与其他多数扩展不同，`Flask-WTF`无须在应用层初始化，但是它要求应用配置一个密钥。密钥是一个由随机字符构成的唯一字符串，通过加密或签名以不同的方式提升应用的安全性。Flask使用这个密钥保护用户会话，以防被篡改。每个应用的密钥应该不同，而且不能让任何人知道。

```python
app = Flask(__name__)
app.config['SECRET_KEY'] = 'hard to guess string'
```

app.config字典可用于存储Flask、扩展和应用自身的配置变量。使用标准的字典句法就能把配置添加到app.config对象中。这个对象还提供了一些方法，可以从文件或环境中导入配置。

Flask-WTF之所以要求应用配置一个密钥，是为了防止表单遭到跨站请求伪造（CSRF, cross-site request forgery）攻击。恶意网站把请求发送到被攻击者已登录的其他网站时，就会引发CSRF攻击。Flask-WTF为所有表单生成安全令牌，存储在用户会话中。令牌是一种加密签名，根据密钥生成。

> 为了增强安全性，密钥不应该直接写入源码，而要保存在环境变量中。

## 表单类

使用Flask-WTF时，在服务器端，每个Web表单都由一个继承自`FlaskForm`的类表示。这个类定义表单中的一组字段，每个字段都用对象表示。字段对象可附属一个或多个验证函数。验证函数用于验证用户提交的数据是否有效。

定义表单类：

```python
# 定义一个表单类
class LoginForm(FlaskForm):
    # 定义一个文本框，提示信息为用户名，validators设置验证规则，参数是一个列表，可以添加多个验证函数，DataRequired不能为空验证函数
    name = StringField("用户名：", validators=[DataRequired()])
    # 定义一个提交按钮
    submit = SubmitField("提交")
```

`LoginForm`表单中有一个名为`name`的文本字段和一个名为`submit`的提交按钮。StringField类表示属性为`type="text"`的HTML `<input>`元素。SubmitField类表示属性为type="submit"的HTML `<input>`元素。字段构造函数的第一个参数是把表单渲染成HTML时使用的标注（label）。

StringField构造函数中的可选参数validators指定一个由验证函数组成的列表，在接受用户提交的数据之前验证数据。验证函数DataRequired()确保提交的字段内容不为空。

> FlaskForm基类由Flask-WTF扩展定义，所以要从flask wtf中导入。然而，字段和验证函数却是直接从WTForms包中导入的。

WTForms支持的HTML标准字段。

| 字段类型                | 说明                                |
| ----------------------- | ----------------------------------- |
| BooleanField            | 复选框，值为TRUE和FALSE             |
| DateField               | 文本字段，值为datetime.date格式     |
| DateTimeField           | 文本字段，值为datetime.datetime格式 |
| DecimalField            | 文本字段，值为decimal.Decimal       |
| FileField               | 文件上传字段                        |
| HiddenField             | 隐藏的文本字段                      |
| MultipleFileField       | 多文件上传字段                      |
| Fieldlist               | 一组指定类型的字段                  |
| FloatField              | 文本字段，值为浮点数                |
| FormField               | 把一个表单作为字段嵌入另一个表单    |
| IntegerField            | 文本字段，值为整数                  |
| PasswordField           | 密码文本字段                        |
| RadiField               | 一组单选按钮                        |
| SelectField             | 下拉列表                            |
| SelectMultipleField     | 下拉列表，可选择多个值              |
| SubmitField             | 表单提交按钮                        |
| StringField             | 文本字段                            |
| TextAreaField           | 多行文本字段                        |
| WTForms内建的验证函数。 |                                     |

| 验证函数      | 说明                                                   |
| ------------- | ------------------------------------------------------ |
| DataRequired  | 确保转换类型后字段中有数据                             |
| Email         | 验证电子邮件地址                                       |
| EqualTo       | 比较两个字段的值，常用于要求输入两次密码进行确认的情况 |
| InputReqiured | 确保转换类型前字段中有数据                             |
| IPAddress     | 验证IPv4网络地址                                       |
| Length        | 验证输入字符串的长度                                   |
| MacAddress    | 验证MAC地址                                            |
| NumberRange   | 验证输入的值在数字范围之内                             |
| Optional      | 允许字段中没有输入，将跳过其他验证函数                 |
| Regexp        | 使用正则表达式验证输入值                               |
| URL           | 验证URL                                                |
| UUID          | 验证UUID                                               |
| AnyOf         | 确保输入值在一组可能的值中                             |
| NoneOf        | 确保输入值不在一组可能的值中                           |

## 把表单渲染成HTML

表单字段是可调用的，在模板中调用后会渲染成HTML。假设视图函数通过form参数把一个NameForm实例传入模板，在模板中可以生成一个简单的HTML表单，这里使用了boostrap提供的form模板，如下所示：

```html
{% extends 'bootstrap/base.html' %}
{% import "bootstrap/wtf.html" as wtf %}
<div class="content">
    <h1>Hello, {% if name %}{{ name }}{% else %}Stranger{% endif %}!</h1>
    {{ wtf.quick_form(form) }}
</div>
```

## 在视图函数中处理表单

```python
@app.route('/', methods=['GET', 'POST'])
def index():
    name = None
    form = LoginForm()
    if form.validate_on_submit:
        name = form.name.data
        form.name.data = ''
    return render_template('index.html', form=form, name=name, current_time=datetime.utcnow())
```

app.route装饰器中多出的methods参数告诉Flask，在URL映射中把这个视图函数注册为GET和POST请求的处理程序。如果没指定methods参数，则只把视图函数注册为GET请求的处理程序。

用户提交表单后，服务器会收到一个包含数据的POST请求。`validate on submit()`会调用名字字段上依附的`DataRequired()`验证函数。如果名字不为空，就能通过验证，`validate on submit()`返回True。

![](image-20210731132446805.png)

## 重定向和用户会话

使用重定向作为POST请求的响应，而不是使用常规响应。重定向是一种特殊的响应，响应内容包含的是URL，而不是HTML代码的字符串。浏览器收到这种响应时，会向重定向的URL发起GET请求，显示页面的内容。这个页面的加载可能要多花几毫秒，因为要先把第二个请求发给服务器。除此之外，用户不会察觉到有什么不同。现在，前一个请求是GET请求，所以刷新命令能像预期的那样正常运作了。这个技巧称为`Post /重定向/Get模式`。

但这种方法又会引起另一个问题。应用处理POST请求时，可以通过`form.name.data`获取用户输入的名字，然而一旦这个请求结束，数据也就不见了。因为这个POST请求使用重定向处理，所以应用需要保存输入的名字，这样重定向后的请求才能获得并使用这个名字，从而构建真正的响应。

应用可以把数据存储在用户会话中，以便在请求之间“记住”数据。用户会话是一种私有存储，每个连接到服务器的客户端都可访问。用户会话，它是请求上下文中的变量，名为`session`，像标准的Python字典一样操作。

> 默认情况下，用户会话保存在客户端cookie中，使用前面设置的密钥加密签名。如果篡改了cookie的内容，签名就会失效，会话也将随之失效。

实现重定向和用户会话。

```python
@app.route('/', methods=['GET', 'POST'])
def index():
    form = LoginForm()
    print(form.validate_on_submit)
    if form.validate_on_submit():
        # name = form.name.data
        session['name'] = form.name.data
        return redirect(url_for('index'))
        print(url_for('index'))
    return render_template('index.html', form=form, name=session.get('name'), current_time=datetime.utcnow())
```

应用的前一个版本在局部变量name中存储用户在表单中输入的名字。这个变量现在保存在用户会话中，即`session['name']`，所以在两次请求之间能记住输入的值。

现在，包含有效表单数据的请求最后会使视图函数调用`redirect()`函数。这是`Flask`提供的辅助函数，用于生成`HTTP`重定向响应。`redirect()`函数的参数是重定向的URL，这里使用的重定向`URL`是应用的根`URL`，因此重定向响应本可以写得更简单一些，写成`redirect('/')`，不过这里却使用了Flask提供的URL生成函数`url_for()`。`url_for()`函数的第一个且唯一必须指定的参数是端点名，即路由的内部名称。默认情况下，路由的端点是相应视图函数的名称。在这个示例中，处理根URL的视图函数是`index()`，因此传给`url_for()`函数的名字是index。

最后一处改动位于`render_template()`函数中，现在我们使用`session.get('name')`直接从会话中读取name参数的值。与普通的字典一样，这里使用get()获取字典中键对应的值，可以避免未找到键时抛出异常。如果指定的键不存在，则`get()`方法返回默认值None。

## 闪现消息

请求完成后，有时需要让用户知道状态发生了变化，可以是确认消息、警告或者错误提醒。一个典型例子是，用户提交有一项错误的登录表单后，服务器发回的响应重新渲染登录表单，并在表单上面显示一个消息，提示用户名或密码无效。Flask本身内置这个功能。`flash()`函数可实现这种效果。

```python
from flask import flash

@app.route('/', methods=['GET', 'POST'])
def index():
    form = LoginForm()
    if form.validate_on_submit():
        # name = form.name.data
        old_name = session.get('name')
        if old_name is not None and old_name != form.name.data:
            flash('密码不正确，请好好想想！')
        session['name'] = form.name.data
        return redirect(url_for('index'))
        print(url_for('index'))
    return render_template('index.html', form=form, name=session.get('name'), current_time=datetime.utcnow())
```

每次提交的名字都会和存储在用户会话中的名字进行比较，而会话中存储的名字是前一次在这个表单中提交的数据。如果两个名字不一样，就会调用`flash()`函数，在发给客户端的下一个响应中显示一个消息。

仅调用`flash()`函数并不能把消息显示出来，应用的模板必须渲染这些消息。最好在基模板中渲染闪现消息，因为这样所有页面都能显示需要显示的消息。Flask把`get_flashed_messages()`函数开放给模板，用于获取并渲染闪现消息。

```html
 {% for message in get_flashed_messages() %}
	<div class="alert alert-warning">
      <button type="button" class="close">&times;</button>
        {{ message }}
    </div>
{% endfor %}
```

这里使用了循环，因为在之前的请求循环中每次调用`flash()`函数时都会生成一个消息，所以可能有多个消息在排队等待显示。`get_flashed_messages()`函数获取的消息在下次调用时不会再次返回，因此闪现消息只显示一次，然后就消失了。





[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


