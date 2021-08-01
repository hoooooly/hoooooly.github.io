---
title: 使用Flask-Moment本地化日期和时间
tags:
  - flask
  - python
comments: true
typora-root-url: 使用Flask-Moment本地化日期和时间
date: 2021-07-30 11:15:25
categories: Flask
index_img:
banner_img:
---

有一个使用JavaScript开发的优秀客户端开源库，名为Moment.js，它可以在浏览器中渲染日期和时间。Flask-Moment是一个Flask扩展，能简化把Moment.js集成到Jinja2模板中的过程。Flask-Moment使用pip安装：

```shell
(venv)$ pip install flask-moment
```

初始化

```python
from flask_moment import Moment
# ...

Moment(app)

# ...
```

除了Moment.js, Flask-Moment还依赖jQuery.js。因此，要在HTML文档的某个地方引入这两个库，可以直接引入，这样可以选择使用哪个版本，也可以使用扩展提供的辅助函数，从内容分发网络（CDN, content delivery network）中引入通过测试的版本。Bootstrap已经引入了jQuery.js，因此只需引入Moment.js即可。

templates/index.html：引入Moment.js库

```html
{% block scripts %}}
    {{ super() }}
    {{ moment.include_moment()}}
{% endblock %}}
```

为了处理时间戳，Flask-Moment向模板开放了moment对象。把变量current time传入模板进行渲染。

```python
from datetime import datetime

@app.route('/')
def index():
    return render_template('index.html', name="world", current_time=datetime.utcnow())
```

在模板中渲染：

```html
<p>The local date and time is {{ moment(current_time).format('LLL') }}.</p>
<p>That was {{ moment(current_time).fromNow(refresh=True) }}</p>
```

实现效果：

![](image-20210730113140244.png)

Flask-Moment实现了Moment.js的format()、fromNow()、fromTime()、calendar()、valueOf()和unix()等方法。请查阅Moment.js的文档（http://momentjs.com/docs/#/displaying/），学习这个库提供的全部格式化选项。

Flask-Moment渲染的时间戳可实现多种语言的本地化。语言可在模板中选择，方法是在引入Moment.js之后，立即把两个字母的语言代码传给locale()函数。例如，配置Moment. js使用汉语的方式如下：

```html
{% block scripts %}}
    {{ super() }}
    {{ moment.include_moment()}}
    {{ moment.locale('zh-cn') }}
{% endblock %}}
```

![](image-20210730113419711.png)



[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


