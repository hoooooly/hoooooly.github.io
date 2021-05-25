---
title: Ajax的原理和解析
tags:
  - ajax
  - javascript
comments: true
toc: true
only:
  - home
  - category
  - tag
date: 2021-05-24 16:33:38
categories: 爬虫
pic:
---

## 什么是Ajax？

`Ajax`，全称为`Asynchronous JavaScriptand XML`，即异步的`JavaScript`和`XML`。它不是一门编程语言，而是利用`JavaScript`在保证页面不被刷新、页面链接不改变的情况下与服务器交换数据并更新部分网页的技术。

传统的网页，如果你想更新其内容，那么必须要刷新整个页面。有了`Ajax`，便可以在页面不被全部刷新的情况下更新其内容。在这个过程中，页面实际上在后台与服务器进行了数据交互，获取到数据之后，再 利用`JavaScript`改变网页，这样网页内容就会更新了。

到`W3School`上体验几个`Demo`来感受一下：[http://www.w3school.com.cn/ajax/ajax_xmlhttprequest_send.asp](http://www.w3school.com.cn/ajax/ajax_xmlhttprequest_send.asp)。

## 基本原理

初步了解了`Ajax`之后，来详细了解它的基本原理。发送`Ajax`请求到网页更新的过程可以简单分为以下3步：

- 发送请求
- 解析内容
- 渲染网页

### 发送请求

`JavaScript`可以实现页面的各种交互功能，`Ajax`也不例外，它是由`JavaScript`实现的，实际上执行了如下代码：

```javascript
var xmlhttp;
if (window.XMLHttpRequest)
  {// code for IE7+, Firefox, Chrome, Opera, Safari
  xmlhttp=new XMLHttpRequest();
  }
else
  {// code for IE6, IE5
  xmlhttp=new ActiveXObject("Microsoft.XMLHTTP");
  }
xmlhttp.onreadystatechange=function()
  {
  if (xmlhttp.readyState==4 && xmlhttp.status==200)
    {
    document.getElementById("myDiv").innerHTML=xmlhttp.responseText;
    }
  }
xmlhttp.open("GET","/ajax/demo_get.asp",true);
xmlhttp.send();
```

这是`JavaScript`对`Ajax`最底层的实现，这个过程实际上是新建了`XMLHttpRequest`对象，然后调用`onreadystatechange`属性设置监听，最后调用`open()`和`send()`方法向某个链接（也就是服务器）发送请求。

用`Python`实现请求发送之后，可以得到响应结果，但这里请求的发送由`JavaScript`来完成。由于设置了监听，所以当服务器返回响应时，`onreadystatechange`对应的方法便会被触发，在这个方法里面 解析响应内容即可。

### 解析内容

解得到响应之后，`onreadystatechange`属性对应的方法会被触发，此时利用`xmlhttp`的`responseText`属性便可取到响应内容。这类似于`Python`中利用`requests`向服务器发起请求，然后得到响应的过程。

返回的内容可能是`HTML`，也可能是`JSON`，接下来只需要在方法中用`JavaScript`进一步处理即可。比如，如果返回的内容是`JSON`的话，可以对它进行解析和转化。

### 渲染网页

`JavaScript`有改变网页内容的能力，解析完响应内容之后，就可以调用`JavaScript`针对解析完的内容对网页进行下一步处理。比如，通过`document.getElementById().innerHTML`这样的操作，对某个元素内的源代码进行更改，这样网页显示的内容就改变了，这种对`Document`网页文档进行如更改、删除等操作也被称作**`DOM`操作**。

上例中，`document.getElementById("myDiv").innerHTML=xmlhttp.responseText`这个操作便将`ID`为`myDiv`的节点内部的`HTML`代码更改为服务器返回的内容，这样`myDiv`元素内部便会呈现出服务器返回的新数据，网页的部分内容看上去就更新了。

可以看到，**发送请求**、**解析内容**和**渲染网页**这3个步骤其实都是由`JavaScript`完成的。

## Ajax分析

用浏览器打开微博链接[https://weibo.com/u/5902654742](https://weibo.com/u/5902654742)，随后在页面中点击鼠标右键，从弹出的快捷菜单中选择“检查”选项，此时便会弹出开发者工具，如图所示：

![](Screenshot_1.webp)

```python

```

`Ajax`有其特殊的请求类型，它叫作`xhr`。在图中我们可以发现一个以`getIndex`开头的请求，其`Type`为`xhr`，这就是一个`Ajax`请求。用鼠标点击这个请求，可以查看这个请求的详细信息。

![](Screenshot_2.webp)

在右侧可以观察到`Request Headers`、`URL`和`Response Headers`等信息。`Request Headers`中有一个信息为`X-Requested-With:XMLHttpRequest`，这就标记了此请求是`Ajax`请求，如图所示：

![](Screenshot_3.webp)

![](Screenshot_4.webp)

点击`Preview`，即可看到响应的内容，它是`JSON`格式的。这里`Chrome`为我们自动做了解析，点击箭头即可展开和收起相应内容。

可以观察到，返回结果是我的个人信息，包括昵称、简介、头像等，这也是用来渲染个人主页所使用的数据。`JavaScript`接收到这些数据之后，再执行相应的渲染方法，整个页面就渲染出来了。

![](Screenshot_5.webp)

切换到`Response`选项卡，从中观察到真实的返回数据，如图所示：

![](Screenshot_6.webp)

所以说，我们看到的微博页面的真实数据并不是最原始的页面返回的，而是在执行 JavaScript 后再次向后台发送 Ajax请求，浏览器拿到数据后进一步渲染出来的。

## 过滤请求

利用`Chrome`开发者工具的筛选功能筛选出所有的`Ajax`请求。在请求的上方有一层筛选栏，直接点击`XHR`，此时在下方显示的所有请求便都是`Ajax`请求了，如图所示：

![](Screenshot_7.webp)

接下来，不断滑动页面，可以看到页面底部有一条条新的微博被刷出，而开发者工具下方也不断地出现`Ajax`请求，这样我们就可以捕获到所有的`Ajax`请求了。 随意点开一个条目，都可以清楚地看到其`Request URL、Request Headers、Response Headers、Response Body`等内容，此时想要模拟请求和提取就非常简单了。

![](Screenshot_8.webp)

