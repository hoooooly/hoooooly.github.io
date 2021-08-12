---
title: Flask-Mail实现邮件功能
tags:
  - flask
  - mail
  - python
comments: true
typora-root-url: Flask-Mail实现邮件功能
date: 2021-08-05 13:21:20
categories: Flask
index_img:
banner_img:
---

# 使用Flask-Mail提供电子邮件支持

虽然Python标准库中的`smtplib`包可用于在Flask应用中发送电子邮件，但包装了`smtplib`的`Flask-Mail`扩展能更好地与Flask集成。`Flask-Mail`使用`pip`安装：

```shell
(venv)$ pip install falsk-mail
```

`Flask-Mail`连接到简单邮件传输协议`（SMTP, simple mail transfer protocol）`服务器，把邮件交给这个服务器发送。如果不进行配置，则`Flask-Mail`连接`localhost`上的`25`端口，无须验证身份即可发送电子邮件。

`Flask-Mail` SMTP服务器的配置:

| 配置          | 默认值    | 说明                                                  |
| ------------- | --------- | ----------------------------------------------------- |
| MAIL_SERVER   | localhost | 电子邮件服务器的主机名或IP地址                        |
| MAIL_PORT     | 25        | 电子邮件服务器的端口                                  |
| MAIL_USE_TLS  | FALSE     | 启用传输层安全（`TLS，transoprt layer security`）协议 |
| MAIL_USE_SSL  | False     | 启用安全套接层（SSL，secure sockets layer）协议       |
| MAIL_USERNAME | None      | 邮件账户的用户名                                      |
| MAIL_PASSWORD | None      | 邮件账户的密码                                        |

在开发过程中，连接到外部SMTP服务器可能更方便。

配置Flask-Mail使用`GMAIL`。

```python
app.config['MAIL_SERVER'] = 'smtp.googlemail.com'
app.config['MAIL_PORT'] = 587
app.config['MAIL_USE_TLS'] = True
app.config['MAIL_USERNAME'] = os.environ.get('MAIL_USERNAME')
app.config['MAIL_PASSWORD'] = os.environ.get('MAIL_PASSWORD')
```

初始化方法：

```python
from flask import Mail
mail = Mail(app)
```

## 在python shell中发送电子邮件

```python
(venv)$falsk shell
>>>from flask_mail import Message
>>>from run import mail
>>>msg = Message('test mail', sender='you@example.com', recipients=['you@example.com'])
>>>msg.body = 'This is the plain text body'
>>>msg.html = 'This is the <b>HTML</b> body'
>>>with app.app_context():
...		mail.send(msg)
...		
```

注意，`Flask-Mail`的`send()`函数使用`current_app`，因此要在激活的应用上下文中执行。



[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


