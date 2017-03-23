---
title: 【Celery】Windows 下 backend='amqp' 配置无效的解决方法
date: 2015-09-30 16:30:54
tags:
- Linux
- Celery
- 原创
categories: 
- Python
---
最近在Windows下用Celery的时候，碰到个问题：
 ```python
app = Celery('tasks', backend = 'amqp', broker='amqp://guest@localhost//')
```
这样建立C对象后，backend的配置“未生效”——不能追溯任务的结果，但在启动的状态和检查status时又显示“生效了”，怪哉。
tasks.py如下
 ```python
from celery import Celery
app = Celery('tasks', backend = 'amqp', broker='amqp://guest@localhost//')
@app.task
def add(x, y):
return x + y
```
启动界面如下：
```python
-------------- celery@Lenovo-PC v3.1.18 (Cipater)
---- **** -----
--- * ***  * -- Windows-8-6.2.9200
-- * - **** ---
- ** ---------- [config]
- ** ---------- .> app:         tasks:0x34025f8
- ** ---------- .> transport:   amqp://guest:**@localhost:5672//
- ** ---------- .> results:     amqp
- *** --- * --- .> concurrency: 4 (prefork)
-- ******* ----
--- ***** ----- [queues]
 -------------- .> celery           exchange=celery(direct) key=celery
```
在python shell中做如下操作
```python
>>> res.backend
<celery.backends.amqp.AMQPBackend object at 0x000000000349E438>
>>> res = tasks.add.delay(2,2)
>>> res.ready()
False
>>> res.backend
<celery.backends.amqp.AMQPBackend object at 0x000000000349E438>
>>> res.ready()
False
>>> res.state
'PENDING'
>>> res.state
'PENDING'
```
不能获取到结果
 通过尝试，在tasks.py中app创建后面加上
app.conf.CELERY_RESULT_BACKEND = 'amqp'
手动配置BACKEND后，再次尝试，可以解决问题。
```python
>>> res = tasks.add.delay(2,2)
>>> res.backend
<celery.backends.amqp.AMQPBackend object at 0x000000000349E438>
>>> res.get()
4
>>> res.ready()
True
```
但是问题的原因是啥，没搞懂，今天是国庆假期前一天，晚上没啥事，搜了一下，百度是没戏了，都是些互相抄的基础教程，试了下google，搜到这么一个页面，https://github.com/celery/celery/issues/897  里面的问题就是我的问题，看了一下，发现两个解决方法，一个和我的一样，一个是在celery启动的时候后面加--pool=solo，对比了一下发现加了--pool=solo后启动页面配置有变动：
Concurrency 分别是prefork和solo
查了Celery手册，
celery.concurrency.solo
Single-threaded pool implementation.
[原文](http://celery.readthedocs.org/en/latest/_modules/celery/concurrency/solo.html#TaskPool)  
celery.concurrency.prefork
Pool implementation using multiprocessing.
[原文](http://celery.readthedocs.org/en/latest/_modules/celery/concurrency/prefork.html#TaskPool)

综合一下所得信息：
- 1，linux下没有问题，windows下才会出现
- 2，单线程的时候backend可以获取到结果，多进程的时候获取不到
- 3，多进程的时候可以通过app.conf.CELERY_RESULT_BACKEND配置，使得结果可以获取到
感觉像是两种配置方法不同，受到多进程的影响了，结果没有顺利到达backend。
