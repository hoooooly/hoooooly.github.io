---
title: Scrapy中的Request和Response对象
tags:
  - scrpay
  - Fluid
comments: true
typora-root-url: Scrapy中的Request和Response对象
date: 2021-07-27 22:22:22
categories: 爬虫
index_img:
banner_img:
---

# `Request`对象

`Request`对象用来描述一个`HTTP`请求，下面是其构造器方法的参数列表：

```python
Resquest(url, callback=None, method='GET', headers=None, body=None,
                 cookies=None, meta=None, encoding='utf-8', priority=0,
                 dont_filter=False, errback=None, flags=None, cb_kwargs=None)
```

- `url`：必选参数，请求的地址。
- `callback`：页面解析函数，`Callable`类型，Request对象请求的页面下载完成后，由该参数指定的页面解析函数被调用。如果未传递该参数，默认调用`Spider`的`parse`方法。
- `method`：HTTP请求的方法，默认为`’GET'`。
- `headers`：HTTP请求的头部字典，`dict`类型，例如`{'Accept': 'text/html', 'User-Agent':Mozilla/5.0'}`。如果其中某项的值为`None`，就表示不发送该项HTTP头部，例如`{'Cookie': None}`，禁止发送`Cookie`。
- `body`：HTTP请求的正文，`bytes`或`str`类型。
- `cookies`：Cookie信息字典，`dict`类型，例如`{'currency': 'USD', 'country': 'UY'}`
- `meta`：Request的元数据字典，`dict`类型，用于给框架中其他组件传递信息，比如中间件`ItemPipeline`。其他组件可以使用Request对象的`meta`属性访问该元数据字典`（request.meta）`，也用于给响应处理函数传递信息，详见Response的meta属性。
- `encoding`：`url`和`body`参数的编码默认为`’utf-8'`。如果传入的`url`或`body`参数是`str`类型，就使用该参数进行编码。
-  `priority`：请求的优先级默认值为0，优先级高的请求优先下载。
-  `dont_filter`：默认情况下（`dont_filter=False`），对同一个`url`地址多次提交下载请求，后面的请求会被去重过滤器过滤（避免重复下载）。如果将该参数置为`True`，可以使请求避免被过滤，强制下载。例如，在多次爬取一个内容随时间而变化的页面时（每次使用相同的`url`），可以将该参数置为`True`。
- `errback`：请求出现异常或者出现HTTP错误时（如404页面不存在）的回调函数。

# Response对象

Response对象用来描述一个HTTP响应，Response只是一个基类，根据响应内容的不同有如下子类：

- `TextResponse`
-  `HtmlResponse`
-  `XmlResponse`

当一个页面下载完成时，下载器依据HTTP响应头部中的`Content-Type`信息创建某个Response的子类对象。我们通常爬取的网页，其内容是HTML文本，创建的便是`HtmlResponse`对象，其中`HtmlResponse`和`XmlResponse`是`TextResponse`的子类。实际上，这3个子类只有细微的差别，这里以`HtmlResponse`为例进行讲解。

下面介绍`HtmlResponse`对象的属性及方法。

- `url`：HTTP响应的`url`地址，`str`类型。

- `status`：HTTP响应的状态码，`int`类型，例如200, 404。

- `headers`：HTTP响应的头部，类字典类型，可以调用`get`或`getlist`方法对其进行访问。

  ```python
  response.headers.get('Content-Type')
  response.headers.getlist('Set-Cookie')
  ```

- `body`：HTTP响应正文，`bytes`类型。

-  `text`：文本形式的HTTP响应正文，`str`类型，它是由`response.body`使用`response.encoding`解码得到的，即`response.text=response.body.decode(response.encoding)`。

- `encoding`：HTTP响应正文的编码，它的值可能是从HTTP响应头部或正文中解析出来的。

- `request`：产生该HTTP响应的`Request`对象。

- `meta`：即`response.request.meta`，在构造`Request`对象时，可将要传递给响应处理函数的信息通过`meta`参数传入；响应处理函数处理响应时，通过`response.meta`将信息取出。

- `selector`：Selector对象用于在`Response`中提取数据（选择器相关话题在后面章节详细讲解）。

- `xpath(query)`：使用`XPath`选择器在`Response`中提取数据，实际上它是`response.selector.xpath`方法的快捷方式。

- `css(query)`：使用`CSS选择器`在`Response`中提取数据，实际上它是`response.selector.css`方法的快捷方式（选择器相关话题在后面章节详细讲解）。

- `urljoin（url）`：用于构造绝对`url`。当传入的`url`参数是一个相对地址时，根据`response.url`计算出相应的绝对`url`。例如，`response.url`为`http://www.example.com/a`, `url`为`b/index.html`，调用`response.urljoin(url)`的结果为`http://www.example.com/a/b/index.html`。







[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


