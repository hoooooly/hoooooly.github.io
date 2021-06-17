---
title: Requests库
tags:
  - requests
  - http
comments: true
toc: true
only:
  - home
  - category
  - tag
date: 2021-06-17 19:53:38
categories: Python库
pic: requests-sidebar.png
---

> Requests is an elegant and simple HTTP library for Python, built for human beings.

`Requests` 唯一的一个非转基因的 `Python HTTP` 库，人类可以安全享用。

## 安装

`Requests`是一个第三方的库，建议在虚拟环境中安装。

```python
pip install requests
```

## 快速上手

### 发送请求

```python
# 安装requests模块
# pip install requests

# 导入requests
import requests

# 目标url
url = "https://www.baidu.com"

# 向目标url发送请求
response = requests.get(url)

# 打印响应内容
print(response.text)
```

`Requests` 会自动解码来自服务器的内容。大多数 unicode 字符集都能被无缝地解码。

请求发出后， `Requests` 会基于 `HTTP` 头部对响应的编码作出有根据的推测。当你访问 `response.text` 之时，`Requests` 会使用其推测的文本编码。你可以找出 `Requests` 使用了什么编码，并且能够使用 `response.encoding` 属性来改变它：

```python
# 设置响应编码方式
response.encoding = 'ISO-8859-1'
```

### 二进制响应内容

你也能以字节的方式访问请求响应体，对于非文本请求：

```python
# 打印响应内容, 二进制格式
print(response.content)`
```

### 传递 URL 参数

你也许经常想为 `URL` 的查询字符串(query string)传递某种数据。如果你是手工构建 `URL`，那么数据会以键/值对的形式置于 `URL` 中，跟在一个问号的后面。例如，` httpbin.org/get?key=val`。 `Requests` 允许你使用 `params` 关键字参数，以一个字符串字典来提供这些参数。举例来说，如果你想传递` key1=value1` 和 `key2=value2` 到 `httpbin.org/get` ，那么你可以使用如下代码：

```python
>>> payload = {'key1': 'value1', 'key2': 'value2'}
>>> r = requests.get("http://httpbin.org/get", params=payload)
```

通过打印输出该 `URL`，你能看到 `URL` 已被正确编码：

```python
>>> print(r.url)
http://httpbin.org/get?key2=value2&key1=value1
```

注意字典里值为 `None` 的键都不会被添加到 `URL` 的查询字符串里。

你还可以将一个列表作为值传入：

```python
>>> payload = {'key1': 'value1', 'key2': ['value2', 'value3']}

>>> r = requests.get('http://httpbin.org/get', params=payload)
>>> print(r.url)
http://httpbin.org/get?key1=value1&key2=value2&key2=value3
```

### JSON 响应内容

`Requests` 中也有一个内置的 `JSON` 解码器，助你处理 `JSON` 数据：

```python
>>> import requests

>>> r = requests.get('https://api.github.com/events')
>>> r.json()
[{u'repository': {u'open_issues': 0, u'url': 'https://github.com/...
>>> r.status_code
200
```

如果 `JSON` 解码失败， `r.json()` 就会抛出一个异常。例如，响应内容是 `401 (Unauthorized)`，尝试访问 `r.json()` 将会抛出 `ValueError: No JSON object could be decoded `异常。

需要注意的是，成功调用 `r.json()` 并**不**意味着响应的成功。有的服务器会在失败的响应中包含一个 `JSON` 对象（比如` HTTP 500 `的错误细节）。这种 `JSON` 会被解码返回。要检查请求是否成功，请使用 `r.raise_for_status()` 或者检查 `r.status_code` 是否和你的期望相同。

### 原始响应内容

在某些情况下，如果你想获取来自服务器的原始套接字响应，那么你可以访问 `r.raw`。 如果你确实想这么干，那请你确保在初始请求中设置了 `stream=True`。具体你可以这么做：

```python
>>> r = requests.get('https://api.github.com/events', stream=True)
>>> r.raw
<urllib3.response.HTTPResponse object at 0x00000197FAA56AC0>
>>> r.raw.read(10)
b'\x1f\x8b\x08\x00\x00\x00\x00\x00\x00\x03'
```

但一般情况下，你应该以下面的模式将文本流保存到文件：

```python
with open(filename, 'wb') as fd:
    for chunk in r.iter_content(chunk_size):
        fd.write(chunk)
```

使用 `Response.iter_content` 将会处理大量你直接使用 `Response.raw` 不得不处理的。 当流下载时，上面是优先推荐的获取内容方式。 

### 定制请求头

如果你想为请求添加 `HTTP` 头部，只要简单地传递一个 `dict` 给 `headers` 参数就可以了。

例如，在前一个示例中我们没有指定 `content-type`:

```python
>>> url = 'https://api.github.com/some/endpoint'
>>> headers = {'user-agent': 'my-app/0.0.1'}

>>> r = requests.get(url, headers=headers)
```

注意: 定制 `header` 的优先级低于某些特定的信息源，例如：

- 如果在 `.netrc` 中设置了用户认证信息，使用 `headers=` 设置的授权就不会生效。而如果设置了` auth=` 参数，`.netrc` 的设置就无效了。
- 如果被重定向到别的主机，授权 `header` 就会被删除。
- 代理授权 `header` 会被 URL 中提供的代理身份覆盖掉。
- 在我们能判断内容长度的情况下，`header` 的 `Content-Length` 会被改写。

更进一步讲，`Requests` 不会基于定制 `header` 的具体情况改变自己的行为。只不过在最后的请求中，所有的 `header` 信息都会被传递进去。

注意: 所有的 head`er 值必须是 `string、bytestring` 或者 `unicode`。尽管传递 `unicode header` 也是允许的，但不建议这样做。

### 更加复杂的 POST 请求

通常，你想要发送一些编码为表单形式的数据——非常像一个 `HTML` 表单。要实现这个，只需简单地传递一个字典给 `data` 参数。你的数据字典在发出请求时会自动编码为表单形式：

```python
>>> payload = {'key1': 'value1', 'key2': 'value2'}

>>> r = requests.post("http://httpbin.org/post", data=payload)
>>> print(r.text)
{
  ...
  "form": {
    "key2": "value2",
    "key1": "value1"
  },
  ...
}
```

你还可以为 `data` 参数传入一个元组列表。在表单中多个元素使用同一 `key` 的时候，这种方式尤其有效：

```python
>>> payload = (('key1', 'value1'), ('key1', 'value2'))
>>> r = requests.post('http://httpbin.org/post', data=payload)
>>> print(r.text)
{
  ...
  "form": {
    "key1": [
      "value1",
      "value2"
    ]
  },
  ...
```

很多时候你想要发送的数据并非编码为表单形式的。如果你传递一个 `string` 而不是一个 `dict` ，那么数据会被直接发布出去。

例如，`Github API v3` 接受编码为 `JSON` 的 `POST/PATCH` 数据：

```python
>>> import json

>>> url = 'https://api.github.com/some/endpoint'
>>> payload = {'some': 'data'}

>>> r = requests.post(url, data=json.dumps(payload))
```

此处除了可以自行对 `dict` 进行编码，你还可以使用 `json` 参数直接传递，然后它就会被自动编码。这是 `2.4.2 `版的新加功能：

```python
>>> url = 'https://api.github.com/some/endpoint'
>>> payload = {'some': 'data'}

>>> r = requests.post(url, json=payload)
```

### 响应状态码

可以检测响应状态码：

```python
>>> r = requests.get('http://httpbin.org/get')
>>> r.status_code
200
```

为方便引用，`Requests`还附带了一个内置的状态码查询对象：

```
>>> r.status_code == requests.codes.ok
True
```

如果发送了一个错误请求(一个 4XX 客户端错误，或者 5XX 服务器错误响应)，我们可以通过 `Response.raise_for_status()` 来抛出异常：

```
>>> bad_r = requests.get('http://httpbin.org/status/404')
>>> bad_r.status_code
404

>>> bad_r.raise_for_status()
Traceback (most recent call last):
  File "requests/models.py", line 832, in raise_for_status
    raise http_error
requests.exceptions.HTTPError: 404 Client Error
```

如果响应对象 `r` 的 `status_code` 是 `200` ，当我们调用 `raise_for_status()` 时，得到的是：

```python
>>> r.raise_for_status()
None
```

### 响应头

我们可以查看以一个 `Python` 字典形式展示的服务器响应头：

```python
>>> r.headers
{
    'content-encoding': 'gzip',
    'transfer-encoding': 'chunked',
    'connection': 'close',
    'server': 'nginx/1.0.4',
    'x-runtime': '148ms',
    'etag': '"e1ca502697e5c9317743dc078f67693f"',
    'content-type': 'application/json'
}
```

但是这个字典比较特殊：它是仅为 `HTTP` 头部而生的。根据 `RFC 2616`， `HTTP` 头部是大小写不敏感的。

因此，我们可以使用任意大写形式来访问这些响应头字段：

```python
>>> r.headers['Content-Type']
'application/json'

>>> r.headers.get('content-type')
'application/json'
```

它还有一个特殊点，那就是服务器可以多次接受同一 `header`，每次都使用不同的值。

### Cookie

如果某个响应中包含一些 `cookie`，你可以快速访问它们：

```python
>>> url = 'http://www.baidu.com'
>>> r = requests.get(url)
>>> r.cookies
<RequestsCookieJar[Cookie(version=0, name='BDORZ', value='27315', port=None, port_specified=False, domain='.baidu.com', domain_specified=True, domain_initial_dot=True, path='/', path_specified=True, secure=False, expires=1624028209, discard=False, comment=None, comment_url=None, rest={}, rfc2109=False)]>

>>> r.cookies.get_dict()
{'BDORZ': '27315'}
```

要想发送你的`cookies`到服务器，可以使用 `cookies` 参数：

```
>>> url = 'http://httpbin.org/cookies'
>>> cookies = dict(cookies_are='working')

>>> r = requests.get(url, cookies=cookies)
>>> r.text
'{"cookies": {"cookies_are": "working"}}'
```

`Cookie` 的返回对象为 `RequestsCookieJar` ，它的行为和字典类似，但接口更为完整，适合跨域名跨路径使用。你还可以把 `Cookie Jar` 传到 `Requests` 中：

```python
>>> jar = requests.cookies.RequestsCookieJar()
>>> jar.set('tasty_cookie', 'yum', domain='httpbin.org', path='/cookies')
>>> jar.set('gross_cookie', 'blech', domain='httpbin.org', path='/elsewhere')
>>> url = 'http://httpbin.org/cookies'
>>> r = requests.get(url, cookies=jar)
>>> r.text
'{"cookies": {"tasty_cookie": "yum"}}'
```

### 重定向与请求历史

默认情况下，除了 `HEAD, Requests` 会自动处理所有重定向。

可以使用响应对象的 `history` 方法来追踪重定向。

`Response.history` 是一个 `Response` 对象的列表，为了完成请求而创建了这些对象。这个对象列表按照从最老到最近的请求进行排序。

例如，Github 将所有的 `HTTP` 请求重定向到 `HTTPS`：

```python
>>> r = requests.get('http://github.com')

>>> r.url
'https://github.com/'

>>> r.status_code
200

>>> r.history
[<Response [301]>]
```

如果你使用的是`GET、OPTIONS、POST、PUT、PATCH` 或者 `DELETE`，那么你可以通过 `allow_redirects` 参数禁用重定向处理：

```python
>>> r = requests.get('http://github.com', allow_redirects=False)
>>> r.status_code
301
>>> r.history
[]
```

如果你使用了 `HEAD`，你也可以启用重定向：

```python
>>> r = requests.head('http://github.com', allow_redirects=True)
>>> r.url
'https://github.com/'
>>> r.history
[<Response [301]>]
```

### 超时处理

你可以告诉 `requests` 在经过以 `timeout` 参数设定的秒数时间之后停止等待响应。基本上所有的生产代码都应该使用这一参数。如果不使用，你的程序可能会永远失去响应：

```python
>>> requests.get('http://github.com', timeout=0.001)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
requests.exceptions.Timeout: HTTPConnectionPool(host='github.com', port=80): Request timed out. (timeout=0.001)
```

> 注意
 `timeout` 仅对连接过程有效，与响应体的下载无关。 `timeout` 并不是整个下载响应的时间限制，而是如果服务器在 `timeout` 秒内没有应答，将会引发一个异常（更精确地说，是在 `timeout` 秒内没有从基础套接字上接收到任何字节的数据时）。

### 错误与异常

遇到网络问题（如：DNS 查询失败、拒绝连接等）时， `Requests` 会抛出一个 `ConnectionError` 异常。

如果 `HTTP` 请求返回了不成功的状态码， `Response.raise_for_status()` 会抛出一个 `HTTPError` 异常。

若请求超时，则抛出一个 `Timeout` 异常。

若请求超过了设定的最大重定向次数，则会抛出一个 `TooManyRedirects` 异常。

所有`Requests`显式抛出的异常都继承自`requests.exceptions.RequestException` 。


## 高级用法

接下来介绍一些 `requests` 的高级用法。

### 会话对象

会话对象让你能够跨请求保持某些参数。它也会在同一个 `Session` 实例发出的所有请求之间保持 `cookie` ， 期间使用 `urllib3` 的 `connection pooling` 功能。所以如果你向同一主机发送多个请求，底层的 `TCP` 连接将会被重用，从而带来显著的性能提升。

会话对象具有主要的 `Requests API` 的所有方法。

我们来跨请求保持一些 `cookie`:

```python
import requests

# 创建一个会话对象
s = requests.Session()

s.get('http://httpbin.org/cookies/set/sessioncookie/123456789')

r = s.get("https://httpbin.org/cookies")

print(r.text)

# {
#   "cookies": {
#     "sessioncookie": "123456789"
#   }
# }
```

会话也可用来为请求方法提供缺省数据。这是通过为会话对象的属性提供数据来实现的：

```python
s = requests.Session()
s.auth = ('user', 'pass')
s.headers.update({'x-test': 'true'})

# both 'x-test' and 'x-test2' are sent
s.get('http://httpbin.org/headers', headers={'x-test2': 'true'})
```


任何你传递给请求方法的字典都会与已设置会话层数据合并。方法层的参数覆盖会话的参数。

不过需要注意，就算使用了会话，方法级别的参数也不会被跨请求保持。下面的例子只会和第一个请求发送 `cookie` ，而非第二个：

```python
s = requests.Session()

r = s.get('http://httpbin.org/cookies', cookies={'from-my': 'browser'})
print(r.text)
# '{"cookies": {"from-my": "browser"}}'

r = s.get('http://httpbin.org/cookies')
print(r.text)
# '{"cookies": {}}'
```

如果你要手动为会话添加 `cookie` ，就使用 `Cookie utility` 函数 来操纵 `Session.cookies`。

会话还可以用作前后文管理器：

```python
with requests.Session() as s:
    s.get('http://httpbin.org/cookies/set/sessioncookie/123456789')
```

这样就能确保 `with` 区块退出后会话能被关闭，即使发生了异常也一样。

从字典参数中移除一个值
有时你会想省略字典参数中一些会话层的键。要做到这一点，你只需简单地在方法层参数中将那个键的值设置为 `None` ，那个键就会被自动省略掉。

### 请求与响应对象

任何时候进行了类似 `requests.get()` 的调用，你都在做两件主要的事情。其一，你在构建一个 `Request` 对象， 该对象将被发送到某个服务器请求或查询一些资源。其二，一旦 `requests` 得到一个从服务器返回的响应就会产生一个 `Response` 对象。该响应对象包含服务器返回的所有信息，也包含你原来创建的 `Request` 对象。如下是一个简单的请求，从 `Baidu` 的服务器得到一些非常重要的信息：

```python
>>> r = requests.get("https://www.baidu.com")
``` 

如果想访问服务器返回给我们的响应头部信息，可以这样做：

```python
>>> r.headers
{'Cache-Control': 'private, no-cache, no-store, proxy-revalidate, no-transform', 'Connection': 'keep-alive',
        'Content-Encoding': 'gzip', 'Content-Type': 'text/html', 'Date': 'Thu, 17 Jun 2021 15:52:48 GMT',
        'Last-Modified': 'Mon, 23 Jan 2017 13:23:55 GMT', 'Pragma': 'no-cache', 'Server': 'bfe/1.0.8.18',
        'Set-Cookie': 'BDORZ=27315; max-age=86400; domain=.baidu.com; path=/', 'Transfer-Encoding': 'chunked'}
```

然而，如果想得到发送到服务器的请求的头部，我们可以简单地访问该请求，然后是该请求的头部：

```python
>>> r.request.headers
{'User-Agent': 'python-requests/2.25.1', 'Accept-Encoding': 'gzip, deflate', 'Accept': '*/*',
     'Connection': 'keep-alive'}
```

### 准备的请求 （Prepared Request）

当你从 `API` 或者会话调用中收到一个 `Response` 对象时，`request` 属性其实是使用了 `PreparedRequest` 。有时在发送请求之前，你需要对 `body` 或者 `header` （或者别的什么东西）做一些额外处理，下面演示了一个简单的做法：

```python
from requests import Request, Session

s = Session()
req = Request('GET', url,
    data=data,
    headers=header
)
prepped = req.prepare()

# do something with prepped.body
# do something with prepped.headers

resp = s.send(prepped,
    stream=stream,
    verify=verify,
    proxies=proxies,
    cert=cert,
    timeout=timeout
)

print(resp.status_code)
```

由于你没有对 `Request` 对象做什么特殊事情，你立即准备和修改了 `PreparedRequest` 对象，然后把它和别的参数一起发送到 `requests.*` 或者 `Session.*`。

然而，上述代码会失去 `Requests Session` 对象的一些优势， 尤其 `Session` 级别的状态，例如 `cookie` 就不会被应用到你的请求上去。要获取一个带有状态的 `PreparedRequest`， 请用 `Session.prepare_request()` 取代 `Request.prepare()` 的调用，如下所示：

```python
from requests import Request, Session

s = Session()
req = Request('GET',  url,
    data=data
    headers=headers
)

prepped = s.prepare_request(req)

# do something with prepped.body
# do something with prepped.headers

resp = s.send(prepped,
    stream=stream,
    verify=verify,
    proxies=proxies,
    cert=cert,
    timeout=timeout
)

print(resp.status_code)
```

### SSL 证书验证

`Requests` 可以为 `HTTPS` 请求验证 `SSL` 证书，就像 `web` 浏览器一样。 `SSL` 验证默认是开启的，如果证书验证失败，`Requests` 会抛出 `SSLError`:

```python
>>> requests.get('https://requestb.in')
requests.exceptions.SSLError: hostname 'requestb.in' doesn't match either of '*.herokuapp.com', 'herokuapp.com'
```

在该域名上我没有设置 SSL(现在已经设置了，返回的`<Response [200]>`)，所以失败了。但 Github 设置了 SSL:

```python
>>> requests.get('https://github.com', verify=True)
<Response [200]>
```

你可以为 `verify` 传入 `CA_BUNDLE` 文件的路径，或者包含可信任 `CA` 证书文件的文件夹路径：

```python
>>> requests.get('https://github.com', verify='/path/to/certfile')
```

或者将其保持在会话中：

```
s = requests.Session()
s.verify = '/path/to/certfile'
```

**注解**

如果 `verify` 设为文件夹路径，文件夹必须通过 `OpenSSL` 提供的 `c_rehash` 工具处理。

你还可以通过 `REQUESTS_CA_BUNDLE` 环境变量定义可信任 `CA` 列表。

如果你将 `verify` 设置为 `False`，`Requests` 也能忽略对 `SSL` 证书的验证。

```python
>>> requests.get('https://kennethreitz.org', verify=False)
<Response [200]>
```

默认情况下， `verify` 是设置为 `True` 的。选项 `verify` 仅应用于主机证书。

> 对于私有证书，你也可以传递一个 `CA_BUNDLE` 文件的路径给 `verify`。你也可以设置 `# REQUEST_CA_BUNDLE `环境变量。

### 流式上传

`Requests`支持流式上传，这允许你发送大的数据流或文件而无需先把它们读入内存。要使用流式上传，仅需为你的请求体提供一个类文件对象即可：

```python
with open('massive-body') as f:
    requests.post('http://some.url/streamed', data=f)
```

强烈建议你用`二进制模式（binary mode）`打开文件。这是因为 `requests` 可能会为你提供 `header` 中的 `Content-Length`，在这种情况下该值会被设为文件的字节数。如果你用文本模式打开文件，就可能碰到错误。

### POST 多个分块编码的文件

你可以在一个请求中发送多个文件。例如，假设你要上传多个图像文件到一个 `HTML` 表单，使用一个多文件 `field` 叫做 `"images"`:

```html
<input type="file" name="images" multiple="true" required="true"/>
```

要实现，只要把文件设到一个元组的列表中，其中元组结构为 `(form_field_name, file_info)`:

```python
>>> url = 'http://httpbin.org/post'
>>> multiple_files = [
        ('images', ('foo.png', open('foo.png', 'rb'), 'image/png')),
        ('images', ('bar.png', open('bar.png', 'rb'), 'image/png'))]
>>> r = requests.post(url, files=multiple_files)
>>> r.text
{
  ...
  'files': {'images': 'data:image/png;base64,iVBORw ....'}
  'Content-Type': 'multipart/form-data; boundary=3131623adb2043caaeb5538cc7aa0b3a',
  ...
}
```

同样的，建议使用二进制打开文件。

### 代理

如果需要使用代理，你可以通过为任意请求方法提供 `proxies` 参数来配置单个请求:

```python
import requests

proxies = {
  "http": "http://10.10.1.10:3128",
  "https": "http://10.10.1.10:1080",
}

requests.get("http://example.org", proxies=proxies)
```

你也可以通过环境变量 `HTTP_PROXY` 和 `HTTPS_PROXY` 来配置代理。

```python
$ export HTTP_PROXY="http://10.10.1.10:3128"
$ export HTTPS_PROXY="http://10.10.1.10:1080"

$ python
>>> import requests
>>> requests.get("http://example.org")
```

若你的代理需要使用`HTTP Basic Auth`，可以使用 `http://user:password@host/` 语法：

```python
proxies = {
    "http": "http://user:pass@10.10.1.10:3128/",
}
```

要为某个特定的连接方式或者主机设置代理，使用 `scheme://hostname` 作为 `key`， 它会针对指定的主机和连接方式进行匹配。

```python
proxies = {'http://10.20.1.128': 'http://10.10.1.10:5323'}
```

注意，代理 `URL` 必须包含连接方式。

















[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


