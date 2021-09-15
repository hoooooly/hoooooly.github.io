---
title: feapder框架上手体验
tags:
  - python
  - 爬虫
  - feapder
comments: true

date: 2021-09-15 14:09:33
categories: 爬虫
index_img: 2021/09/15/feapder框架上手体验/feapder.png
banner_img:
---

## 简介

**feapder** 是一款上手简单，功能强大的Python爬虫框架，使用方式类似scrapy，方便由scrapy框架切换过来，框架内置3种爬虫：

- `AirSpider`爬虫比较轻量，学习成本低。面对一些数据量较少，无需断点续爬，无需分布式采集的需求，可采用此爬虫。
- `Spider`是一款基于redis的分布式爬虫，适用于海量数据采集，支持断点续爬、爬虫报警、数据自动入库等功能
- `BatchSpider`是一款分布式批次爬虫，对于需要周期性采集的数据，优先考虑使用本爬虫。

**feapder**支持**断点续爬**、**数据防丢**、**监控报警**、**浏览器渲染下载**、数据自动入库**Mysql**或**Mongo**，还可通过编写[pipeline](http://boris.org.cn/feapder/#/source_code/pipeline)对接其他存储

- 官方文档：[http://feapder.com](http://feapder.com/)
- 国内文档：https://boris-code.gitee.io/feapder
- github：https://github.com/Boris-code/feapder
- 更新日志：https://github.com/Boris-code/feapder/releases
- 爬虫管理系统：https://boris.org.cn/feapder/#/feapder_platform/爬虫管理系统

## 安装

### 安装要求

- Python 3.6.0+
- Works on Linux, Windows, macOS

### 安装

这里使用pip安装即可。如果要安装完整版`pip3 install feapder[all]`，可能会报错参见[官方文档]([安装问题 - feapder-document (boris.org.cn)](http://boris.org.cn/feapder/#/question/安装问题))。

```python
pip3 install feapder
```

## 使用

创建一个爬虫

```shell
feapder create -s first_spider
```

创建后的爬虫代码如下：

```python
import feapder


class FirstSpider(feapder.AirSpider):
    def start_requests(self):
        yield feapder.Request("https://www.baidu.com")

    def parse(self, request, response):
        print(response)


if __name__ == "__main__":
    FirstSpider().start()
```

直接运行，打印如下：

```shel
C:\Users\q\AppData\Local\Programs\Python\Python39\python.exe D:/PycharmProjects/Spider_learning/feapder_eaxmple/first/first_spider.py
2021-09-15 15:17:03.613 | DEBUG    | feapder.core.parser_control:run:464 - parser 等待任务...
2021-09-15 15:17:04.625 | DEBUG    | feapder.network.request:get_response:305 - 
                -------------- FirstSpider.parse request for ----------------
                url  = https://www.baidu.com
                method = GET
                body = {'timeout': 22, 'stream': True, 'verify': False, 'headers': {'User-Agent': 'Mozilla/5.0 (Windows NT 5.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/31.0.1650.16 Safari/537.36'}}
                
<Response [200]>
2021-09-15 15:17:04.908 | DEBUG    | feapder.core.parser_control:run:464 - parser 等待任务...
2021-09-15 15:17:08.663 | INFO     | feapder.core.spiders.air_spider:run:104 - 无任务，爬虫结束
```

输出日志还是中文的，应该是国人开发的框架。

## 上手

测下官方文档中的列子，手刃[糗事百科 - 超搞笑的原创糗事笑话分享社区 (qiushibaike.com)](https://www.qiushibaike.com/8hr/page/1/)

直接在上面创建的项目中修改start_requests：

```python
class FirstSpider(feapder.AirSpider):
    def start_requests(self):
        for i in range(0, 20):
            yield feapder.Request("https://www.qiushibaike.com/8hr/page/{}/".format(i))
```

再改改parse：

```python
def parse(self, request, response):
    article_list = response.xpath('//a[@class="recmd-content"]')
    for article in article_list:
        title = article.xpath("./text()").extract_first()
        url = article.xpath("./@href").extract_first()
        print(title, url)
```

![](image-20210915153021188.png)

细心的把文章的连接都给补全了。

提取详情页：

在parse中添加回调函数：

```python
yield feapder.Request(
                url, callback=self.parse_detail, title=title
            )  # callback 为回调函数
```

再定义一个parse_detail进行详情页的解析。

> 注意： 这是设置response.encoding_errors = 'ignore'	# 无视编码错误，避免报错

```python
    def parse_detail(self, request, response):
        """
        解析详情
        """
        response.encoding_errors = 'ignore'	# 无视编码错误
        # esponse.code = '网页编码' #  已将encoding简写为code，指定编码格式
        # 取url
        url = request.url
        # 取title
        title = request.title
        # 解析正文
        content = response.xpath('string(//div[@class="content"])').extract_first()  # string 表达式是取某个标签下的文本，包括子标签文本

        print("url", url)
        print("title", title)
        print("content", content)
```

嫌弃跑的太慢? 我们可以加一个参数轻松解决，这是开15个线程。

```python
if __name__ == "__main__":
    SpiderTest(thread_count=15).start()
```

![](image-20210915153728403.png)

不得不说，确实很方便，对于这种没有限制的网站傻瓜式爬取。





[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


