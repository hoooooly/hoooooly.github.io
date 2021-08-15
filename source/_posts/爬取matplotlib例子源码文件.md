---
title: 爬取matplotlib例子源码文件
tags:
  - scrapy
comments: true
typora-root-url: 爬取matplotlib例子源码文件
date: 2021-08-14 14:07:07
categories: 爬虫
index_img:
banner_img:
---

# 项目实战：爬取matplotlib例子源码文件

matplotlib是一个非常著名的Python绘图库，广泛应用于科学计算和数据分析等领域。在matplotlib网站上提供了许多应用例子代码，在浏览器中访问[Matplotlib Examples — Matplotlib 2.0.2 documentation](https://matplotlib.org/2.0.2/examples/index.html)，可看到下图页面。

![](image-20210814140923483.png)

其中有几百个例子，被分成多个类别，单击第一个例子，进入其页面，如图所示。

<img src="/image-20210814141011589.png" style="zoom:67%;" />

用户可以在每个例子页面中阅读源码，也可以点击页面中的`source code`按钮下载源码文件。如果我们想把所有例子的源码文件都下载到本地，可以编写一个爬虫程序完成这个任务。

## 项目需求

下载http://matplotlib.org网站中所有例子的源码文件到本地。

## 页面分析

先来看如何在例子列表页面[https://matplotlib.org/2.0.2/examples/index.html](https://matplotlib.org/2.0.2/examples/index.html)中获取所有例子页面的链接。使用`scrapy shell`命令下载页面，然后调用`view`函数在浏览器中查看页面，如图所示。

![](image-20210814141609306.png)

观察发现，所有例子页面的链接都在`<div class="toctree-wrappercompound">`下的每一个`<li class="toctree-l2">`中，例如：

```python
<a class="reference internal" href="animation/index.html">animation Examples</a>
```

使用`LinkExtractor`提取所有例子页面的链接，代码如下：

```python
>>> from scrapy.linkextractors import LinkExtractor
>>> le = LinkExtractor(restrict_css='div.toctree-wrapper.compound li.toctree-l2')
>>> links = le.extract_links(response)
>>> [link.url for link in links]
['https://matplotlib.org/2.0.2/examples/animation/animate_decay.html', 'https://matplotlib.org/2.0.2/examples/animation/basic_example.html', 'https
://matplotlib.org/2.0.2/examples/animation/basic_example_writer.html', 'https://matplotlib.org/2.0.2/examples/animation/bayes_update.html', 'https:
//matplotlib.org/2.0.2/examples/animation/double_pendulum_animated.html', 'https://matplotlib.org/2.0.2/examples/animation/dynamic_image.html', 'ht
tps://matplotlib.org/2.0.2/examples/animation/dynamic_image2.html', 
 ...
```

例子列表页面分析完毕，总共找到了507个例子。

接下来分析例子页面。调用fetch函数下载第一个例子页面，并调用view函数在浏览器中查看页面，如图所示。

```pyt
>>> fetch('https://matplotlib.org/2.0.2/examples/animation/animate_decay.html')
2021-08-14 15:00:33 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://matplotlib.org/2.0.2/examples/animation/animate_decay.html> (referer: No
ne)
>>> view(response)
True
```

![](image-20210814150107493.png)

在一个例子页面中，例子源码文件的下载地址可在`<a class="referenceexternal">`中找到：

```python
>>> href = response.css('a.reference.external::attr(href)').extract_first()
>>> href
'animate_decay.py'
>>> response.urljoin(href)
'https://matplotlib.org/2.0.2/examples/animation/animate_decay.py'
```

到此，页面分析的工作完成了。

### 编码实现

接下来，我们按以下4步完成该项目：

（1）创建Scrapy项目，并使用`scrapy genspider`命令创建`Spider`。

（2）在配置文件中启用FilesPipeline，并指定文件下载目录。

（3）实现`ExampleItem`（可选）。

（4）实现`ExamplesSpider`。

**步骤01** 首先创建`Scrapy`项目，取名为`matplotlib_examples`，再使用`scrapygenspider`命令创建`Spider`：

```python
$ scrapy startproject matplotlib_examples
$ cd matplotlib_examples
$ scrapy genspider examples matplotlib.org
```

**步骤02** 在配置文件`settings.py`中启用`FilesPipeline`，并指定文件下载目录，代码如下：

```python
ITEM_PIPELINES = {
    'scrapy.pipelines.files.FilesPipeline': 1,
}
FILES_STORE = 'examples_src'
```

**步骤03** 实现`ExampleItem`，需定义`file_urls`和`files`两个字段，在`items.py`中完成如下代码：

```python
class ExampleItem(scrapy.Item):
    file_urls = scrapy.Field()
    files = scrapy.Field()
```

**步骤04** 实现`ExamplesSpider`。首先设置起始爬取点：

```python
import scrapy


class ExamplesSpider(scrapy.Spider):
    name = 'examples'
    allowed_domains = ['matplotlib.org']
    start_urls = ['https://matplotlib.org/2.0.2/examples/index.html']

    def parse(self, response):
        pass
```

`parse`方法是例子列表页面的解析函数，在该方法中提取每个例子页面的链接，用其构造`Request`对象并提交，实现parse方法的代码如下：

```python
import scrapy
from scrapy.linkextractors import LinkExtractor


class ExamplesSpider(scrapy.Spider):
    name = 'examples'
    allowed_domains = ['matplotlib.org']
    start_urls = ['https://matplotlib.org/2.0.2/examples/index.html']

    def parse(self, response):
        le = LinkExtractor(restrict_css='div.toctree-wrapper.compound', deny='/index.html')
        print(len(le.extract_links(response)))
        for link in le.extract_links(response):
            yield scrapy.Request(link.url, callback=self.parse_example)

    def parse_example(self, response):
        pass
```

面代码中，我们将例子页面的解析函数设置为`parse_example`方法，下面来实现这个方法。例子页面中包含了例子源码文件的下载链接，在`parse_example`方法中获取源码文件的url，将其放入一个列表，赋给`ExampleItem`的`file_urls`字段。实现`parse_example`方法的代码如下：

 ```python
 import scrapy
 from scrapy.linkextractors import LinkExtractor
 from ..items import ExampleItem
 
 
 class ExamplesSpider(scrapy.Spider):
     name = 'examples'
     allowed_domains = ['matplotlib.org']
     start_urls = ['https://matplotlib.org/2.0.2/examples/index.html']
 
     def parse(self, response):
         le = LinkExtractor(restrict_css='div.toctree-wrapper.compound', deny='/index.html')
         print(len(le.extract_links(response)))
         for link in le.extract_links(response):
             yield scrapy.Request(link.url, callback=self.parse_example)
 
     def parse_example(self, response):
         href = response.css('a.reference.external::attr(href)').extract_first()
         url = response.urljoin(href)
         example = ExampleItem()
         example['file_urls'] = [url]
         return example
 ```

编码完成后，运行爬虫，并观察结果：

```python
scrapy crawl examples -o examples.json
```

运行结束后，在文件examples.json中可以查看到文件下载结果信息：

![](image-20210814160121208.png)

再来查看文件下载目录exmaples_src：

![](image-20210814155737440.png)

507个源码文件被下载到了`examples_src/full`目录下，并且每个文件的名字都是一串长度相等的奇怪数字，这些数字是下载文件url的`sha1`散列值。

这种命名方式可以防止重名的文件相互覆盖，但这样的文件名太不直观了，无法从文件名了解文件内容，我们期望把这些例子文件按照类别下载到不同目录下，为完成这个任务，可以写一个单独的脚本，依据`examples.json`文件中的信息将文件重命名，也可以修改`FilesPipeline`为文件命名的规则，这里采用后一种方式。

阅读`FilesPipeline`的源码发现，原来是其中的`file_path`方法决定了文件的命名，相关代码如下：

```python
def file_path(self, request, response=None, info=None, *, item=None):
    media_guid = hashlib.sha1(to_bytes(request.url)).hexdigest()
    media_ext = os.path.splitext(request.url)[1]
    # Handles empty and wild extensions by trying to guess the
    # mime type then extension or default to empty string otherwise
    if media_ext not in mimetypes.types_map:
        media_ext = ''
        media_type = mimetypes.guess_type(request.url)[0]
        if media_type:
            media_ext = mimetypes.guess_extension(media_type)
    return f'full/{media_guid}{media_ext}'
```

现在，我们实现一个`FilesPipeline`的子类，覆写`file_path`方法来实现所期望的文件命名规则，这些源码文件url`的`最后两部分是类别和文件名，例如：

```python
https://matplotlib.org/2.0.2/examples/animation/(animate_decay.py)
```

可用以上括号中的部分作为文件路径，在`pipelines.py`实现`MyFilesPipeline`，代码如下：

```python
from urllib.parse import urlparse
from os.path import basename, dirname, join

class MyFilePipline(FilesPipeline):
    def file_path(self, request, response=None, info=None):
        path = urlparse(request.url).path
        return join(basename(dirname(path)), basename(path))
```

修改配置文件，使用`MyFilesPipeline`替代`FilesPipeline`：

```python
ITEM_PIPELINES = {
    # 'scrapy.pipelines.files.FilesPipeline': 1,
    'matplotlib_examples.pipelines.MyFilePipline': 1,
}
```

删除之前下载的所有文件，重新运行爬虫后，再来查看`examples_src`目录：

![](image-20210814161729928.png)

从上述结果看出，507个文件按类别被下载到26个目录下，这正是我们所期望的。

到此，文件下载的项目完成了。

[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


