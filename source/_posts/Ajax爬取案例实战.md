---
title: Ajax爬取案例实战
tags:
  - ajax
  - python
comments: true
toc: true
only:
  - home
  - category
  - tag
date: 2021-05-24 20:13:23
categories: 爬虫
pic:
---

## 准备工作

- 安装好`Python 3`（最低为 3.6 版本），并能成功运行`Python 3`程序。
- 了解`PythonHTTP`请求库`requests`的基本用法。
- 了解`Ajax`的基础知识和分析`Ajax`的基本方法。

## 爬取目标

其链接为：[https://dynamic1.scrape.cuiqingcai.com/](https://dynamic1.scrape.cuiqingcai.com/)，页面如图所示。

![](Screenshot_1.webp)

需要完成的目标有：

- 分析页面数据的加载逻辑。
- 用`requests`实现`Ajax`数据的爬取。
- 将每部电影的数据保存成一个`JSON`数据文件。

## 爬取列表页

打开浏览器开发者工具，切换到`Network`面板，勾选上`「Preserve Log」`并切换到`「XHR」`选项卡，如图所示。

![](Screenshot_2.webp)

点开第`2`个结果，观察到其`Ajax`接口请求的`URL`地址为：`https://dynamic1.scrape.cuiqingcai.com/api/movie/?limit=10&offset=10`，这里有两个参数，一个是`limit`，其值为`10`，一个是`offset`，它的值也是`10`。

切换到`Preview`选项卡，结果如图所示。

![](Screenshot_3.webp)

可以看到结果是一些`JSON`数据，它有一个`results`字段，这是一个列表，列表的每一个元素都是一个字典。观察一下字典的内容，可以看到对应的电影数据的字段了，如`name`、`alias`、`cover`、 `categories`，对比下浏览器中的真实数据，各个内容是完全一致的，而且这个数据已经非常结构化了，完全就是我们想要爬取的数据。

导入一些所需的库并定义一些配置，代码如下：

```python
import requests
import logging

logging.basicConfig(level=logging.INFO,
                    format='%(asctime)s-%(levelname)s:%(message)s')

INDEX_URL = 'https://dynamic1.scrape.cuiqingcai.com/api/movie/?limit={limit}&offset={offset}'
```

引入了`requests`和`logging`库，并定义了`logging`的基本配置，接着定义`INDEX_URL`，这里把`limit`和`offset`预留出来变成占位符，可以动态传入参数构造成一个完整的列表页`URL`。

下面来实现一下列表页的爬取，先定义一个通用的爬取方法，代码如下：

```python
def scrape_api(url):
    logging.info('scraping %s...', url)
    try:
        response = requests.get(url, verify=False)
        if response.status_code == 200:
            return response.json()
        logging.error('get invalid status code %s while scraping %s',
                      response.status_code, url)
    except requests.RequestException:
        logging.error('error occurred while scraping %s', url, exc_info=True)
```

定义一个`scrape_api`方法，和之前不同的是，这个方法专门用来处理`JSON`接口，最后的`response`调用的是`json`方法，它可以解析响应的内容并将其转化成`JSON`字符串。

在这个基础之上，定义一个爬取列表页的方法，代码如下：

```python
LIMIT = 10

def scrape_index(page):
    url = INDEX_URL.format(limit=LIMIT, offset=LIMIT*(page-1))
    return scrape_api(url)
```

定义了一个`scrape_index`方法，用来接收参数`page`，`page`代表列表页的页码。

先构造了一个`URL`，通过字符串的`format`方法，传入`limit`和`offset`的值。这里的`limit`直接使用了全局变量`LIMIT`的值，`offset`则是动态计算的，计算方法是页码数减1再乘以`limit`，比如第1页的`offset`值就是`0`，第2页的`offset`值就是`10`，以此类推。构造好`URL`之后，直接调用`scrape_api`方法并返回结果即可。

这样就完成了列表页的爬取，每次请求都会得到一页`10`部的电影数据。

由于这时爬取到的数据已经是`JSON`类型了，所以不用像之前一样去解析`HTML`代码来提取数据，爬到的数据就是想要的结构化数据，因此解析这一步这里就可以直接省略。

## 爬取详情页

这时候已经可以拿到每一页的电影数据了，但是实际上这些数据还缺少一些想要的信息，如剧情简介等，所以需要进一步进入到详情页来获取这些内容。

这时候点击任意一部电影，如《教父》，进入到其详情页面，这时候可以发现页面的`URL`已经变成了[](https://dynamic1.scrape.cuiqingcai.com/detail/40)，页面也成功展示了详情页的信息

先定义一个详情页的爬取逻辑吧，代码如下：

```python
DETAIL_URL = 'https://dynamic1.scrape.cuiqingcai.com/api/movie/{id}'

def scrape_detail(id):
    url = DETAIL_URL.format(id=id)
    return scrape_api(url)
```

定义了一个`scrape_detail`方法，它接收参数`id`。这里的实现也非常简单，先根据定义好的`DETAIL_UR`L加上`id`，构造一个真实的详情页`Ajax`请求的`URL`，然后直接调用`scrape_api`方法传入这个`URL`即可。接着，定义一个总的调用方法，将以上的方法串联调用起来，代码如下：

```python
def main():
    for page in range(1, TOTAL_PAGE+1):
        index_data = scrape_index(page)
        for item in index_data.get('results'):
            id = item.get('id')
            print(id)
            detail_data = scrape_detail(id)
            logging.info('detail data %s', detail_data)
```

定义了一个`main`方法，首先遍历获取页码`page`，然后把`page`当成参数传递给`scrape_index`方法，得到列表页的数据。接着我遍历所有列表页的结果，获取每部电影的`id`，然后把`id`当作参数传递给`scrape_detail`方法，来爬取每部电影的详情数据，赋值为`detail_data`，输出即可。

## 保存数据

定义一个数据保存的方法，代码如下：

```python
import json
from os import makedirs
from os.path import exists

RESULT_DIR = 'results'

exists(RESULT_DIR) or makedirs(RESULT_DIR)

def save_data(data):
    name = data.get('name')
    data_path = f'{RESULT_DIR}/{name}.json'
    json.dump(data, open(data_path, 'w', encoding='utf-8'), ensure_ascii=False, indent=2)
```

首先定义了数据保存的文件夹`RESULTS_DIR`，注意，先要判断这个文件夹是否存在，如果不存在则需要创建。

接着，定义了保存数据的方法`save_data`，首先获取数据的`name`字段，即电影的名称，把电影名称作为`JSON`文件的名称，接着构造`JSON`文件的路径，然后用`json`的`dump`方法将数据保存成文本格式。`dump`的方法设置了两个参数，一个是`ensure_ascii`，将其设置为`False`，它可以保证中文字符在文件中能以正常的中文文本呈现，而不是`unicode`字符；另一个是`indent`，它的数值为2，这代表生成的`JSON`数据结果有两个空格缩进，让它的格式显得更加美观。

最后，`main`方法再调用下`save_data`方法即可，实现如下：

```python
def main():
    for page in range(1, TOTAL_PAGE+1):
        index_data = scrape_index(page)
        for item in index_data.get('results'):
            id = item.get('id')
            print(id)
            detail_data = scrape_detail(id)
            logging.info('detail data %s', detail_data)
            save_data(detail_data)
```

本地`results`文件夹下出现了各个电影的`JSON`文件，如图所示。

![](Screenshot_4.webp)
