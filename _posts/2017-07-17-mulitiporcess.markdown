---
layout:     post
title:      "PYTHON多进程总结"
subtitle:   "python效率提高的多种方法"
date:       2017-07-17
author:     "Diane"
header-img: "img/post-bg-js-module.jpg"
tags:

---

## 前言

[这是一个简单而又痛苦的开始...](#build) 

继续总结python多进程的内容。

<p id = "build"></p>
---

######正文
<br>
由于GIL的原因，在python中涉及到多线程处理问题都需要使用多进程。
<br>
Unix/Linux操作系统提供了一个fork()系统调用，它非常特殊。普通的函数调用，调用一次，返回一次，但是fork()调用一次，返回两次，因为操作系统自动把当前进程（称为父进程）复制了一份（称为子进程），然后，分别在父进程和子进程内返回。
<br>
子进程永远返回0，而父进程返回子进程的ID。这样做的理由是，一个父进程可以fork出很多子进程，所以，父进程要记下每个子进程的ID，而子进程只需要调用getppid()就可以拿到父进程的ID。
<br>
Python的os模块封装了常见的系统调用，其中就包括fork，可以在Python程序中轻松创建子进程。
<br>
由于Windows没有fork调用，上面的代码在Windows上无法运行。由于Mac系统是基于BSD（Unix的一种）内核，所以，在Mac下运行是没有问题的，推荐大家用Mac学Python！
<br>
有了fork调用，一个进程在接到新任务时就可以复制出一个子进程来处理新任务，常见的Apache服务器就是由父进程监听端口，每当有新的http请求时，就fork出子进程来处理新的http请求。
<br>
如果你打算编写多进程的服务程序，Unix/Linux无疑是正确的选择。由于Windows没有fork调用，难道在Windows上无法用Python编写多进程的程序？<br>

由于Python是跨平台的，自然也应该提供一个跨平台的多进程支持。multiprocessing模块就是跨平台版本的多进程模块。
<br>
创建子进程时，只需要传入一个执行函数和函数的参数，创建一个Process实例，用start()方法启动，这样创建进程比fork()还要简单。join()方法可以等待子进程结束后再继续往下运行，通常用于进程间的同步。
<br>
如果要启动大量的子进程，可以用进程池的方式批量创建子进程。对Pool对象调用join()方法会等待所有子进程执行完毕，调用join()之前必须先调用close()，调用close()之后就不能继续添加新的Process了。
<br>
Process之间肯定是需要通信的，操作系统提供了很多机制来实现进程间的通信。Python的multiprocessing模块包装了底层的机制，提供了Queue、Pipes等多种方式来交换数据。
<br>
在Unix/Linux下，multiprocessing模块封装了fork()调用，使我们不需要关注fork()的细节。由于Windows没有fork调用，因此，multiprocessing需要“模拟”出fork的效果，父进程所有Python对象都必须通过pickle序列化再传到子进程去，所有，如果multiprocessing在Windows下调用失败了，要先考虑是不是pickle失败了。
<br>

######代码字段

```python
import multiprocessing
import time

def worker(interval):
    n = 5
    while n > 0:
        print("The time is {0}".format(time.ctime()))
        time.sleep(interval)
        n -= 1

if __name__ == "__main__":
    p = multiprocessing.Process(target = worker, args = (3,))
    p.start()
    print("p.pid:", p.pid)
    print("p.name:", p.name)
    print("p.is_alive:", p.is_alive())
```

    p.pid: 4548
    p.name: Process-1
    p.is_alive: True
    The time is Mon Jul 17 15:27:33 2017
    The time is Mon Jul 17 15:27:36 2017
    The time is Mon Jul 17 15:27:39 2017
    The time is Mon Jul 17 15:27:42 2017
    The time is Mon Jul 17 15:27:45 2017



```python
import multiprocessing
import time

def worker_1(interval):
    print("worker_1")
    time.sleep(interval)
    print("end worker_1")

def worker_2(interval):
    print("worker_2")
    time.sleep(interval)
    print("end worker_2")

def worker_3(interval):
    print("worker_3")
    time.sleep(interval)
    print("end worker_3")

if __name__ == "__main__":
    p1 = multiprocessing.Process(target = worker_1, args = (2,))
    p2 = multiprocessing.Process(target = worker_2, args = (3,))
    p3 = multiprocessing.Process(target = worker_3, args = (4,))

    p1.start()
    p2.start()
    p3.start()

    print("The number of CPU is:" + str(multiprocessing.cpu_count()))
    for p in multiprocessing.active_children():
        print("child   p.name:" + p.name + "\tp.id" + str(p.pid))
    print("END!!!!!!!!!!!!!!!!!")
```

    The number of CPU is:4
    child   p.name:Process-4	p.id4560
    child   p.name:Process-3	p.id4559
    child   p.name:Process-2	p.id4558
    END!!!!!!!!!!!!!!!!!
    worker_2
    worker_1
    worker_3
    end worker_1
    end worker_2
    end worker_3



```python
import multiprocessing
import time

class ClockProcess(multiprocessing.Process):
    def __init__(self, interval):
        multiprocessing.Process.__init__(self)
        self.interval = interval

    def run(self):
        n = 5
        while n > 0:
            print("the time is {0}".format(time.ctime()))
            time.sleep(self.interval)
            n -= 1

if __name__ == '__main__':
    p = ClockProcess(3)
    p.start()
```

    the time is Mon Jul 17 15:29:14 2017
    the time is Mon Jul 17 15:29:17 2017
    the time is Mon Jul 17 15:29:20 2017
    the time is Mon Jul 17 15:29:23 2017
    the time is Mon Jul 17 15:29:26 2017



```python
import multiprocessing
import time

def worker(interval):
    print("work start:{0}".format(time.ctime()));
    time.sleep(interval)
    print("work end:{0}".format(time.ctime()));

if __name__ == "__main__":
    p = multiprocessing.Process(target = worker, args = (3,))
    p.start()
    print("end!")
```

    end!
    work start:Mon Jul 17 15:30:40 2017
    work end:Mon Jul 17 15:30:43 2017



```python
import multiprocessing
import time

def worker(interval):
    print("work start:{0}".format(time.ctime()));
    time.sleep(interval)
    print("work end:{0}".format(time.ctime()));

if __name__ == "__main__":
    p = multiprocessing.Process(target = worker, args = (3,))
    p.daemon = True
    p.start()
    print("end!")
```

    end!
    work start:Mon Jul 17 15:31:36 2017
    work end:Mon Jul 17 15:31:39 2017



```python
import multiprocessing
import time

def worker(interval):
    print("work start:{0}".format(time.ctime()));
    time.sleep(interval)
    print("work end:{0}".format(time.ctime()));

if __name__ == "__main__":
    p = multiprocessing.Process(target = worker, args = (3,))
    p.daemon = True
    p.start()
    p.join()
    print("end!")
```

    work start:Mon Jul 17 15:33:05 2017
    work end:Mon Jul 17 15:33:09 2017
    end!



```python
import multiprocessing
import sys

def worker_with(lock, f):
    with lock:
        fs = open(f, 'a+')
        n = 10
        while n > 1:
            fs.write("Lockd acquired via with\n")
            n -= 1
        fs.close()
        
def worker_no_with(lock, f):
    lock.acquire()
    try:
        fs = open(f, 'a+')
        n = 10
        while n > 1:
            fs.write("Lock acquired directly\n")
            n -= 1
        fs.close()
    finally:
        lock.release()
    
if __name__ == "__main__":
    lock = multiprocessing.Lock()
    f = "file.txt"
    w = multiprocessing.Process(target = worker_with, args=(lock, f))
    nw = multiprocessing.Process(target = worker_no_with, args=(lock, f))
    w.start()
    nw.start()
    print("end")
```

    end



```python
import multiprocessing
import time

def worker(s, i):
    s.acquire()
    print(multiprocessing.current_process().name + "acquire");
    time.sleep(i)
    print(multiprocessing.current_process().name + "release\n");
    s.release()

if __name__ == "__main__":
    s = multiprocessing.Semaphore(2)
    for i in range(5):
        p = multiprocessing.Process(target = worker, args=(s, i*2))
        p.start()
```

    Process-11acquire
    Process-12acquire
    Process-11release
    
    Process-13acquire
    Process-12release
    
    Process-14acquire
    Process-13release
    
    Process-15acquire
    Process-14release
    
    Process-15release
    



```python
import multiprocessing
import time

def wait_for_event(e):
    print("wait_for_event: starting")
    e.wait()
    print("wairt_for_event: e.is_set()->" + str(e.is_set()))

def wait_for_event_timeout(e, t):
    print("wait_for_event_timeout:starting")
    e.wait(t)
    print("wait_for_event_timeout:e.is_set->" + str(e.is_set()))

if __name__ == "__main__":
    e = multiprocessing.Event()
    w1 = multiprocessing.Process(name = "block",
            target = wait_for_event,
            args = (e,))

    w2 = multiprocessing.Process(name = "non-block",
            target = wait_for_event_timeout,
            args = (e, 2))
    w1.start()
    w2.start()

    time.sleep(3)

    e.set()
    print("main: event is set")
```

    wait_for_event: starting
    wait_for_event_timeout:starting
    wait_for_event_timeout:e.is_set->False
    wairt_for_event: e.is_set()->True
    main: event is set



```python
import multiprocessing

def writer_proc(q):      
    try:         
        q.put(1, block = False) 
    except:         
        pass   

def reader_proc(q):      
    try:         
        print(q.get(block = False))
    except:         
        pass

if __name__ == "__main__":
    q = multiprocessing.Queue()
    writer = multiprocessing.Process(target=writer_proc, args=(q,))  
    writer.start()   

    reader = multiprocessing.Process(target=reader_proc, args=(q,))  
    reader.start()  

    reader.join()  
    writer.join()
```

    1



```python
import multiprocessing
import time

def proc1(pipe):
    while True:
        for i in arrange(0,10000):
            print("send: %s" %(i))
            pipe.send(i)
            time.sleep(1)

def proc2(pipe):
    while True:
        print("proc2 rev:", pipe.recv())
        time.sleep(1)

def proc3(pipe):
    while True:
        print("PROC3 rev:", pipe.recv())
        time.sleep(1)

if __name__ == "__main__":
    pipe = multiprocessing.Pipe()
    p1 = multiprocessing.Process(target=proc1, args=(pipe[0],))
    p2 = multiprocessing.Process(target=proc2, args=(pipe[1],))
    #p3 = multiprocessing.Process(target=proc3, args=(pipe[1],))

    p1.start()
    p2.start()
    #p3.start()

    p1.join()
    p2.join()
    #p3.join()
```


```python
import multiprocessing
import time

def func(msg):
    print("msg:", msg)
    time.sleep(3)
    print("end")

if __name__ == "__main__":
    pool = multiprocessing.Pool(processes = 3)
    for i in xrange(4):
        msg = "hello %d" %(i)
        pool.apply_async(func, (msg, ))   #维持执行的进程总数为processes，当一个进程执行完毕后会添加新的进程进去

    print("Mark~ Mark~ Mark~~~~~~~~~~~~~~~~~~~~~~")
    pool.close()
    pool.join()   #调用join之前，先调用close函数，否则会出错。执行完close后不会有新的进程加入到pool,join函数等待所有子进程结束
    print("Sub-process(es) done.")
```


```python
import multiprocessing
import time

def func(msg):
    print("msg:", msg)
    time.sleep(3)
    print("end")

if __name__ == "__main__":
    pool = multiprocessing.Pool(processes = 3)
    for i in range(0,3):
        msg = "hello %d" %(i)
        pool.apply(func, (msg, ))   #维持执行的进程总数为processes，当一个进程执行完毕后会添加新的进程进去

    print("Mark~ Mark~ Mark~~~~~~~~~~~~~~~~~~~~~~")
    pool.close()
    pool.join()   #调用join之前，先调用close函数，否则会出错。执行完close后不会有新的进程加入到pool,join函数等待所有子进程结束
    print("Sub-process(es) done.")
```

    msg: hello 0
    end
    msg: hello 1
    end
    msg: hello 2
    end
    Mark~ Mark~ Mark~~~~~~~~~~~~~~~~~~~~~~
    Sub-process(es) done.



```python
import multiprocessing
import time

def func(msg):
    print("msg:", msg)
    time.sleep(3)
    print("end")
    return "done" + msg

if __name__ == "__main__":
    pool = multiprocessing.Pool(processes=4)
    result = []
    for i in range(0,3):
        msg = "hello %d" %(i)
        result.append(pool.apply_async(func, (msg, )))
    pool.close()
    pool.join()
    for res in result:
        print(":::", res.get())
    print("Sub-process(es) done.")
```

    msg: hello 1
    msg: hello 0
    msg: hello 2
    end
    end
    end
    ::: donehello 0
    ::: donehello 1
    ::: donehello 2
    Sub-process(es) done.


    import multiprocessing
    import os, time, random
    
    def Lee():
        print("\nRun task Lee-%s" %(os.getpid())) #os.getpid()获取当前的进程的ID
        start = time.time()
        time.sleep(random.random() * 10) #random.random()随机生成0-1之间的小数
        end = time.time()
        print('Task Lee, runs %0.2f seconds.' %(end - start))
    
    def Marlon():
        print("\nRun task Marlon-%s" %(os.getpid()))
        start = time.time()
        time.sleep(random.random() * 40)
        end=time.time()
        print('Task Marlon runs %0.2f seconds.' %(end - start))
    
    def Allen():
        print("\nRun task Allen-%s" %(os.getpid()))
        start = time.time()
        time.sleep(random.random() * 30)
        end = time.time()
        print('Task Allen runs %0.2f seconds.' %(end - start))
    
    def Frank():
        print("\nRun task Frank-%s" %(os.getpid()))
        start = time.time()
        time.sleep(random.random() * 20)
        end = time.time()
        print('Task Frank runs %0.2f seconds.' %(end - start))
            
    if __name__=='__main__':
        function_list=  [Lee, Marlon, Allen, Frank] 
        print("parent process %s" %(os.getpid()))

    pool=multiprocessing.Pool(4)
    for func in function_list:
        pool.apply_async(func)     #Pool执行函数，apply执行函数,当有一个进程执行完毕后，会添加一个新的进程到pool中

    print('Waiting for all subprocesses done...')
    pool.close()
    pool.join()    #调用join之前，一定要先调用close() 函数，否则会出错, close()执行后不会有新的进程加入到pool,join函数等待素有子进程结束
    print('All subprocesses done.')
    

######后记

python多进程的具体效果还是需要继续尝试。