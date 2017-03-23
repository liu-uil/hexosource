---
title: Python装饰器分析
date: 2015-11-29 16:37:29
tags:
- 装饰器
- 原理
- 原创
categories: 
- Python 

---

>**装饰器**是一个很著名的设计模式，经常被用于有切面需求的场景，较为经典的有插入日志、性能测试、事务处理等。装饰器是解决这类问题的绝佳设计，有了装饰器，我们就可以抽离出大量函数中与函数功能本身无关的雷同代码并继续重用。概括的讲，装饰器的作用就是为已经存在的对象添加额外的功能。

### 1.装饰器的原理
装饰器实现上是将函数作为参数传递给装饰器函数，然后返回一个新的函数，调用的时候实际上调用的是新的函数。
下面的两段代码作用是相同的：
```python
#encoding=utf-8
import time
 
def foo():
    print 'in foo()'
 
# 定义一个计时器，传入一个，并返回另一个附加了计时功能的方法
def timeit(func):
     
    # 定义一个内嵌的包装函数，给传入的函数加上计时功能的包装
    def wrapper():
        start = time.clock()
        func()
        end =time.clock()
        print 'used:', end - start
     
    # 将包装后的函数返回
    return wrapper
 
foo = timeit(foo)
foo()
```
```python
import time
 
def timeit(func):
    def wrapper():
        start = time.clock()
        func()
        end =time.clock()
        print 'used:', end - start
    return wrapper
 
@timeit
def foo():
    print 'in foo()'
 
foo()
```
重点关注第11行的@timeit，在定义上加上这一行与另外写foo = timeit(foo)完全等价，没有另外的黑魔法。

### 2.装饰器的执行时机
在定义了装饰器后，装饰函数的动作是在什么时候执行的也需要清除，是否每次执行函数的时候都会包装一下？可以看下下面这段代码：
```python 
# encoding: utf-8

def decorator(foo):
    print "this is the decorator"
    def wrap(*arg):
        print "before foo "
        foo(*arg)
        print "after foo"

    return wrap
@decorator
def foo(b, a = []):
    a.append(b)
    print a
    return a

r1 = foo(1)

@decorator
def foo2(*args):
    a = []
    for item in args:
        a.append(item)
    print a
    return a

r2 = foo2([1,2,3])
r1 = foo(2)
```
输出结果：
```
this is the decorator
before foo
[1]
after foo
this is the decorator
before foo
[[1, 2, 3]]
after foo
before foo
[1, 1]
after foo
```
可以看到装饰器是在函数定义的时候执行的，且每个被装饰的函数都执行且只执行一次装饰器。

### 3.python 内置的装饰器
#### 3.1 staticmethod、classmethod、property

python中内置了3个装饰器：staticmethod、classmethod、property，作用分别是把类中定义的方法变成类的静态方法、类方法和类属性。下面是一个实例：
```python
class Rabbit(object):
     
    def __init__(self, name):
        self._name = name
     
    @staticmethod
    def newRabbit(name):
        return Rabbit(name)
     
    @classmethod
    def newRabbit2(cls):
        return Rabbit('')
     
    @property
    def name(self):
        return self._name
```
此外，functools模块中有两个很有用的装饰器。
#### 3.2 wraps(wrapped[, assigned][, updated]): 

:   函数是有几个特殊属性比如函数名，在被装饰后，上例中的函数名foo会变成包装函数的名字wrapper，如果你希望使用反射，可能会导致意外的结果。这个装饰器可以解决这个问题，它能将装饰过的函数的特殊属性保留。

下面是实例：
```python
import time
import functools
 
def timeit(func):
    @functools.wraps(func)
    def wrapper():
        start = time.clock()
        func()
        end =time.clock()
        print 'used:', end - start
    return wrapper
 
@timeit
def foo():
    print 'in foo()'
 
foo()
print foo.__name__
```

这样foo的__name__属性就还是'foo'

#### 3.3 total_ordering(cls): 

:   这个装饰器在特定的场合有一定用处，但是它是在Python 2.7后新增的。它的作用是为实现了至少__lt__、__le__、__gt__、__ge__其中一个的类加上其他的比较方法，这是一个类装饰器。

例子：
下面的这段代码是不加这个装饰器的时候的python自带扩展比较的效果：
```python
#!/usr/bin/env python
# encoding: utf-8


class test:
    def __init__(self, var):
        self.var = var

    def __lt__(self, other):
        return self.var > other.var


t1 = test(2)
t2 = test(3)


print t1 < t2
print t1 > t2
print t1 <= t2
print t1 >= t2

```
输出是：
```
False
True
True
False
```
但是实际是定义的大于的时候小于才成立，而这里的<=和期望的值是不符的，说明python只能自己重新定义gt，没有重新定义le和ge。
下面加上total_ordering:
```python
import functools

@functools.total_ordering
class test:
    def __init__(self, var):
        self.var = var

    def __lt__(self, other):
        return self.var > other.var


t1 = test(2)
t2 = test(3)


print t1 < t2
print t1 > t2
print t1 <= t2
print t1 >= t2
```
输出是：
```
False
True
False
True
```
total_ordering自己把le和ge也用自定义的lt扩展了。



