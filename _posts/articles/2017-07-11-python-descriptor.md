---
layout: post
title: Python 描述器解析
excerpt: "描述器语法分析，及其在实现 property()、 staticmethod() 等内置函数中的作用"
categories: articles
tags: [Python]
comments: true
share: true
---

# 语法简析

一般来说，描述器（descriptor）是一个有”绑定行为”的对象属性（object attribute），它的属性访问被描述器协议方法重写。这些方法是 `__get__()`、 `__set__()` 和 `__delete__()` 。如果一个对象定义了以上任意一个方法，它就是一个描述器。而描述器协议的具体形式如下：

```python
descr.__get__(self, obj, type=None) --> value

descr.__set__(self, obj, value) --> None

descr.__delete__(self, obj) --> None
```

描述器本质上是一个类对象，该对象定义了描述器协议三种方法中至少一种。而这三种方法只有当类的实例出现在一个所有者类（owner class）之内时才有效，也就是说，描述器必须出现在所有者类或其父类的字典 `__dict__` 里。这里提到了两个类，一是定义了描述器协议的描述器类，另一个是使用描述器的所有者类。

描述器往往以装饰器的方式被使用，导致二者常被混淆。描述器类和不带参数的装饰器类一样，都传入函数对象作为参数，并返回一个类实例，所不同的是，装饰器类返回 callable 的实例，描述器则返回描述器实例。

记住上面的话，下面我们举例说明。

# @Property

Python 内置的 `property` 函数可以说是最著名的描述器之一，几乎所有讲述描述器的文章都会拿它做例子。

`property` 是用 C 实现的，不过这里有一份等价的 Python 实现：

```python
class Property(object):
    "Emulate PyProperty_Type() in Objects/descrobject.c"

    def __init__(self, fget=None, fset=None, fdel=None, doc=None):
        self.fget = fget
        self.fset = fset
        self.fdel = fdel
        if doc is None and fget is not None:
            doc = fget.__doc__
        self.__doc__ = doc

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        if self.fget is None:
            raise AttributeError("unreadable attribute")
        return self.fget(obj)

    def __set__(self, obj, value):
        if self.fset is None:
            raise AttributeError("can't set attribute")
        self.fset(obj, value)

    def __delete__(self, obj):
        if self.fdel is None:
            raise AttributeError("can't delete attribute")
        self.fdel(obj)

    def getter(self, fget):
        return type(self)(fget, self.fset, self.fdel, self.__doc__)

    def setter(self, fset):
        return type(self)(self.fget, fset, self.fdel, self.__doc__)

    def deleter(self, fdel):
        return type(self)(self.fget, self.fset, fdel, self.__doc__)
```

`Property` 怎么用呢？看下面的例子：

```python
class C(object):
    def __init__(self):
        self._x = None

    @Property
    def x(self):
        """I'm the 'x' property."""
        return self._x

    @x.setter
    def x(self, value):
        assert value > 0
        self._x = value

    @x.deleter
    def x(self):
        del self._x
```

我们结合源代码和用法来分析 `Property`。

`@Property` 的用法就是一个装饰器。我们可以将其等价转化为：

```python
x = Property(x)
```

函数 `x` 作为位置参数被赋给 `Property.__init__()` 的 `fget`，得到新的 `x` 已经不是个函数而是个完整实现了 `__get__()` 方法的描述器实例了。

`@x.setter` 的用法略有不同。它实际上是利用上面定义的描述器实例 `x` 的 `setter` 方法，重新创建了新的实例。这时变量 `x` 再次被更新，指向了一个完整实现 `__get__()` 和 `__set__()` 方法的新描述器。传入 `setter` 方法的函数名必须是 `x`，否则如果是 `y`，按照装饰器的性质，

```python
y = x.setter(y)
```

新描述器就被 `y` 引用了，与需求不符。

`Property` 提供了像访问类“成员变量”一样访问 get、set 方法的能力。

```python
In [123]: c = C()

In [124]: c.x = 1

In [125]: c.x
Out[125]: 1

In [126]: c.x = 0
---------------------------------------------------------------------------
AssertionError                            Traceback (most recent call last)
<ipython-input-126-b03deb420dcb> in <module>()
----> 1 c.x = 0

<ipython-input-50-95b8686aa4bd> in __set__(self, obj, value)
     20         if self.fset is None:
     21             raise AttributeError("can't set attribute")
---> 22         self.fset(obj, value)
     23
     24     def __delete__(self, obj):

<ipython-input-116-379a4e5fa639> in x(self, value)
     10     @x.setter
     11     def x(self, value):
---> 12         assert value > 0
     13         self._x = value
     14

AssertionError:
```

与一般的属性访问不同，`c.x` 访问的已经不是简单的属性，而是相当于 `x.__get__(c)`，可以调用各种复杂方法对属性作检查、包装 。

那么，描述器是怎样被访问到的呢？

# 调用描述器

有两类描述器：如果同时定义了 `__get__()` 和 `__set__()` 方法的描述器称为资料描述器(data descriptor)，仅定义了 `__get__()` 的描述器称为非资料描述器(non-data descriptor)。非资料描述器常用于类的方法，如常见的 `staticmethod` 和 `classmethod`，都是其应用。

如前文所说，描述器常在所有者类或其实例中被调用。

对于实例对象，`object.__getattribute__()` 会把 `c.x` 转化为 `type(c).__dict__['x'].__get__(c, type(c))`。如果实例中有和描述器重名的属性 `x` 怎么办？资料和非资料描述器的区别在于，相对于实例字典的优先级不同。当描述器和实例字典中的某个属性重名，按访问优先级，资料描述器 > 同名实例字典中的属性 > 非资料描述器，优先级小的会被大的覆盖。上面的类 `C` 中，会优先访问资料描述器 `x`。下面将讲到，类的方法实际就是一个仅实现了 `__get__()` 的非资料描述器，所以如果实例 `c` 中同时定义了名为 `foo` 的方法和属性，那么 `c.foo` 访问的是属性而非方法。

对于类，`type.__getattribute__()` 把 `C.x` 转化为 `C.__dict__['x'].__get__(None, C)`。

有几点需要牢记的：

1. 描述器被 `__getattribute__()` 方法调用
2. 因而，重载 `__getattribute__()` 可能会妨碍描述器被自动调用
3. `__getattribute__()` 仅存在于继承自 `object` 的新式类之中
4. `object.__getattribute__()` 和 `type.__getattribute__()` 对 `__get__()` 的调用不一样
5. 资料描述器总会覆盖实例字典，即资料描述器具有最高优先级
6. 非资料描述器可能会被实例字典覆盖，即非资料描述器具有最低优先级

# 非资料描述器与类方法

Python 面向对象的特征建立在基于函数的环境之上。Python 用非资料描述器将二者无缝结合。

方法和普通函数唯一的区别就是，一般方法的第一个参数引用了当前实例，即通常命名为 `self` 的变量。

Python 中的函数，可以被认为是一个实现了 `__get__()` 的非资料描述器，用 Python 来描述就是：

```python
class Function(object):
    . . .
    def __get__(self, obj, objtype=None):
        "Simulate func_descr_get() in Objects/funcobject.c"
        return types.MethodType(self, obj, objtype)
```

当函数作为属性被访问时，非资料描述器把函数变为一个方法，把实例调用 `obj.f(*args)` 转化成 `f(obj, *args)`，把类调用 `klass.f(*args)` 转化为 `f(*args)`。

更多绑定和转换参见下表。

| 转换   | 从对象调用               | 从类调用            |
| :--- | :------------------ | :-------------- |
| 函数   | f(obj, *args)       | f(*args)        |
| 静态方法 | f(*args)            | f(*args)        |
| 类方法  | f(type(obj), *args) | f(klass, *args) |

静态方法是特殊的方法，可以无须实例化而在类中被直接调用，这时当然无法提供合法的 `self`。为此，需要实现 `staticmethod` 描述器，其 `__get__()` 返回的函数无需实例参数，其实也就是原样返回即可，可以用 Python 这样实现

```python
class StaticMethod(object):
    "Emulate PyStaticMethod_Type() in Objects/funcobject.c"

    def __init__(self, f):
        self.f = f

    def __get__(self, obj, objtype=None):
        return self.f
```

类方法是另一种特殊的方法，无需当前实例 `self`， 但是需要当前类 `klass` （通常也写成 `cls`），纯 Python 实现如下：

```python
class ClassMethod(object):
    "Emulate PyClassMethod_Type() in Objects/funcobject.c"

    def __init__(self, f):
        self.f = f

    def __get__(self, obj, klass=None):
        if klass is None:
            klass = type(obj)
        def newfunc(*args):
            return self.f(klass, *args)
        return newfunc
```



# 参考资料

* [Descriptor HowTo Guide](https://docs.python.org/2/howto/descriptor.html#id1) 及其[中文翻译](http://pyzh.readthedocs.io/en/latest/Descriptor-HOW-TO-Guide.html)