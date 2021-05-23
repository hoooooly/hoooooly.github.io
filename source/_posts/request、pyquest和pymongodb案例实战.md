---
title: request、pyquest和pymongodb案例实战
tags:
  - request
  - pyquest
  - pymongodb
comments: true
toc: true
only:
  - home
  - category
  - tag
date: 2021-05-23 21:12:36
categories: 爬虫
pic:
---

## 准备工作

在本节课开始之前，我们需要做好如下的准备工作：

- 安装好`Python3`（最低为 3.6 版本），并能成功运行`Python3`程序。
- 了解`Python`多进程的基本原理。
- 了解`PythonHTTP`请求库`requests`的基本用法。
- 了解正则表达式的用法和`Python`中正则表达式库`re`的基本用法。
- 了解`PythonHTML`解析库`pyquery`的基本用法。
- 了解`MongoDB`并安装和启动`MongoDB`服务。
- 了解`Python`的`MongoDB`操作库`PyMongo`的基本用法。

## 爬虫目标

一个基本的静态网站作为案例进行爬取，需要爬取的链接为：[https://static1.scrape.cuiqingcai.com/](https://static1.scrape.cuiqingcai.com/)，这个网站里面包含了一些电影信息。

要完成的目标是：

- 用`requests`爬取这个站点每一页的电影列表，顺着列表再爬取每个电影的详情页。
- 用`pyquery`和正则表达式提取每部电影的名称、封面、类别、上映时间、评分、剧情简介等内容。
- 把以上爬取的内容存入`MongoDB`数据库。
- 使用多进程实现爬取的加速。

### 爬取列表页

爬取的第一步肯定要从列表页入手，首先观察一下列表页的结构和翻页规则。在浏览器中访问[](https://static1.scrape.cuiqingcai.com/)，然后打开浏览器开发者工具，观察每一个电影信息区块对应的`HTML`，以及进入到详情页的`URL`是怎样的，如图所示：

![](Screenshot_1.webp)

每部电影对应的区块都是一个`div`节点，它的`class`属性都有`el-card`这个值。每个列表页有10个这样的`div`节点，也就对应着10部电影的信息。

再分析下从列表页是怎么进入到详情页的，选中电影的名称，看下结果：

![](Screenshot_2.webp)

这个名称实际上是一个`h2`节点，其内部的文字就是电影的标题。`h2`节点的外面包含了一个`a`节点，这个`a`节点带有`href`属性，这就是一个超链接，其中`href`的值为`/detail/1`，这是一个相对网站的根`URL`[https://static1.scrape.cuiqingcai.com/](https://static1.scrape.cuiqingcai.com/)路径，加上网站的根`URL`就构成了[https://static1.scrape.cuiqingcai.com/detail/1](https://static1.scrape.cuiqingcai.com/detail/1)，也就是这部电影详情页的`URL`。这样只需要提取这个`href`属性就能构造出详情页的`URL`并接着爬取了。

接下来分析下翻页的逻辑，拉到页面的最下方，可以看到分页页码，如图所示：

![](Screenshot_3.webp)

页面显示一共有`100`条数据，10页的内容，因此页码最多是10。接着我们点击第2页，如图所示：

![](Screenshot_4.webp)

可以看到网页的`URL`变成了`https://static1.scrape.cuiqingcai.com/page/2`，相比根`URL`多了`/page/2`这部分内容。网页的结构还是和原来一模一样，所以我们可以和第1页一样处理。

接着查看第3页、第4页等内容，可以发现有这么一个规律，每一页的`URL`最后分别变成了`/page/3`、`/page/4`。所以，`/page`后面跟的就是列表页的页码，当然第1页也是一样，在根`URL`后面加上`/page/1`也是能访问的，只不过网站做了一下处理，默认的页码是1，所以显示第1页的内容。

分析到这里，逻辑基本就清晰了。

如果要完成列表页的爬取，可以这么实现：

- 遍历页码构造`10`页的索引页`URL`。
- 从每个索引页分析提取出每个电影的详情页`URL`。

先定义一些基础的变量，并引入一些必要的库，写法如下：

```python
import requests
import logging
import re
import pymongo
from pyquery import PyQuery as pq
from urllib.parse import urljoin

logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s: %(message)s')

BASE_URL = 'https://static1.scrape.cuiqingcai.com'
TOTAL_PAGE = 10
```

引入`requests`用来爬取页面，`logging`用来输出信息，`re`用来实现正则表达式解析，`pyquery`用来直接解析网页，`pymongo`用来实现`MongoDB`存储，`urljoin`用来做`URL`的拼接。

接着定义日志输出级别和输出格式，完成之后再定义`BASE_URL`为当前站点的根`URL`，`TOTAL_PAGE`为需要爬取的总页码数量。

定义好了之后，来实现一个页面爬取的方法，实现如下：

```python
def scrape_page(url):
    logging.info('scraping %s...', url)
    try:
        response = requests.get(url, verify=False)
        if response.status_code == 200:
            return response.text
        logging.error('get invalid status code %s while scraping %s', response.status_code, url)
    except requests.RequestException:
        logging.error('error occurred while scraping %s', url, exc_info=True)
```

考虑到不仅要爬取列表页，还要爬取详情页，在这里定义一个较通用的爬取页面的方法，叫作`scrape_page`，它接收一个`url`参数，返回页面的`html`代码。

首先判断状态码是不是`200`，如果是，则直接返回页面的`HTML`代码，如果不是，则会输出错误日志信息。另外，这里实现了`requests`的异常处理，如果出现了爬取异常，则会输出对应的错误日志信息。这时将`logging`的`error`方法的`exc_info`参数设置为`True`则可以打印出`Traceback`错误堆栈信息。 

有了`scrape_page`方法之后，给这个方法传入一个`url`，正常情况下它就可以返回页面的`HTML`代码。

在这个基础上，来定义列表页的爬取方法吧，实现如下：

```python
def scrape_index(page):
    index_url = f'{BASE_URL}/page/{page}'
    return scrape_page(index_url)
```

方法名称叫作`scrape_index`，这个方法会接收一个`page`参数，即列表页的页码，在方法里面实现列表页的`URL`拼接，然后调用`scrape_page`方法爬取即可得到列表页的`HTML`代码了。

获取了`HTML`代码后，下一步就是解析列表页，并得到每部电影的详情页的`URL`了，实现如下：

```python
def parse_index(html):
    doc = pq(html)
    links = doc('.el-card .name')
    for link in links.items():
        href = link.attr('href')
        detail_url = urljoin(BASE_URL, href)
        logging.info('get detail info %s', detail_url)
        yield detail_url
```

这里我们定义了`parse_index`方法，它接收一个`html`参数，即列表页的`HTML`代码。接着用`pyquery`新建一个`PyQuery`对象，完成之后再用`.el-card .name`选择器选出来每个电影名称对应的超链接节点。遍历这些节点，通过调用`attr`方法并传入`href`获得详情页的`URL`路径，得到的`href`就是上文所说的类似`/detail/1`这样的结果。这并不是一个完整的`URL`，所以需要借助`urljoin`方法把`BASE_URL`和`href`拼接起来，获得详情页的完整`URL`，得到的结果就是类似`https://static1.scrape.cuiqingcai.com/detail/1`这样完整的`URL`了，最后`yield`返回即可。

通过调用`parse_index`方法传入列表页的`HTML`代码就可以获得该列表页所有电影的详情页`URL`了，接下来把上面的方法串联调用一下，实现如下：

```python
def main():
    for page in range(1, TOTAL_PAGE+1):
        index_html = scrape_index(page)
        detail_urls = parse_index((index_html))
        logging.info('detail urls %s', list(detail_urls))

if __name__ == '__main__':
    main()
```

定义了`main`方法来完成上面所有方法的调用，首先使用`range`方法遍历一下页码，得到的`page`是`1~10`，接着把`page`变量传给`scrape_index`方法，得到列表页的`HTML`，赋值为`index_html`变量。接下来再将`index_html`变量传给`parse_index`方法，得到列表页所有电影的详情页`URL`，赋值为`detail_urls`，结果是一个生成器，调用`list`方法就可以将其输出出来。

```python
2021-05-23 23:23:04,059 - INFO: get detail info https://static1.scrape.cuiqingcai.com/detail/1
2021-05-23 23:23:04,060 - INFO: get detail info https://static1.scrape.cuiqingcai.com/detail/2
2021-05-23 23:23:04,061 - INFO: get detail info https://static1.scrape.cuiqingcai.com/detail/3
2021-05-23 23:23:04,062 - INFO: get detail info https://static1.scrape.cuiqingcai.com/detail/4
2021-05-23 23:23:04,063 - INFO: get detail info https://static1.scrape.cuiqingcai.com/detail/5
2021-05-23 23:23:04,064 - INFO: get detail info https://static1.scrape.cuiqingcai.com/detail/6
2021-05-23 23:23:04,065 - INFO: get detail info https://static1.scrape.cuiqingcai.com/detail/7
2021-05-23 23:23:04,067 - INFO: get detail info https://static1.scrape.cuiqingcai.com/detail/8
2021-05-23 23:23:04,070 - INFO: get detail info https://static1.scrape.cuiqingcai.com/detail/9
2021-05-23 23:23:04,072 - INFO: get detail info https://static1.scrape.cuiqingcai.com/detail/10
2021-05-23 23:23:04,074 - INFO: detail urls ['https://static1.scrape.cuiqingcai.com/detail/1', 'https://static1.scrape.cuiqingcai.com/detail/2', 'https://static1.scrape.cuiqingcai.com/detail/3', 'https://static1.scrape.cuiqingcai.com/detail/4', 'https://static1.scrape.cuiqingcai.com/detail/5', 'https://static1.scrape.cuiqingcai.com/detail/6', 'https://static1.scrape.cuiqingcai.com/detail/7', 'https://static1.scrape.cuiqingcai.com/detail/8', 'https://static1.scrape.cuiqingcai.com/detail/9', 'https://static1.scrape.cuiqingcai.com/detail/10']
2021-05-23 23:23:04,076 - INFO: scraping https://static1.scrape.cuiqingcai.com/page/2...
....
```

由于输出内容比较多，这里只贴了一部分。可以看到，在这个过程中程序首先爬取了第`1`页列表页，然后得到了对应详情页的每个`URL`，接着再接着爬第2页、第3页，一直到第10页，依次输出了每一页的详情页`URL`。这样，就成功获取到所有电影详情页`URL`。

### 爬取详情页

首先观察一下详情页的`HTML`代码，如图所示：

![](Screenshot_5.webp)

经过分析，要提取的内容和对应的节点信息如下：

- 封面：是一个`img`节点，其`class`属性为`cover`。
- 名称：是一个`h2`节点，其内容便是名称。
- 类别：是`span`节点，其内容便是类别
- 内容，其外侧是`button`节点，再外侧则是`class`为`categories`的`div`节点。
- 上映时间：是`span`节点，其内容包含了上映时间，其外侧是包含了`class`为`info`的`div`节点。但注意这个`div`前面还有一个`class`为`info`的`div`节点，可以使用其内容来区分，也可以使用`nth-child`或`nth- of-type`这样的选择器来区分。另外提取结果中还多了「上映」二字，可以用正则表达式把日期提取出来。
- 评分：是一个`p`节点，其内容便是评分，`p`节点的`class`属性为`score`。
- 剧情简介：是一个`p`节点，其内容便是
- 剧情简介，其外侧是`class`为`drama`的`div`节点。

刚才已经成功获取了详情页的`URL`，接下来要定义一个详情页的爬取方法，实现如下：

```python
def scrape_detail(url): 
    return scrape_page(url)
```

未完待续。。。20210523