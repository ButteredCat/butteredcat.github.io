---
layout: post
title: Python 对函数式编程的支持 
excerpt: "Python 对函数式编程中的一些概念，如匿名函数、柯里化、偏函数、尾递归、惰性求值的支持情况"
modified: 2017-07-04
categories: articles
tags: [python]
comments: true
share: true
---
> Python does not promote functional programming even though it works fairly well.

## Anonymous function
匿名函数是一种语法糖（syntactic sugar）：所有使用匿名函数的地方，都可以用普通函数代替，少了它程序一样写，匿名函数只是让这一过程更加简便可读。我国程序员们为了给小函数起一个能准确描述功能的英文名称，做归纳、查字典，花的时间比实现函数本身还要长。匿名函数一出，问题迎刃而解。不过从简便可读的角度来讲，Python 的匿名函数，即 lambda 函数，真是有点名不副实。`lambda` 关键字包含6个字母，外加一个空格，本身就很长，写出来的匿名函数更长，严重影响阅读。比如下面这个：
```python
map(lambda x: x * x, lst)
```
同样的功能，Scala 写出来就清晰了不少：
```scala
lst map (x => x * x)
```
另外，lambda 函数体只支持一条语句，这也限制了它的功能。不过从另一方面来说，需要两条以上语句实现的函数就不该再作为匿名函数了。

## Currying & partial application
Python 并不原生支持柯里化（currying），如果你一定要，那也可以有。

举例来说，你有一个函数 f：
```python
def f(a, b, c):
    print(a, b, c)
```
如果要将其柯里化，你可以改写成这样：
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
得益于函数在 Python 中一等公民的地位，即所谓 first-class function，你可以在函数中返回另一个函数，柯里化也由此实现。[《Currying in Python》](https://mtomassoli.wordpress.com/2012/03/18/currying-in-python/)一文提供了一种将普通函数柯里化的方法。

Partial application（偏函数）与柯里化是相关但不同的概念。你可以用 functools.partial 得到一个函数的偏函数：
```python
In [1]: def f(a, b, c):
   ...:     print(a, b, c)
   ...:

In [2]: from functools import partial

In [3]: g = partial(partial(f, 1), 2)

In [4]: g(3)
1 2 3
```

## Recursion & tail recursion
Python 不支持对尾递归函数的优化（tail recursion elimination， TRE），也根本不建议程序员在生产代码中使用递归。

摘录一段 Python 领导核心 Guido van Rossum [关于尾递归的讲话](http://neopythonic.blogspot.com.au/2009/04/tail-recursion-elimination.html)，供大家学习领会：
>  So let me defend my position (which is that I don't want TRE in the language). If you want a short answer, it's simply unpythonic. 

## Lazy evaluation
生成器（generator）就是 Python 中惰性求值（lazy evaluation）的一个例子：一个值只有当请求到来时，才会被计算。生成器的使用，可以看我的另一篇文章[《用 Python 写一个函数式编程风格的斐波那契序列生成器》](http://www.jianshu.com/p/920bd6cdde61)。

但是 Python 没有一些函数式编程语言中那种精确到每个值的惰性计算。比如在 Scala 中，你可以用关键字 `lazy` 定义一个值：
```scala
scala> lazy val a = 2 * 2
a: Int = <lazy>

scala> a
res0: Int = 4
```
直到第一次调用，值 a 才会被计算出来。

Python 可以曲折地实现类似的惰性计算特性。Python Cookbook 第三版 8.10 节展示了如何利用描述符（descriptor）实现类属性的惰性计算：
```python
class lazyproperty(object):
    def __init__(self, func):
        self.func = func

    def __get__(self, instance, cls):
        if instance is None:
            return self
        else:
            value = self.func(instance) 
            setattr(instance, self.func.__name__, value) 
            return value
```
定义好的 lazyproperty 作为装饰器（decorator），被装饰的属性就只有在被调用时才会求值，求得的值会被缓存供以后使用：
```python
class Circle(object):
    def __init__(self, radius):
        self.radius = radius

    @lazyproperty
    def area(self):
        print('Computing area')
        return math.pi * self.radius ** 2

    @lazyproperty
    def perimeter(self):
        print('Computing perimeter')
        return 2 * math.pi * self.radius

In [16]: import math

In [17]: c = Circle(4.0)

In [18]: c.area
Computing area
Out[18]: 50.26548245743669

In [19]: c.area
Out[19]: 50.26548245743669
```

## map, reduce & filter
好消息是，Python 支持这些基本的操作；而坏消息是，Python 不建议你使用它们。

在 Python 2 中，`map`、`reduce` 和 `filter` 都是 build-in functions（内置函数），返回的都是列表或计算结果，使用起来很方便；但到了 Python 3，`reduce` 不再是内置函数，而被放入 `functools`，`map` 和 `filter` 返回的不再是列表，而是可迭代的对象。假如你需要的恰恰是列表，那么使用起来略显繁琐。
```python
In [1]: from platform import python_version

In [2]: print('Python', python_version())
Python 3.5.2

In [3]: n1 = [1, 2, 3]

In [4]: n2 = [4, 5, 6]

In [5]: import operator

In [6]: map(operator.add, n1, n2)
Out[6]: <map at 0x104749748>

In [7]: list(map(operator.add, n1, n2))
Out[7]: [5, 7, 9]

In [8]: import functools

In [9]: functools.reduce(operator.add, n1)
Out[9]: 6
```
如果不拘泥于 `map`、`reduce` 的形式，实际上，list comprehension （列表推导）和 generator expression（生成器表达式）可以优雅地代替 `map` 和 `filter`：
```python
In [10]: [i + j for i, j in zip(n1, n2)] # list(map(operator.add, n1, n2))
Out[10]: [5, 7, 9]

In [11]: [i for i in n1 if i > 1] # list(filter(lambda x: x > 1, n1))
Out[11]: [2, 3]
```
一个 `for` 循环也总是可以代替 `reduce`，当然，你得多写几行代码，看上去也不那么 functional 了。

此外，[itertools](https://docs.python.org/3/library/itertools.html) 中还提供了 `takewhile`、`dropwhile`、`filterfalse`、`groupby`、`permutations`、`combinations` 等诸多函数式编程中常用的函数。

## 其他
另外，Python 不支持 pattern matching，return 语句怎么看怎么像 imperative programming （命令式编程）余孽，这些都限制了 Python 成为一种更好的函数式编程语言。

---
综上所述，相信你对于如何使用 Python 编写函数式代码已经入门，考虑到 Python 语言设计上对函数式编程的诸多限制，下一步就该考虑放弃了……

