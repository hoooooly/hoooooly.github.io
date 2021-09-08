---
title: 项目实战：Scrapy爬取京东商城书籍信息
tags:
  - scrapy
  - splash
comments: true
typora-root-url: 项目实战：Scrapy爬取京东商城书籍信息
date: 2021-08-17 09:35:24
categories: 爬虫
index_img:
banner_img:
---

# 项目实战：爬取京东商城中的书籍信息

## 项目需求

爬取京东商城中所有Python书籍的名字和价格信息。

## 页面分析

在京东网站（http://www.jd.com）的书籍分类下搜索Python关键字得到的页面。

![](image-20210817093807431.png)

结果有很多页，在每一个书籍列表页面中可以数出有60本书，但我们在`scrapyshell`中爬取该页面时遇到了问题，仅在页面中找到了30本书，少了30本，代码如下：

```python
$ scrapy shell
```











[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


