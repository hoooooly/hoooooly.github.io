---
title: Scrapy项目练习爬取books
tags:
  - Hexo
  - Fluid
comments: true
typora-root-url: Scrapy项目练习爬取books
date: 2021-08-11 17:59:45
categories: 爬虫
index_img:
banner_img:
---

## 项目需求

目标网站：http://books.toscrape.com

网站截图：

![](image-20210812145527146.png)

项目目标：

爬取http://books.toscrape.com网站中的书籍信息。

（1）其中每一本书的信息包括：

​		➢ 书名

​		➢ 价格

​		➢ 评价等级

​		➢ 产品编码

​		➢ 库存量

​		➢ 评价数量

（2）将爬取的结果保存到`csv`文件中。

<img src="image-20210812145846244.png" />

## 页面分析

首先，我们对一本书的页面进行分析。在进行页面分析时，除了之前使用过的`Chrome`开发者工具外，另一个常用的工具是`scrapy shell <URL>`命令，它使用户可以在交互式命令行下操作一个`Scrapy`爬虫，通常我们利用该工具进行前期爬取实验，从而提高开发效率。

接下来分析第一本书的页面，以页面的`url`地址为参数运行`scrapy shell`命令：

```python
(venv) $ scrapy shell https://books.toscrape.com/catalogue/a-light-in-the-attic_1000/index.html
2021-08-12 15:04:28 [scrapy.utils.log] INFO: Scrapy 2.5.0 started (bot: books)
2021-08-12 15:04:28 [scrapy.utils.log] INFO: Versions: lxml 4.6.3.0, libxml2 2.9.5, cssselect 1.1.0, parsel 1.6.0, w3lib 1.22.0, Twisted 21.7.0, Py
thon 3.9.5 (tags/v3.9.5:0a7dcbd, May  3 2021, 17:27:52) [MSC v.1928 64 bit (AMD64)], pyOpenSSL 20.0.1 (OpenSSL 1.1.1k  25 Mar 2021), cryptography 3
.4.7, Platform Windows-10-10.0.19041-SP0
2021-08-12 15:04:28 [scrapy.utils.log] DEBUG: Using reactor: twisted.internet.selectreactor.SelectReactor
2021-08-12 15:04:28 [scrapy.crawler] INFO: Overridden settings:
{'BOT_NAME': 'books',
 'DUPEFILTER_CLASS': 'scrapy.dupefilters.BaseDupeFilter',
 'LOGSTATS_INTERVAL': 0,
 'NEWSPIDER_MODULE': 'books.spiders',
 'ROBOTSTXT_OBEY': True,
 'SPIDER_MODULES': ['books.spiders']}
2021-08-12 15:04:28 [scrapy.extensions.telnet] INFO: Telnet Password: a610410a1da32370
2021-08-12 15:04:28 [scrapy.middleware] INFO: Enabled extensions:
['scrapy.extensions.corestats.CoreStats',
 'scrapy.extensions.telnet.TelnetConsole']
2021-08-12 15:04:29 [scrapy.middleware] INFO: Enabled downloader middlewares:
['scrapy.downloadermiddlewares.robotstxt.RobotsTxtMiddleware',
 'scrapy.downloadermiddlewares.httpauth.HttpAuthMiddleware',
 'scrapy.downloadermiddlewares.downloadtimeout.DownloadTimeoutMiddleware',
 'scrapy.downloadermiddlewares.defaultheaders.DefaultHeadersMiddleware',
 'scrapy.downloadermiddlewares.useragent.UserAgentMiddleware',
 'scrapy.downloadermiddlewares.retry.RetryMiddleware',
 'scrapy.downloadermiddlewares.redirect.MetaRefreshMiddleware',
 'scrapy.downloadermiddlewares.httpcompression.HttpCompressionMiddleware',
 'scrapy.downloadermiddlewares.redirect.RedirectMiddleware',
 'scrapy.downloadermiddlewares.cookies.CookiesMiddleware',
 'scrapy.downloadermiddlewares.httpproxy.HttpProxyMiddleware',
 'scrapy.downloadermiddlewares.stats.DownloaderStats']
2021-08-12 15:04:29 [scrapy.middleware] INFO: Enabled spider middlewares:
['scrapy.spidermiddlewares.httperror.HttpErrorMiddleware',
 'scrapy.spidermiddlewares.offsite.OffsiteMiddleware',
 'scrapy.spidermiddlewares.referer.RefererMiddleware',
 'scrapy.spidermiddlewares.urllength.UrlLengthMiddleware',
 'scrapy.spidermiddlewares.depth.DepthMiddleware']
2021-08-12 15:04:29 [scrapy.middleware] INFO: Enabled item pipelines:
[]
2021-08-12 15:04:29 [scrapy.extensions.telnet] INFO: Telnet console listening on 127.0.0.1:6023
2021-08-12 15:04:29 [scrapy.core.engine] INFO: Spider opened
2021-08-12 15:04:31 [scrapy.core.engine] DEBUG: Crawled (404) <GET https://books.toscrape.com/robots.txt> (referer: None)
2021-08-12 15:04:32 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://books.toscrape.com/catalogue/a-light-in-the-attic_1000/index.html> (referer: None)
[s] Available Scrapy objects:
[s]   scrapy     scrapy module (contains scrapy.Request, scrapy.Selector, etc)
[s]   crawler    <scrapy.crawler.Crawler object at 0x00000172F53B3100>
[s]   item       {}
[s]   request    <GET https://books.toscrape.com/catalogue/a-light-in-the-attic_1000/index.html>
[s]   response   <200 https://books.toscrape.com/catalogue/a-light-in-the-attic_1000/index.html>
[s]   settings   <scrapy.settings.Settings object at 0x00000172F53AF940>
[s]   spider     <DefaultSpider 'default' at 0x172f56b95b0>
[s] Useful shortcuts:
[s]   fetch(url[, redirect=True]) Fetch URL and update local objects (by default, redirects are followed)
[s]   fetch(req)                  Fetch a scrapy.Request and update local objects
[s]   shelp()           Shell help (print this help)
[s]   view(response)    View response in a browser
```

运行这条命令后，`scrapy shell`会使用`url`参数构造一个`Request`对象，并提交给`Scrapy`引擎，页面下载完成后，程序进入一个`python shell`当中，在此环境中已经创建好了一些变量（对象和函数），以下几个最为常用：

● request最近一次下载对应的Request对象。

● response最近一次下载对应的Response对象。

● `fetch(req_or_url)`该函数用于下载页面，可传入一个Request对象或`url`字符串，调用后会更新变量request和response。

● `view(response)`该函数用于在浏览器中显示response中的页面。

接下来，在`scrapy shell`中调用`view`函数，在浏览器中显示response所包含的页面：

```python
view(response)
```

可能在很多时候，使用view函数打开的页面和在浏览器直接输入`url`打开的页面看起来是一样的，但需要知道的是，前者是由`Scrapy`爬虫下载的页面，而后者是由浏览器下载的页面，有时它们是不同的。在进行页面分析时，使用view函数更加可靠。

![](image-20210812152753401.png)

从图中看出，我们可在`<div class="col-sm-6 product_main">`中提取书名、价格、评价等级，在`scrapy shell`中尝试提取这些信息:

```python
>>> sel = response.css('div.product_main')
>>> sel.xpath('./h1/text()').extract_first()
'A Light in the Attic'
>>> sel.css('p.price_color::text').extract_first()
'£51.77'
>>> sel.css('p.star-rating::attr(class)').re_first('star-rating ([A-Za-z]+)')
'Three'
```

![](image-20210812223121867.png)

另外，可在页面下端位置的`<table class="table table-striped">`中提取产品编码、库存量、评价数量，在`scrapy shell`中尝试提取这些信息：

```python
>>> sel.xpath('(.//tr)[1]/td/text()').extract_first()
'a897fe39b1053632'

>>> sel.xpath('(.//tr)[last()-1]/td/text()').re_first('\((\d+) available\)')
'22'

>>> sel.xpath('(.//tr)[last()]/td/text()').extract_first()
'0'
```

分析完书籍页面后，接着分析如何在书籍列表页面中提取每一个书籍页面的链接。在`scrapy shell`中，先调用`fetch`函数下载第一个书籍列表页面（http://books.toscrape.com/），下载完成后再调用view函数在浏览器中查看页面：

```python
>>> fetch('http://books.toscrape.com/')
2021-08-12 22:43:09 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://books.toscrape.com/> (referer: None)
>>> view(response)
True
```

![](image-20210812224736039.png)

每个书籍页面的链接可以在每个`<article class="product_pod">`中找到，在`scrapy shell`中使用`LinkExtractor`提取这些链接：

```python
>>> from scrapy.linkextractors import LinkExtractor
>>> le = LinkExtractor(restrict_css='article.product_pod')
>>> le.extract_links(response)
[Link(url='http://books.toscrape.com/catalogue/a-light-in-the-attic_1000/index.html', text='', fragment='', nofollow=False), Link(url='http://books
.toscrape.com/catalogue/tipping-the-velvet_999/index.html', text='', fragment='', nofollow=False), Link(url='http://books.toscrape.com/catalogue/so
umission_998/index.html', text='', fragment='', nofollow=False), Link(url='http://books.toscrape.com/catalogue/sharp-objects_997/index.html', text=
'', fragment='', nofollow=False), Link(url='http://books.toscrape.com/catalogue/sapiens-a-brief-history-of-humankind_996/index.html', text='', frag
ment='', nofollow=False), Link(url='http://books.toscrape.com/catalogue/the-requiem-red_995/index.html', text='', fragment='', nofollow=False), Lin
k(url='http://books.toscrape.com/catalogue/the-dirty-little-secrets-of-getting-your-dream-job_994/index.html', text='', fragment='', nofollow=False
), Link(url='http://books.toscrape.com/catalogue/the-coming-woman-a-novel-based-on-the-life-of-the-infamous-feminist-victoria-woodhull_993/index.ht
ml', text='', fragment='', nofollow=False), Link(url='http://books.toscrape.com/catalogue/the-boys-in-the-boat-nine-americans-and-their-epic-quest-
for-gold-at-the-1936-berlin-olympics_992/index.html', text='', fragment='', nofollow=False), Link(url='http://books.toscrape.com/catalogue/the-blac
k-maria_991/index.html', text='', fragment='', nofollow=False), Link(url='http://books.toscrape.com/catalogue/starving-hearts-triangular-trade-tril
ogy-1_990/index.html', text='', fragment='', nofollow=False), Link(url='http://books.toscrape.com/catalogue/shakespeares-sonnets_989/index.html', t
ext='', fragment='', nofollow=False), Link(url='http://books.toscrape.com/catalogue/set-me-free_988/index.html', text='', fragment='', nofollow=Fal
se), Link(url='http://books.toscrape.com/catalogue/scott-pilgrims-precious-little-life-scott-pilgrim-1_987/index.html', text='', fragment='', nofol
low=False), Link(url='http://books.toscrape.com/catalogue/rip-it-up-and-start-again_986/index.html', text='', fragment='', nofollow=False), Link(ur
l='http://books.toscrape.com/catalogue/our-band-could-be-your-life-scenes-from-the-american-indie-underground-1981-1991_985/index.html', text='', f
ragment='', nofollow=False), Link(url='http://books.toscrape.com/catalogue/olio_984/index.html', text='', fragment='', nofollow=False), Link(url='h
ttp://books.toscrape.com/catalogue/mesaerion-the-best-science-fiction-stories-1800-1849_983/index.html', text='', fragment='', nofollow=False), Lin
k(url='http://books.toscrape.com/catalogue/libertarianism-for-beginners_982/index.html', text='', fragment='', nofollow=False), Link(url='http://bo
oks.toscrape.com/catalogue/its-only-the-himalayas_981/index.html', text='', fragment='', nofollow=False)]
```

## 编码实现

首先创建一个`Scrapy`项目，取名为`toscrape_book`。

```python
scrapy startproject toscrape_book
```

通常，我们不需要手工创建Spider文件以及Spider类，可以使用`scrapygenspider<SPIDER_NAME> <DOMAIN>`命令生成（根据模板）它们，该命令的两个参数分别是Spider的名字和所要爬取的域（网站）：

```python
cd toscrape_book
scrapy genspider books books.toscrape.com
```

运行后，`scrapy genspider`命令创建了文件`toscrape_book/spiders/books.py`，并在其中创建了一个`BooksSpider`类，代码如下：

```python
import scrapy


class BooksSpider(scrapy.Spider):
    name = 'books'
    allowed_domains = ['books.toscrape.com']
    start_urls = ['http://books.toscrape.com/']

    def parse(self, response):
        pass
```

实现Spider之前，先定义封装书籍信息的Item类，在`toscrape_book/items.py`中添加如下代码：

```python
import scrapy


class BookItem(scrapy.Item):
    name = scrapy.Field()  # 书名
    price = scrapy.Field()  # 价格
    review_rating = scrapy.Field()  # 评价等级1-5星
    review_num = scrapy.Field()  # 评价数量
    upc = scrapy.Field()       # 产品编码
    stock = scrapy.Field()  # 库存量
```

接下来，按以下5步完成`BooksSpider`。

- **步骤01** 继承Spider创建`BooksSpider`类（已完成）。
- **步骤02** 为Spider取名（已完成）。
- **步骤03** 指定起始爬取点（已完成）。
- **步骤04** 实现书籍列表页面的解析函数。
- **步骤05** 实现书籍页面的解析函数。

其中前3步已经由`scrapy genspider`命令帮我们完成，不需做任何修改。

第4步和第5步的工作是实现两个页面解析函数，因为起始爬取点是一个书籍列表页面，我们就将parse方法作为书籍列表页面的解析函数，另外，还需要添加一个`parse_book`方法作为书籍页面的解析函数，代码如下：

```python
class BooksSpider(scrapy.Spider):
    name = 'books'
    allowed_domains = ['books.toscrape.com']
    start_urls = ['http://books.toscrape.com/']

    # 书籍列表页面的解析函数
    def parse(self, response):
        pass

    # 数据页面的解析函数
    def parse_book(self, response):
        pass
```

先来完成第4步，实现书籍列表页面的解析函数（parse方法），需要完成以下两个任务：

（1）提取页面中每一个书籍页面的链接，用它们构造Request对象并提交。

（2）提取页面中下一个书籍列表页面的链接，用其构造Request对象并提交。提取链接的具体细节在页面分析时已经讨论过，实现代码如下：

```python
```





[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


