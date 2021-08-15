---
title: Scrapy爬取动态页面
tags:
  - scrapy
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












[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


