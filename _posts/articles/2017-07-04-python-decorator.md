---
layout: post
title: Python 装饰器详解 
excerpt: "本文提供了将装饰器转化成高阶函数表达的一种范式，根据这一范式，可以直接推导出四种装饰器的写法。"
categories: articles
tags: [python]
comments: true
share: true
---
Python 装饰器(decorator)是一种语法糖(*Syntactic sugar*)，

```python
@d
def foo():
    pass
```

以上代码和以下代码是等价的：

```python
foo = d(foo)
```

进一步说，

```python
@d(param)
def foo():
    pass
```

和下面代码是等价的

```python
foo = (d(param))(foo)
```

他们的实质都是传入一个函数对象，进行“装饰”后返回装饰好的另一个函数对象。

## 返回函数对象的函数

Python 中的函数是一个对象(object)，它可以像任何其他对象一样被创建、引用、传递和销毁。引用函数对象的方法是使用它的函数名，如 foo 引用的是一个函数对象，而 foo() 则是其返回值了。

得益于函数在 Python 中一等公民的地位，你可以在一个函数中返回另一个函数，接受函数作为参数或返回函数的函数，称为高阶函数(high-order function)。

看下面的例子：

```python
def f(a):
    def g(b):
        def h(c):
            print(a, b, c)
        return h
    return g

In [2]: f(1)(2)(3)
1 2 3

In [3]: g = f(4)

In [4]: g(5)(6)
4 5 6
```

这是在函数式编程中被称为柯里化(currying)的东西，被柯里化后的高阶函数 f 其实是这样执行的：

```python
((f(1))(2))(3)
```

首先执行的 f(1)，返回其内部定义的函数对象 g，接着执行 g(2)，返回了 g 内部定义的对象 h，以此类推。

## 不带参数的装饰器函数

最简单的情形就是不带参数的装饰器函数。
给函数 foo 使用装饰器 d，等价于

```python
foo = d(foo)
```
为了定义装饰器 d，必须使其接受一个函数对象参数，并返回一个函数对象，而要返回一个函数对象，就需要在 d 内部新定义一个函数对象。这个内部的函数对象有什么特点呢？它接受 foo 的参数，运行 foo 的代码，并返回 foo 的返回值，当然还做了装饰器需要做的额外工作。

按照以上需求，装饰器 d 的代码就可以写出了。

```python
def d(f):
    def wrapper(*args, **kwargs):
        # do something
        res = f(*args, **kwargs)
        # do something
        return res
    
    return wrapper
```


d(foo) 返回了函数对象 wrapper，而变量 foo 指向了它。foo 不再是原来的那个 foo，装饰器赋予了它额外的功能。

如果需要的话，d 其实是可以这样被调用的：

```python
In [35]: def foo(n):
    ...:     return n + 1
    ...:

In [36]: d(foo)(2)
Out[36]: 3
```

是不是就是上面柯里化的形式？

另一种装饰器就更容易理解了：

```python
def d(f):
    f.__some_attr__ = 'attr'
    return f
```

根本就没有重新定义一个函数对象，改了 f 的某个属性就把它扔出来了。

## 带参数的装饰器函数

带参数的装饰器函数这样使用：

```python
@d(param)
def foo(n):
    return n + 1
```

它等价于

```python
foo = d(param)(foo)
```

为此，首先需要定义一个接受装饰器参数 param 的函数，接着再定义接受 foo 为参数的函数，最后是接受函数 foo 本身参数为参数的函数，除了最内层函数返回 foo() 运行结果外，其他函数都返回一个函数对象。

```python
def d(param):
    def outter(f):
        def inner(*args, **kwargs):
            # do something
            res = f(*args, **kwargs)
            # do something
            return res
        return inner
    return outter
```

你当然也可以这样调用 d：

```python
d(param)(foo)(3)
```

## callable

如果一个类的实例 instance 具有 \_\_call\_\_ 方法，那么这个实例就是一个 callable，instance() 就相当于 instance.\_\_call\_\_()。

## 不带参数的装饰器类

仍然沿用以上对装饰器语法糖的解释，对函数 foo 使用装饰器类 D，等价于

```python
foo = D(foo)
```

foo 对象应该在 D.\_\_init\_\_() 中被传入，得到一个 callable，而 foo 在 D.\_\_call\_\_() 中被执行。所以一个不带参数的装饰器类应该写成这样：

```python
class D(object):
    def __init__(self, f):
        self.f = f
        
    def __call__(self, *args, **kwargs):
        # do something
        res = self.f(*args, **kwargs)
        # do something
        return res
```

## 带参数的装饰器类

使用带参数的装饰器类，等价于

```python
foo = D(params)(foo)
```

可以知道，装饰器的参数 params 在 D.\_\_init\_\_() 中被传入，而 D.\_\_call\_\_() 接受函数对象 foo， 返回装饰后的函数对象。因而装饰器类是这样的：

```python
class D(object):
    def __init__(self, param):
        self.param = param
        
    def __call__(self, f):
        def wrapper(*args, **kwargs):
            # do something
            res = f(*args, **kwargs)
            # do something
            return res
        
        return wrapper
```



## 总结

本文提供了将装饰器转化成高阶函数表达的一种范式，根据这一范式，可以直接推导出四种装饰器（带参/不带参、函数/类）的写法。
