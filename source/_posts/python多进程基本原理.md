---
title: python多进程基本原理
tags:
  - python
  - multiprocessing
comments: true
toc: true
only:
  - home
  - category
  - tag
date: 2021-05-14 20:04:49
categories: 爬虫
pic:
---

## 多进程的含义

多进程（`Process`）是具有一定独立功能的程序关于某个数据集合上的一次运行活动，是系统进行资源分配和调度的一个独立单位。

顾名思义，多进程就是启用多个进程同时运行。由于进程是线程的集合，而且进程是由一个或多个线程构成的，所以多进程的运 行意味着有大于或等于进程数量的线程在运行。

## Python多进程的优势

由于进程中`GIL`的存在，`Python`中的多线程并不能很好地发挥多核优势，一个进程中的多个线程，在同一时刻只能有一个线程运行。

对于多进程来说，每个进程都有属于自己的`GIL`，所以，在多核处理器下，多进程的运行是不会受`GIL`的影响的。因此，多进程能更好地发挥多核的优势。

当然，对于爬虫这种`IO`密集型任务来说，多线程和多进程影响差别并不大。对于计算密集型任务来说，`Python`的多进程相比多线程，其多核运行效率会有成倍的提升。

总的来说，`Python`的多进程整体来看是比多线程更有优势的。所以，在条件允许的情况下，能用多进程就尽量用多进程。

不过值得注意的是，由于进程是系统进行资源分配和调度的一个独立单位，所以各个进程之间的数据是无法共享的，如多个进程无法共享一个全局变量，进程之间的数据共享需要有单独的机制来实现到。

## 多进程的实现

在`Python`中也有内置的库来实现多进程，它就是`multiprocessing`。

`multiprocessing`提供了一系列的组件，如`Process（进程`、`Queue（队列）`、`Semaphore（信号量）`、`Pipe（管道）`、`Lock（锁）`、`Pool（进程池）`等，接下来让我们来了解下它们的使用方法。

### 直接使用Process类

在`multiprocessing`中，每一个进程都用一个`Process`类来表示。它的`API`调用如下：

```python
Process([group [, target [, name [, args [, kwargs]]]]])
```

- `target`表示调用对象，你可以传入方法的名字。
- `args`表示被调用对象的位置参数元组，比如`target`是函数`func`，他有两个参数`m`，`n`，那么`args`就传入`[m, n]`即可。
- `kwargs`表示调用对象的字典。
- `name`是别名，相当于给这个进程取一个名字。
- `group`分组。
  
先用一个实例来感受一下：

```python
import multiprocessing

def process(index):
    print(f'process: {index}')

if __name__ == '__main__':
    for i in range(5):
        p = multiprocessing.Process(target=process, args=(i,))
        p.start()
```

这是一个实现多进程最基础的方式：通过创建`Process`来新建一个子进程，其中`target`参数传入方法名，`args`是方法的参数，是以 元组的形式传入，其和被调用的方法`process`的参数是一一对应的。

> 注意：这里`args `必须要是一个元组，如果只有一个参数，那也要在元组第一个元素后面加一个逗号，如果没有逗号则和单个元素本身没有区别，无法构成元组，导致参数传递出现问题。

创建完进程之后，我们通过调用`start`方法即可启动进程了。运行结果如下：

```python
process: 0
process: 1
process: 2
process: 3
process: 4
```

运行了`5`个子进程，每个进程都调用了`process`方法。`process`方法的`index`参数通过`Process`的`args`传入，分别是`0~4`这`5`个序号，最后打印出来，`5`个子进程运行结束。

由于进程是`Python`中最小的资源分配单元，因此这些进程和线程不同，各个进程之间的数据是不会共享的，每启动一个进程，都会独立分配资源。

在当前`CPU`核数足够的情况下，这些不同的进程会分配给不同的`CPU`核来运行，实现真正的并行执行。

运行结果如下：

```python
CPU number: 4
Child process name: Process-2 id: 12112
Child process name: Process-3 id: 14708
Child process name: Process-1 id: 11208
Child process name: Process-5 id: 9968
Child process name: Process-4 id: 11784
Process ended
process: 0
process: 1
process: 2
process: 3
process: 4
````

通过`cpu_count`成功获取了`CPU`核心的数量：`4`个，不同的机器结果可能不同。通过 `active_children`获取到了当前正在活跃运行的进程列表。然后遍历每个进程，并将它们的*名称*和*进程号*打印出来了，这里进程号直接使用`pid`属性即可获取，进程名称直接通过`name`属性即可获取。

## 继承继Process类

在上面的例子中，创建进程是直接使用`Process`这个类来创建的，这是一种创建进程的方式。不过，创建进程的方式不止这一 种，同样，也可以像线程`Thread`一样来通过继承的方式创建一个进程类，进程的基本操作我们在子类的`run`方法中实现即可。

通过一个实例来看一下：

```python
from multiprocessing import Process
import time

class MyProcess(Process):
    def __init__(self, loop):
        Process.__init__(self)
        self.loop = loop

    def run(self):
        for count in range(self.loop):
            time.sleep(1)
            print(f'Pid: {self.pid} LoopCount: {count}')

if __name__ == '__main__':
    for i in range(2,5):
        p = MyProcess(i)
        p.start()
```

声明了一个构造方法，这个方法接收一个`loop`参数，代表循环次数，并将其设置为全局变量。在`run`方法中，又使用这个`loop`变量循环了`loop`次并打印了当前的进程号和循环次数。

在调用时，用`range`方法得到了`2、3、4`三个数字，并把它们分别初始化了`MyProcess`进程，然后调用`start`方法将进程启动起来。

> 注意：这里进程的执行逻辑需要在`run`方法中实现，启动进程需要调用`start`方法，调用之后`run`方法便会执行。

运行结果如下：

```python
Pid: 13728 LoopCount: 0
Pid: 13560 LoopCount: 0
Pid: 9908 LoopCount: 0 
Pid: 9908 LoopCount: 1
Pid: 13560 LoopCount: 1
Pid: 13728 LoopCount: 1
Pid: 13728 LoopCount: 2
Pid: 9908 LoopCount: 2
Pid: 13728 LoopCount: 3
```

三个进程分别打印出了`2、3、4`条结果，即进程`13560`打印了`2`次结果，进程`9908` 打印了`3`次结果，进程`13728`打印了`4`次结果。

> 注意，这里的进程`pid`代表进程号，不同机器、不同时刻运行结果可能不同。 

通过上面的方式，非常方便地实现了一个进程的定义。为了复用方便，可以把一些方法写在每个进程类里封装好，在使用时直接初始化一个进程类运行即可。

## 守护进程

在多进程中，同样存在守护进程的概念，如果一个进程被设置为守护进程，当父进程结束后，子进程会自动被终止，我们可以通过设置`daemon`属性来控制是否为守护进程。

还是原来的例子，增加了`deamon`属性的设置：

```python
from multiprocessing import Process
import time

class MyProcess(Process):
    def __init__(self, loop):
        Process.__init__(self)
        self.loop = loop

    def run(self):
        for count in range(self.loop):
            time.sleep(1)
            print(f'Pid: {self.pid} LoopCount: {count}')

if __name__ == '__main__':
    for i in range(2,5):
        p = MyProcess(i)
        p.daemon = True
        p.start()
    print('Main process is ended.')
```

运行结果如下：

```python
Main process is ended.
```

结果很简单，因为主进程没有做任何事情，直接输出一句话结束，所以在这时也直接终止了子进程的运行。

这样可以有效防止无控制地生成子进程。这样的写法可以让我们在主进程运行结束后无需额外担心子进程是否关闭，避免了独立子进程的运行。

## 进程等待

上面的运行效果其实不太符合我们预期：主进程运行结束时，子进程（守护进程）也都退出了，子进程什么都没来得及执行。

能不能让所有子进程都执行完了然后再结束呢？当然是可以的，只需要加入`join`方法即可，可以将代码改写如下：

```python
    process = []
    for i in range(2,5):
        p = MyProcess(i)
        process.append(p)
        p.daemon = True
        p.start()
    for p in process:
        p.join()
    print('Main process is ended.')
```

运行结果如下：

```python
Pid: 13220 LoopCount: 0
Pid: 10488 LoopCount: 0
Pid: 4652 LoopCount: 0
Pid: 13220 LoopCount: 1
Pid: 10488 LoopCount: 1
Pid: 4652 LoopCount: 1
Pid: 10488 LoopCount: 2
Pid: 4652 LoopCount: 2
Pid: 4652 LoopCount: 3
Main process is ended.
```

在调用`start`和`join`方法后，父进程就可以等待所有子进程都执行完毕后，再打印出结束的结果。

默认情况下，`join`是无限期的。也就是说，如果有子进程没有运行完毕，主进程会一直等待。这种情况下，如果子进程出现问题陷入了死循环，主进程也会无限等待下去。怎么解决这个问题呢？可以给`join`方法传递一个超时参数，代表最长等待秒数。如果子进程没有在这个指定秒数之内完成，会被强制返回，主进程不再会等待。也就是说这个参数设置了主进程等待该子进程的最长时间。

例如这里传入`1`，代表最长等待`1`秒，代码改写如下：

```python
Pid: 11936 LoopCount: 0
Pid: 968 LoopCount: 0
Pid: 11572 LoopCount: 0
Pid: 11936 LoopCount: 1
Pid: 968 LoopCount: 1
Pid: 11572 LoopCount: 1
Main process is ended.
```

可以看到，有的子进程本来要运行`3`秒，结果运行`1`秒就被强制返回了，由于是守护进程，该子进程被终止了。

