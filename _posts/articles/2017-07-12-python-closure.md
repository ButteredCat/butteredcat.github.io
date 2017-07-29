---
layout: post
title: 闭包与自由变量
excerpt: "闭包简介，以及修改闭包外部自由变量的几种方法"
modified: 2017-07-14
categories: articles
tags: [Python]
comments: true
share: true
---

[之前](/articles/python-decorator)分析了装饰器的语法，由此可以直接推导出其基本框架。但为了写出一个功能完整的装饰器，还需要了解一个概念——闭包。

# 闭包

**闭包(closure)**，是引用了自由变量的函数。*这个被引用的自由变量将和这个函数一同存在，即使已经离开了创造它的环境也不例外。* 

看下面的例子

```python
def f(a):
    def g(b):
        print(a, b)
    return g

In [3]: g = f(4)

In [4]: g(5)
4 5
```

对 `f` 内部的函数 `g` 来说，参数 `a` 既不是它的参数，也不是它的局部变量，而是它的自由变量。该自由变量可以

1. 被 `g` 所引用，
2. 即使已经离开了创造它的函数 `f`，也不例外。


闭包和嵌套函数的概念有所区别。闭包当然是嵌套函数，但没有引用自由变量的嵌套函数却不是闭包。

Python 的函数有一个只读属性 `__closure__`，存储的就是函数所引用的自由变量，

```python
In [5]: g = f(1)

In [6]: g.__closure__
Out[6]: (<cell at 0x10f8cad98: int object at 0x10dbf8b20>,)
```

如果仅仅是嵌套函数，它的 `__closure__` 应该是 `None`。

# 修改自由变量

闭包有个重要的特性：内部函数只能引用而不能修改外部函数中定义的自由变量。试图直接修改只有两种结果，要么运行出错，要么你以为你修改了，实际并没有。

不能修改不是因为 Python 设计者故意限制，不给它权限，而是外部的自由变量被内部的局部变量覆盖了；被覆盖了也不是闭包独有的特性，从普通函数内部同样也不能直接修改全局变量。Python 命名空间的查找规则简写为 LEGB，四个字母分别代表 local、enclosed、global 和 build-in，闭包外层函数的命名空间就是 enclosed。Python 在检索变量时，按照 L -> E -> G -> B 的顺序依次查找，如果在 L 中找到了变量，就不会继续向后查找。

## 修改失败的情形

在示例 1 中，你的本意是修改自由变量 `number` ，然而并不能：由于存在对 `number` 的赋值语句（ `number += 1` ），Python 会认为 `number` 是 `printer` 的局部变量，可是在局部变量字典中又查找不到它的定义，只好抛出异常。抛出的异常不是因为不能修改自由变量，而是局部变量还没赋值就被引用了。

```python
# 示例1
def print_msg(number):
     def printer():
         number += 1
         print(number)
     printer()
     print(number)

In [10]: print_msg(9)
...
UnboundLocalError: local variable 'number' referenced before assignment
```

在示例 2 中，Python 成功地在 `printer` 内定义了局部变量 `number`，并覆盖了同名自由变量，你可能以为自己成功修改了 `print_msg` 中的 `number`，然而并没有。

```python
# 示例 2
def print_msg(number):
    def printer():
        number = 3
        print(number)
    printer()
    print(number)

In [14]: print_msg(9)
3
9
```

## 解决办法

怎么才能修改呢？

一种做法是利用可变类型(mutable)的特性，把变量存放在列表(List)之中。对可变的列表的修改并不需要对列表本身赋值，`number[0] = 3` 只是修改了列表元素。虽然列表发生了变化，但引用列表的变量却并没有改变，巧妙地“瞒”过了 Python。见示例3。

```python
# 示例 3
def print_msg(number):
    number = [number]
    def printer():
        number[0] = 3
        print(number[0])
    printer()
    print(number[0])
    
In [18]: print_msg(9)
3
3
```

Python 3 引入了 `nonlocal` 关键字，明确告诉解释器：这不是局部变量，要找上外头找去。在示例 4 中，`nonlocal` 帮助我们实现了所期望的对自由变量的修改。

```python
# 示例 4
def print_msg(number):
    def printer():
        "Here we are using the nonlocal keyword"
        nonlocal number
        number = 3
        print(number)
    printer()
    print(number)

In [20]: print_msg(9)
3
3
```

其实，在 Python 2 中，用 `global` 代替 `nonlocal`，也能达到类似的效果，但由于全局变量的不易控制，这种做法不被提倡。

# 实例

下面的例子很好地展示了自由变量的特点：与引用它的函数一同存在，而想要修改它，得小心谨慎。

```python
import time
from functools import wraps


def rate_limited(max_per_sec):
    min_interval = 1.0 / max_per_sec
    def decorated(func):
        last_time_called = [0.0]
        @wraps(func)
        def rate_limited_func(*args, **kwargs):
            elapsed = time.time() - last_time_called[0]
            to_wait = min_interval - elapsed
            if to_wait > 0:
                time.sleep(to_wait)
            res = func(*args, **kwargs)
            last_time_called[0] = time.time()
            return res
        return rate_limited_func
    return decorated


@rate_limited(2)
def print_n(n):
    print('>> %s' % time.time())


if __name__ == "__main__":
    for i in range(10):
        print_n(i)
```

装饰器 `rate_limit` 的作用，是限制被装饰的函数每秒内最多被访问 `max_per_sec` 次。为此，需要维护一个变量用以记录上次被调用的时刻，它独立于函数之外，和被修饰的函数一同存在，还能在每次被调用的时候更新。`last_time_called` 就是这样的变量。为了正确地更新，`last_time_called` 以列表的形式存在。如果在 Python 3 中，它也可以直接存为 `float`，只要在内部函数中声明为 `nonlocal`，也可以达到同样的目的。