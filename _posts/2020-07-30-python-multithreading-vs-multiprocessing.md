---
layout: post
title: "Python多线程 vs 多进程"
subtitle: ''
author: "heleiqiu"
header-style: text
tags: [Python, 多线程, threading, 多进程, multiprocessing, GIL]
---

> 注：参考[Python多线程多进程那些事儿看这篇就够了~~](https://www.cnblogs.com/-qing-/p/11291581.html)

## 进程、线程
进程和线程简单举例:
对于操作系统来说，一个任务就是一个**进程**（Process），比如打开一个浏览器就是启动一个浏览器进程。  
有些进程还不止同时干一件事，比如Word，它可以同时进行打字、拼写检查、打印等事情。在一个进程内部，要同时干多件事，就需要同时运行多个“子任务”，我们把进程内的这些“子任务”称为**线程**（Thread）。

进程与线程的区别：简而言之，一个程序至少有一个进程，一个进程至少有一个线程。进程就是一个应用程序在处理机上的一次执行过程，它是一个动态的概念，而线程是进程中的一部分，进程包含多个线程在运行。多线程可以共享全局变量，多进程不能。多线程中，所有子线程的进程号相同；多进程中，不同的子进程进程号不同。

### 1. 并行和并发

- 并行处理：是计算机系统中能同时执行两个或更多个处理的一种计算方法。并行处理可同时工作于同一程序的不同方面。并行处理的主要目的是节省大型和复杂问题的解决时间。  
- 并发处理：指一个时间段中有几个程序都处于已启动运行到运行完毕之间，且这几个程序都是在同一个处理机(CPU)上运行，但任一个时刻点上只有一个程序在处理机(CPU)上运行。

### 2. 同步与异步
- 同步：指一个进程在执行某个请求的时候，若该请求需要一段时间才能返回信息，那么这个进程将会一直等待下去，直到收到返回信息才继续执行下去。  
- 异步：指进程不需要一直等待下去，而是继续执行下面的操作，不管其他进程的状态，当有消息返回时系统会通知进程进行处理，这样可以提高执行效率。

为了高效率的执行，就有了并发编程、多线程、多进程等概念，这里就要提一下“GIL(全局解释锁：Global Interpreter Lock)”了

### 3. GIL

首先需要明确的一点是 GIL 并不是Python的特性，它是在实现Python解析器(CPython)时所引入的一个概念，所以Python完全可以不依赖于GIL。那么CPython实现中的GIL又是什么呢？使用Python多线程的人都知道，Python中由于GIL的存在，在多线程时并没有真正的进行多线程计算。GIL说白了就是伪多线程，一个线程运行其他线程阻塞，使你的多线程代码不是同时执行，而是交替执行。

下面用代码来说明GIL的多线程是伪多线程。
```python
# 单线程执行代码
from threading import Thread
import time
 
def my_counter():
    i = 0
    for _ in range(100000000):
        i = i + 1
    return True
 
def main():
    thread_array = {}
    start_time = time.time()
    for tid in range(2):
        t = Thread(target=my_counter)
        t.start()
        t.join()
    end_time = time.time()
    print("Total time: {}".format(end_time - start_time))
 
if __name__ == '__main__':
    main()
```
> 结果: Total time: 26.5897

```python
# 两个线程并发执行代码
from threading import Thread
import time
 
def my_counter():
    i = 0
    for _ in range(100000000):
        i = i + 1
    return True
 
def main():
    thread_array = {}
    start_time = time.time()
    for tid in range(2):
        t = Thread(target=my_counter)
        t.start()
        thread_array[tid] = t
    for i in range(2):
        thread_array[i].join()
    end_time = time.time()
    print("Total time: {}".format(end_time - start_time))
 
if __name__ == '__main__':
    main()
```
> 结果: Total time: 39.5884

可以看到多线程反而慢了十几秒。但并不是说Python多线程完全无用，其多用于IO密集型的应用。

### 4. 应用类型
- IO密集型：对于IO密集型的应用，涉及到网络、磁盘IO的任务都是IO密集型任务，大多消耗都是硬盘读写和网络传输的消耗。  
- 计算密集型 ：计算密集型，顾名思义就是应用需要非常多的CPU计算资源，在计算密集型任务的特点是要进行大量的计算，消耗CPU资源，比如计算圆周率、对视频进行高清解码等等，全靠CPU的运算能力。

那么GIL多线程的不足，其实是对于计算密集型的不足，这个解决可以利用多进程进行解决，而对于IO密集型的任务，仍可以使用多线程进行提升效率。