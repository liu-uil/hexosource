---
title: Python 多线程模块Threading和Thread
date: 2015-09-27 16:30:08
tags:
- 线程
- 语法
- 原创
categories:
- Python
---
Python 中的多线程有两种模块实现：**Thread**和**Threading**。区别是，Thread 是更老的实现模块，同步原语很少（只有锁），而Threading模块则较多；Thread 实现的线程，主线程结束，所有的线程都会强制结束，没有警告也不会有正常的清理，Threading因为有守护线程的概念，可以确保非守护线程结束后进程才退出。简单来说，就是Threading的同步机制更加完善，下面是两个模块各自的函数对比：
表1 Thread模块函数和Lock类型锁对象方法

|函数|描述|
|---|---|
|Thread模块函数|-|
|start_new_thread(function, args[, kwargs])|Start a new thread and return its identifier. The thread executes the function function with the argument list args (which must be a tuple). The optional kwargs argument specifies a dictionary of keyword arguments. When the function returns, the thread silently exits. When the function terminates with an unhandled exception, a stack trace is printed and then the thread exits (but other threads continue to run).|
|allocate_lock()|Return a new lock object. Methods of locks are described below. The lock is initially unlocked.|
|exit()|Raise the SystemExit exception. When not caught, this will cause the thread to exit silently.
|Lock类型锁对象方法|-|
|acquire([waitflag])|Without the optional argument, this method acquires the lock unconditionally, if necessary waiting until it is released by another thread (only one thread at a time can acquire a lock — that’s their reason for existence). If the integer waitflag argument is present, the action depends on its value: if it is zero, the lock is only acquired if it can be acquired immediately without waiting, while if it is nonzero, the lock is acquired unconditionally as before. The return value is True if the lock is acquired successfully, False if not.
|locked()|Return the status of the lock: True if it has been acquired by some thread, False if not.
|release()|Releases the lock. The lock must have been acquired earlier, but not necessarily by the same thread.

表2 Threading 模块对象和Thread类对象方法

|函数|描述|
|---|---|
|Threading模块对象|-|
|Thread(group=None, target=None, name=None, args=(), kwargs={})|This constructor should always be called with keyword arguments. Arguments are:<br>group should be None; reserved for future extension when a ThreadGroup class is implemented.<br>target is the callable object to be invoked by the run() method. Defaults to None, meaning nothing is called.<br>name is the thread name. By default, a unique name is constructed of the form “Thread-N” where N is a small decimal number.<br>args is the argument tuple for the target invocation. Defaults to ().<br>kwargs is a dictionary of keyword arguments for the target invocation. Defaults to {}.<br>If the subclass overrides the constructor, it must make sure to invoke the base class constructor (Thread.__init__()) before doing anything else to the thread.
|同步原语|Lock, RLock, Condition, Event, Semaphore, BoundSemaphore, TImer
|Thread 类对象方法|-|
|start()|Start the thread’s activity.
|run()|Method representing the thread’s activity.<br>You may override this method in a subclass.
|join([timeout])|Wait until the thread terminates or timeout.
|getName() , setName()|Get the thread’s name; set the thread’s name; Multiple threads may be given the same name.
|isAlive()|Return whether the thread is alive.
|isDaemon(), setDaemon()|A boolean value indicating whether this thread is a daemon thread (True) or not (False). This must be set before start() is called, otherwise RuntimeError is raised. Its initial value is inherited from the creating thread; the main thread is not a daemon thread and therefore all threads created in the main thread default to daemon = False.

用Thread和Threading实现并行执行打印时间
Talk is cheap，下面动手写一下，先是用Thread模块实现2个线程：
```python
#encoding = utf-8
from time import ctime, sleep
import thread
loopNum = 2
def loop(loopIndex,nSec, lock):
    print 'Loop %s starts at %s'%(loopIndex, ctime())
    sleep(nSec)
    print 'Loop %s ends at %s'%(loopIndex, ctime())
    lock.release()
def main():
    print "Main starts at %s"%ctime()
    locks = []
    for i in range(loopNum):
        lock = thread.allocate_lock()
        lock.acquire()
        locks.append(lock)
    for i in range(loopNum):
        thread.start_new_thread(loop,(i+1, (i+1)*2, locks[i]))
#    sleep(6)
    for i in range(loopNum):
        while locks[i].locked():
            pass
    print "Main ends at %s"%ctime()
if __name__=="__main__":
    main()
```
输出如下：
```python
Main starts at Sun Sep 27 10:49:57 2015
Loop 1 starts at Sun Sep 27 10:49:57 2015
Loop 2 starts at Sun Sep 27 10:49:57 2015
Loop 1 ends at Sun Sep 27 10:49:59 2015
Loop 2 ends at Sun Sep 27 10:50:01 2015
Main ends at Sun Sep 27 10:50:01 2015
```
可以看到Thread对于同步的实现是很原始的，需要手动控制锁的开关，下面用Threading模块实现同样的功能：
```python
#encoding = utf-8
import threading
from time import sleep,ctime
loopNum = 2
class MyThread(threading.Thread):
    def __init__(self, func, args, name=''):
        threading.Thread.__init__(self)
        self.func = func
        self.args = args
        self.name = name
    def run(self):
        apply(self.func, self.args)
def loop(loopIndex, nSec):
    print 'Loop %s starts at %s'%(loopIndex, ctime())
    sleep(nSec)
    print 'Loop %s ends at %s'%(loopIndex, ctime())  
def main():
    print "Main starts at %s"%ctime()
    Threads = []
    for i in range(loopNum):
        t = MyThread(loop, (i+1,(i+1)*2), loop.__name__)
        Threads.append(t)
    for i in range(loopNum):
        Threads[i].start()
    for i in range(loopNum):
        Threads[i].join()
    print "Main ends at %s"%ctime()
if __name__=="__main__":
    main()
```
输出的结果：
```python
Main starts at Sun Sep 27 10:50:48 2015
Loop 1 starts at Sun Sep 27 10:50:48 2015
Loop 2 starts at Sun Sep 27 10:50:48 2015
Loop 1 ends at Sun Sep 27 10:50:50 2015
Loop 2 ends at Sun Sep 27 10:50:52 2015
Main ends at Sun Sep 27 10:50:52 2015
```
Threading有3种使用方法：给Threading.Thread的实例传递函数和参数；给Threading.Thread的实例传递可调用的类实例；继承Threading.Thread类，重写run方法，用新类实例化对象。上面用了第3种方法，从代码可以看出，用Threading 实现时，只需要用到join()方法实现同步，不需要手动控制锁的开关，相比Thread更方便，更不容易出错。
关于Threading模块的几个小点：
Daemon的继承
子线程默认继承上一级线程的Daemon状态，默认所有线程Daemon都是False，如果设置了子线程是True的，默认子线程的子线程（在子线程中新起的线程）是True的。
在程序2的loop函数前加函数subLoop:
```python
def subLoop():
pass
```
在loop函数的最后加2行：
```python
def loop(loopIndex, nSec):
    print 'Loop %s starts at %s'%(loopIndex, ctime())
    sleep(nSec)
    print 'Loop %s ends at %s'%(loopIndex, ctime()) 
    t = threading.Thread(target=subLoop)
print 'subLoop\'s daemon status is ', t.isDaemon()
```
再次执行，输出：
```python
ain starts at Sun Sep 27 11:23:49 2015
Loop 1 starts at Sun Sep 27 11:23:49 2015
Loop 2 starts at Sun Sep 27 11:23:49 2015
Loop 1 ends at Sun Sep 27 11:23:51 2015
subLoop's daemon status is  False
Loop 2 ends at Sun Sep 27 11:23:53 2015
subLoop's daemon status is  False
Main ends at Sun Sep 27 11:23:53 2015
```
将main函数的34行加上setDaemon：
```python
    for i in range(loopNum):
        t = MyThread(loop, (i+1,(i+1)*2), loop.__name__)
        t.setDaemon(True)
        Threads.append(t)
```
再次执行，输出：
```python
Main starts at Sun Sep 27 11:30:15 2015
Loop 1 starts at Sun Sep 27 11:30:15 2015
Loop 2 starts at Sun Sep 27 11:30:15 2015
Loop 1 ends at Sun Sep 27 11:30:17 2015
subLoop's daemon status is  True
Loop 2 ends at Sun Sep 27 11:30:19 2015
subLoop's daemon status is  True
Main ends at Sun Sep 27 11:30:19 2015
```
新增subSubLoop，subLoop和loop修改如下：
```python
def subSubLoop():
    pass
def subLoop():
    t = threading.Thread(target = subSubLoop)
    print 'subSubLoop\'s daemon status is ', t.isDaemon()
    pass
def loop(loopIndex, nSec):
    print 'Loop %s starts at %s'%(loopIndex, ctime())
    sleep(nSec)
    print 'Loop %s ends at %s'%(loopIndex, ctime()) 
    t = threading.Thread(target=subLoop)
    print 'subLoop\'s daemon status is ', t.isDaemon()
    t.start()
t.join()
```
再次执行，输出：
```python
Main starts at Sun Sep 27 11:34:26 2015
Loop 1 starts at Sun Sep 27 11:34:26 2015
Loop 2 starts at Sun Sep 27 11:34:26 2015
Loop 1 ends at Sun Sep 27 11:34:28 2015
subLoop's daemon status is  True
subSubLoop's daemon status is  True
Loop 2 ends at Sun Sep 27 11:34:30 2015
subLoop's daemon status is  True
subSubLoop's daemon status is  True
Main ends at Sun Sep 27 11:34:30 2015
```
可以看到子线程和孙线程都是True了，说明Daemon的继承具有传递性。
setName 设置多个线程同名（没发现有啥用，后续补充吧）
使用多个join，可能不是你需要的结果，值得注意
如果有多个线程，程序需要等多个线程都结束再继续（如上文中的程序2），33行开始的for循环里的start()是同时开启所有线程，但是到36开始的for循环里join()并不是同时开始执行的，而是逐个执行。将37行join()前加一行打印：
```python
    for i in range(loopNum):
        print 'join %d start at %s'%(i+1, ctime())
        Threads[i].join()
```
输出如下：
```python
Main starts at Sun Sep 27 11:11:38 2015
Loop 1 starts at Sun Sep 27 11:11:38 2015
Loop 2 starts at Sun Sep 27 11:11:38 2015
join 1 start at Sun Sep 27 11:11:38 2015
Loop 1 ends at Sun Sep 27 11:11:40 2015
join 2 start at Sun Sep 27 11:11:40 2015
Loop 2 ends at Sun Sep 27 11:11:42 2015
Main ends at Sun Sep 27 11:11:42 2015
```

