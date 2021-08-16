---
title: Scrapy爬取动态页面-scrapy-splash
tags:
  - scrapy
  - splash
comments: true
typora-root-url: Scrapy爬取动态页面
date: 2021-08-15 09:42:11
categories: 爬虫
index_img:
banner_img:
---

在现实中，目前绝大多数网站的页面都是动态页面，动态页面中的部分内容是浏览器运行页面中的`JavaScript`脚本动态生成的，爬取相对困难，这一章来学习如何爬取动态页面。

先来看一个简单的动态页面的例子，在浏览器中打开http://quotes.toscrape.com/js，显示如图所示。

![](image-20210815094432155.png)

页面中有10条名人名言，每一条都包含在一个`<div class="quote">`元素中。在`scrapy shell`环境下尝试爬取页面中的名人名言：

```python
$ scrapy shell http://quotes.toscrape.com/js
>>> response.css('div.quote')
[]
```

![](image-20210815094734907.png)

爬取失败了，在页面中没有找到任何包含名人名言的`<divclass="quote">`元素。这些`<div class="quote">`就是动态内容，从服务器下载的页面中并不包含它们（所以我们爬取失败），浏览器执行了页面中的一段`JavaScript`代码后，它们才被生成出来。

![](image-20210815094857124.png)

```js
var data = [
{
    "tags": [
        "change",
        "deep-thoughts",
        "thinking",
        "world"
    ],
    "author": {
        "name": "Albert Einstein",
        "goodreads_link": "/author/show/9810.Albert_Einstein",
        "slug": "Albert-Einstein"
    },
    "text": "\u201cThe world as we have created it is a process of our thinking. It cannot be changed without changing our thinking.\u201d"
},
{
    "tags": [
        "abilities",
        "choices"
    ],
    "author": {
        "name": "J.K. Rowling",
        "goodreads_link": "/author/show/1077326.J_K_Rowling",
        "slug": "J-K-Rowling"
    },
    "text": "\u201cIt is our choices, Harry, that show what we truly are, far more than our abilities.\u201d"
},
{
    "tags": [
        "inspirational",
        "life",
        "live",
        "miracle",
        "miracles"
    ],
    "author": {
        "name": "Albert Einstein",
        "goodreads_link": "/author/show/9810.Albert_Einstein",
        "slug": "Albert-Einstein"
    },
    "text": "\u201cThere are only two ways to live your life. One is as though nothing is a miracle. The other is as though everything is a miracle.\u201d"
},
{
    "tags": [
        "aliteracy",
        "books",
        "classic",
        "humor"
    ],
    "author": {
        "name": "Jane Austen",
        "goodreads_link": "/author/show/1265.Jane_Austen",
        "slug": "Jane-Austen"
    },
    "text": "\u201cThe person, be it gentleman or lady, who has not pleasure in a good novel, must be intolerably stupid.\u201d"
},
{
    "tags": [
        "be-yourself",
        "inspirational"
    ],
    "author": {
        "name": "Marilyn Monroe",
        "goodreads_link": "/author/show/82952.Marilyn_Monroe",
        "slug": "Marilyn-Monroe"
    },
    "text": "\u201cImperfection is beauty, madness is genius and it's better to be absolutely ridiculous than absolutely boring.\u201d"
},
{
    "tags": [
        "adulthood",
        "success",
        "value"
    ],
    "author": {
        "name": "Albert Einstein",
        "goodreads_link": "/author/show/9810.Albert_Einstein",
        "slug": "Albert-Einstein"
    },
    "text": "\u201cTry not to become a man of success. Rather become a man of value.\u201d"
},
{
    "tags": [
        "life",
        "love"
    ],
    "author": {
        "name": "Andr\u00e9 Gide",
        "goodreads_link": "/author/show/7617.Andr_Gide",
        "slug": "Andre-Gide"
    },
    "text": "\u201cIt is better to be hated for what you are than to be loved for what you are not.\u201d"
},
{
    "tags": [
        "edison",
        "failure",
        "inspirational",
        "paraphrased"
    ],
    "author": {
        "name": "Thomas A. Edison",
        "goodreads_link": "/author/show/3091287.Thomas_A_Edison",
        "slug": "Thomas-A-Edison"
    },
    "text": "\u201cI have not failed. I've just found 10,000 ways that won't work.\u201d"
},
{
    "tags": [
        "misattributed-eleanor-roosevelt"
    ],
    "author": {
        "name": "Eleanor Roosevelt",
        "goodreads_link": "/author/show/44566.Eleanor_Roosevelt",
        "slug": "Eleanor-Roosevelt"
    },
    "text": "\u201cA woman is like a tea bag; you never know how strong it is until it's in hot water.\u201d"
},
{
    "tags": [
        "humor",
        "obvious",
        "simile"
    ],
    "author": {
        "name": "Steve Martin",
        "goodreads_link": "/author/show/7103.Steve_Martin",
        "slug": "Steve-Martin"
    },
    "text": "\u201cA day without sunshine is like, you know, night.\u201d"
}];
    for (var i in data) {
        var d = data[i];
        var tags = $.map(d['tags'], function(t) {
            return "<a class='tag'>" + t + "</a>";
        }).join(" ");
        document.write("<div class='quote'><span class='text'>" + d['text'] + "</span><span>by <small class='author'>" + d['author']['name'] + "</small></span><div class='tags'>Tags: " + tags + "</div></div>");
        }
```
所有名人名言信息被保存在数组data中，最后的for循环迭代data中的每项信息，使用`document.write`生成每条名人名言对应的`<div class="quote">`元素。

上面是动态网页中最简单的一个例子，数据被硬编码于`JavaScript`代码中，实际中更常见的是`JavaScript`通过HTTP请求跟网站动态交互获取数据（`AJAX`），然后使用数据更新HTML页面。爬取此类动态网页需要先执行页面中的`JavaScript`代码渲染页面，再进行爬取。

## Splash渲染引擎

Splash是`Scrapy`官方推荐的JavaScript渲染引擎，它是使用`Webkit`开发的轻量级无界面浏览器，提供基于HTTP接口的JavaScript渲染服务，支持以下功能：

为用户返回经过渲染的HTML页面或页面截图。

- 并发渲染多个页面。
-  关闭图片加载，加速渲染。
-  在页面中执行用户自定义的JavaScript代码。
- 执行用户自定义的渲染脚本（`lua`），功能类似于`PhantomJS`。

首先安装`Splash`，在`linux`下使用`docker`安装十分方便：

```
sudo docker pull scrapinghub/splash
```
安装完成后，在本机的8050和8051端口开启Splash服务：

```shell
[root@VM-8-16-centos docker]#  sudo docker run -it -p 8050:8050 --rm scrapinghub/splash
2021-08-15 02:22:30+0000 [-] Log opened.
2021-08-15 02:22:30.880640 [-] Xvfb is started: ['Xvfb', ':1396582278', '-screen', '0', '1024x768x24', '-nolisten', 'tcp']
QStandardPaths: XDG_RUNTIME_DIR not set, defaulting to '/tmp/runtime-splash'
2021-08-15 02:22:31.273181 [-] Splash version: 3.5
2021-08-15 02:22:32.250100 [-] Qt 5.14.1, PyQt 5.14.2, WebKit 602.1, Chromium 77.0.3865.129, sip 4.19.22, Twisted 19.7.0, Lua 5.2
2021-08-15 02:22:32.251646 [-] Python 3.6.9 (default, Jul 17 2020, 12:50:27) [GCC 8.4.0]
2021-08-15 02:22:32.251796 [-] Open files limit: 1048576
2021-08-15 02:22:32.251837 [-] Can't bump open files limit
2021-08-15 02:22:32.352114 [-] proxy profiles support is enabled, proxy profiles path: /etc/splash/proxy-profiles
2021-08-15 02:22:32.353014 [-] memory cache: enabled, private mode: enabled, js cross-domain access: disabled
2021-08-15 02:22:32.735663 [-] verbosity=1, slots=20, argument_cache_max_entries=500, max-timeout=90.0
2021-08-15 02:22:32.735875 [-] Web UI: enabled, Lua: enabled (sandbox: enabled), Webkit: enabled, Chromium: enabled
2021-08-15 02:22:32.736222 [-] Site starting on 8050
2021-08-15 02:22:32.737289 [-] Starting factory <twisted.web.server.Site object at 0x7f1700b435c0>
2021-08-15 02:22:32.740763 [-] Server listening on http://0.0.0.0:8050
```

Splash功能丰富，包含多个服务端点，由于篇幅有限，这里只介绍其中两个最常用的端点：

- `render.html`提供`JavaScript`页面渲染服务。
- `execute`执行用户自定义的渲染脚本（`lua`），利用该端点可在页面中执行JavaScript代码。

Splash文档地址：http://splash.readthedocs.io/en/latest/api.html。

### `render.html`端点

JavaScript页面渲染服务是`Splash`中最基础的服务:

| 服务端点 | render.html                       |
| -------- | --------------------------------- |
| 请求地址 | http://localhost:8050/render.html |
| 请求方式 | GET/POST                          |
| 返回类型 | html                              |

`render.html`端点支持的参数:

| 参数      | 是否必选 | 类型     | 描述                                         |
| --------- | -------- | -------- | -------------------------------------------- |
| url       | 必选     | string   | 需要渲染页面的url                            |
| timeout   | 可选     | float    | 渲染页面的超时时间                           |
| proxy     | 可选     | string   | 代理服务器地址                               |
| wait      | 可选     | float    | 等待页面渲染的时间                           |
| images    | 可选     | interger | 是否下载图片，默认为1                        |
| js_source | 可选     | string   | 用户自定义的JavaScript代码，在页面渲染前执行 |



下面是使用requests库调用render.html端点服务对页面http://quotes.toscrape.com/js/进行渲染的示例代码。

```python
>>> import requests
>>> from scrapy.selector import Selector
>>> splash_url = 'http://localhost:8050/render.html'
>>> args = {'url': 'http://quotes.toscrape.com/js', 'timeout': '5', 'image': 0}
>>> response = requests.get(splash_url, params=args)
>>> sel = Selector(response)
>>> sel.css('div.quote span.text::text').extract()
['“The world as we have created it is a process of our thinking. It cannot be changed without changing our thinking.”', '“It is our choices, Harry, that show what we truly are, far more than our abilities.”', '“There are only two ways to live your life. One is as though nothing is a miracle. The other is as though everything is a miracle.”', '“The person, be it gentleman or lady, who has not pleasure in a good novel, must be intolerably stupid.”', "“Imperfection is beauty, madness is genius and it's better to be absolutely ridiculous than absolutely boring.”", '“Try not to become a man of success. Rather become a man of value.”', '“It is better to be hated for what you are than to be loved for what you are not.”', "“I have not failed. I've just found 10,000 ways that won't work.”", "“A woman is like a tea bag; you never know how strong it is until it's in hot water.”", '“A day without sunshine is like, you know, night.”']
```

在上述代码中，依据文档中的描述设置参数`url`、`timeout`、`images`，然后发送HTTP请求到服务接口地址。从运行结果看出，页面渲染成功，我们爬取到了页面中的10条名人名言。

### execute端点

在爬取某些页面时，我们想在页面中执行一些用户自定义的JavaScript代码，例如，用JavaScript模拟点击页面中的按钮，或调用页面中的JavaScript函数与服务器交互，利用Splash的execute端点提供的服务可以实现这样的功能。

| 服务端点 | execute                       |
| -------- | ----------------------------- |
| 请求地址 | http://localhost:8050/execute |
| 请求方式 | POST                          |
| 返回类型 | 自定义                        |

execute端点支持的参数：

| 参数       | 必选/可选 | 类型   | 描述                |
| ---------- | --------- | ------ | ------------------- |
| lua_source | 必选      | string | 用户自定义的lua脚本 |
| timeout    | 可选      | float  | 渲染页面的超时时间  |
| proxy      | 可选      | string | 代理服务器地址      |

可以将execute端点的服务看作一个可用`lua`语言编程的浏览器，功能类似于`PhantomJS`。使用时需传递一个用户自定义的`lua`脚本给Splash，该`lua`脚本中包含用户想要模拟的浏览器行为，例如：

- 打开某`url`地址的页面
- 等待页面加载及渲染
- 执行JavaScript代码
- 获取HTTP响应头部
- 获取Cookie

下面是使用requests库调用execute端点服务的示例代码。

```python
>>> import requests
>>> import json
>>> lua_scrpit = '''
...     function main(splash)
...         splash:go("http://example.com")
...         splash: wait(0.5)
...         local title=splash:evaljs("document.title")
...         return{title=title)
...     end
... '''
>>> splash_url = 'http://localhost:8050/execute'
>>> headers = {'content-type': 'application/json'}
>>> data = json.dumps({'lua_source': lua_scrpit})
>>> response = requests.post(splash_url, headers=headers, data=data)
>>> response
<Response [400]>
>>> response.json()
{'error': 400, 'type': 'ScriptError', 'description': 'Error happened while executing Lua script', 'info': {'source': '[string "..."]', 'line_number': 6, 'error': "'}' expected near ')'", 'type': 'LUA_INIT_ERROR', 'message': '[string "..."]:6: \'}\' expected near \')\''}}
```

用户自定义的`lua`脚本中必须包含一个`main`函数作为程序入口，`main`函数被调用时会传入一个`splash`对象（`lua`中的对象），用户可以调用该对象上的方法操纵`Splash`。例如，在上面的例子中，先调用go方法打开某页面，再调用wait方法等待页面渲染，然后调用`evaljs`方法执行一个JavaScript表达式，并将结果转化为相应的`lua`对象，最终Splash根据main函数的返回值构造HTTP响应返回给用户，main函数的返回值可以是字符串，也可以是`lua`中的表（类似Python字典），表会被编码成`json`串。

splash对象常用的属性和方法。

- `splash.args`属性

  用户传入参数的表，通过该属性可以访问用户传入的参数，如`splash.args.url`、`splash.args.wait`。

-  `splash.js_enabled`属性

  用于开启/禁止JavaScript渲染，默认为true。

- `splash.images_enabled`属性

  用于开启/禁止图片加载，默认为true。

- `splash:go`方法

  `splash:go{url, baseurl=nil, headers=nil, http_method="GET",body=nil, formdata=nil}`

  类似于在浏览器中打开某url地址的页面，页面所需资源会被加载，并进行JavaScript渲染，可以通过参数指定HTTP请求头部、请求方法、表单数据等。

- `splash:wait`方法

  `splash:wait{time, cancel_on_redirect=false, cancel_on_error=true}`

  等待页面渲染，time参数为等待的秒数。

- `splash:evaljs`方法

  `splash:evaljs(snippet)`

  在当前页面下，执行一段JavaScript代码，并返回最后一句表达式的值。

-  `splash:runjs`方法

  `splash:runjs(snippet)`

  在当前页面下，执行一段JavaScript代码，与evaljs方法相比，该函数只执行JavaScript代码，不返回值。

- `splash:url`方法

  `splash:url()`

  获取当前页面的`url`。

- `splash:html`方法

  `splash:html()`

  获取当前页面的HTML文本。

- `splash:get_cookies`方法

  `splash:get_cookies()`

  获取全部Cookie信息．

## 在Scrapy中使用Splash

如何在`Scrapy`中调用`Splash`服务，Python库的`scrapy-splash`是非常好的选择。

使用pip安装`scrapy-splash`：

```python
pip install scrapy-splash
```

在项目环境中讲解`scrapy-splash`的使用，创建一个`Scrapy`项目，取名为`splash_examples`：

```python
scrapy startproject scrapy-splash
```

首先在项目配置文件`settings.py`中对`scrapy-splash`进行配置，添加内容如下：

```python
# Splash服务器地址
SPLASH_URL = 'http://localhost:8050'

# 开启Splash的两个下载中间件并调整HttpCompressionMiddleware的次序
DOWNLOADER_MIDDLEWARES = {
    'scrapy_splash.SplashCookiesMiddleware': 723,
    'scrapy_splash.SplashMiddleware': 725,
    'scrapy.downloadermiddlewares.httpcompression.HttpCompressionMiddleware': 810,
}

# 设置去重过滤器
DUPEFILTER_CLASS = 'scrapy_splash.SplashAwareDupeFilter'

# 用来支持cache_args(可选）
SPIDER_MIDDLEWARES = {
    'scrapy_splash.SplashDeduplicateArgsMiddleware': 100
}
```

编写Spider代码过程中，使用`scrapy_splash`调用Splash服务非常简单，`scrapy_splash`中定义了一个`SplashRequest`类，用户只需使用`scrapy_splash.SplashRequest`（替代`scrapy.Request`）提交请求即可。下面是`SplashRequest`构造器方法中的一些常用参数。

- `url`

  与`scrapy.Request`中的`url`相同，也就是待爬取页面的`url`（注意，不是Splash服务器地址）。

- `headers`

  与`scrapy.Request`中的`headers`相同。

- `cookies`

  与`scrapy.Request`中的`cookies`相同。

- `args`

  传递给Splash的参数（除`url`以外），如`wait`、`timeout`、`images`、`js_source`等。

- `cache_args`

  如果`args`中的某些参数每次调用都重复传递并且数据量较大（例如一段JavaScript代码），此时可以把该参数名填入`cache_args`列表中，让Splash服务器缓存该参数，如`SplashRequest(url,args= {'js_source': js,'wait': 0.5}, cache_args = ['js_source'])`。

- `endpointSplash`

  服务端点，默认为`’render.html'`，即JavaScript页面渲染服务，该参数可以设置为`’render.json'`、`'render.har'`、`'render.png'`、`'render.jpeg'`、`'execute’`等

- `splash_url`

  Splash服务器地址，默认为None，即使用配置文件中`SPLASH_URL`的地址。

## 项目实战：爬取toscrape中的名人名言

首先，在`splash_examples`项目目录下创建Spider：

```python
scrapy genspider quotes quotes.toscrape.com
```

接下来只需使用Splash的`render.html`端点渲染页面，再进行爬取即可实现`QuotesSpider`，代码如下：

```python
# coding:utf-8
import scrapy
from scrapy_splash import SplashRequest


class QuotesSpider(scrapy.Spider):
    name = 'quotes'
    allowed_domains = ['quotes.toscrape.com']
    start_urls = ['http://quotes.toscrape.com/js/']

    def start_requests(self):
        for url in self.start_urls:
            # 不下载图片，超时时间3秒
            yield SplashRequest(url, args={'images': 0, 'timeout': 3})

    def parse(self, response):
        for sel in response.css('div.quote'):
            quote = sel.css('span.text::text').extract_first()
            author = sel.css('small.author::text').extract_first()
            yield {'quote': quote, 'author': author}
        href = response.css('li.next > a::attr(href)').extract_first()
        if href:
            url = response.urljoin(href)
            yield SplashRequest(url, args={'images': 0, 'timeout': 3})
```

上述代码中，使用`SplashRequest`提交请求，在`SplashRequest`的构造器中无须传递`endpoint`参数，因为该参数默认值便是`’render.html'`。使用`args`参数禁止`Splash`加载图片，并设置渲染超时时间。

运行爬虫，观察结果：

```python
(Spiders) [root@VM-8-16-centos splash_examples]# scrapy crawl quotes -o quotes.csv
(Spiders) [root@VM-8-16-centos splash_examples]# cat quotes.csv 
quote,author
“The world as we have created it is a process of our thinking. It cannot be changed without changing our thinking.”,Albert Einstein
"“It is our choices, Harry, that show what we truly are, far more than our abilities.”",J.K. Rowling
“There are only two ways to live your life. One is as though nothing is a miracle. The other is as though everything is a miracle.”,Albert Einstein
"“The person, be it gentleman or lady, who has not pleasure in a good novel, must be intolerably stupid.”",Jane Austen
"“Imperfection is beauty, madness is genius and it's better to be absolutely ridiculous than absolutely boring.”",Marilyn Monroe
“Try not to become a man of success. Rather become a man of value.”,Albert Einstein
“It is better to be hated for what you are than to be loved for what you are not.”,André Gide
"“I have not failed. I've just found 10,000 ways that won't work.”",Thomas A. Edison
“A woman is like a tea bag; you never know how strong it is until it's in hot water.”,Eleanor Roosevelt
"“A day without sunshine is like, you know, night.”",Steve Martin
"“This life is what you make it. No matter what, you're going to mess up sometimes, it's a universal truth. But the good part is you get to decide how you're going to mess it up. Girls will be your friends - they'll act like it anyway. But just remember, some come, some go. The ones that stay with you through everything - they're your true best friends. Don't let go of them. Also remember, sisters make the best friends in the world. As for lovers, well, they'll come and go too. And baby, I hate to say it, most of them - actually pretty much all of them are going to break your heart, but you can't give up because if you give up, you'll never find your soulmate. You'll never find that half who makes you whole and that goes for everything. Just because you fail once, doesn't mean you're gonna fail at everything. Keep trying, hold on, and always, always, always believe in yourself, because if you don't, then who will, sweetie? So keep your head high, keep your chin up, and most importantly, keep smiling, because life's a beautiful thing and there's so much to smile about.”",Marilyn Monroe
...
```

运行结果显示，我们成功爬取了10个页面中的100条名人名言。


[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


