---
title: Selenium的基本使用
tags:
  - selenium
comments: true
toc: true
only:
  - home
  - category
  - tag
date: 2021-06-19 21:15:51
categories: 爬虫
index_img: /2021/06/19/Selenium的基本使用/selenium.png
---

在很多情况下，`Ajax`请求的接口通常会包含加密的参数，如`token`、`sign`等，如：[https://dynamic2.scrape.cuiqingcai.com/](https://dynamic2.scrape.cuiqingcai.com/)，它的`Ajax`接口是包含一个`token`参数的，如图所示。

![](Screenshot_5.webp)

由于接口的请求加上了`token`参数，如果不深入分析并找到`token`的构造逻辑，是难以直接模拟这些`Ajax`请求的。

`Selenium`是一个自动化测试工具，利用它可以驱动浏览器执行特定的动作，如点击、下拉等操作，同时还可以获取浏览器当前呈现的页面源代码，做到可见即可爬。对于一些使用`JavaScript`动态渲染的页面来说，此种抓取方式非常有效。

## 准备工作

确保已经正确安装好了`Chrome`浏览器并配置好了`ChromeDriver`。另外，还需要正确安装好`Python`的`Selenium`库。

### Selenium安装

```python
pip install selenium
```

### chrome driver安装

查看安装的浏览器版本，在`chrome`搜索栏输入`chrome://version/`，会显示带当前浏览器的版本信息。

![](Screenshot_1.webp)

在[https://npm.taobao.org/mirrors/chromedriver/](https://npm.taobao.org/mirrors/chromedriver/)下载对应版本的`chrome driver`，一定要对应，否则不能使用。

![](Screenshot_2.webp)

下载到本地后，将`chromedriver.exe`分别拷贝到`chrome.exe`和`python`安装路径下。

![](Screenshot_3.webp)

![](Screenshot_4.webp)

以上，安装就完成了！

## 基本使用

首先来看一下`Selenium`有一些怎样的功能。示例如下：

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.wait import WebDriverWait

browser = webdriver.Chrome()
try:
    browser.get('https://www.baidu.com')
    input = browser.find_element_by_id('kw')
    input.send_keys('Python')
    input.send_keys(Keys.ENTER)
    wait = WebDriverWait(browser, 10)
    wait.until(EC.presence_of_element_located((By.ID, 'content_left')))
    print(browser.current_url)
    print(browser.get_cookies())
    print(browser.page_source)
finally:
    browser.close()
```

运行代码后会自动弹出一个`Chrome`浏览器，浏览器会跳转到百度，然后在搜索框中输入`Python`，接着跳转到搜索结果页，如图所示。

![](Screenshot_6.webp)

此时在控制台的输出结果如下：

```shell
href="http://www.baidu.com/link?url=CYEq_OJxbiLcx5LothJSzfcQI9xeTzMYkPSFg2HYyceDRTQi8o6Fk8oLyRG5BTI8remsEFjvd4qN8vGLA-EBg_" target="_blank">你都用 <em>Python</em> 来做什么? - 知乎</a></h3><div class="c-row c-gap-top-small"><div class="general_image_pic c-span3" style="position:relative;top:2px;"><a class="c-img c-img3 c-img-radius-large" style="height:85px" href="http://www.baidu.com/link?url=CYEq_OJxbiLcx5LothJSzfcQI9xeTzMYkPSFg2HYyceDRTQi8o6Fk8oLyRG5BTI8remsEFjvd4qN8vGLA-EBg_" target="_blank"><img class="c-img c-img3 c-img-radius-large" src="https://dss1.bdstatic.com/6OF1bjeh1BF3odCf/it/u=958012630,1445793633&amp;fm=218&amp;app=92&amp;f=JPEG?w=121&amp;h=75&amp;s=44A638725CB7469C82F4FFF40200D025" style="height:85px;"><span class="c-img-border c-img-radius-large"></span></a></div><div class="c-span9 c-span-last"><div class="c-abstract"><span class=" newTimeFactor_before_abs c-color-gray2 m">2019年10月17日&nbsp;</span>所有项目的代码和数据在Github:interesting-<em>python</em> 如果你也想用<em>Python</em>获取数据,进行有趣的 
数据分析,Alfred数据室应众多读者要求出品的《实战玩转<em>python</em>爬虫》课程将会是你的好帮手...</div><style>.user-avatar {
        display: flex;
        flex-direction: row;
        align-items: center;
        justify-content: flex-start;
}</style><div class="f13 c-gap-top-xsmall se_st_footer user-avatar"><a target="_blank" href="http://www.baidu.com/link?url=CYEq_OJxbiLcx5LothJSzfcQI9xeTzMYkPSFg2HYyceDRTQi8o6Fk8oLyRG5BTI8remsEFjvd4qN8vGLA-EBg_" class="c-showurl c-color-gray" style="text-decoration:none;position:relative;"><div class="c-img c-img-circle c-gap-right-xsmall" style="display: inline-block;width: 16px;height: 16px;position: relative;top: 3px;vertical-align:top;"><span class="c-img-border c-img-source-border c-img-radius-large"></span><img src="https://ss0.baidu.com/6ONWsjip0QIZ8tyhnq/it/u=493147230,3096476255&amp;fm=195&amp;app=88&amp;f=JPEG?w=200&amp;h=200"></div>知乎</a><div class="c-tools c-gap-left" id="tools_9637258987948217910_8" data-tools="{&quot;title&quot;:&quot;你都用 Python 来做什么? - 知乎&quot;,&quot;url&quot;:&quot;http://www.baidu.com/link?url=CYEq_OJxbiLcx5LothJSzfcQI9xeTzMYkPSFg2HYyceDRTQi8o6Fk8oLyRG5BTI8remsEFjvd4qN8vGLA-EBg_&quot;}"><i class="c-icon f13"></i></div><span class="c-icons-outer"><span class="c-icons-inner"></span></span><style>.snapshoot, .snapshoot:visited {
        color: #9195A3!important;
    }
    .snapshoot:active, .snapshoot:hover {
        color: #626675!important;
    }</style><a data-click="{'rsv_snapshot':'1'}" href="http://cache.baiducontent.com/c?m=JgFgeas2iHrh6pppvD-MHkvHPj9DsOz-wd718JIlv_IJy7d0KdQ2HvVIYznaRWwGrKHLEtVbqjR4uoZg6X_h4g62yFf5FgBVDUBkRfcWrOG&amp;p=8b2a9715d9c647f904aec5344f57&amp;newp=8b2a9729c5df1bb508e291784f4a92695d0fc20e3ad0d001298ffe0cc4241a1a1a3aecbf2c261600d7c279650aab4a5feff336743d0034f1f689df08d2ecce7e439b3b6e6a&amp;s=45c48cce2e2d7fbd&amp;user=baidu&amp;fm=sc&amp;query=Python&amp;qid=dc4c1a0300139b42&amp;p1=8" target="_blank" class="m c-gap-left c-color-gray kuaizhao snapshoot">百度快照</a></div></div></div></div>
```

源代码过长，在此省略。可以看到，当前得到的`URL`、`Cookies`和源代码都是浏览器中的真实内容。所以说，如果用`Selenium`来驱动浏览器加载网页的话，就可以直接拿到`JavaScript`渲染的结果了，不用担心使用的是什么加密系统。

了解一下`Selenium`的用法。

## 声明浏览器对象

`Selenium`支持非常多的浏览器，如`Chrome`、`Firefox`、`Edge`等，还有`Android`、`BlackBerry`等手机端的浏览器。

可以用如下方式进行初始化：

```python
from selenium import webdriver
browser = webdriver.Chrome()
browser = webdriver.Firefox()
browser = webdriver.Edge()
browser = webdriver.Safari()

# 如果你没有把chromedriver添加到系统环境变量中，executable_path参数要附上chromedriver的绝对路径
browser = webdriver.Chrome(executable_path="D:/tools/chromedriver")
```

这样就完成了浏览器对象的初始化并将其赋值为`browser`对象。接下来，要做的就是调用`browser`对象，让其执行各个动作以模拟浏览器操作。

## 访问页面

可以用`get`方法请求页面，只需要把参数传入连接`URL`即可。比如，这里用`get`方法访问淘宝，代码如下：

```python
from selenium import webdriver
browser = webdriver.Chrome()
browser.get('https://www.taobao.com')
print(browser.page_source)
browser.close()
```

运行后会弹出`Chrome`浏览器并且自动访问淘宝，然后控制台会输出淘宝页面的源代码，随后浏览器关闭。

## 查找节点

`Selenium`可以驱动浏览器完成各种操作，比如填充表单、模拟点击等。举个例子，当想要完成向某个输入框输入文字的操作时，首先需要知道这个输入框在哪，而 `Selenium`提供了一系列查找节点的方法，可以用这些方法来获取想要的节点，以便执行下一步动作或者提取信息。

### 单个节点

想要从淘宝页面中提取搜索框这个节点，首先要观察它的源代码。

可以发现，它的`id`是`q`，`name`也是`q`，此外还有许多其他属性。此时就可以用多种方式获取它了。比如，`find_element_by_name`代表根据`name`值获取，`find_element_by_id`则是根据`id`获取，另外，还有根据`XPath`、`CSS`选择器等获取的方式。

这里使用`3`种方式获取输入框，分别是根据`id`、`CSS`选择器和`XPath`获取，它们返回的结果完全一致。运行结果如下：

```python
<selenium.webdriver.remote.webelement.WebElement (session="fbca625cd8bb4eb98ae76bb68a23bd4f",element="de630823-02d3-409a-ba19-4e892a0fad15")> 
<selenium.webdriver.remote.webelement.WebElement (session="fbca625cd8bb4eb98ae76bb68a23bd4f", element="de630823-02d3-409a-ba19-4e892a0fad15")> 
[<selenium.webdriver.remote.webelement.WebElement (session="fbca625cd8bb4eb98ae76bb68a23bd4f", element="de630823-02d3-409a-ba19-4e892a0fad15")>]
```

这3个节点的类型是一致的，都是`WebElement`。

这里列出所有获取单个节点的方法：

```python
find_element_by_id 
find_element_by_name 
find_element_by_xpath 
find_element_by_link_text 
find_element_by_partial_link_text 
find_element_by_tag_name 
find_element_by_class_name 
find_element_by_css_selector
```

### 多个节点

多如果在网页中只查找一个目标，那么完全可以用`find_element`方法。但如果有多个节点需要查找，再用`find_element`方法，就只能得到第1个节点了。如果要查找所有满足条件的节点，需要用`find_elements`这样的 方法。注意，在这个方法的名称中， 注`element`多了一个多`s`，注意区分。

举个例子，假如要查找淘宝左侧导航条的所有条目，就可以这样来实现：

```python
from selenium import webdriver

browser = webdriver.Chrome()
browser.get('https://www.taobao.com')
lis = browser.find_elements_by_css_selector('.service-bd li')
print(lis)
browser.close()
```

结果如下：

```python
[<selenium.webdriver.remote.webelement.WebElement (session="f9550f7bf1204e4d481803cf7a69c736", element="01e78303-6103-4f1a-96dc-0386a755a9f4")>, <selenium.webdriver.remote.webelement.WebElement (session="f9550f7bf1204e4d481803cf7a69c736", element="7a4e764e-9a05-4dad-9188-636512591835")>, <selenium.webdriver.remote.webelement.WebElement (session="f9550f7bf1204e4d481803cf7a69c736", element="16219266-ff57-4f42-95ab-4b8dbb4b811a")>, <selenium.webdriver.remote.webelement.WebElement (session="f9550f7bf1204e4d481803cf7a69c736", element="4f23598b-3ca6-4b20-9192-3e06e98984df")>, <selenium.webdriver.remote.webelement.WebElement (session="f9550f7bf1204e4d481803cf7a69c736", element="d837e630-e546-4f8b-babd-e3fdc539d2a6")>, <selenium.webdriver.remote.webelement.WebElement (session="f9550f7bf1204e4d481803cf7a69c736", element="1da195c9-d959-4d47-b729-21d8eac2756f")>, <selenium.webdriver.remote.webelement.WebElement (session="f9550f7bf1204e4d481803cf7a69c736", element="175c2217-dbe6-4d27-997f-58fbe46cbf1a")>, <selenium.webdriver.remote.webelement.WebElement (session="f9550f7bf1204e4d481803cf7a69c736", element="4747359f-d93c-4a7a-976e-501087d96e5f")>, <selenium.webdriver.remote.webelement.WebElement (session="f9550f7bf1204e4d481803cf7a69c736", element="2537bb34-3a26-4ccd-8f53-4ebbef5c70b2")>, <selenium.webdriver.remote.webelement.WebElement (session="f9550f7bf1204e4d481803cf7a69c736", element="04210bc7-ebba-44ef-8757-d88dfd0d023f")>, <selenium.webdriver.remote.webelement.WebElement (session="f9550f7bf1204e4d481803cf7a69c736", element="7bd1b6e4-31e5-4120-8017-afc59ca56f48")>, <selenium.webdriver.remote.webelement.WebElement (session="f9550f7bf1204e4d481803cf7a69c736", element="930a7b6c-26ff-468c-896e-6204b0555747")>, <selenium.webdriver.remote.webelement.WebElement (session="f9550f7bf1204e4d481803cf7a69c736", element="81305b6e-0893-46d2-b7a7-89a85c00132c")>, <selenium.webdriver.remote.webelement.WebElement (session="f9550f7bf1204e4d481803cf7a69c736", element="763237aa-5925-426d-9a76-90b3567870eb")>, <selenium.webdriver.remote.webelement.WebElement (session="f9550f7bf1204e4d481803cf7a69c736", element="60ca3595-3f74-4f1d-9224-7beb9e381a4b")>]
```

得到的内容变成了列表类型，列表中的每个节点都是`WebElement`类型。

也就是说，如果用`find_element`方法，只能获取匹配的第一个节点，结果是`WebElement`类型。如果用`find_elements`方法，则结果是列表类型，列表中的每个节点是`WebElement`类型。

这里列出所有获取多个节点的方法：

```python
find_elements_by_id
find_elements_by_name
find_elements_by_xpath
find_elements_by_link_text
find_elements_by_partial_link_text
find_elements_by_tag_name
find_elements_by_class_name
find_elements_by_css_selector
```

当然，也可以直接用`find_elements`方法来选择，这时可以这样写：

```python
lis = browser.find_elements(By.CSS_SELECTOR, '.service-bd li') 
```

结果是完全一致的。

### 节点交互

`Selenium`可以驱动浏览器来执行一些操作，或者说可以让浏览器模拟执行一些动作。比较常见的用法有：输入文字时用`send_keys`方法，清空文字时用`clear`方法，点击按钮时用`click`方法。示例如下：

```python
import time

browser = webdriver.Chrome()
browser.get('https://www.taobao.com')
input = browser.find_element_by_id('q')
input.send_keys('apple')
time.sleep(1)
input.clear()
input.send_keys('小米')
button = browser.find_element_by_class_name('btn-search')
button.click()
```

这里首先驱动浏览器打开淘宝，用`find_element_by_id`方法获取输入框，然后用`send_keys`方法输入`iPhone`文字，等待一秒后用`clear`方法清空输入框，接着再次调用`send_keys`方法输入`iPad`文字，之后再用`find_element_by_class_name`方法获取搜索按钮，最后调用`click`方法完成搜索动作。

### 动作链

在上面的实例中，一些交互动作都是针对某个节点执行的。比如，对于输入框，调用它的输入文字和清空文字方法；对于按钮，调用它的点击方法。其实，还有另外一些操作，它们没有特定的执行对象，比如鼠标拖拽、键盘按键等，这些动作用另一种方式来执行，那就是动作链。

比如，现在要实现一个节点的拖拽操作，将某个节点从一处拖拽到另外一处，可以这样实现：

```python
from selenium import webdriver
from selenium.webdriver import ActionChains

browser = webdriver.Chrome()
url = 'http://www.runoob.com/try/try.php?filename=jqueryui-api-droppable'

browser.get(url)
browser.switch_to.frame('iframeResult')
source = browser.find_element_by_css_selector("#draggable")
target = browser.find_element_by_css_selector("#droppable")
actions = ActionChains(browser)
actions.drag_and_drop(source, target)
actions.perform()
```

打开网页中的一个拖拽实例，依次选中要拖拽的节点和拖拽到的目标节点，接着声明`ActionChains`对象并将其赋值为`actions`变量，然后通过调用`actions`变量的`drag_and_drop`方法，再调用`perform`方法执行动作，此时就完成了拖拽操作，如图所示：

![](Screenshot_7.webp)

拖拽前页面：

![](Screenshot_8.webp)

### 执行执`JavaScript`

`SeleniumAPI`并没有提供实现某些操作的方法，比如，下拉进度条。但它可以直接模拟运行`JavaScript`，此时使用`execute_script`方法即可实现，代码如下：

```python
from selenium import webdriver
browser = webdriver.Chrome()
browser.get('https://www.zhihu.com/explore')
browser.execute_script('window.scrollTo(0, document.body.scrollHeight)')
browser.execute_script('alert(" To Bottom)')
```

这里利用`execute_script`方法将进度条下拉到最底部，然后弹出`alert`提示框。

有了这个方法，基本上`API`没有提供的所有功能都可以用执行`JavaScript`的方式来实现了。

## 获取节点信息

`Selenium`已经提供了选择节点的方法，并且返回的是`WebElement`类型，那么它也有相关的方法和属性来直接提取节点信息，如属性、文本等。这样的话，就可以不用通过解析源代码来提取信息了，非常方便。

### 获取属性

可以使用`get_attribute`方法来获取节点的属性，但是前提是得先选中这个节点，示例如下：

```python
from selenium import webdriver

browser = webdriver.Chrome()
url = 'https://www.baidu.com/'
browser.get(url)
logo = browser.find_element_by_class_name('index-logo-src')
print(logo)
print(logo.get_attribute('src'))
```

运行之后，程序便会驱动浏览器打开该页面，然后获取`class`为`logo-image`的节点，最后打印出它的`src`属性。

返回结果：

```python
<selenium.webdriver.remote.webelement.WebElement (session="c90d6158ff9e3e6ba88d5f9b9358a358", element="c99170cd-b0c1-44b6-a0a0-f4ccf1105e91")>
https://www.baidu.com/img/PCtm_d9c8750bed0b3c7d089fa7d55720d6cf.png
```

通过`get_attribute`方法，只需要传入想要获取的属性名，就可以得到它的值了。

### 获取文本值

每个`WebElement`节点都有`text`属性，直接调用这个属性就可以得到节点内部的文本信息，这相当于`pyquery`的`text`方法，示例如下：

```python
from selenium import webdriver

browser = webdriver.Chrome()
url = 'https://holychan.ltd/archives/'
browser.get(url)
title = browser.find_element_by_class_name('title-h2')
print(title)
print(title.text)
```

结果如下：

```python
<selenium.webdriver.remote.webelement.WebElement (session="79f97c0aa935723e276438a5fff2610c", element="b86ec344-cb00-4538-b9e9-8959a076bc75")>
文章归档
```

### 获取获ID、位置、标签名、大小

`WebElement`节点还有一些其他属性，比如`id`属性可以获取节点`id`，`location`属性可以获取该节点在页面中的相对位置，`tag_name`属性可以获取标签名称，`size`属性可以获取节点的大小，也就是宽高，这些 属性有时候还是很有用的。示例如下：

```python
from selenium import webdriver
browser = webdriver.Chrome() 
url = 'https://dynamic2.scrape.cuiqingcai.com/' 
browser.get(url) 
input = browser.find_element_by_class_name('logo-title') 
print(input.id) 
print(input.location) 
print(input.tag_name) 
print(input.size) 
```

这里首先获得`class`为`logo-title`这个节点，然后调用其`id`、`location`、`tag_name`、`size`属性来获取对应的属性值。

## 切换Frame

网页中有一种节点叫作`iframe`，也就是子`Frame`，相当于页面的子页面，它的结构和外部网页的结构完全一致。`Selenium`打开页面后，默认是在父级`Frame`里面操作，而此时如果页面中还有子`Frame`，`Selenium`是不能获取到子`Frame`里面的节点的。这时就需要使用`switch_to.frame`方法来切换`Frame`。示例如下：

```python
from selenium import webdriver
from selenium.common.exceptions import NoSuchElementException
browser = webdriver.Chrome()
url = 'http://www.runoob.com/try/try.php?filename=jqueryui-api-droppable'
browser.get(url)
browser.switch_to('iframeResult')
try:
    logo = browser.find_element_by_class_name('logo')
except NoSuchElementException:
    print('No logo found')
browser.switch_to.parent_frame()
logo = browser.find_element_by_class_name('logo')
print(logo)
print(logo.text)
```

控制台输出：

```python
No logo found
<selenium.webdriver.remote.webelement.WebElement (session="ce74c2d83e0aca64c9ef8c4540b2fe16", element="1d2c3c25-bd01-4b9a-b11b-16b8534bf6c2")>
RUNOOB.COM
```

首先通过`switch_to.frame`方法切换到子`Frame`里面，然后尝试获取子`Frame`里的`logo`节点（这是不能找到的），如果找不到的话，就会抛出`NoSuchElementException`异常，异常被捕捉之后，就会输出`No logo found`。接下来，我们需要重新切换回父级`Frame`，然后再次重新获取节点，发现此时可以成功获取了。

所以，当页面中包含子`Frame`时，如果想获取子`Frame`中的节点，需要先调用`switch_to.frame`方法切换到对应的`Frame`，然后再进行操作。

## 延时等待

在`Selenium`中，`get`方法会在网页框架加载结束后结束执行，此时如果获取`page_source`，可能并不是浏览器完全加载完成的页面，如果某些页面有额外的`Ajax`请求，在网页源代码中也不一定能成功获取 \到。所以，这里需要延时等待一定时间，确保节点已经加载出来。

这里等待的方式有两种：一种是隐式等待，一种是显式等待。

### 隐式等待 

当使用隐式等待执行测试的时候，如果`Selenium`没有在`DOM`中找到节点，将继续等待，超出设定时间后，则抛出找不到节点的异常。换句话说，隐式等待可以在我们查找节点而节点并没有立即出现的时候， 等待一段时间再查找`DOM`，默认的时间是`0`。示例如下：

```python
from selenium import webdriver

browser = webdriver.Chrome()
browser.implicitly_wait(10)
browser.get('https://dynamic2. scrape.cuiqingcai.com/') 
input = browser.find_element_by_class_name('logo-image') 
print(input)
```

## selenium提取数据

### driver对象的常用属性和方法

1. `driver.page_source` 当前标签页游览器渲染后的网页源代码
2. `driver.current_url` 当前标签页的url
3. `drive.close` 关闭当前标签页，如果只有一个标签页则关闭整个浏览器
4. `drice.quit` 关闭浏览器
5. `drive.forward` 页面前进
6. `drive.back` 页面后退
7. `drive.save_screenshot(imgname)` 页面截图

```python
import time
from selenium import webdriver

driver = webdriver.Chrome(executable_path="D:/tools/chromedriver")

driver.get("https://www.baidu.com")

# # 在对话框输入python
driver.find_element_by_id("kw").send_keys('python')

# # 点击百度搜索
driver.find_element_by_id("su").click()

# time.sleep(6)
# 保存搜索截图
driver.save_screenshot("baidu_python.png")


# 显示源码
print(driver.page_source)

# 显示响应对应的url
print(driver.current_url)
# https://www.baidu.com/

# 显示标题
print(driver.title)
# 百度一下，你就知道

time.sleep(2)
driver.get('http://www.douban.com')
time.sleep(2)
driver.back()
time.sleep(2)
driver.foward()
time.sleep(2)
driver.close()

driver.quit()
```

### driver对象定位标签元素获取标签对象的方法

```python
driver.find_element(s)_by_id()     # 返回一个元素
driver.find_element(s)_by_class_name()   # 根据类名获取元素列表
driver.find_element(s)_by_name()       # 根据标签的name属性返回包含标签对象元素的列表
driver.find_element(s)_by_xpath()      # 返回一个包含元素的列表
driver.find_element(s)_by_link_text()      # 根据链接文本获取元素列表
driver.find_element(s)_by_partial_link_text()  # 根据链接包含的文本获取元素列表
driver.find_element(s)_by_tag_name()   # 根据标签名获取元素列表
driver.find_element(s)_by_css_selector()   #根据css选择器来获取元素
```

`find_element`和`find_elements`的区别在于，多了s就返回列表，没有s就返回匹配到的第一个标签对象。

```python
import time
from selenium import webdriver


driver = webdriver.Chrome(executable_path="D:/tools/chromedriver")

url = "https://www.baidu.com"

driver.get(url)

# 通过xpath进行元素定位
# driver.find_element_by_xpath('//*[@id="kw"]').send_keys("python")

# 通过css选择器进行元素定位
# driver.find_element_by_css_selector("#kw").send_keys("python")

# 通过name属性值进行元素定位
driver.find_element_by_name("wd").send_keys("python")

# 目标元素在当前HTML是唯一标签的时候或者是众多定位出来的标签中的第一个的时候
driver.find_element_by_tag_name("")

driver.find_element_by_id("su").click()

time.sleep(2)

driver.quit()
```

### 标签对象提取文本内容和属性值

- 获取文本`element.text`
   - 通过定位获取的标签对象的`text`属性，获取文本内容

- 获取属性值`element.get_attribute('属性名')
   - 通过定位获取标签对象的`get_attribute`函数，获取文本内容

代码实现：

```python
import time
from selenium import webdriver


driver = webdriver.Chrome(executable_path="D:/tools/chromedriver")

driver.get("https://holychan.ltd/")

title = driver.find_element_by_tag_name("h2")

print(title.text)
# Holy的个人站点

email = driver.find_element_by_xpath("//*[@id='footer']/div[2]/div/div/ul[1]/li/a")

print(email.get_attribute('href'))
# mailto:espholychan@outlook.com

driver.quit()
```

## selenium的其他用法

### 标签页的切换

当selenium打开多个标签页时，如何控制浏览器在不同的标签页中进行切换呢?

- 获取所有标签页的窗口句柄

- 利用窗口句柄切换到句柄指向的标签页


具体方法：

1. 获取当前所有标签页的句柄构成的列表

```python
current_windows = driver.window_handles
```

2. 根据标签页的句柄列表索引下标进行切换

```python
driver.switch_to.window(current_windows[0])
```

案列：

```python
import time
from selenium import webdriver

url = "https://sz.58.com/"

driver = webdriver.Chrome(executable_path="D:/tools/chromedriver")

driver.get(url)

print(driver.current_url)
print(driver.window_handles)

# 定位并点击租房按钮
driver.find_element_by_xpath('//*[@id="fcNav"]/em/a[1]').click()


print(driver.current_url)
print(driver.window_handles)

time.sleep(3)
current_windows = driver.window_handles

driver.switch_to.window(current_windows[0])

time.sleep(3)

driver.quit()
```

### switch_to切换frame标签

>iframe是html中常用的一种技术，即一个页面中嵌套着另外一个网页，selenium默认是访问不了frame中的内容的，对应的解决思路是 driver.switch_to.frame(frame_element)。

 登录QQ邮箱：

 ```python
import time
from selenium import webdriver

url = "https://mail.qq.com/"

driver = webdriver.Chrome(executable_path="D:/tools/chromedriver")

driver.get(url)

# 切换当登陆的frame
driver.switch_to.frame('login_frame')

# 输入账号
driver.find_element_by_xpath('//*[@id="u"]').send_keys("admin")
# 输入密码
driver.find_element_by_xpath('//*[@id="p"]').send_keys("=pass")
# 点击登录
driver.find_element_by_xpath('//*[@id="login_button"]').click()

time.sleep(2)
# 输入独立密码, 没设置独立密码的注释下面
driver.find_element_by_xpath('//*[@id="pp"]').send_keys("pass")

# 点击登录
driver.find_element_by_xpath('//*[@id="btlogin"]').click()

time.sleep(6)

driver.quit()
```

### selenium对cookie的处理

1. 获取cookie

`driver.get_cookies()`返回列表信息，其中包含的是完整的cookie信息，光有Name、Value还有domain等cookie其他维度的信息。所以如果想要把获取到cookie信息和requests模块配合使用的话，需要转换为name、value作为键值对的cookie字典。

```python
# 获取当前标签页的全部cookie信息
driver.get_cookies()
# 把cookies转化成字典
cookies_dict = {cookie['name']:cookie['value'] for cookie in driver.get_cookies()}
```

### selenium控制浏览器执行js代码

`selelnium` 可以让浏览器执行我们规定的`js代码`，运行下列代码查看运行效果：

```python
import time
from selenium import webdriver

url = "https://www.holychan.ltd/"

driver = webdriver.Chrome(executable_path="D:/tools/chromedriver")
driver.get(url)
time.sleep(1)

# 定义一个js语句，滑动到页面底部
js = "window.scrollTo(0, document.body.scrollHeight)"
# 执行js的方法
driver.execute_script(js)

time.sleep(2)
driver.quit()
```

### 页面等待

**分类**

1. 强制等待（了解）

- 使用`time.sleep()`
- 缺点：不智能，设置时间太短，元素还没有加载出来。设置时间太长，则会浪费时间。

2. 隐式等待

- 隐式等待针对的是元素定位，隐式等待设置了一个等待时间，在一段时间内判断元素是否定位成功，如果完成了就进行下一步。
- 在设置的时间内没有定位成功，则会报超时加载

示例代码：

```python
import time
from selenium import webdriver

url = "https://www.baidu.com"

driver = webdriver.Chrome(executable_path="D:/tools/chromedriver")

# 设置位置之后的所有元素定位操作都有最大等待时间十秒，在10秒内会定期进行元素定位，超出设置时间后将会报错
driver.implicitly_wait(10)

driver.get(url)

el = driver.find_element_by_xpath('//*[@id="lg"]/img[10000]')
print(el)

driver.quit()
```

3. 显式等待(了解)

每经过多少秒就查看一次等待条件是否达成，如果达成就停止等待，继续执行后续代码。

4. 手动实现页面等待



