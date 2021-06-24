---
title: scrapy框架
tags:
  - python
  - scrapy
comments: true
toc: true
only:
  - home
  - category
  - tag
date: 2021-06-18 14:34:41
categories: Python库
index_img: /2021/06/18/scrapy框架详解/1.webp
---

## 安装

建议在虚拟环境中安装。

```python
pip install scrapy
```

## scrapy使用教程

### 创建一个scrapy项目

在你开始使用 `scrapy` 前，你必须先创建一个`Scrpay project`。输入你要创建的项目目录名称，比如这里的`tutorial`然后运行:

```python
scrapy startproject tutorial
```

这将会创建一个名字为 tutorial 的目录，包含以下内容：

```python
tutorial/
    scrapy.cfg            # 部署的配置文件

    tutorial/             # 项目的python模块
        __init__.py

        items.py          # 项目item定义文件

        middlewares.py    # 项目的中间件文件

        pipelines.py      # 项目的管道文件

        settings.py       # 项目的配置文件

        spiders/          # 存放spider文件的目录
            __init__.py
```

### 开始第一个爬虫

`Spiders`是一些类的集合，他们都是`Spider`的子类。

接下来在`tutorial/spiders`文件下新建一个`quotes_spider.py`文件，写下如下内容：

```python
import scrapy


class QuotesSpider(scrapy.Spider):
    name = "quotes"

    def start_requests(self):
        urls = [
            'http://quotes.toscrape.com/page/1/',
            'http://quotes.toscrape.com/page/2/',
        ]

        for url in urls:
            yield scrapy.Request(url=url, callback=self.parse)

    def parse(self, response):
        page = response.url.split("/")[-2]
        filename = f'qoutes-{page}.html'
        with open(filename, 'wb') as f:
            f.write(response.body)
        self.log(f'Saved file {filename}')
```

如你所见，我们的蜘蛛子类 `scrapy.Spider` 并定义了一些属性和方法：

- `name` ：标识蜘蛛。它在一个项目中必须是唯一的，也就是说，不能为不同的蜘蛛设置相同的名称。

- `start_requests()` ：必须返回一个`ITable of requests`（您可以返回一个请求列表或编写一个生成器函数），蜘蛛将从中开始爬行。随后的请求将从这些初始请求中依次生成。

- `parse()` ：将调用的方法，用于处理为每个请求下载的响应。响应参数是的实例 `TextResponse` 它保存页面内容，并有进一步有用的方法来处理它。

这个 `parse()` 方法通常解析响应，将抓取的数据提取为`dict`，并查找新的URL以跟踪和创建新的请求。

### 运行爬虫

在项目的顶级目录中运行，比如这里的 `tutorial/` ：

```python
scrapy crawl quotes
```

你将看到以下输出：

```python
2021-06-18 22:30:22 [scrapy.utils.log] INFO: Scrapy 2.5.0 started (bot: tutorial)
2021-06-18 22:30:22 [scrapy.utils.log] INFO: Versions: lxml 4.6.3.0, libxml2 2.9.5, cssselect 1.1.0, parsel 1.6.0, w3lib 1.22.0, Twisted 21.2.0, Python 3.9.5 (tags/v3.9.5:0a7dcbd, May  3 2021, 17:27:52) [MSC v.1928 64 bit (AMD64)], pyOpenSSL 20.0.1 (OpenSSL 1.1.1k  25 Mar 2021), cryptography 3.4.7, Platform Windows-10-10.0.19041-SP0
2021-06-18 22:30:22 [scrapy.utils.log] DEBUG: Using reactor: twisted.internet.selectreactor.SelectReactor
2021-06-18 22:30:22 [scrapy.crawler] INFO: Overridden settings:
{'BOT_NAME': 'tutorial',
 'NEWSPIDER_MODULE': 'tutorial.spiders',
 'ROBOTSTXT_OBEY': True,
 'SPIDER_MODULES': ['tutorial.spiders']}
......
```

检查当前目录中的文件。您应该注意到已经创建了两个新文件： `quotes-1.html` 和 `引用-2.HTML`。

## scrapy的工作流程



上面的流程在 `scrapy` 中分为下面几个部分：

- Scheduler: 用来接受引擎发过来的请求并加入队列中，并在引擎再次请求的时候提供给引擎。
- Downloader: 下载器
- ScrapyEngine: 用来处理整个系统的数据流处理、触发事务，是整个框架的核心。
- ItemPipeline：项目管道
- Sipders：定义爬取的逻辑



[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


