---
title: 使用python实现下载器
tags:
  - python
  - download
comments: true
typora-root-url: 使用python实现下载器
date: 2021-09-03 14:41:41
categories:
index_img:
banner_img:
---



## 需要安装的库

```python
tqdm
requests
retry
multitasking
```

`Tqdm` 是一个快速，可扩展的Python进度条，可以在 Python 长循环中添加一个进度提示信息，用户只需要封装任意的迭代器 `tqdm(iterator)`。

`retry`是一个用于错误处理的模块，功能类似`try-except`。

安装：

```python
pip install tqdm requests multitasking retry
```

找个文件下载下：

```python
https://issuecdn.baidupcs.com/issue/netdisk/yunguanjia/BaiduNetdisk_7.2.8.9.exe
```

## 简易版下载器

```python
import requests

url = "https://issuecdn.baidupcs.com/issue/netdisk/yunguanjia/BaiduNetdisk_7.2.8.9.exe"
# 文件名
file_name = "BaiduNetdisk_7.2.8.9.exe"
# 请求头
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.132 Safari/537.36 QIHU 360SE'
}
print('正在下载文件......')
# 发起请求
response = requests.get(url, headers=headers)

content = response.content

# 以wb形式打开文件
with open(file_name, mode='wb') as f:
    # 写入响应内容
    f.write(content)
print(f'文件下载成功！文件名{file_name}')
```

输出结果：

```python
正在下载文件......
文件下载成功！文件名BaiduNetdisk_7.2.8.9.exe
```

打开代码运行目录即可看到文件：`BaiduNetdisk_7.2.8.9.exe`

## 带进度条的文件下载器

### 基础知识

**获取文件大小** 首先要知道文件的大小和每次写入文件的大小。

```python
import requests

url = 'https://issuecdn.baidupcs.com/issue/netdisk/yunguanjia/BaiduNetdisk_7.2.8.9.exe'

headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.132 Safari/537.36 QIHU 360SE'
}

response = requests.head(url, headers=headers)
# 文件大小
file_size = response.headers.get('Content-Length')
if file_size is not None:
    file_size = int(file_size)

print(f"文件的大小:{file_size}")
```

输出结果：

```python
文件的大小:67765560
```

### 分块获取响应内容

一次获取整个文件，进度条的进度就没什么意义，所以分段连续地读取响应知道读取完毕这样才是比较合理的。

```python
import requests

url = 'https://issuecdn.baidupcs.com/issue/netdisk/yunguanjia/BaiduNetdisk_7.2.8.9.exe'

headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.132 Safari/537.36 QIHU 360SE'
}

# 发起head请求，即只获取响应头信息
head = requests.head(url, headers=headers)
# 文件大小，以B为单位
file_size = head.headers.get('Content-Length')
if file_size is not None:
    file_size = int(file_size)
response = requests.get(url, headers=headers, stream=True)
# 一块文件的大小
chunk_size = 2048
# 记录已经读取的文件大小
read = 0
for chunk in response.iter_content(chunk_size=chunk_size):
    read += chunk_size
    read = min(read, file_size)
    print(f'已读取: {read} 总大小: {file_size}')
```

输出结果：

```python
已读取: 2048 总大小: 67765560
已读取: 4096 总大小: 67765560
已读取: 6144 总大小: 67765560
已读取: 8192 总大小: 67765560
...
```

#### **一个 tqdm 的使用例子**

```python
import time
from tqdm import tqdm
total = 100
for _ in tqdm(range(total)):
    time.sleep(0.1)
```

![](Animation.gif)

如果想一次更新指定次数个进度怎么办呢？比如说进度总共是 100 ，每 10 个刷新一次进度。

**实现代码例子如下**

```python
import time
from tqdm import tqdm
# 总进度
total = 100
# 每次刷新的进度
step = 10
# 总共要刷新的次数
flush_count = total//step
bar = tqdm(total=total)
for _ in range(flush_count):
    time.sleep(0.1)
    bar.update(step)
bar.close()
```

以上代码可实现 每 10 个单位刷新一次直进度满 100。

```python
import time
from tqdm import tqdm
# 总进度
total = 100
# 每次刷新的进度
step = 10
# 总共刷新的次数
flush_count = total//step
bar = tqdm(total=total)
for _ in range(flush_count):
    time.sleep(0.1)
    bar.update(step)
bar.close()
```

## 下载器实现进度条

```python
import requests
from tqdm import tqdm

# 目标URL
url = "https://issuecdn.baidupcs.com/issue/netdisk/yunguanjia/BaiduNetdisk_7.2.8.9.exe"
# 文件名
file_name = "BaiduNetdisk_7.2.8.9.exe"
# 请求头
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.132 Safari/537.36 QIHU 360SE'
}
# 发起head请求，获取响应头部信息
head = requests.head(url, headers=headers)
# 文件大小
file_size = head.headers.get('Content-Length')
if file_size is not None:
    file_size = int(file_size)

response = requests.get(url, headers=headers, stream=True)
# 一块文件的大小
chunk_size = 1024
bar = tqdm(total=file_size, desc=f"下载文件{file_name}")
with open(file_name, mode='wb') as f:
    # 写入分块文件
    for chunk in response.iter_content(chunk_size=chunk_size):
        f.write(chunk)
        bar.update(chunk_size)
# 关闭进度条
bar.close()
```

![](Animation-16306593412281.gif)

这样就成功实现了进度条下载器。

## 单进程实现

```python
import requests
from tqdm import tqdm


def download(url: str, filename: str):
    """
    根据文件链和文件名下载文件

    url:文件直链
    filename:文件名（文件路径）
    """

    headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.132 Safari/537.36 QIHU 360SE'
    }

    # 获取响应头部信息
    head = requests.head(url, headers=headers)

    file_size = head.headers.get('Content-Length')
    if file_size is not None:
        file_size = int(file_size)

    response = requests.get(url, headers=headers, stream=True)

    chunk_size = 1024
    bar = tqdm(total=file_size, desc=f'下载文件{filename}')
    with open(filename, mode='wb') as f:
        # 写入分块文件
        for chunk in response.iter_content(chunk_size=chunk_size):
            f.write(chunk)
            bar.update(chunk_size)
    bar.close()


if __name__ == '__main__':
    url = 'https://issuecdn.baidupcs.com/issue/netdisk/yunguanjia/BaiduNetdisk_7.2.8.9.exe'
    filename = 'BaiduNetdisk.exe'
    download(url, filename)
```

## 多线程文件下载器

### 多线程和单线程比较

单线程程序

```python 
import time


def say(number: int):
    print(number)
    time.sleep(0.5)


for i in range(5):
    say(i)
```

输出如下：

```python
0
1
2
3
4
```

这段程序耗时在 2.5 秒左右。 如果我们使用多线程呢？每一次操作均开启一个线程，结果会怎样？

首先，安装一个多线程库，简化写法：

```bash
pip install multitasking
```

使用它之后，我们只需要给自定义的函数前面加上一行代码（装饰器）即可在调用函数时，为被调用的这个函数开启新的线程。

```python
import time
import multiprocessing
import signal

# ctrl + c 终止已经开启的全部线程
import multitasking

signal.signal(signal.SIGINT, multitasking.killall)


@multitasking.task
def say(number: int):
    print(number)
    time.sleep(0.5)


start_time = time.time()
for i in range(5):
    say(i)
# 等待线程全部执行完成
multitasking.wait_for_tasks()
end_time = time.time()
print("耗时：", end_time - start_time, "秒")
```

输出结果：

```python
0
1
2
3
4
耗时： 0.5024490356445312 秒
```

同样是每 0.5 秒输出一个数字，上面的代码因为使用多线程，耗时只有 0.5 秒左右，而之前的单线程版本耗时是 2.5 秒左右。

要实现多线程下载同一个文件，就需要为每一个线程分配属于自己的任务，比如一个100B文件，总共分配五个线程，每个线程下载20B。

```python
# 总大小
total = 100
# 每一块的大小
step = 20
# 分多块
parts = [(start, min(start+step,total)) for start in range(0, total, step)]
print(parts)
```

运行上面的代码，会达到以下输出

```text
[(0, 20), (20, 40), (40, 60), (60, 80), (80, 100)]
```

了能够更好地维护代码，我们可以尝试把它抽取为函数，示例如下

```python
def split(start: int, end: int, step: int) -> list[tuple[int, int]]:
    parts = [(start, min(start+step, end))
             for start in range(0, end, step)]

    return parts


if "__main__" == __name__:
    # 起始位置
    start = 1
    # 终止位置
    total = 102
    # 区间长度
    step = 20
    parts = split(start, total, step)
    print(parts)
```

以上操作是为了后续分段下载文件做准备。

### 下载部分文件

演示如何下载某一部分的文件

```python
import requests

# 起始位置
start = 0
# 终止位置
end = 10000
# 每次读取大小
chunk_size = 1024
# 记录下载位置
seek = start
# 请求头
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.132 Safari/537.36 QIHU 360SE'
}
# 设定下载的范围
headers['Range'] = f'bytes={start}-{end}'
# 下载链接
url = 'https://issuecdn.baidupcs.com/issue/netdisk/yunguanjia/BaiduNetdisk_7.2.8.9.exe'

response = requests.get(url, headers=headers, stream=True)
for chunk in response.iter_content(chunk_size=chunk_size):
    _seek = min(seek + chunk_size, end)
    print(f'下载: {seek}-{_seek}')
    seek = _seek
```

输出情况：

```python
下载: 0-1024
下载: 1024-2048
下载: 2048-3072
下载: 3072-4096
下载: 4096-5120
下载: 5120-6144
下载: 6144-7168
下载: 7168-8192
下载: 8192-9216
下载: 9216-10000
```

### 多线程下载文件

```python
import requests  # 发起网络请求库
import multitasking  # 用于多线程操作
import signal
from tqdm import tqdm  # 显示进度条库
from retry import retry  # 导入retry库方便出错重试

signal.signal(signal.SIGINT, multitasking.killall)

# 请求头
headers = {
    "user-agent": "Mozilla/5.0 (Linux; Android 6.0; Nexus 5 Build/MRA58N) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.159 Mobile Safari/537.36 Edg/92.0.902.84"
}

# 定义1MB等于多少B
MB = 1024 ** 2


def split(start: int, end: int, step: int) -> list[tuple[int, int]]:
    """
    :description 分块
    :param start 起始字节
    :param end  终止字节
    :param step 步长
    """
    parts = [(start, min(start + step, end)) for start in range(0, end, step)]
    return parts


def get_file_size(url: str, raise_error: bool = False) -> int:
    """
    :description 获取文件大小
    :param url
    :param raise_error
    """
    response = requests.head(url)
    file_size = response.headers.get('Content-Length')
    if file_size is None:
        if raise_error is True:
            raise ValueError("该文件不支持多线程下载")
        return file_size
    return int(file_size)


def download(url: str, file_name: str, retry_times: int = 3, each_size=16 * MB) -> None:
    """
    :description    根据文件直链和文件名下载文件
    :param url  文件直链
    :param file_name    文件名
    :param retry_times  重试次数, 默认3次
    :param each_size 每次下载大小
    """
    f = open(file_name, 'wb')
    file_size = get_file_size(url)

    @retry(tries=retry_times)
    @multitasking.task
    def start_download(start: int, end: int) -> None:
        """
        :description 根据文件起止位置下载文件
        :param start   开始位置
        :param end  结束位置
        """
        _headers = headers.copy()
        # 分段下载的核心
        _headers['Range'] = f'bytes={start}-{end}'
        # 发起请求获取流式响应
        response = requests.get(url, headers=_headers, stream=True)
        # 每次读取的流式响应大小
        chunk_size = 1024
        # 暂存获取的响应，后续循环写入
        chunks = []
        for chunk in response.iter_content(chunk_size=chunk_size):
            # 把获取的响应存到列表中
            chunks.append(chunk)
            # 更新进度条
            bar.update(chunk_size)
        f.seek(start)
        for chunk in chunks:
            f.write(chunk)
        # 释放写入的资源
        del chunks

    session = requests.Session()
    # 分块文件如果比文件大，就取文件大小为分块大小
    each_size = min(each_size, file_size)

    # 分块
    parts = split(0, file_size, each_size)
    print(f'分快数：{len(parts)}')
    # 创建进度条
    bar = tqdm(total=file_size, desc=f'下载文件：{file_name}')
    for part in parts:
        start, end = part
        start_download(start, end)
    # 等待全部线程结束
    multitasking.wait_for_tasks()

    f.close()
    bar.close()


if __name__ == '__main__':
    url = 'https://issuecdn.baidupcs.com/issue/netdisk/yunguanjia/BaiduNetdisk_7.2.8.9.exe'
    file_name = 'BaiduNetdisk_7.2.8.9.exe'
    download(url, file_name)
```







[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


