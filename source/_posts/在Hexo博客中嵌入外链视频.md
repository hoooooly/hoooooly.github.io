---
title: 在Hexo博客中嵌入外链视频
date: 2021-05-11 09:12:11
categories: 博客教程
tags:
- Hexo
# sticky: 100
pic:
comments: true
toc: true
only:
- home
- category
- tag
---

# 嵌入的视频

Hexo支持Youtube视频的嵌入，可以参考其实现方式。

首先，在`node_modules/hexo/lib/plugins/tag/index.js`中添加以下代码。

```javascript
  // 添加哔哩哔哩
  tag.register('bilibili', require('./bilibili'));
```

然后在`node_modules/hexo/lib/plugins/tag/`目录下新建`bilibili.js`文件，打开并添加如下代码：

```javascript
'use strict';

const { htmlTag } = require('hexo-util');

/**
* bilibili tag
*
* Syntax:
*   {% bilibili id %}
*/

function bilibiliTag(args, content) {
  var id = args[0];
  return `<div style="position: relative; width: 100%; height: 0; padding-bottom: 75%;"><iframe src="//player.bilibili.com/player.html?${id}" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true" style="position: absolute; width: 100%; height: 100%; left: 0; top: 0;"> </iframe></div>`;
}

module.exports = bilibiliTag;

```

重新启动下`Hexo Server`,在`md`页面中添加下列：

```javascript
{% bilibili aid=586894170&bvid=BV1uz4y1m72t&cid=305401685&page=1 %}
```

重新刷新页面，就可以看到视频正常加载了。

{% bilibili aid=586894170&bvid=BV1uz4y1m72t&cid=305401685&page=1 %}