---
title: asyncio异步I/O模块
tags:
  - asyncio
  - python
comments: true
toc: true
only:
  - home
  - category
  - tag
date: 2021-06-03 07:40:51
categories: Python标准库
pic:
---

`asyncio` 是用来编写并发代码的库，使用 `async/await` 语法。

`asyncio` 被用作多个提供高性能 `Python` 异步框架的基础，包括网络和网站服务，数据库连接库，分布式任务队列等等。

`asyncio` 往往是构建 `IO` 密集型和高层级 `结构化` 网络代码的最佳选择。

`asyncio` 提供一组 `高层级 API` 用于:

- 并发地运行 `Python` 协程 并对其执行过程实现完全控制;
- 执行网络IO和IPC;
- 控制子进程;
- 通过队列 实现分布式任务;
- 同步并发代码;

此外，还有一些 `低层级 API` 以支持库和框架的开发者实现:

- 创建和管理事件循环，以提供异步 API 用于`网络化`, 运行`子进程`，处理 `OS信号` 等等;
- 使用 `transports` 实现高效率协议;
- 通过 `async/await` 语法 `桥接` 基于回调的库和代码。

## 协程与任务

### 协程

协程 通过 `async/await` 语法进行声明，是编写 `asyncio` 应用的推荐方式。

```python
 import asyncio

async def main():
    print('hello')
    await asyncio.sleep(1)
    print('world')

asyncio.run(main())
```

输出hello后停止1秒后输出world。

直接调用协程并不会执行，比如这里直接`main()`是运行不了的，吃屎的`main`不是函数而是一个协程对象。

运行协程，有三种机制：

- `asyncio.run()`函数来运行最高层级的入口点“main”函数。
- 等待一个协程，下面的代码等待1秒后输出hello，再等待2秒后输出world。

```python
import asyncio
import time

async def say_after(delay, what):
    await asyncio.sleep(delay)
    print(f"%s at {time.strftime('%X')}"%what)

async def main():
    print(f"startd at {time.strftime('%X')}")
    await say_after(1, "hello")
    await say_after(2, "world")
    print(f"ended at {time.strftime('%X')}")

asyncio.run(main())
```

输出结果：

```python
startd at 15:04:17
hello at 15:04:18
world at 15:04:20
ended at 15:04:20
```

- `asyncio.create_task()`函数用来并发运行作为 `asyncio` 任务的多个协程。

```python
import asyncio
import time

async def say_after(delay, what):
    await asyncio.sleep(delay)
    print(f"%s at {time.strftime('%X')}"%what)

async def main():
    task1 = asyncio.create_task(say_after(1, "hello"))
    task2 = asyncio.create_task(say_after(2, "world"))

    print(f"startd at {time.strftime('%X')}")
    await task1
    await task2
    print(f"ended at {time.strftime('%X')}")

asyncio.run(main())
```

输出结果，注意，输出显示代码段的运行时间比之前快了 1 秒:：

```python
startd at 15:17:35
hello at 15:17:36
world at 15:17:37
ended at 15:17:37
```

### 可等待对象

如果一个对象可以在 `await` 语句中使用，那么它就是 `可等待` 对象。

`可等待` 对象有三种主要类型: `协程`, `任务` 和 `Future`。

#### 协程

`Python` 协程属于 `可等待` 对象，因此可以在其他协程中被等待:

```python
import asyncio

async def nested():
    return 42

async def main():
    nested()     # 不会直接执行

    print(await nested()) # 打印42

asyncio.run(main())
```

{% colorpanel info 重要 %}

"协程" 可用来表示两个紧密关联的概念:
- 协程函数: 定义形式为 `async def` 的函数;

- 协程对象: 调用 `协程函数` 所返回的对象。

{% endcolorpanel %}

#### 任务

*任务* 被用来“并行的”调度协程

当一个协程通过 `asyncio.create_task()` 等函数被封装为一个 *任务*，该协程会被自动调度执行:

```python
import asyncio

async def hello():
    print('hello World')

async def main():
    task = asyncio.create_task(hello())
    await task

asyncio.run(main())

>>hello World
```

### 运行 asyncio 程序

`asyncio.run(coro, *, debug=False)`

执行 `coroutine coro` 并返回结果。

此函数会运行传入的协程，负责管理 `asyncio` 事件循环，终结异步生成器，并关闭线程池。

当有其他 `asyncio` 事件循环在同一线程中运行时，此函数不能被调用。

如果 `debug` 为 `True`，事件循环将以调试模式运行。

此函数总是会创建一个新的事件循环并在结束时关闭之。它应当被用作 `asyncio` 程序的主入口点，理想情况下应当只被调用一次。

示例:

```python
async def main():
    await asyncio.sleep(1)
    print('hello')

asyncio.run(main())
```

### 创建任务

`asyncio.create_task(coro, *, name=None)`

将 `coro` 协程 封装为一个 `Task` 并调度其执行。返回 `Task` 对象。

`name` 不为 `None`，它将使用 `Task.set_name()` 来设为任务的名称。

该任务会在 `get_running_loop()` 返回的循环中执行，如果当前线程没有在运行的循环则会引发 `RuntimeError`。

此函数 在 `Python 3.7` 中被加入。在 `Python 3.7 `之前，可以改用低层级的 `asyncio.ensure_future()` 函数。

```python
async def coro():
    print("hello")

# python3.7
task = asyncio.create_task(coro())
...

# python3.7以前
task = asyncio.ensure_future(coro())
```

### 休眠

`coroutine asyncio.sleep(delay, result=None, *, loop=None)`

阻塞 `delay` 指定的秒数。

如果指定了 `result`，则当协程完成时将其返回给调用者。

`sleep()` 总是会挂起当前任务，以允许其他任务运行。

将 `delay` 设为 `0` 将提供一个经优化的路径以允许其他任务运行。 这可供长期间运行的函数使用以避免在函数调用的全过程中阻塞事件循环。

```python
import asyncio
import datetime

async def display_date():
    loop = asyncio.get_running_loop()
    end_time = loop.time() + 5.0
    while True:
        print(datetime.datetime.now())
        if (loop.time() + 1.0) >= end_time:
            break
        await asyncio.sleep(1)

asyncio.run(display_date())
```

输出结果：

```python
2021-06-04 16:05:16.005762
2021-06-04 16:05:17.010811
2021-06-04 16:05:18.022837
2021-06-04 16:05:19.038106
2021-06-04 16:05:20.038414
```

### 并发运行任务

`awaitable asyncio.gather(*aws, loop=None, return_exceptions=False)`

并发 运行 `aws` 序列中的 可等待对象。

如果 `aws` 中的某个可等待对象为协程，它将自动被作为一个任务调度。

如果所有可等待对象都成功完成，结果将是一个由所有返回值聚合而成的列表。结果值的顺序与 `aws` 中可等待对象的顺序一致。

如果 `return_exceptions` 为 `False` (默认)，所引发的首个异常会立即传播给等待 `gather()` 的任务。`aws` 序列中的其他可等待对象 不会被取消 并将继续运行。

如果 `return_exceptions` 为 `True`，异常会和成功的结果一样处理，并聚合至结果列表。

如果 `gather()` 被取消，所有被提交 (尚未完成) 的可等待对象也会 被取消。

如果 `aws` 序列中的任一 `Task` 或 `Future` 对象 被取消，它将被当作引发了 `CancelledError` 一样处理 -- 在此情况下 `gather()` 调用 *不会* 被取消。这是为了防止一个已提交的 `Task/Future` 被取消导致其他 `Tasks/Future` 也被取消。

```python
import asyncio

async def factorial(name, number):
    f = 1
    for i in range(2, number + 1):
        print(f'任务名 {name}: factorial({number}), currently i = {i}...')
        await asyncio.sleep(1)
        f *= i
    print(f"任务 {name}: factorial({number}) = {f}")
    return f

async def main():
    L = await asyncio.gather(
        factorial("A", 2),
        factorial("B", 3),
        factorial("C", 4),
    )
    print(L)

asyncio.run(main())

# output
# 任务名 A: factorial(2), currently i = 2...
# 任务名 B: factorial(3), currently i = 2...
# 任务名 C: factorial(4), currently i = 2...
# 任务 A: factorial(2) = 2
# 任务名 B: factorial(3), currently i = 3...
# 任务名 C: factorial(4), currently i = 3...
# 任务 B: factorial(3) = 6
# 任务名 C: factorial(4), currently i = 4...
# 任务 C: factorial(4) = 24
# [2, 6, 24]
```



