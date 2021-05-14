---
title: python多线程基本原理
tags: 
- python
- 多线程
- threading
comments: true
toc: true
only:
- home
- category
- tag
date: 2021-05-1303:42:12
categories: Python
pic:
---


## 多线程的含义

说起多线程，就不得不先说什么是线程。然而想要弄明白什么是线程，又不得不先说什么是进程。

进程可以理解为是一个可以独立运行的程序单位，比如打开一个浏览器，这就开启了一个浏览器进程；打开一个文本编辑器，这就开启了一个文本编辑器进程。但一个进程中是可以同时处理很多事情的，比如在浏览器中，可以在多个选项卡中打开多个页面，有的页面在播放音乐，有的页面在播放视频，有的网页在播放动画，它们可以同时运行，互不干扰。为什么能同时做到同时运行这么多的任务呢？这里就需要引出线程的概念了，其实这一个个任务，实际上就对应着一个个线程的执行。

而进程呢？它就是**线程的集合**，进程就是由一个或多个线程构成的，线程是操作系统进行运算调度的最小单位，是进程中的一个最小运行单元。比如上面所说的浏览器进程，其中的播放音乐就是一个线程，播放视频也是一个线程，当然其中还有很多其他的线程在同时运行，这些线程的并发或并行执行最后使得整个浏览器可以同时运行这么多的任务。

了解了线程的概念，多线程就很容易理解了，多线程就是一个进程中同时执行多个线程，前面所说的浏览器的情景就是典型的多线程执行。

## 并发和并行

并说到多进程和多线程，这里就需要再讲解两个概念，那就是并发和并行。知道，一个程序在计算机中运行，其底层是处理器通过运行一条条的指令来实现的。

并发，英文叫作`concurrency`。它是指同一时刻只能有一条指令执行，但是多个线程的对应的指令被快速轮换地执行。比如一个处理器，它先执行线程`A`的指令一段时间，再执行线程`B`的指令一段时间，再切回到线程`A`执行一段时间。

由于处理器执行指令的速度和切换的速度非常非常快，人完全感知不到计算机在这个过程中有多个线程切换上下文执行的操作，这就使得宏观上看起来多个线程在同时运行。但微观上只是这个处理器在连续不断地在多个线程之间切换和执行，每个线程的执行一定会占用这个处理器一个时间片段，同一时刻，其实只有一个线程在执行。

并行，英文叫作`parallel`。它是指同一时刻，有多条指令在多个处理器上同时执行，并行必须要依赖于多个处理器。不论是从宏观上还是微观上，多个线程都是在同一时刻一起执行的。

并行只能在多处理器系统中存在，如果的计算机处理器只有一个核，那就不可能实现并行。而并发在单处理器和多处理器系统中都是可以存在的，因为仅靠一个核，就可以实现并发。

举个例子，比如系统处理器需要同时运行多个线程。如果系统处理器只有一个核，那它只能通过并发的方式来运行这些线程。如果系统处理器有多个核，当一个核在执行一个线程时，另一个核可以执行另一个线程，这样这两个线程就实现了并行执行，当然其他的线程也可能和另外的线程处在同一个核上执行，它们之间就是并发执行。具体的执行方式，就取决于操作系统的调度了。

## 多线程适用场景

多在一个程序进程中，有一些操作是比较耗时或者需要等待的，比如等待数据库的查询结果的返回，等待网页结果的响应。如果使用单线程，处理器必须要等到这些操作完成之后才能继续往下执行其他操作，而这个线程在等待的过程中，处理器明显是可以来执行其他的操作的。如果使用多线程，处理器就可以在某个线程等待的时候，去执行其他的线程，从而从整体上提高执行效率。

像上述场景，线程在执行过程中很多情况下是需要等待的。比如网络爬虫就是一个非常典型的例子，爬虫在向服务器发起请求之后，有一段时间必须要等待服务器的响应返回，这种任务就属于`IO`密集型任务。对于这种任务，如果启用多线程，处理器就可以在某个线程等待的过程中去处理其他的任务，从而提高整体的爬取效率。

但并不是所有的任务都是`IO`密集型任务，还有一种任务叫作计算密集型任务，也可以称之为`CPU`密集型任务。顾名思义，就是任务的运行一直需要处理器的参与。此时如果开启了多线程，一个处理器从一个计算密集型任务切换到切换到另一个计算密集型任务上去，处理器依然不会停下来，始终会忙于计算，这样并不会节省总体的时间，因为需要处理的任务的计算总量是不变的。如果线程数目过多，反而还会在线程切换的过程中多耗费一些时间，整体效率会变低。

所以，如果任务不全是计算密集型任务，可以使用多线程来提高程序整体的执行效率。尤其对于网络爬虫这种IO密集型任务来说，使用多线程会大大提高程序整体的爬取效率。

## Python实现多线程

实在`Python`中，实现多线程的模块叫作`threading`，是`Python`自带的模块。使用`threading`实现多线程的方法。

###Thread直接创建子线程

首先，可以使用`Thread`类来创建一个线程，创建时需要指定`target`参数为运行的方法名称，如果被调用的方法需要传入额外的参数，则可以通过`Thread`的`args`参数来指定。示例如下：

```python
import threading
import time

def target(second):
    print(f'Threading {threading.currentThread().name} id runing')
    print(f'Threading {threading.current_thread().name} sleep {second}s')
    time.sleep(second)
    print(f'Threading {threading.currentThread().name} is ended')

print(f'Threading {threading.currentThread().name} id runing')
for i in [1, 5]:
    thread = threading.Thread(target=target, args=[i])
    thread.start()
print(f'Threading {threading.currentThread().name} is ended')
```

运行结果如下：

```python
Threading MainThread id runing
Threading Thread-1 id runing
Threading Thread-1 sleep 1s
Threading Thread-2 id runingThreading MainThread is ended

Threading Thread-2 sleep 5s
Threading Thread-1 is ended
Threading Thread-2 is ended
```

在这里首先声明了一个方法，叫作`target`，它接收一个参数为`second`，通过方法的实现可以发现，这个方法其实就是执行了一个`time.sleep`休眠操作，`second`参数就是休眠秒数，其前后都`print`了一些内容，其中线程的名字通过`threading.current_thread().name`来获取出来，如果是主线程的话，其值就是`MainThread`，如果是子线程的话，其值就是`Thread-*`。

然后通过`Thead`类新建了两个线程，`target`参数就是刚才所定义的方法名，`args`以列表的形式传递。两次循环中，这里`i`分别就是`1`和`5`，这样两个线程就分别休眠`1`秒·和5秒，声明完成之后，调用`start`方法即可开始线程的运行。

观察结果可以发现，这里一共产生了三个线程，分别是主线程`MainThread`和两个子线程`Thread-1`、`Thread-2`。另外观察到，主线程首先运行结束，紧接着`Thread-1`、`Thread-2`才接连运行结束，分别间隔了`1`秒和`4`秒。这说明主线程并没有等待子线程运行完毕才结束运行，而是直接退出了，有点不符合常理。如果想要主线程等待子线程运行完毕之后才退出，可以让每个子线程对象都调用下`join`方法，实现如下：

```python
threads = []
for i in [1, 5]:
    thread = threading.Thread(target=target, args=[i])
    threads.append(thread)
    thread.start()
for thread in threads:
    thread.join()
```

运行结果如下：

```python
Threading MainThread id runing
Threading Thread-1 id runing
Threading Thread-1 sleep 1s
Threading Thread-2 id runing
Threading Thread-2 sleep 5s
Threading Thread-1 is ended
Threading Thread-2 is ended
Threading MainThread is ended
```

主线程必须等待子线程都运行结束，主线程才继续运行并结束。

## 继承Thread类创建子线程

另外，也可以通过继承`Thread`类的方式创建一个线程，该线程需要执行的方法写在类的`run`方法里面即可。上面的例子的等价改写为：

```python
import time
import threading

class MyThread(threading.Thread):
    def __init__(self, second):
        threading.Thread.__init__(self)
        self.second = second

    def run(self):
        print(f'Threading {threading.current_thread().name} is runing')
        print(f'Threding {threading.current_thread().name} sleep {self.second}s')
        time.sleep(self.second)
        print(f'Threading {threading.current_thread().name} is ended')

print(f'Threading {threading.current_thread().name} is running')
threads = []
for i in [1, 5]:
    thread = MyThread(i)
    threads.append(thread)
    thread.start()
for thread in threads:
    thread.join()
print(f'Threading {threading.current_thread().name} is ended')
```

运行结果如下：

```python
Threading MainThread id runing
Threading Thread-1 id runing
Threading Thread-1 sleep 1s
Threading Thread-2 id runing
Threading Thread-2 sleep 5s
Threading Thread-1 is ended
Threading Thread-2 is ended
Threading MainThread is ended
```

可以看到，两种实现方式，其运行效果是相同的。

## 守护进程

在线程中有一个叫作守护线程的概念，如果一个线程被设置为守护线程，那么意味着这个线程是“不重要”的，这意味着，如果主线程结束了而该守护线程还没有运行完，那么它将会被强制结束。在`Python`中可以通过`setDaemon`方法来将某个线程设置为守护线程。

示例如下：

```python
Threding MainThread is running
Threading Thread-1 is running
Threding Thread-1 sleep 2s
Threading Thread-2 is running
Threding Thread-2 sleep 5s
Threading MainThread is ended
Threading Thread-1 is ended
```

在这里通过 setDaemon方法将 t2 设置为了守护线程，这样主线程在运行完毕时，t2 线程会随着线程的结束而结束。

运行结果如下：

```python
Threding MainThread is running
Threading Thread-1 is running
Threding Thread-1 sleep 2s
Threading Thread-2 is running
Threding Thread-2 sleep 5s
Threading MainThread is ended
Threading Thread-1 is ended
```

可以看到，没有`Thread-2`打印退出的消息，`Thread-2`随着主线程的退出而退出了。

这里并没有调用`join`方法，如果让`t1`和`t2`都调用`join`方法，主线程就会仍然等待各个子线程执行完毕再退出，不论其是否是守护线程。

## 互斥锁

互在一个进程中的多个线程是共享资源的，比如在一个进程中，有一个全局变量`count`用来计数，声明多个线程，每个线程运行时都给`count`加`1`，代码实现如下：

在这里，声明了`1000`个线程，每个线程都是现取到当前的全局变量`count`值，然后休眠一小段时间，然后对`count`赋予新的值。

按照常理来说，最终的`count`值应该为`1000`。但其实不然。运行结果如下：

```python
Final count: 15
```

最后的结果居然只有`15`，而且多次运行或者换个环境运行结果是不同的.这是为什么呢？因为`count`这个值是共享的，每个线程都可以在执行`temp = count`这行代码时拿到当前`count`的值，但是这些线程中的一些线程可能是并发或者并行执行的，这就导致不同的线程拿到的可能是同一个`count`值，最后导致有些线程的`count`的加`1`操作并没有生效，导致最后的结果偏小。

所以，如果多个线程同时对某个数据进行读取或修改，就会出现不可预料的结果。为了避免这种情况，我们需要对多个线程进行同步，要实现同步，我们可以对需要操作的数据进行**加锁保护**，这里就需要用到`threading.Lock`了。

加锁保护是什么意思呢？就是说，某个线程在对数据进行操作前，需要先加锁，这样其他的线程发现被加锁了之后，就无法继续向下执行，会一直等待锁被释放，只有加锁的线程把锁释放了，其他的线程才能 继续加锁并对数据做修改，修改完了再释放锁。这样可以确保同一时间只有一个线程操作数据，多个线程不会再同时读取和修改同一个数据，这样最后的运行结果就是对的了。

将代码修改为如下内容：

```python
import time
import threading

count = 0

class MyThread(threading.Thread):
    def __init__(self):
        threading.Thread.__init__(self)

    def run(self):
        global count
        lock.acquire()  # 加锁
        temp = count + 1
        time.sleep(0.001)
        count = temp
        lock.release() # 解锁

threads = []
lock = threading.Lock() # 创建一把锁
for _ in range(1000):
    thread = MyThread()
    thread.start()
    threads.append(thread)

for thread in threads:
    thread.join()
print(f'Final count: {count}')

```

运行结果如下：

```python
Final count: 1000
```

## Python多线程的问题

由于`Python`中`GIL`的限制，导致不论是在单核还是多核条件下，在同一时刻只能运行一个线程，导致`Python`多线程无法发挥多核并行的优势。

`GIL`全称为`GlobalInterpreter Lock`，中文翻译为全局解释器锁，其最初设计是出于数据安全而考虑的。

在`Python`多线程下，每个线程的执行方式如下：

- 获取`GIL`执行对应线程的代码
- 释放`GIL`可见，某个线程想要执行，必须先拿到 GIL，我们可以把 GIL看作是通行证，并且在一个 Python进程中，GIL只有一个。拿不到通行证的线程，就不允许执行。这样就会导致，即使是多核条件下，一个 Python 进程下的多个线程，同一时刻也只能执行一个线程。 不过对于爬虫这种 IO 密集型任务来说，这个问题影响并不大。而对于计算密集型任务来说，由于 GIL的存在，多线程总体的运行效率相比可能反而比单线程更低