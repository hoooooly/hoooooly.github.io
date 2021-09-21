---
title: 使用scrapy爬取新笔趣阁小说
tags:
  - scrapy
comments: true
typora-root-url: 使用scrapy爬取新笔趣阁小说
date: 2021-09-15 23:33:38
categories:
index_img:
banner_img:
---

## 简单介绍

本次爬取了单本小说[诸神定式](http://www.xbiquge.la/72/72098/)。

![img.png](img.png)

## 实现源码

首先，spider代码，这里命名为biquge.py文件，页面逻辑很简单，不加heades也没什么影响。

```python
import scrapy
from scrapy_learn.items import NovelItem
import time


class BiqugeSpider(scrapy.Spider):
    name = 'biquge'
    allowed_domains = ['www.xbiquge.la/72/72098/']
    start_urls = ['http://www.xbiquge.la/72/72098/']  # 小说页地址，有章节列表的那种

    def start_requests(self):
        for url in self.start_urls:  # 将cookie交给scrapy
            headers = {
                "user-agent": "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.159 Safari/537.36 Edg/92.0.902.84"
            }
            yield scrapy.Request(url=url, headers=headers, meta={"headers": headers}, dont_filter=True,
                                 callback=self.parse)

    def parse(self, response):
        chapter_url_list = response.css('#list dd a::attr(href)').extract()
        novel_item = NovelItem()
        novel_item['novel_title'] = response.css('#info h1::text').extract()    # 小说名

        # for url in chapter_url_list[0:2]: # 调试只获取2章
        for url in chapter_url_list:
            url = 'https://www.xbiquge.la' + url
            time.sleep(1)
            yield scrapy.Request(url=url, meta={"chapter_title_list": chapter_url_list, 'item': novel_item},
                                 dont_filter=True, callback=self.parse_page)

    def parse_page(self, response):
        novel_item = response.meta.get('item')
        novel_item['chapter_name'] = response.css('.bookname h1::text').extract()
        novel_item['chapter_content'] = response.css('#content::text').extract()

        yield novel_item
```

items.py文件：

```python
import scrapy


class ScrapyLearnItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    pass


class NovelItem(scrapy.Item):
    """
    小说名
    章节名
    章节内容
    """
    novel_title = scrapy.Field()
    chapter_name = scrapy.Field()
    chapter_content = scrapy.Field()
```

pipelines.py:

```python
import os
import codecs

class Save2TxtPipeline:
    """保存到txt"""

    def process_item(self, item, spider):
        novel_title = item.get('novel_title')[0]  # 获取小说名创建同名目录
        if os.path.exists(novel_title):
            pass
        else:
            os.mkdir(novel_title)

        chapter_name = item.get('chapter_name')  # 章节名
        chapter_content = item.get('chapter_content')  # 章节内容
        print(chapter_name)
        print(chapter_content)
        chapter_content = map(str.lstrip, chapter_content)   # 去掉列表中头部空白字符
        file = codecs.open(r"{}/{}.txt".format(novel_title, str(chapter_name[0])), 'w',
                           encoding="utf-8")  # 打开文件，增量内容
        file.writelines(chapter_content)  # 写入字符串列表
        file.close()  # 别忘了关闭
        return item
```

这里采用了同步写入文件的写法，速度很慢，不建议这么做，练手的话随便。

最好还是采用异步的方式存到数据库里面，这里就不做过多介绍了。

这个pipeline一定要在settings.py中启用，这里没别的优先级设置为1。

```python
ITEM_PIPELINES = {
   'scrapy_learn.pipelines.Save2TxtPipeline': 1,
}
```

## 运行调试

main.py:

```python
from scrapy.cmdline import execute

import sys
import os

# 添加当前目录到系统路径
sys.path.append(os.path.dirname(os.path.abspath(__file__)))

# 启动scrapy，便于调试
execute(["scrapy", "crawl", "biquge"])

```

这里定义了main入口文件，使用pycharm来执行这个main文件，方便断点Debug。

## 结果

![img_1.png](img_1.png)





[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


