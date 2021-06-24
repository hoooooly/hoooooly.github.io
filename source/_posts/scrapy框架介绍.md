---
title: scrapy框架介绍
tags:
  - scrapy
  - python
comments: true
toc: true
only:
  - home
  - category
  - tag
date: 2021-05-29 18:13:04
categories: 爬虫
pic:
---

## Scrapy介绍

`Scrapy`是一个基于`Twisted`的异步处理框架，是纯`Python`实现的爬虫框架，其架构清晰，模块之间的耦合程度低，可扩展性极强，可以灵活完成各种需求。只需要定制开发几个模块就可以轻松实现一个爬虫。

`Scrapy`框架主要实现下面的流程：

```mermaid
graph RL
    A[内容提取]--url--->B[url队列]
    B--url--->C[发送请求]
    C--respone--->A
    D[提取URL]--->A
    E[提取数据]--->A
    A--数据--->F[数据队列]
    F--->G[保存数据]
```

来看下`Scrapy`框架的架构，如图所示：

![](Screenshot_1.webp)

它可以分为如下的几个部分。

- `Engine（引擎）`：用来处理整个系统的数据流处理、触发事务，是整个框架的核心。
- `Item（项目）`：定义了爬取结果的数据结构，爬取的数据会被赋值成该对象。
- `Scheduler（调度器）`：用来接受引擎发过来的请求并加入队列中，并在引擎再次请求的时候提供给引擎。
- `Downloader（下载器）`：用于下载网页内容，并将网页内容返回给蜘蛛。
- `Spiders（蜘蛛）`：其内定义了爬取的逻辑和网页的解析规则，它主要负责解析响应并生成提取结果和新的请求。
- `ItemPipeline（项目管道）`：负责处理由蜘蛛从网页中抽取的项目，它的主要任务是清洗、验证和存储数据。
- `Downloader Middlewares（下载器中间件）`：位于引擎和下载器之间的钩子框架，主要是处理引擎与下载器之间的请求及响应。
- `Spider Middlewares（蜘蛛中间件）`：位于引擎和蜘蛛之间的钩子框架，主要工作是处理蜘蛛输入的响应和输出的结果及新的请求。

## 数据流

了解了架构，就是要了解它是怎么进行数据爬取和处理的，所以接下来介绍`Scrapy`的数据流机制。

`Scrapy`中的数据流由引擎控制，其过程如下：

- `Engine`首先打开一个网站，找到处理该网站的`Spider`并向该`Spider`请求第一个要爬虫的`URL`。
- `Engine`从`Spider`中获取到第一个要爬取的`URL`并通过`Scheduler`以`Request`的形式调度。
- `Engine`向`Scheduler`请求下一个要爬取的`URL`。
- `Scheduler`返回下一个要爬取的`URL`给`Engine`，`Engine`将`URL`通过`Downloader Middlewares`转发给`Downloader`下载。
- 一旦页面下载完毕， `Downloader`生成一个该页面的`Response`，并将其通过`Downloader Middlewares`发送给`Engine`。
- `Engine`从下载器中接收到`Response`并通过`Spider Middlewares`发送给`Spider`处理。
- `Spider`处理`Response`并返回爬取到的`Item`及新的`Request`给`Engine`。
- `Engine`将`Spider`返回的`Item`给`ItemPipeline`，将新的`Request`给`Scheduler`。
- 重复第二步到最后一步，直到`Scheduler`中没有更多的`Request`，`Engine`关闭该网站，爬取结束。

通过多个组件的相互协作、不同组件完成工作的不同、组件对异步处理的支持，`Scrapy`最大限度地利用了网络带宽，大大提高了 数据爬取和处理的效率。


## 安装

使用官方文档推荐的方式安装就可以了：

```python
pip install scrapy
```

## 项目结构

安装完成后，就可以使用`Scrapy`预先配置好的很多可用的组件和编写爬虫时所用的脚手架。

创建项目的命令如下：

```python
scrapy startproject demo
```

执行完成之后，在当前运行目录下便会出现一个文件夹，叫作`demo`，这就是一个`Scrapy`项目框架，可以基于这个项目框架来编写爬虫。

项目文件结构如下所示：

```python
scrapy.cfg 
project/ 
  __init__.py
  items.py 
  pipelines.py 
  settings.py 
  middlewares.py 
  spiders/
    __init__.py
    spider1.py
    spider2.py
    ...
```

在此要将各个文件的功能描述如下：

- `scrapy.cfg`：它是`Scrapy`项目的配置文件，其内定义了项目的配置文件路径、部署相关信息等内容。
- `items.py`：它定义`Item`数据结构，所有的`Item`的定义都可以放这里。
- `pipelines.py`：它定义`ItemPipeline`的实现，所有的`ItemPipeline`的实现都可以放这里。
- `settings.py`：它定义项目的全局配置。
- `middlewares.py`：它定义`Spider Middlewares`和`Downloader Middlewares`的实现。
- `spiders`：其内包含一个个`Spider`的实现，每个`Spider`都有一个文件。

