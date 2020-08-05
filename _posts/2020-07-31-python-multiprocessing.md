---
layout: post
title: "Python多进程"
subtitle: ''
author: "heleiqiu"
header-style: text
tags: [Python, 多进程, multiprocessing]
---

由于Python多线程的弊端和GIL，适合IO密集型，那么对于计算密集型的应用应该怎么办呢？那就是多进程。（每个进程的GIL互不影响，多进程来并行编程。）

### 1. multiprocessing模块

multiprocessing模块提供了一个Process类来代表一个进程对象。Process类描述如下：  
`Process([group [, target [, name [, args [, kwargs]]]]])`  

|  函数    |  用途  |  
|  -----   | ----  |  
| `group`  | 参数未使用，值始终为`None` |   
| `target`| 表示调用对象，即子进程要执行的任务  |  
| `args`  | 表示调用对象的位置参数元组，`args=(1,2,'egon',)`  |  
| `kwargs`| 表示调用对象的字典，`kwargs={'name':'egon','age':18}`|    
| `name`  | 为子进程的名称|  

- 一些常用的函数

|  函数    |  用途  |  
|  -----   | ----  |  
| `p.start()`| 启动进程，并调用该子进程中的`p.run()`|   
| `p.run()`| 进程启动时运行的方法，正是它去调用`target`指定的函数，我们自定义类的类中一定要实现该方法 |   
| `p.terminate()` | 强制终止进程`p`，不会进行任何清理操作，如果`p`创建了子进程，该子进程就成了僵尸进程，使用该方法需要特别小心这种情况。如果p还保存了一个锁那么也将不会被释放，进而导致死锁|  
| `p.is_alive()`| 如果`p`仍然运行，返回`True`|  
| `p.join([timeout])`| 主线程等待p终止（强调：是主线程处于等的状态，而`p`是处于运行的状态）。`timeout`是可选的超时时间，需要强调的是，`p.join`只能`join`住`start`开启的进程，而不能`join`住`run`开启的进程|  

- 一些属性
   
|  属性   |  用途  |  
|  -----   | ----  |  
| `p.daemon`| 默认值为`False`，如果设为`True`，代表`p`为后台运行的守护进程，当`p`的父进程终止时，`p`也随之终止，并且设定为`True`后，`p`不能创建自己的新进程，必须在`p.start()`之前设置|  
| `p.name`| 进程的名称|  
| `p.pid`| 进程的`pid`|   
| `p.exitcode`| 进程在运行时为`None`、如果为`–N`，表示被信号`N`结束(了解即可)|  
| `p.authkey`| 进程的身份验证键，默认是由`os.urandom()`随机生成的32字符的字符串。这个键的用途是为涉及网络连接的底层进程间通信提供安全性，这类连接只有在具有相同的身份验证键时才能成功）|  

```python
# 启动子进程
from multiprocessing import Process
import os

# 子进程要执行的代码
def run_proc(name):
    print('Run Subprocess %s (%s)...' % (name, os.getpid()))

if __name__=='__main__':
    print('Parent process %s.' % os.getpid())
    p = Process(target=run_proc, args=('test',))
    print('Subprocess will start.')
    p.start()
    p.join()
    print('Subprocess end.')
```

执行结果如下：  
> Parent process 4424.  
Subprocess will start.  
Run Subprocess test (14896)...  
Subprocess end.  

创建子进程时，只需要传入一个执行函数和函数的参数，创建一个Process实例，用start()方法启动。join()方法可以等待子进程结束后再继续往下运行，通常用于进程间的同步。



### 2. 进程池

如果要启动大量的子进程，可以用进程池的方式批量创建子进程：

```python
from multiprocessing import Pool
import os, time, random

def long_time_task(name):
    print('Run task %s (%s)...' % (name, os.getpid()))
    start = time.time()
    time.sleep(random.random() * 3)
    end = time.time()
    print('Task %s runs %0.2f seconds.' % (name, (end - start)))

if __name__=='__main__':
    print('Parent process %s.' % os.getpid())
    p = Pool(4)
    for i in range(4):
        p.apply_async(long_time_task, args=(i,))
    print('Waiting for all subprocesses done...')
    p.close()
    p.join()
    print('All subprocesses done.')
```
 
 > Parent process 15148.  
Waiting for all subprocesses done...  
Run task 0 (2400)...  
Run task 1 (13564)...  
Run task 2 (6820)...  
Run task 3 (15068)...  
Task 1 runs 0.40 seconds.  
Task 3 runs 0.78 seconds.  
Task 2 runs 1.72 seconds.  
Task 0 runs 1.84 seconds.   
All subprocesses done.  

对Pool对象调用join()方法会等待所有子进程执行完毕，调用join()之前必须先调用close()，调用close()之后就不能继续添加新的Process了。   
当某个进程fork一个子进程后，该进程必须要调用`wait`等待子进程结束发送的`sigchld`信号，对子进程进行资源回收等相关工作，否则，子进程会成为僵死进程，被`init`收养。所以，在`multiprocessing.Process`实例化一个对象之后，该对象有必要调用`join`方法，因为在`join`方法中完成了对底层`wait`的处理。

### 3. 进程间通信

Process需要通信，操作系统提供了很多机制来实现进程间的通信。Python的multiprocessing模块包装了底层的机制，提供了Queue、Pipes等多种方式来交换数据。我们以Queue为例，在父进程中创建两个子进程，一个往Queue里写数据，一个从Queue里读数据：

```python
from multiprocessing import Process, Queue
import os, time, random

# 写数据进程执行的代码:
def write(q):
    print('Process to write: %s' % os.getpid())
    for value in ['A', 'B', 'C']:
        print('Put %s to queue...' % value)
        q.put(value)
        time.sleep(random.random())

# 读数据进程执行的代码:
def read(q):
    print('Process to read: %s' % os.getpid())
    while True:
        value = q.get(True)
        print('Get %s from queue.' % value)

if __name__=='__main__':
    # 父进程创建Queue，并传给各个子进程：
    q = Queue()
    pw = Process(target=write, args=(q,))
    pr = Process(target=read, args=(q,))
    # 启动子进程pw，写入:
    pw.start()
    # 启动子进程pr，读取:
    pr.start()
    # 等待pw结束:
    pw.join()
    # pr进程里是死循环，无法等待其结束，只能强行终止:
    pr.terminate()
```

> Process to write: 14132  
Put A to queue...  
Process to read: 14664  
Get A from queue.  
Put B to queue...  
Get B from queue.  
Put C to queue...  
Get C from queue.  