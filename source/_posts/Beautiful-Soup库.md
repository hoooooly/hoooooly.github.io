---
title: Beautiful Soup4库
tags:
  - bs4
  - python
comments: true
toc: true
only:
  - home
  - category
  - tag
date: 2021-06-05 16:15:33
categories: Python库
pic:
---



`Beautiful Soup` 是一个可以从`HTML`或`XML`文件中提取数据的`Python`库。

## 安装

```python
pip install beautifulsoup4
```

`Beautiful Soup`支持`Python`标准库中的`HTML解析器`,还支持一些第三方的解析器,其中一个是 `lxml` .根据操作系统不同,可以选择下列方法来安装`lxml`:

```python
pip install lxml
```

## 使用

示例HTML；

```html
html_doc = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title"><b>The Dormouse's story</b></p>

<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>

<p class="story">...</p>
"""
```

使用`BeautifulSoup`解析这段代码,能够得到一个 `BeautifulSoup` 的对象,并能按照标准的缩进格式的结构输出:

output:

```
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title"><b>The Dormouse's story</b></p>
<p class="story">Once upon a time there were three little sisters; and their names were
<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a> and
<a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
<p class="story">...</p>
</body></html>
```

几个简单的浏览结构化数据的方法:

```python
soup.title
# <title>The Dormouse's story</title>

soup.title.name
# u'title'

soup.title.string
# u'The Dormouse's story'

soup.title.parent.name
# u'head'

soup.p
# <p class="title"><b>The Dormouse's story</b></p>

soup.p['class']
# u'title'

soup.a
# <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>

soup.find_all('a')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.find(id="link3")
# <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>
```

从文档中找到所有<a>标签的链接:

```python
for link in soup.find_all('a'):
    print(link.get('href'))
    # http://example.com/elsie
    # http://example.com/lacie
    # http://example.com/tillie
```

从文档中获取所有文字内容:

```python
print(soup.get_text())
# The Dormouse's story
#
# The Dormouse's story
#
# Once upon a time there were three little sisters; and their names were
# Elsie,
# Lacie and
# Tillie;
# and they lived at the bottom of a well.
#
# ...
```

## 对象的种类

`Beautiful Soup`将复杂*HTML文档*转换成一个复杂的树形结构,每个节点都是*Python对象*,所有对象可以归纳为4种: `Tag` , `NavigableString` , `BeautifulSoup` , `Comment` .

### Tag

`Tag` 对象与`XML`或`HTML`原生文档中的`tag`相同:

```python
soup = BeautifulSoup('<b class="boldest">hello world</b>', features="lxml")
tag = soup.b

print(tag)
print(type(tag))

# <b class="boldest">hello world</b>
# <class 'bs4.element.Tag'>
```

### Name

每个`tag`都有自己的名字,通过 `.name` 来获取:

```python
tag.name
# u'b'
```

如果改变了`tag`的`name`,那将影响所有通过当前`Beautiful Soup`对象生成的`HTML`文档:

```python
tag.name = "blockquote"
tag
# <blockquote class="boldest">Extremely bold</blockquote>
```

### Attributes

一个`tag`可能有很多个属性`. tag <b class="boldest"> `有一个 `“class”` 的属性,值为 `“boldest” . tag`的属性的操作方法与字典相同:

```python
tag['class']
# u'boldest'
```

也可以直接”点”取属性, 比如: `.attrs` :

```python
tag.attrs
# {u'class': u'boldest'}
```

`tag`的属性可以被添加,删除或修改. 再说一次, `tag`的属性操作方法与字典一样

```python
tag['class'] = 'verybold'
tag['id'] = 1
tag
# <blockquote class="verybold" id="1">Extremely bold</blockquote>

del tag['class']
del tag['id']
tag
# <blockquote>Extremely bold</blockquote>

tag['class']
# KeyError: 'class'
print(tag.get('class'))
# None
```

### 多值属性

还有一些属性 `rel` , `rev` , `accept-charset` , `headers` , `accesskey` `.` 在`Beautiful Soup``中多值属性的返回类型是list`:

```python
css_soup = BeautifulSoup('<p class="body strikeout"></p>')
css_soup.p['class']
# ["body", "strikeout"]

css_soup = BeautifulSoup('<p class="body"></p>')
css_soup.p['class']
# ["body"]
```

如果某个属性看起来好像有多个值,但在任何版本的`HTML`定义中都没有被定义为多值属性,那么`Beautiful Soup`会将这个属性作为字符串返回。

```python
id_soup = BeautifulSoup('<p id="my id"></p>')
id_soup.p['id']
# 'my id'
```

将`tag`转换成字符串时,多值属性会合并为一个值。

```python
rel_soup = BeautifulSoup('<p>Back to the <a rel="index">homepage</a></p>')
rel_soup.a['rel']
# ['index']
rel_soup.a['rel'] = ['index', 'contents']
print(rel_soup.p)
# <p>Back to the <a rel="index contents">homepage</a></p>
```

如果转换的文档是`XML`格式,那么`tag`中不包含多值属性

```python
xml_soup = BeautifulSoup('<p class="body strikeout"></p>', 'xml')
xml_soup.p['class']
# u'body strikeout'
```


[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


