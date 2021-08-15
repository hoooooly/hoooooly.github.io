---
title: Cookie登录
tags:
  - cookie
comments: true
typora-root-url: Cookie登录
date: 2021-08-14 22:49:31
categories: 爬虫
index_img:
banner_img:
---

在使用浏览器登录网站后，包含用户身份信息的`Cookie`会被浏览器保存在本地，如果`Scrapy`爬虫能直接使用浏览器中的`Cookie`发送`HTTP`请求，就可以绕过提交表单登录的步骤。

## 获取浏览器Cookie

使用第三方Python库`browsercookie`可以获取Chrome和Firefox浏览器中的Cookie。

使用pip安装`browsercookie`：

```python
pip install browsercookie
```

`browsercookie`的使用非常简单，示例代码如下：

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


