---
title: XPath库解析
tags:
  - xpath
  - python
  - lxml
comments: true
toc: true
only:
  - home
  - category
  - tag
date: 2021-06-06 14:44:22
categories: Python库
pic:
---



`XPath`，全称 `XML Path Language`，即 `XML` 路径语言，它是一门在*XML文档*中查找信息的语言。`XPath` 最初设计是用来搜寻*XML文档*的，`HTML`是`XML`的子集，它同样适用于*HTML文档*的搜索。

## 安装

```python
pip install lxml
```

## XPath 术语

### 节点（Node）

在 `XPath` 中，有七种类型的节点：*元素*、*属性*、*文本*、*命名空间*、*处理指令*、*注释*以及*文档节点（或称为根节点）*。

请看下面这个 `XML` 文档：

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>

<bookstore>

<book>
  <title lang="en">Harry Potter</title>
  <author>J K. Rowling</author> 
  <year>2005</year>
  <price>29.99</price>
</book>

</bookstore>
```

上面的`XML`文档中的节点例子：

```xml
<bookstore> （文档节点）
<author>J K. Rowling</author> （元素节点）
lang="en" （属性节点） 
```

#### 基本值（或称原子值，Atomic value）

基本值是无父或无子的节点。

基本值的例子：

```xml
J K. Rowling
"en"
```

#### 项目（Item）

项目是基本值或者节点。

### 节点关系

#### 父（Parent）

每个元素以及属性都有一个父。

在下面的例子中，`book` 元素是 `title`、`author`、`year` 以及 `price` 元素的父：

```xml
<book>
  <title>Harry Potter</title>
  <author>J K. Rowling</author>
  <year>2005</year>
  <price>29.99</price>
</book>
```

####　子（Children）

元素节点可有零个、一个或多个子。

在下面的例子中，`title`、`author`、`year` 以及 `price` 元素都是 `book` 元素的子：

```xml
<book>
  <title>Harry Potter</title>
  <author>J K. Rowling</author>
  <year>2005</year>
  <price>29.99</price>
</book>
```

#### 同胞（Sibling）

拥有相同的父的节点

在下面的例子中，`title`、`author`、`year` 以及 `price` 元素都是同胞：

```xml
<book>
  <title>Harry Potter</title>
  <author>J K. Rowling</author>
  <year>2005</year>
  <price>29.99</price>
</book>
```

#### 先辈（Ancestor）

某节点的父、父的父，等等。

在下面的例子中，`title` 元素的先辈是 `book` 元素和 `bookstore` 元素：

```xml
<bookstore>

<book>
  <title>Harry Potter</title>
  <author>J K. Rowling</author>
  <year>2005</year>
  <price>29.99</price>
</book>

</bookstore>
```

#### 后代（Descendant）

某个节点的子，子的子，等等。

在下面的例子中，`bookstore `的后代是 `book`、`title`、`author`、`year` 以及 `price` 元素：

```xml
<bookstore>

<book>
  <title>Harry Potter</title>
  <author>J K. Rowling</author>
  <year>2005</year>
  <price>29.99</price>
</book>

</bookstore>
```

## XPath 语法

目标实列：

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>

<bookstore>

<book category="COOKING">
  <title lang="en">Everyday Italian</title>
  <author>Giada De Laurentiis</author>
  <year>2005</year>
  <price>30.00</price>
</book>

<book category="CHILDREN">
  <title lang="en">Harry Potter</title>
  <author>J K. Rowling</author>
  <year>2005</year>
  <price>29.99</price>
</book>

<book category="WEB">
  <title lang="en">XQuery Kick Start</title>
  <author>James McGovern</author>
  <author>Per Bothner</author>
  <author>Kurt Cagle</author>
  <author>James Linn</author>
  <author>Vaidyanathan Nagarajan</author>
  <year>2003</year>
  <price>49.99</price>
</book>

<book category="WEB">
  <title lang="en">Learning XML</title>
  <author>Erik T. Ray</author>
  <year>2003</year>
  <price>39.95</price>
</book>

</bookstore>
```

### 选取节点

`XPath` 使用路径表达式在 `XML` 文档中选取节点。节点是通过沿着路径或者 `step` 来选取的。

下面列出了最有用的路径表达式：

| 表达式	| 描述 |
| :--- | :---- |
| nodename	| 选取此节点的所有子节点。 |
| /	| 从根节点选取。 |
| //	| 从匹配选择的当前节点选择文档中的节点，而不考虑它们的位置。 |
| .	| 选取当前节点。 |
| ..	| 选取当前节点的父节点。 |
| @	| 选取属性。 |

**实例**

在下面的表格中，列出了一些路径表达式以及表达式的结果：

| 路径表达式	| 结果 |
| :--- | :---- |
| bookstore |	选取 bookstore 元素的所有子节点。 |
| /bookstore | 选取根元素 bookstore。<br>注释：假如路径起始于正斜杠( / )，则此路径始终代表到某元素的绝对路径！ |
| bookstore/book	| 选取属于 bookstore 的子元素的所有 book 元素。 |
| //book	| 选取所有 book 子元素，而不管它们在文档中的位置。 |
| bookstore//book	| 选择属于 bookstore 元素的后代的所有 book 元素，而不管它们位于 bookstore 之下的什么位置。 |
| //@lang	| 选取名为 lang 的所有属性。 |

###　谓语（Predicates）

谓语用来查找某个特定的节点或者包含某个指定的值的节点。

谓语被嵌在方括号中。

**实例**
在下面的表格中，我们列出了带有谓语的一些路径表达式，以及表达式的结果：

| 路径表达式	| 结果 |
| :--- | :---- |
| /bookstore/book[1]	| 选取属于 bookstore 子元素的第一个 book 元素。 |
| /bookstore/book[last()]	| 选取属于 bookstore 子元素的最后一个 book 元素。 |
| /bookstore/book[last()-1]	| 选取属于 bookstore 子元素的倒数第二个 book 元素。 |
| /bookstore/book[position()<3]	| 选取最前面的两个属于 bookstore 元素的子元素的 book 元素。 |
| //title[@lang]	| 选取所有拥有名为 lang 的属性的 title 元素。 |
| //title[@lang='eng']	| 选取所有 title 元素，且这些元素拥有值为 eng 的 lang 属性。 |
| /bookstore/book[price>35.00]	| 选取 bookstore 元素的所有 book 元素，且其中的 price 元素的值须大于 35.00。 |
| /bookstore/book[price>35.00]/title	| 选取 bookstore 元素中的 book 元素的所有 title 元素，且其中的 price 元素的值须大于 35.00。 |

### 选取未知节点

`XPath` 通配符可用来选取未知的 `XML` 元素。

| 通配符	| 描述 |
| :--- | :---- |
| *	| 匹配任何元素节点。 |
| @*	| 匹配任何属性节点。 |
| node()	| 匹配任何类型的节点。 |

**实例**

在下面的表格中，列出了一些路径表达式，以及这些表达式的结果：

| 路径表达式	| 结果 |
| :--- | :---- |
| /bookstore/*	| 选取 bookstore 元素的所有子元素。 |
| //*	| 选取文档中的所有元素。 |
| //title[@*] | 选取所有带有属性的 title 元素。 |

### 选取若干路径

通过在路径表达式中使用“|”运算符，您可以选取若干个路径。

**实例**
在下面的表格中，我们列出了一些路径表达式，以及这些表达式的结果：

| 路径表达式	| 结果 |
| :----- | :---- |
| //book/title \| //book/price	|选取 book 元素的所有 title 和 price 元素。 |
| //title \| //price	| 选取文档中的所有 title 和 price 元素。 |
| /bookstore/book/title \| //price	| 选取属于 bookstore 元素的 book 元素的所有 title 元素，以及文档中所有的 price 元素。 |

## XPath 实例

对`xml`文档的匹配。

```python
# coding: utf-8
from lxml import etree

tree = etree.XML(xml,)
result = tree.xpath("/bookstore/book/title/text()")
# ['Everyday Italian', 'Harry Potter', 'XQuery Kick Start', 'Learning XML']

result = tree.xpath("//book[3]/author[2]/text()")
# ['Per Bothner']
```

对`html`文档的匹配。

```html
<div>
    <ul>
        <li class="item-0"><a href="link1.html">first item</a></li>
        <li class="item-1"><a href="link2.html">second item</a></li>
        <li class="item-inactive"><a href="link3.html">third item</a></li>
        <li class="item-1"><a href="link4.html">fourth item</a></li>
        <li class="item-0"><a href="link5.html">fifth item</a>
    </ul>
</div>
```

引入`xlxml`模块， 从文件中导入`html`。

```python
from lxml import etree

tree = etree.parse('./test.html', etree.HTMLParser())
```

### 通过字符串转换成html格式

利用`etree.HTML`解析字符串，将字符串解析从`html`格式的文件， 经过处理后，部分缺失的节点可以自动修复，并且还自动添加了 `body`、`html` 节点。

```python
from lxml import etree

text = """
<html>
    <body>
        <div>
            <ul>
                <li class="item-0"><a href="link1.html">first item</a></li>
                <li class="item-1"><a href="link2.html">second item</a></li>
                <li class="item-inactive"><a href="link3.html">third item</a></li>
                <li class="item-1"><a href="link4.html">fourth item</a></li>
                <li class="item-0"><a href="link5.html">fifth item</a></li>
            </ul>
        </div>
    </body>
</html>
"""

html = etree.HTML(text)
s = etree.tostring(html).decode()
print(s)

# <html>
#     <body>
#         <div>
#             <ul>
#                 <li class="item-0"><a href="link1.html">first item</a></li>
#                 <li class="item-1"><a href="link2.html">second item</a></li>
#                 <li class="item-inactive"><a href="link3.html">third item</a></li>
#                 <li class="item-1"><a href="link4.html">fourth item</a></li>
#                 <li class="item-0"><a href="link5.html">fifth item</a></li>
#             </ul>
#         </div>
#     </body>
# </html>
```

### 所有节点

用以 `//` 开头的 `XPath` 规则来选取所有符合要求的字符串：

```python
result = html.xpath("//text()")
# ['\n    ', '\n        ', '\n            ', '\n                ', 'first item', '\n                ', 'second item', '\n                ', 'third item', '\n
#       ', 'fourth item', '\n                ', 'fifth item', '\n            ', '\n        ', '\n    ', '\n']
```

### 绝对路径查找

- 获取某个标签的内容

注意，获取`a`标签的所有内容，`a`后面就不用再加正斜杠，否则报错。

```python
result = html.xpath("/html/body/div/ul/li/a")
for i in result:
    print(i.text)

# first item
# second item
# third item
# fourth item
# fifth item
```

或者

```python
result = html.xpath("/html/body/div/ul/li/a/text()")
for i in result:
    print(i)

first item
second item
third item
fourth item
fifth item
```

- 打印指定路径下a标签的属性

这里可以通过遍历拿到某个属性的值，查找标签的内容，通过`@属性名`获取。

```python
result = html.xpath("/html/body/div/ul/li/a/@href")
for i in result:
    print(i)

# link1.html
# link2.html
# link3.html
# link4.html
# link5.html
```

- 获取指定标签对应属性值的内容

使用`xpath`拿到得都是一个个的`ElementTree`对象，如果需要查找内容的话，还需要遍历拿到数据的列表。

```python
result = html.xpath('/html/body/div/ul/li/a[@href="link3.html"]/text()')
for i in result:
    print(i)

# third item
```

## 相对路径查找（常用）

- 查找所有li标签下的a标签内容

```python
result = html.xpath('//li/a/text()')
print(result)
# ['first item', 'second item', 'third item', 'fourth item', 'fifth item']
```

- 查找一下l相对路径下li标签下的a标签下的href属性的值

注意，`a`标签后面需要双`//`。

```python
result = html.xpath('//li/a/@href')
print(result)
```

- 查找`a`标签下属性`href`值为`link5.html`的内容。

```python
result = html.xpath('//li/a/@href')
print(result)
```

- 查找`a`标签下属性`href`值为""的内容。

```python
result = html.xpath('//li/a[@href="link5.html"]/text()')
print(result)

# ['fifth item']
```

## 子节点

通过 `/` 或 `//` 即可查找元素的子节点或子孙节点，选择 `li` 节点的所有直接 `a` 子节点`xpath`为：`//li/a`。

## 父节点

知道子节点，查询父节点可以用 `..` 来实现。

```python
result = html.xpath('//li/a[@href="link5.html"]/../@class')
print(result)
# ['item-0']
```

## 属性匹配

匹配时可以用`@`符号进行属性过滤，匹配`li`下属性`class`为`item-5`的内容。

```python
result = html.xpath('//li/a[@href="link5.html"]/../@class')
print(result)
# ['item-0']
```

## 文本获取

有两种方法：一是获取文本所在节点后直接获取文本，二是使用 `//`。第二种方法会获取到补全代码时换行产生的特殊字符，推荐使用第一种方法，可以保证获取的结果是整洁的。

```python
# first
result = html.xpath('//li[@class="item-0"]/a/text()')
print(result)
# ['first item', 'fifth item']

# second
result = html.xpath('//li[@class="item-0"]//text()')
print(result)
# ['first item', 'fifth item']
```

## 属性获取

`@`符号相当于过滤器，可以直接获取节点的属性值。

```python
result = html.xpath('//li/a/@href')
print(result)
# ['link1.html', 'link2.html', 'link3.html', 'link4.html', 'link5.html']
```

## 属性多值匹配

某些节点的某个属性可能有多个值：

```python
from lxml import etree

text = '''
<li class="zxc  asd  wer"><a href="https://s2.bdstatic.com/">1 item</a></li>
<li class="ddd  asd  eee"><a href="https://s3.bdstatic.com/">2 item</a></li>
'''

html = etree.HTML(text)
result = html.xpath('//li[contains(@class, "asd")]/a/text()')
print(result)

# ['1 item', '2 item']
```

## 多属性匹配

```python
from lxml import etree

text = '''
<li class="zxc  asd  wer" name="222"><a href="https://s2.bdstatic.com/">1 item</a></li>
<li class="ddd  zxc  eee" name="111"><a href="https://s3.bdstatic.com/">2 item</a></li>
'''
html = etree.HTML(text)
result = html.xpath('//li[contains(@class, "zxc") and @name="111"]/a/text()')
print(result)

# 运行结果：['2 item']
```

## 函数

查找最后一个`li`标签里的`a`标签的`href`属性（`last()函数`）

```python
from lxml import etree

text = '''
<li class="zxc  asd  wer"><a href="https://s2.bdstatic.com/">1 item</a></li>
<li class="ddd  asd  eee"><a href="https://s3.bdstatic.com/">2 item</a></li>
'''
html = etree.HTML(text)
result = html.xpath('//li[last()]/a/text()')
print(result)
# ['2 item']
```


[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>