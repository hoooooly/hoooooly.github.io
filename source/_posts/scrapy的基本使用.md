---
title: scrapy的基本使用
tags:
  - scrapy
comments: true
toc: true
only:
  - home
  - category
  - tag
date: 2021-05-29 22:49:55
categories: 爬虫
pic:
---

## 抓取目标

要完成的任务如下。

- 创建一个`Scrapy`项目。
- 创建一个`Spider`来抓取站点和处理数据。
- 通过命令行将抓取的内容导出。
- 将抓取的内容保存到`MongoDB`数据库。

抓取的目标站点为[](http://quotes.toscrape.com/)。

## 准备工作

安装好`scrapy`框架、`MongoDB`和`PyMongo`库。

## 创建项目

创建一个`Scrapy`项目，项目文件可以直接用`Scrapy`命令生成，命令如下所示：

```python
scrapy startproject tutorial
```

这个命令将会创建一个名为`tutorial`的文件夹，文件夹结构如下所示：

```python
scrapy.cfg    # Scrapy 部署时的配置文件 
tutorial    # 项目的模块，引入的时候需要从这里引入 
  __init__.py 
  items.py    # Items 的定义，定义爬取的数据结构 
  middlewares.py     # Middlewares 的定义，定义爬取时的中间件 
  pipelines.py    # Pipelines 的定义，定义数据管道 
  settings.py     # 配置文件 
  spiders     # 放置 Spiders 的文件夹 
    __init__.py
```

## 创建Spider

`Spider`是自己定义的类，`Scrapy`用它从网页里抓取内容，并解析抓取的结果。不过这个类必须继承`Scrapy`提供的`Spider`类`scrapy.Spider`，还要定义`Spider`的名称和起始请求，以及怎样处理爬取后的结果的方法。

也可以使用命令行创建一个`Spider`。比如要生成`Quotes`这个`Spider`，可以执行如下命令：

```python
>>> scrapy genspider quotes quotes.toscrape.com     
Created spider 'quotes' using template 'basic' in module:
  tutorial.spiders.quotes
```

进入刚才创建的`tutorial`文件夹，然后执行`genspider`命令。第一个参数是`Spider`的名称，第二个参数是`网站域名`。执行完毕之后，`spiders`文件夹中多了一个`quotes.py`，它就是刚刚创建的`Spider`，内容如下所示：

```python
import scrapy

class QuotesSpider(scrapy.Spider):
    name = 'quotes'
    allowed_domains = ['quotes.toscrape.com']
    start_urls = ['http://quotes.toscrape.com/']

    def parse(self, response):
        pass
```

这里有三个属性——`name`、`allowed_domains`和`start_urls`，还有一个方法`parse`。

- `name`：它是每个项目唯一的名字，用来区分不同的`Spider`。
- `allowed_domains`：它是允许爬取的域名，如果初始或后续的请求链接不是这个域名下的，则请求链接会被过滤掉。
- `start_urls`：它包含了`Spider`在启动时爬取的`url`列表，初始请求是由它来定义的。
- `parse`：它是`Spider`的一个方法。默认情况下，被调用时`start_urls`里面的链接构成的请求完成下载执行后，返回的响应就会作为唯一的参数传递给这个函数。该方法负责解析返回的响应、提取数据或者进一 步生成要处理的请求。

## 创建Item

`Item`是保存爬虫数据的容器，它的使用方法和字典类似，不过相比字典，`Item`多了额外的保护机制，可以避免拼写错误或者定义字段错误。

创建`Item`需要继承`scrapy.Item`类，并且定义类型为`scrapy.Field`的字段。观察目标网站，可以获取到的内容有`text`、`author`、`tags`。

定义`Item`，此时将`items.py`修改如下：

```python
import scrapy

class QuoteItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    text = scrapy.Field()
    author = scrapy.Field()
    tags = scrapy.Field()
```

这里定义了三个字段，将类的名称修改为`QuoteItem`，接下来爬取时会使用到这个`Item`。

## 解析Response

`parse`方法的参数`response`是`start_urls`里面的链接爬取后的结果。所以在`parse`方法中，可以直接对`response`变量包含的内容进行解析，比如浏览请求结果的网页源代码，或者进一步分析源代码内容，或者找出结果中的链接而得到下一个请求。

首先看看网页结构，如图所示。每一页都有多个`class`为`quote`的区块，每个区块内都包含`text`、`author`、`tags`。那么先找出所有的`quote`，然后提取每一个`quote`中的内容。

![](Screenshot_1.webp)

提取的方式可以是`CSS`选择器或`XPath`选择器。在这里使用`CSS`选择器进行选择，`parse`方法的改写如下所示：

```python
class QuotesSpider(scrapy.Spider):
    name = 'quotes'
    allowed_domains = ['http://quotes.toscrape.com/']
    start_urls = ['http://http://quotes.toscrape.com//']

    def parse(self, response):
        quotes = response.css('.quote')
        for quote in quotes:
            text = quote.css('.text::text').extract_first()
            author = quote.css('.author::text').extract_first()
            tags = quote.css('.tags .tag::text').extract()
```

对于`text`，获取结果的第一个元素即可，所以使用`extract_first`方法，对于`tags`，要获取所有结果组成的列表，所以使用`extract`方法。

## 使用Item

上文定义了`Item`，接下来就要使用它了。`Item`可以理解为一个字典，不过在声明的时候需要实例化。然后依次用刚才解析的结果赋值`Item`的每一个字段，最后将`Item`返回即可。

`QuotesSpider`的改写如下所示：

```python
    def parse(self, response):
        quotes = response.css('.quote')
        for quote in quotes:
            item = QuoteItem()
            item['text'] = quote.css('.text::text').extract_first()
            item['author'] = quote.css('.author::text').extract_first()
            item['tags'] = quote.css('.tags .tag::text').extract()
            yield item
```

如此一来，首页的所有内容被解析出来，并被赋值成了一个个`QuoteItem`。

## 后续Request

接下来就需要从当前页面中找到信息来生成下一个请求，然后在下一个请求的页面里面找到信息再构造下一个请求。这样循环往复迭代，从而实现整站的爬取。

将刚才的页面拉到最底部，如图所示。

![](Screenshot_2.webp)

有一个`Next`按钮，查看一下源代码，可以发现它的链接是`/page/2/`，实际上全链接就是：`http://quotes.toscrape.com/page/2`，通过这个链接我们就可以构造下一个请求。

构造请求时需要用到`scrapy.Request`。这里传递两个参数——`url`和`callback`，这两个参数的说明如下。

- `url`：它是请求链接。
- `callback`：它是回调函数。当指定了该回调函数的请求完成之后，获取到响应，引擎会将该响应作为参数传递给这个回调函数。回调函数进行解析或生成下一个请求，回调函数如上文的`parse()`所示。

由于`parse`就是解析`text`、`author`、`tags`的方法，而下一页的结构和刚才已经解析的页面结构是一样的，所以可以再次使用`parse`方法来做页面解析。

接下来要做的就是利用选择器得到下一页链接并生成请求，在`parse`方法后追加如下的代码：

```python
next = response.css('.pager .next a::attr(href)').extract_first()
url = response.urljoin(next)
yield scrapy.Request(url=url, callback=self.parse, dont_filter=True)
```

第一句代码首先通过`CSS`选择器获取下一个页面的链接，即要获取`a`超链接中的`href`属性。这里用到了`::attr(href)`操作。然后再调用`extract_first`方法获取内容。

第二句代码调用了`urljoin`方法，`urljoin()`方法可以将相对`URL`构造成一个绝对的`URL`。例如，获取到的下一页地址是`/page/2`，`urljoin`方法处理后得到的结果就是：`http://quotes.toscrape.com/page/2/`。

第三句代码通过`url`和`callback`变量构造了一个新的请求，回调函数`callback`依然使用`parse`方法。这个请求完成后，响应会重新经过`parse`方法处理，得到第二页的解析结果，然后生成第二页的下一页，也就是第三页的请求。这样爬虫就进入了一个循环，直到最后一页。`dont_filter`设置为`Ture`不进行域名过滤，这样就能继续爬取。

通过几行代码，就轻松实现了一个抓取循环，将每个页面的结果抓取下来了。现在，改写之后的整个`Spider`类如下所示：

```python
import scrapy
from tutorial.items import QuoteItem


class QuotesSpider(scrapy.Spider):
    name = 'quotes'
    allowed_domains = ['quotes.toscrape.com/']
    start_urls = ['http://quotes.toscrape.com/']

    def parse(self, response):
        quotes = response.css('.quote')
        for quote in quotes:
            item = QuoteItem()
            item['text'] = quote.css('.text::text').extract_first()
            item['author'] = quote.css('.author::text').extract_first()
            item['tags'] = quote.css('.tags .tag::text').extract()
            yield item

        next = response.css('.pager .next a::attr(href)').extract_first()
        if next !== "":
          url = response.urljoin(next)
          yield scrapy.Request(url=url, callback=self.parse, dont_filter=True)
```

## 运行

进入目录，运行如下命令：

```python
scrapy crawl quotes
```

爬虫一边解析，一边翻页，直至将所有内容抓取完毕，然后终止。

最后，`Scrapy`输出了整个抓取过程的统计信息，如请求的字节数、请求次数、响应次数、完成原因等。

## 保存到文件

`Scrapy`提供的`Feed Exports`可以轻松将抓取结果输出。例如，想将上面的结果保存成`JSON`文件，可以执行如下命令：

```python
scrapy crawl quotes -o quotes.json
```

命令运行后，项目内多了一个`quotes.json`文件，文件包含了刚才抓取的所有内容，内容是`JSON`格式。

![](Screenshot_3.webp)

另外还可以每一个`Item`输出一行`JSON`，输出后缀为`jl`，为`jsonline`的缩写，命令如下所示：

```python
scrapy crawl quotes -o quotes.jl
```

或

```python
scrapy crawl quotes -o quotes.jsonlines
```

输出格式还支持很多种，例如`csv`、`xml`、`pickle`、`marshal`等，还支持`ftp`、`s3` 等远程输出，另外还可以通过自定义`ItemExporter`来实现其他的输出。

例如，下面命令对应的输出分别为`csv`、`xml`、`pickle`、`marshal`格式以及`ftp`远程输出：

```python
scrapy crawl quotes -o quotes.csv
scrapy crawl quotes -o quotes.xml
scrapy crawl quotes -o quotes.pickle
scrapy crawl quotes -o quotes.marshal
scrapy crawl quotes -o ftp://user:pass@ftp.example.com/path/to/quotes.csv 
```

其中，`ftp`输出需要正确配置用户名、密码、地址、输出路径，否则会报错。通过`Scrapy`提供的`Feed Exports`，可以轻松地输出抓取结果到文件。对于一些小型项目来说，这应该足够了。不过如果想要更复杂的输出，如输出到数据库等，可以使用`ItemPileline`来完成。

## 使用ItemPileline

如果想进行更复杂的操作，如将结果保存到`MongoDB`数据库，或者筛选某些有用的`Item`，则可以定义`ItemPipeline`来实现。

`ItemPipeline`为项目管道。当`Item`生成后，它会自动被送到`ItemPipeline`进行处理，常用`ItemPipeline`来做如下操作。

- 清洗`HTML`数据；
- 验证爬取数据，检查爬取字段；
- 查重并丢弃重复内容；
- 将爬取结果储存到数据库。

要实现`ItemPipeline`很简单，只需要定义一个类并实现`process_item`方法即可。启用`ItemPipeline`后，`ItemPipeline`会自动调用这个方法。`process_item`方法必须返回包含数据的字典或`Item`对象，或者抛出`DropItem`异常。

`process_item`方法有两个参数。一个参数是`item`，每次`Spider`生成的`Item`都会作为参数传递过来。另一个参数是`spider`，就是`Spider`的实例。

接下来，实现一个`ItemPipeline`，筛掉`text`长度大于`50`的`Item`，并将结果保存到`MongoDB`。

修改项目里的`pipelines`.py文件，之前用命令行自动生成的文件内容可以删掉，增加一个`TextPipeline`类，内容如下所示：

```python
from scrapy.exceptions import DropItem

class TextPipeline(object):
    def __init__(self):
        self.limit = 50
    
    def process_item(self, item, spider):
        if item['text']:
            if len(item['text']) > self.limit:
                item['text'] =item['text'][0:self.limit].rstrip() + '...'
                return item
            else:
                return DropItem
```

这段代码在构造方法里定义了限制长度为50，实现了`process_item`方法，其参数是`item`和`spider`。首先该方法判断`item`的`text`属性是否存在，如果不存在，则抛出`DropItem`异常；如果存在，再判断长度是否大于50，如果大于，那就截断然后拼接省略号，再将`item`返回即可。

将处理后的`item`存入`MongoDB`，定义另外一个`Pipeline`。同样在`pipelines.py`中，实现另一个类`MongoPipeline`，内容如下所示：

```python
import pymongo

    @classmethod
    def from_crawler(cls, crawler):
        return cls(mongo_uri=crawler.settings.get('MONGO_URI'), mongo_db=crawler.settings.get('MONGO_DB'))
        
    def open_spider(self, spider):
        self.client = pymongo.MongoClient(self.mongo_uri)
        self.db = self.client[self.mongo_db]
    
    def process_item(self, item, spider): 
        name = item.__class__.__name__ 
        self.db[name].insert(dict(item)) 
        return item 
    
    def close_spider(self, spider): 
        self.client.close()
```

`MongoPipeline`类实现了`API`定义的另外几个方法。

- `from_crawler`：这是一个类方法，用`@classmethod`标识，是一种依赖注入的方式，方法的参数就是`crawler`，通过`crawler`这个参数我们可以拿到全局配置的每个配置信息，在全局配置`settings.py`中可以定义`MONGO_URI`和`MONGO_DB`来指定`MongoDB`连接需要的地址和数据库名称，拿到配置信息之后返回类对象即可。所以这个方法的定义主要是用来获取`settings.py`中的配置的。
- `open_spider`：当`Spider`被开启时，这个方法被调用。在这里主要进行了一些初始化操作。
- `close_spider`：当`Spider`被关闭时，这个方法会调用，在这里将数据库连接关闭。

最主要的`process_item`方法则执行了数据插入操作。

定义好`TextPipeline`和`MongoPipeline`这两个类后，需要在`settings.py`中使用它们。`MongoDB`的连接信息还需要定义。

```python
ITEM_PIPELINES = {
    'tutorial.pipelines.TextPipeline': 300,
    'tutorial.pipelines.MongoPipeline': 400,
}
MONGO_URI='localhost'
MONGO_DB='tutorial'
```

赋值`ITEM_PIPELINES`字典，键名是`Pipeline`的类名称，键值是调用优先级，是一个数字，数字越小则对应的`Pipeline`越先被调用。
再重新执行爬取，命令如下所示：

```python
scrapy crawl quotes
```

爬取结束后，`MongoDB`中创建了一个`tutorial`的数据库、`QuoteItem`的表。
