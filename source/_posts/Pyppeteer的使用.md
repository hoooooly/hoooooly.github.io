---
title: Pyppeteer的使用
tags:
  - python
  - pypprteer
comments: true
toc: true
only:
  - home
  - category
  - tag
date: 2021-06-02 14:04:50
categories: 爬虫
pic:
---

Selenium功能虽热强大，但还是有些不方便。

## Pyppeteer介绍

`Puppeteer` 是 `Google` 基于 Node.js 开发的一个工具，有了它我们可以通过 `JavaScript` 来控制 `Chrome` 浏览器的一些操作，当然也可以用作网络爬虫上，其 API 极其完善，功能非常强大。

`Pyppeteer` 是 `Puppeteer` 的 Python版本的实现，但它不是 `Google` 开发的，是一位来自于日本的工程师依据 `Puppeteer` 的一些功能开发出来的非官方版本。

`Pyppeteer` 是依赖于 `Chromium`这个浏览器来运行的。那么有了 `Pyppeteer` 之后，我们就可以免去那些烦琐的环境配置等问题。如果第一次运行的时候，`Chromium`浏览器没有安装，那么程序会帮我们自动安装和配置，就免去了烦琐的环境配置等工作。

## 安装

运行环境python 3.7以上。

```python
pip3 install pyppeteer
```

查看安装路径

```python
import pyppeteer

print(pyppeteer.__chromium_revision__)  # 查看版本号
print(pyppeteer.executablePath())   # 查看chromium存放路径
```
output:

```python
588429
C:\Users\espho\AppData\Local\pyppeteer\pyppeteer\local-chromium\588429\chrome-win32\chrome.exe
```

## 使用

`Pyppeteer` 是一款非常高效的 web 自动化测试工具，由于 `Pyppeteer` 是基于 `asyncio` 构建的，它的所有属性和方法几乎都是 `coroutine (协程)` 对象，因此在构建异步程序的时候非常方便，天生就支持异步运行。

程序构建的基本思路是新建一个 `browser` 浏览器 和 一个 页面 `page`。

```python
import asyncio
from pyppeteer import launch

async def main():
    browser = await launch()
    page = await browser.newPage()
    await page.goto('http://baidu.com')
    await page.screenshot({'path': 'example.png'})  # 截图
    await browser.close()   # 关闭

asyncio.get_event_loop().run_until_complete(main())
```

运行上面这段代码会发现并没有浏览器弹出运行，这是因为 `Pyppeteer` `默认使用的是无头浏览器，如果想要浏览器显示，需要在launch` 函数中设置参数 `“headless = False”`，程序运行结束后在同一目录下会出现截取到的网页图片：

![](example.webp)

有个问题就是图片显示不全，网站内容没有办法完全铺满。

```python
# Pyppeteer 支持字典 和 关键字传参，Puppeteer 只支持字典传参。
# 这里使用字典传参
browser = await launch(
    {
        'headless': False, 
        'dumpio': True, 
        'autoClose': False, 
        'args': [
            '--no-sandbox', 
            '--window-size=1366,850'
        ]
    }
)
await page.setViewport({'width': 1366, 'height': 768})
```

### 报错

报错 `OSError: Unable to remove Temporary User Data`
启动浏览器时指定参数 `userDataDir` 存放缓存，保证硬盘够大且不是系统盘。

### 移除Chrome正受到自动测试软件的控制

```python
import asyncio
from pyppeteer import launch

async def main():
    browser = await launch(headless=False, ignoreDefaultArgs=['--enable-automation'])
    input()
    await browser.close()

asyncio.get_event_loop().run_until_complete(main())
```

### 全屏

```python
import tkinter
import asyncio
from pyppeteer import launch


def screen_size():
    tk = tkinter.Tk()
    width = tk.winfo_screenwidth()
    height = tk.winfo_screenheight()
    tk.quit()
    return {'width': width, 'height': height}


async def main():
    browser = await launch(headless=False, args=['--start-maximized'])  # 页面全屏
    page = await browser.newPage()
    await page.setViewport(screen_size())  # 内容全屏
    await page.goto('https://www.baidu.com/')
    input()
    await browser.close()


asyncio.get_event_loop().run_until_complete(main())
```

### 页面内容

Page.content()或Page.evalute()

```python
import asyncio
from pyppeteer import launch


async def main():
    browser = await launch(headless=False)
    page = await browser.newPage()
    url = 'https://www.holychan.ltd/'
    await page.goto(url)
    # content = await page.content()
    content = await page.evaluate('document.body.textContent', force_expr=True)
    print(content)
    input()
    await browser.close()


asyncio.get_event_loop().run_until_complete(main())
```

把整个页面的文字获取下来了。

### 异步运行

`asyncio.wait() `或 `asyncio.gather()`，建议只用在一次性读取的页面，需要滚动的不建议使用。

```python
import asyncio
from pyppeteer import launch

async def crawl(url):
    browser = await  launch(headless=False)
    page = await browser.newPage()
    await page.goto(url)
    title = await page.title()
    print(title)
    await browser.close()

async def main():
    urls = [
        crawl('https://www.baidu.com/'),
        crawl('https://www.bing.com/')
    ]
    await asyncio.wait(urls)
    # await asyncio.gather(*urls)

asyncio.get_event_loop().run_until_complete(main())

#微软 Bing 搜索 - 国内版
#百度一下，你就知道
```

### 反爬虫检测

访问 [https://bot.sannysoft.com/](https://bot.sannysoft.com/)。

```python
import asyncio
from pyppeteer import launch

async def main():
    browser = await launch(
        {
            'headless': False, 
            'dumpio': True, 
            'autoClose': False, 
            'data_dir':'E:/.cache/',
            'args': [
                '--no-sandbox', 
                '--window-size=1366,850'
            ]
            
        }
    )
    page = await browser.newPage()
    await page.setViewport({'width': 1366, 'height': 768})
    await page.goto('https://bot.sannysoft.com/')
    await page.screenshot({'path': 'test.png'})
    await browser.close()

asyncio.get_event_loop().run_until_complete(main())
```
![](example1.webp)