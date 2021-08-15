---
title: Scrapy下载文件和图片
tags:
  - scrapy
comments: true
typora-root-url: Scrapy下载文件和图片
date: 2021-08-14 12:35:30
categories: 爬虫
index_img:
banner_img:
---

# 下载文件和图片

## `FilesPipeline`和`ImagesPipeline`

`Scrapy`框架内部提供了两个`Item Pipeline`，专门用于下载文件和图片：

- `FilesPipeline`
- `ImagesPipeline`

我们可以将这两个`Item Pipeline`看作特殊的下载器，用户使用时只需要通过`item`的一个特殊字段将要下载文件或图片的`url`传递给它们，它们会自动将文件或图片下载到本地，并将下载结果信息存入`item`的另一个特殊字段，以便用户在导出文件中查阅。

### `FilesPipeline`使用说明

通过一个简单的例子讲解`FilesPipeline`的使用，在如下页面中可以下载多本`PDF`格式的小说：

```html
<html>
<body>
    ...
    <a href="/book/sg.pdf">下载《三国演义》</a>
    <a href="/book/shz.pdf">下载《水浒传》</a>
    <a href="/book/hlm.pdf">下载《红楼梦》</a>
    ...
</body>
</html>
```

使用`FilesPipeline`下载页面中所有`PDF`文件，可按以下步骤进行：

**步骤**01 在配置文件`settings.py`中启用`FilesPipeline`，通常将其置于其他`ItemPipeline`之前：

```python
ITEM_PIPELINES = {'scrapy.piplines.files.FilesPipline': 1}
```

**步骤02** 在配置文件`settings.py`中，使用`FILES_STORE`指定文件下载目录，如：

```python
FILES_STORE = '/home/Dowload/Scrapy'
```

`步骤`03 在`Spider`解析一个包含文件下载链接的页面时，将所有需要下载文件的`url`地址收集到一个列表，赋给`item`的`file_urls`字段（`item['file_urls']`）。`FilesPipeline`在处理每一项`item`时，会读取`item['file_urls']`，对其中每一个`url`进行下载，`Spider`示例代码如下：

```python
class DownloadBoosSpider(scrapy.Spider):
    ...

    def parse(self, response):
        item = {}
        # 下载列表
        item['file_urls'] = []
        for url in response.xpath('//a/@href').extract():
            download_url = response.urljoin(url)
            # 将url填入列表
            item['file_urls'].append((download_url))
        yield item
```

当`FilesPipeline`下载完`item['file_urls']`中的所有文件后，会将各文件的下载结果信息收集到另一个列表，赋给`item`的`files`字段（`item['files']`）。下载结果信息包括以下内容：

- `Path`文件下载到本地的路径（相对于`FILES_STORE`的相对路径）。
-  `Checksum`文件的校验和。
-  `url`文件的`url`地址。

### `ImagesPipeline`使用说明

图片也是文件，所以下载图片本质上也是下载文件，`ImagesPipeline`是`FilesPipeline`的子类，使用上和`FilesPipeline`大同小异，只是在所使用的`item`字段和配置选项上略有差别。

|          | FilePipLine                        | ImagePipline                   |
| -------- | ---------------------------------- | ------------------------------ |
| 导入路径 | scrapy.piplines.files.FilesPipline | scrapy.piplines.ImagesPipeline |
| Item字段 | file_urls,files                    | image_urls,umage               |
| 下载目录 | FILES_STORE                        | IMAGES_STORE                   |

`ImagesPipeline`在`FilesPipleline`的基础上针对图片增加了一些特有的功能：

- 为图片生成缩略图

  开启该功能，只需在配置文件`settings.py`中设置`IMAGES_THUMBS`，它是一个字典，每一项的值是缩略图的尺寸，代码如下：

```python
IMAGES_THUMBS = {
    'small': (50, 50),
    'big': (270, 270)
}
```

开启该功能后，下载一张图片时，本地会出现3张图片（1张原图片，2张缩略图）。

- 过滤掉尺寸过小的图片

  开启该功能，需在配置文件`settings.py`中设置`IMAGES_MIN_WIDTH`和`IMAGES_MIN_HEIGHT`，它们分别指定图片最小的宽和高，代码如下：

```python
IMAGES_MIN_WIDTH = 10
IMAGES_MIN_HEIGHT = 10
```

开启该功能后，如果下载了一张105×200的图片，该图片就会被抛弃掉，因为它的宽度不符合标准。



[//]:#(设置表格整体居中显示)

<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


