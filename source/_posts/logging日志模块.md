---
title: logging日志模块
tags:
  - logging
comments: true
toc: true
only:
  - home
  - category
  - tag
date: 2021-06-02 09:34:05
categories: Python标准库
pic:
---

## 基础教程

### 日志级别

| 级别 | 使用场景 |
| :--: | :---: |
| DEBUG | 细节信息，仅当诊断问题时适用 |
| INFO | 程序按预期运行 |
| WARNING | 表明有已经或即将发生的意外 |
| ERROR | 由于严重的问题，程序的某些功能已经不能正常执行 |
| CRITICAL | 严重的错误，表明程序已经不能继续执行 |

默认是WARNING级别，意味着只会追踪该级别以及以上的事件，除非修改日志配置。

simple example:

```python
import logging

logging.warning('警告信息')
logging.info('正常信息')
```

output:

```python
WARNING:root:警告信息
```

INFO信息并没有出现，因为默认级别是WARNING级别。

### 记录日志到文件

```python
import logging
logging.basicConfig(filename='example.log', encoding='utf-8', level=logging.DEBUG)
logging.debug('这个消息会被记录到日志文件中')
logging.info('这个也会被记录')
logging.warning('这个也会被记录')
logging.error('这个也被记录，error')
```

日志信息会记录到`exampe.log`中，3.9版本以后增加了encoding参数，在之前的版本或者没有指定时，编码会使用open()使用的默认值。

对 basicConfig() 的调用应该在 debug() ， info() 等的前面。因为它被设计为一次性的配置，只有第一次调用会进行操作，随后的调用不会产生有效操作。

如果多次运行上述脚本，则连续运行的消息将追加到文件 example.log 。 如果你希望每次运行重新开始，而不是记住先前运行的消息，则可以通过将上例中的调用更改为来指定 filemode 参数:

```python
logging.basicConfig(filename='example.log', filemode='w', level=logging.DEBUG)
```

### 记录变量数据

使用格式字符作为事件描述信息，附加传入变量数据作为参数。

```python
import logging

logging.warning('打印变量1：%s，打印变量2：%s', '100', '1000')
```

output:

```python
WARNING:root:打印变量1：100，打印变量2：1000
```

### 更改显示消息的格式

example:

```python
import logging

logging.basicConfig(format='%(levelname)s:%(message)s', level=logging.DEBUG)
logging.debug("这是debug信息")
logging.info("这是info信息")
logging.warning("这是warning信息")
```

output:

```python
DEBUG:这是debug信息
INFO:这是info信息      
WARNING:这是warning信息
```

### 在消息中显示日期/时间

显示事件的日期和时间，在格式字符串中使用`'%(asctime)s'`。

```python
import logging

logging.basicConfig(format='%(asctime)s %(message)s', level=logging.DEBUG)
logging.warning('这是警告信息')
logging.info('这是提示信息')
```

output:

```python
2021-06-02 10:52:34,738 这是警告信息
2021-06-02 10:52:34,738 这是提示信息
```

`datafmt`参数可以控制时间的格式。

```python
import logging

logging.basicConfig(format='%(asctime)s %(message)s', level=logging.DEBUG, datefmt='%Y-%m-%d %I:%M:%S %p')
logging.warning('这是警告信息')
logging.info('这是提示信息')
```

output:

```python
2021-06-02 10:55:55 AM 这是警告信息
2021-06-02 10:55:55 AM 这是提示信息
```

`datefmt` 参数的格式与 `time.strftime()` 支持的格式相同。

## 进阶教程

日志库采用模块化的方法，提供了几类组件：记录器、处理器、过滤器和格式器。

- 记录器暴露了应用程序代码直接使用的接口。
- 处理器将日志记录（由记录器创建）发送到适当的目标。
- 过滤器提供了更精细的附加功能，用于确定要输出的日志记录。
- 格式器指定最终输出中日志记录的样式。

通过调用 `Logger` 类（以下称为 loggers ， 记录器）的实例来执行日志记录。 每个实例都有一个名称，它们在概念上以点（句点）作为分隔符排列在命名空间的层次结构中。 例如，名为 'scan' 的记录器是记录器 'scan.text' ，'scan.html' 和 'scan.pdf' 的父级。 记录器名称可以是你想要的任何名称，并指示记录消息源自的应用程序区域。

良好实践是在使用日志记录模块中使用模块记录器，命名如下：

```python
logger = logging.getLogger(__name__)
```

记录器层次结构的根称为根记录器。 这是函数 `debug()` 、 `info()` 、 `warning()` 、 `error()` 和 `critical() `使用的记录器，它们就是调用了根记录器的同名方法。函数和方法具有相同的签名。根记录器的名称在输出中打印为 `'root'` 。

可以将消息记录到不同的地方。 软件包中的支持包含，用于将日志消息写入文件、 HTTP GET/POST 位置、通过 `SMTP` 发送电子邮件、通用套接字、队列或特定于操作系统的日志记录机制（如 `syslog` 或 Windows NT 事件日志）。 目标由 `handler` 类提供。 如果你有任何内置处理器类未满足的特殊要求，则可以创建自己的日志目标类。

默认情况下，没有为任何日志消息设置目标。 你可以使用 `basicConfig()` 指定目标（例如控制台或文件），如教程示例中所示。 如果你调用函数 debug() 、 info() 、 warning() 、 error() 和 critical() ，它们将检查是否有设置目标；如果没有设置，将在委托给根记录器进行实际的消息输出之前设置目标为控制台（ sys.stderr ）并设置显示消息的默认格式。

由 `basicConfig()` 设置的消息默认格式为：

```python
severity:logger name:message
```

可以通过使用 `format` 参数将格式字符串传递给 `basicConfig()` 来更改此设置。

### 记录流程

记录器和处理器中的日志事件信息流程如下图所示。

![]()

### 记录器

`Logger` 对象有三重任务。首先，它们向应用程序代码公开了几种方法，以便应用程序可以在运行时记录消息。其次，记录器对象根据严重性（默认过滤工具）或过滤器对象确定要处理的日志消息。第三，记录器对象将相关的日志消息传递给所有感兴趣的日志处理器。

这些是最常见的配置方法：

- Logger.setLevel() 指定记录器将处理的最低严重性日志消息，其中 debug 是最低内置严重性级别， critical 是最高内置严重性级别。 例如，如果严重性级别为 INFO ，则记录器将仅处理 INFO 、 WARNING 、 ERROR 和 CRITICAL 消息，并将忽略 DEBUG 消息。
- Logger.addHandler() 和 Logger.removeHandler() 从记录器对象中添加和删除处理器对象。处理器在以下内容中有更详细的介绍 处理器 。
- Logger.addFilter() 和 Logger.removeFilter() 可以添加或移除记录器对象中的过滤器。 过滤器对象 包含更多的过滤器细节。

### 处理器

`Handler` 对象负责将适当的日志消息（基于日志消息的严重性）分派给处理器的指定目标。 `Logger` 对象可以使用 `addHandler()` 方法向自己添加零个或多个处理器对象。作为示例场景，应用程序可能希望将所有日志消息发送到日志文件，将错误或更高的所有日志消息发送到标准输出，以及将所有关键消息发送至一个邮件地址。 此方案需要三个单独的处理器，其中每个处理器负责将特定严重性的消息发送到特定位置。

常用的配置方法：

- setLevel() 方法，就像在记录器对象中一样，指定将被分派到适当目标的最低严重性。为什么有两个 setLevel() 方法？记录器中设置的级别确定将传递给其处理器的消息的严重性。每个处理器中设置的级别确定该处理器将发送哪些消息。
- setFormatter() 选择一个该处理器使用的 Formatter 对象。
- addFilter() 和 removeFilter() 分别在处理器上配置和取消配置过滤器对象。

### 格式器

格式化器对象配置日志消息的最终顺序、结构和内容。与 `logging.Handler` 类不同，应用程序代码可以实例化格式器类，但如果应用程序需要特殊行为，则可能会对格式化器进行子类化定制。构造函数有三个可选参数 —— `消息格式字符串`、`日期格式字符串`和`样式指示符`。

`logging.Formatter.__init__(fmt=None, datefmt=None, style='%')`

如果没有消息格式字符串，则默认使用原始消息。如果没有日期格式字符串，则默认日期格式为：

```python
%Y-%m-%d %H:%M:%S
```

最后加上毫秒数。 `style` 是 `％`，`'{'` 或 `'$'` 之一。 如果未指定，则将使用 `'％'`。

在 `3.2 `版更改: 添加 `style` 形参。

以下消息格式字符串将以人类可读的格式记录时间、消息的严重性以及消息的内容，按此顺序:

```python
'%(asctime)s - %(levelname)s - %(message)s'
```

格式器通过用户可配置的函数将记录的创建时间转换为元组。 默认情况下，使用 `time.localtime()`；要为特定格式器实例更改此项，请将实例的 `converter` 属性设置为与 `time.localtime()` 或 `time.gmtime()` 具有相同签名的函数。

### 配置日志记录

开发者可以通过三种方式配置日志记录：

1. 使用调用上面列出的配置方法的 `Python` 代码显式创建记录器、处理器和格式器。
2. 创建日志配置文件并使用 `fileConfig()` 函数读取它。
3. 创建配置信息字典并将其传递给 `dictConfig()` 函数。

具体可参考官方文档。

[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>