---
title: 12 步轻松搞定 Python 装饰器 z
date: 2015-07-04
categories: ["4-Python"]
tags: ["4-Python", "4-Decorator", "4-装饰器"]
slug: decorator
---

**原文地址**：<http://www.jianshu.com/p/d68c6da1587a>、<https://dzone.com/articles/understanding-python>

作为一名教 Python 的老师，我发现学生们基本上一开始很难理解 Python 的装饰器，也许因为装饰器确实很难懂。理解装饰器需要你了解一些函数式编程的概念，还需要理解在 Python 中定义和调用函数相关语法的一些特点。我没法让装饰器变得简单，但是通过一步步的剖析，也许能够让你在理解装饰器的时候更自信一点。因为装饰器很复杂，这篇文章将会很长。

<!-- more -->

> **提醒**：这篇文章中用的是 2.X 的语法。

# 函数

在 Python 中，函数通过`def`关键字、`函数名`和`可选的参数列表`定义。通过`return`关键字返回值。举例来说明如何定义和调用一个简单的函数：

```python
>>> def foo():
...     return 1
>>> foo()
1
```

函数定义主体部分和所有的多行 Python 代码类似，需要有缩进，定义好函数后，在方法名的后面加上双括号`()`就能够调用函数。

# 作用域

在 Python 中，函数会创建一个新的作用域。Python 开发者可能会说函数有自己的命名空间，差不多一个意思。这意味着在函数内部碰到一个变量的时候函数会优先在自己的命名空间里面去寻找。我们写一个简单的函数看一下**本地作用域**和 **全局作用域**有什么不同：

```python
>>> a_string = "This is a global variable"
>>> def foo():
...     print locals()
>>> print globals()
{..., 'a_string': 'This is a global variable'}
>>> foo() # 2
{}
```

内置的函数`globals()`返回一个包含所有 Python 解释器知道的变量名称的`dict`（方便起见，这里省略了 Python 自行创建的一些变量）。在`#2`我调用了函数`foo()`把函数内部本地作用域里面的内容打印出来。我们能够看到，函数`foo()`有自己独立的命名空间，虽然暂时命名空间里面什么都还没有。

# 变量解析规则

当然这并不是说我们在函数里面就不能访问外面的全局变量。在 Python 的作用域规则里面，**创建**变量肯定是在当前作用域中进行，但是**访问**或者**修改**变量时会先在当前作用域查找指定的变量，没有找到匹配变量时才依次向上在闭合的作用域里面进行查找。所以如果我们修改函数`foo()`的实现让它打印全局的作用域里的变量也是可以的：

```python
>>> a_string = "This is a global variable"
>>> def foo():
...     print a_string # 1
>>> foo()
This is a global variable
```

在`#1`处，Python 解释器会尝试查找变量`a_string`，当然在函数的本地作用域里面（即`foo()`里面）是找不到的，所以接着会去上层的作用域里面去查找。但是另一方面，假如我们在函数内部给全局变量**赋值**，结果却和我们想的`不一样`：

```python
>>> a_string = "This is a global variable"
>>> def foo():
...     a_string = "test" # 1
...     print locals()
>>> foo()
{'a_string': 'test'}
>>> a_string # 2
'This is a global variable'
```

可以看到：

1. 全局变量能够被`访问`到；
2. 如果是可变数据类型（如`list`, `dict`这些，甚至能够被更改），可能准确来讲并没有改变`list`或`dict`，而是改变了指向的具体`list`或`dict`内部的某个元素；
3. 不可以~~`赋值`~~，这里的赋值相当于在`foo()`内部创建一个名称为`a_string`的变量，因此只能在`foo()`的作用域内进行。

在函数内部的`#1`处，我们实际上新创建了一个局部变量，隐藏全局作用域中的同名变量。可以通过打印出局部命名空间中的内容得出这个结论，显然在`#2`处打印出来的变量`a_string`的值并没有改变。

# 变量生存周期

值得注意的一个点是，变量不仅是生存在一个个的命名空间内，他们都有自己的生存周期，请看下面这个例子：

```python
>>> def foo():
...     x = 1
>>> foo()
>>> print x # 1
Traceback (most recent call last):
  ...
NameError: name 'x' is not defined
```

`#1`处发生的错误不仅仅是因为作用域规则导致的（尽管这是抛出了`NameError`的错误的原因）它还和 Python 以及其它很多编程语言中函数调用实现的机制有关。在这个地方这个执行时间点并没有什么有效的语法让我们能够获取变量`x`的值，因为它这个时候压根不存在！函数`foo()`的命名空间随着函数调用开始而开始，结束而销毁。

# 函数参数

Python 允许我们向函数传递参数，参数会变成本地变量存在于函数内部。

```
>>> def foo(x):
...     print locals()
>>> foo(1)
{'x': 1}
```

在 Python 里有很多的方式来定义和传递参数，完整版可以查看 [Python 官方文档](http://docs.python.org/tutorial/controlflow.html#more-on-defining-functions)。我们这里简略的说明一下：函数的参数可以是`必须的`**位置参数**或者是`可选的`**命名[默认]参数**。

```python
>>> def foo(x, y=0): # 1
...     return x - y
>>> foo(3, 1) # 2
2
>>> foo(3) # 3
3
>>> foo() # 4
Traceback (most recent call last):
  ...
TypeError: foo() takes at least 1 argument (0 given)
>>> foo(y=1, x=3) # 5
2
```

在`#1`处我们定义了函数`foo()`,它有一个位置参数`x`和一个命名参数`y`。

- 在`#2`处我们能够通过常规的方式来调用函数，尽管有一个命名参数，但参数依然可以通过位置传递给函数；
- 在调用函数的时候，对于命名参数`y`我们也可以完全不管就像`#3`处所示的一样。如果命名参数没有接收到任何值的话，Python会自动使用声明的默认值也就是`0`；
- 不能省略第一个位置参数`x`，否则的话就会出现类似`#4`处所示发生错误；

目前还算简洁清晰吧， 但是接下来可能会有点令人困惑。Python支持函数`调用时的命名参数`（个人觉得应该是命名实参）。

- 观察`#5`处的函数调用，我们传递的是两个`命名`实参，这个时候因为有名称标识，参数传递的`顺序也就不用在意`了；
- 相反的情况也是正确的：函数的第二个形参是`y`，但是我们通过位置的方式传递值给它。比如在`#2`处的函数调用`foo(3,1)`，我们把`3`传递给了第一个参数，把`1`传递给了第二个参数，尽管第二个参数是一个命名参数；

简单来讲，我们可以给只定义了位置参数的函数传递命名参数（实参），反之亦然！如果觉得还理解不了，可以查看[官方文档](https://docs.python.org/2/tutorial/controlflow.html#more-on-defining-functions)。

# 嵌套函数

Python 允许创建嵌套函数。这意味着我们可以在函数里面定义函数而且现有的作用域和变量生存周期依旧适用。

```python
>>> def outer():
...     x = 1
...     def inner():
...         print x # 1
...     inner() # 2
...
>>> outer()
1
```

这个例子有一点儿复杂，但是看起来也还行。想一想在`#1`发生了什么：Python 解释器需找一个叫`x`的本地变量，查找失败之后会继续在上层的作用域里面寻找，这个上层的作用域定义在另外一个函数里面。对函数`outer()`来说，变量`x`是一个本地变量，但是如先前提到的一样，函数`inner()`可以访问封闭的作用域（至少可以`读`和`修改`）。

在`#2`处，我们调用函数`inner()`，非常重要的一点是，`inner()`也仅仅是一个遵循 Python 变量解析规则的变量名，Python 解释器会优先在`outer()`的作用域里面对变量名`inner()`查找匹配的变量。

# 函数是 Python 世界里的一级类对象

## 函数作为参数

显然，在 Python 里函数和其他东西一样都是对象。

```python
>>> issubclass(int, object) # all objects in Python inherit from a common baseclass
True
>>> def foo():
...     pass
>>> foo.__class__ # 1
<type 'function'>
>>> issubclass(foo.__class__, object)
True
```

你也许从没有想过，你定义的`函数居然会有属性`，比如`foo`就有一个名称为`__class__`的属性。没办法，函数在 Python 里面就是对象，和其他的东西一样，也许这样描述会太学院派太官方了点：在 Python 里，函数只是一些普通的值而，已和其他的值一样。这就是说你可以把函数像参数一样传递给其他的函数或者给函数一个函数类型的返回值！如果你从来没有这么想过，那看看下面这个例子：

```python
>>> def add(x, y):
...     return x + y
>>> def sub(x, y):
...     return x - y
>>> def apply(func, x, y): # 1
...     return func(x, y) # 2
>>> apply(add, 2, 1) # 3
3
>>> apply(sub, 2, 1)
1
```

这个例子对你来说应该不会很奇怪。`add()`和`sub()`是两个非常普通的 Python 函数，接受两个值，返回一个计算后的结果值。在`#1`处能看到`apply()`第一个参数虽然只是一个普通的位置变量，但实际上在`#2`处`func`最终对应一个特定的函数，整个过程中函数的名称只是根其他变量一样的一个标识符而已。Python 把频繁要用的操作变成函数作为参数进行使用，比如传递一个函数给内置排序函数`sorted()`的`key`参数来自定义排序规则。

## 函数作为返回值

```python
>>> def outer():
...     def inner():
...         print "Inside inner"
...     return inner # 1
...
>>> foo = outer() #2
>>> foo
<function inner at 0x...>
>>> foo()
Inside inner
```

在`#1`处我把作为函数标识符的变量`inner`作为返回值返回出来。在`#2`处我们捕获住返回值：`函数inner()`，将它存在一个新的变量`foo`里。我们能够看到，当对变量`foo`进行求值时，它确实包含了函数`inner()`，而且我们能够对他进行调用。

# 闭包

我们先不急着定义什么是闭包，先来看看一段代码，仅仅是把上一个例子简单的调整了一下：

```python
>>> def outer():
...     x = 1
...     def inner():
...         print x # 1
...     return inner
>>> foo = outer()
>>> foo.func_closure
(<cell at 0x...: int object at 0x...>,)
```

在上一个例子中我们了解到，`inner()`作为一个函数成为`outer()`的返回值，保存在一个变量`foo`中，并且我们能够对它进行调用：`foo()`。不过它会正常的运行吗？我们先来看看作用域规则。

> 所有的东西都在 Python 的作用域规则下进行工作：`x`是函数`outer()`里的一个局部变量。当函数`inner()`在`#1`处打印`x`的时候，Python 解释器会在`inner()`内部查找相应的变量，当然会找不到，所以接着会到封闭作用域里面查找，并且会找到匹配。
但是从变量的生存周期来看，该怎么理解呢？我们的变量`x`是函数`outer()`的一个局部变量，这意味着只有当函数`outer()`正在运行的时候才会存在。根据我们已知的 Python 运行模式，我们没法在函数`outer()`返回之后继续调用函数`inner()`，在函数`inner()`被调用的时候，变量`x`早已不复存在，可能会发生一个运行时错误。
>
> 然而事实上返回的函数`inner()`居然能够正常工作。Python 支持一个叫做**函数闭包**的特性，用人话来讲就是，**嵌套定义在非全局作用域里面的函数能够记住它在被定义的时候它所处的封闭命名空间**。这能够通过查看函数的`func_closure`属性得出结论，这个属性里面包含封闭作用域里面的值（`只会包含被捕捉到的值`，比如`x`，如果在`outer()`里面还定义了其他的值，封闭作用域里面是不会有的）。

记住，每次函数`outer()`被调用的时候，函数`inner()`都会被重新定义。现在变量`x`的值不会变化，所以每次返回的函数`inner()`会是同样的逻辑，假如我们稍微改动一下呢？

```python
>>> def outer(x):
...     def inner():
...         print x # 1
...     return inner
>>> print1 = outer(1)
>>> print2 = outer(2)
>>> print1()
1
>>> print2()
2
```

从上面例子可以看到**闭包**（被函数记住的封闭作用域）这一特性可被用于创建自定义的函数，本质上来说是一个硬编码的参数。事实上我们并不是传递参数`1`或者`2`给函数`inner()`，我们实际上是创建了能够打印各种数字的各种自定义版本。

闭包单独拿出来就是一个非常强大的功能， 在某些方面，你也许会把它当做一个类似于面向对象的技术：`outer()`像是给`inner()`服务的构造器，`x`像一个私有变量。使用闭包的方式也有很多：你如果熟悉 Python 内置排序方法的参数`key`，你说不定已经写过一个`lambda`方法在排序的时候重新指定一个新的排序规则。现在你说不定也可以写一个`itemgetter`方法，接收一个索引值来返回一个完美的函数，传递给排序函数的参数`key`。

不过，我们现在不会用闭包做这么 low 的事，我们要写的是一个高大上的装饰器！

# 装饰器

装饰器其实就是一个闭包，把一个函数当做参数然后返回一个替代版函数，这个新的替代版的函数在保持原有函数功能的前提下会加入一些新的功能。

```python
>>> def outer(some_func):
...     def inner():
...         print "before some_func"
...         ret = some_func() # 1
...         return ret + 1
...     return inner
>>> def foo():
...     return 1
>>> decorated = outer(foo) # 2
>>> decorated()
before some_func
2
```

仔细看看上面这个装饰器的例子。我们定义了一个函数`outer()`，它只有一个`some_func`的参数，在函数里面我们定义了一个嵌套的函数`inner()`，`inner()`会打印一串字符串，然后调用`some_func()`，在`#1`处得到它的返回值。在`outer()`每次调用的时候`some_func`的值可能会不一样，但是不管`some_func`的之如何，我们都会调用它。最后，`inner()`返回`some_func() + 1`的值：我们通过调用在`#2`处存储在变量`decorated`里面的函数能够看到被打印出来的字符串以及返回值`2`，而不是期望中调用函数`foo()`得到的返回值`1`。

我们可以认为变量`decorated`是函数`foo()`的一个装饰版本，一个加强版本。事实上如果打算写一个有用的装饰器的话，我们可能会想愿意用装饰版本完全取代原先的函数`foo()`，这样我们总是会得到我们的`加强版`的`foo()`。想要达到这个效果，压根不需要学习新的语法，直接将`foo`当成`outer()`的参数，然后将函数`outer()`的结果赋值给`foo`即可：

```python
>>> foo = outer(foo)
>>> foo # doctest: +ELLIPSIS
<function inner at 0x...>
```

> **注意**：这里是将`outer(foo)`赋值给变量`foo`，由于`outer()`的返回值是一个函数`inner()`，因此现在`foo`实际上对应于`inner()`，即针对原始`foo()`函数在功能上加强后的新的`foo()`。不可以写成`foo = outer`，这样的话，前面的`decorated`就需要改写成`decorated = foo(foo)`，这样两个`foo`的处理会非常混乱。

现在，任何怎么调用都不会牵扯到原先的函数`foo()`，都会得到新的装饰版本的`foo()`，现在我们还是来写一个有用的装饰器。

想象我们有一个库，这个库能够提供类似坐标的对象，也许它们仅仅是一些`x`和`y`的坐标对。不过可惜的是这些坐标对象不支持数学运算符，而且我们也不能对源代码进行修改，因此也就`不能直接加入运算符的支持`。我们将会做一系列的数学运算，所以我们想要能够对两个坐标对象进行合适加减运算的函数，就需要在原有的基础上用装饰器方法来加强原有坐标对象的功能：

```python
>>> class Coordinate(object):
...     def __init__(self, x, y):
...         self.x = x
...         self.y = y
...     def __repr__(self):
...         return "Coord: " + str(self.__dict__)
>>> def add(a, b):
...     return Coordinate(a.x + b.x, a.y + b.y)
>>> def sub(a, b):
...     return Coordinate(a.x - b.x, a.y - b.y)

>>> one = Coordinate(100, 200)
>>> two = Coordinate(300, 200)
>>> add(one, two)
Coord: {'y': 400, 'x': 400}
```

如果不巧我们的加减函数同时也需要一些边界检查的行为那该怎么办呢？搞不好你只能够对正的坐标对象进行加减操作，任何返回的值也都应该是正的坐标。所以现在的期望是这样：

```python
>>> one = Coordinate(100, 200)
>>> two = Coordinate(300, 200)
>>> three = Coordinate(-100, -100)
>>> sub(one, two)
Coord: {'y': 0, 'x': -200}
>>> add(one, three)
Coord: {'y': 100, 'x': 0}
```

我们期望在不更改坐标对象`one`, `two`, `three`的前提下`one`减去`two`的值是`{x: 0, y: 0}`，`one`加上`three`的值是`{x: 100, y: 200}`。与其给每个方法都加上参数和返回值边界检查的逻辑，我们来写一个边界检查的装饰器！

```python
>>> def wrapper(func):
...     def checker(a, b): # 1
...         if a.x < 0 or a.y < 0:
...             a = Coordinate(a.x if a.x > 0 else 0, a.y if a.y > 0 else 0)
...         if b.x < 0 or b.y < 0:
...             b = Coordinate(b.x if b.x > 0 else 0, b.y if b.y > 0 else 0)
...         ret = func(a, b)
...         if ret.x < 0 or ret.y < 0:
...             ret = Coordinate(ret.x if ret.x > 0 else 0, ret.y if ret.y > 0 else 0)
...         return ret
...     return checker
>>> add = wrapper(add)
>>> sub = wrapper(sub)
>>> sub(one, two)
Coord: {'y': 0, 'x': 0}
>>> add(one, three)
Coord: {'y': 200, 'x': 100}
```

这个装饰器能像先前的装饰器例子一样进行工作，返回一个经过修改的函数，但是在这个例子中，它能够对函数的输入参数和返回值做一些非常有用的检查和格式化工作，将负值的`x`和`y`替换成`0`。

显而易见，通过这样的方式，我们的代码变得更加简洁：将边界检查的逻辑隔离到单独的方法中，然后通过装饰器包装的方式应用到我们需要进行检查的地方。虽然直接在计算的开始处和返回值之前调用边界检查的方法也能够达到同样的目的，但是使用装饰器能够让我们以最少的代码量达到坐标边界检查的目的。事实上，如果我们是在装饰自己定义的方法的话，我们能够让装饰器应用的更加有逼格。

# 使用 @ 标识符将装饰器应用到函数

Python 2.4 支持使用标识符`@`将装饰器应用在函数上，只需要在函数的定义前加上`@`和装饰器的名称。在上一节的例子里我们是将原本的方法用装饰后的方法代替:

```python
>>> add = wrapper(add)
```

这种方式能够在任何时候对任意方法进行包装。但是如果我们自定义一个方法，我们可以使用`@`进行装饰：

```python
>>> @wrapper
... def add(a, b):
...     return Coordinate(a.x + b.x, a.y + b.y)
```

这样的做法和先前简单的用包装方法替代原有方法是一样的， Python 只是加了一些语法糖让装饰的行为更加的直接明确和优雅一点。

# *args 和 **kwargs

我们已经完成了一个有用的装饰器，但是由于硬编码的原因它只能应用在一类具体的函数上，以上一节的`checker()`为例，它接收两个参数并传递给闭包捕获的函数`add()`或`sub()`。如果我们想实现一个能够应用在任何函数上的装饰器要怎么做呢？再比如，如果我们要实现一个能应用在任何函数上的类似于计数器的装饰器，不需要改变原有方法的任何逻辑。这意味着装饰器能够接受任意形式的函数作为自己的被装饰函数，同时能够用传递给它的参数对被装饰的方法进行调用。

非常巧合的是 Python 正好有支持这个特性的语法。可以阅读 [Python Tutorial](http://docs.python.org/tutorial/controlflow.html#arbitrary-argument-lists) 获取更多的细节。当定义函数的时候使用了`*`，意味着那些通过位置传递的参数将会被放在带有`*`前缀的变量中， 所以：

```python
>>> def one(*args):
...     print args # 1
>>> one()
()
>>> one(1, 2, 3)
(1, 2, 3)
>>> def two(x, y, *args): # 2
...     print x, y, args
>>> two('a', 'b', 'c')
a b ('c',)
```

第一个函数`one()`只是简单地讲任何传递过来的位置参数全部打印出来而已，在代码`#1`处我们只是引用了函数内的变量`args`, `*args`仅仅只是用在函数定义的时候用来表示位置参数应该存储在变量`args`里面。Python 允许我们制定一些参数并且通过`args`捕获其他所有剩余的未被捕捉的位置参数，就像`#2`处所示的那样。

`*`操作符在函数被调用的时候也能使用。意义基本是一样的。当调用一个函数的时候，一个用`*`标志的变量意思是变量里面的内容需要被提取出来然后当做位置参数被使用。同样的，来看个例子：

```python
>>> def add(x, y):
...     return x + y
>>> lst = [1,2]
>>> add(lst[0], lst[1]) # 1
3
>>> add(*lst) # 2
3
```

`#1`处的代码和`#2`处的代码所做的事情其实是一样的，在`#2`处，Python 为我们所做的事其实也可以手动完成。这也不是什么坏事，`*args`要么是表示**调用**方法的时候额外的参数可以从一个可迭代列表中取得，要么就是**定义**方法的时候标志这个方法能够接受任意的位置参数。

接下来提到的`**`会稍多更复杂一点，`**`代表着键值对的参数字典，和`*`所代表的意义相差无几，也很简单：

```python
>>> def foo(**kwargs):
...     print kwargs
>>> foo()
{}
>>> foo(x=1, y=2)
{'y': 2, 'x': 1}
```

当我们定义一个函数的时候，我们能够用`**kwargs`来表明，所有未被捕获的关键字参数都应该存储在`kwargs`的字典中。如前所诉，`args`和`kwargs`并不是 Python 语法的一部分，但在定义函数的时候，使用这样的变量名算是一个不成文的约定。和`*`一样，我们同样可以在定义或者调用函数的时候使用`**`。

```python
>>> dct = {'x': 1, 'y': 2}
>>> def bar(x, y):
...     return x + y
>>> bar(**dct)
3
```

# 更通用的装饰器

有了这招新的技能，我们随随便便就可以写一个能够记录下传递给函数参数的装饰器了。先来个简单地把日志输出到界面的例子：

```python
>>> def logger(func):
...     def inner(*args, **kwargs): #1
...         print "Arguments were: %s, %s" % (args, kwargs)
...         return func(*args, **kwargs) #2
...     return inner
```

请注意我们的函数`inner()`，它能够接受任意数量和类型的参数并把它们传递给被包装的方法，这让我们能够用这个装饰器来装饰任何方法。

```python
>>> @logger
... def foo1(x, y=1):
...     return x * y
>>> @logger
... def foo2():
...     return 2
>>> foo1(5, 4)
Arguments were: (5, 4), {}
20
>>> foo1(1)
Arguments were: (1,), {}
1
>>> foo2()
Arguments were: (), {}
2
```

随便调用我们定义的哪个方法，相应的日志也会打印到输出窗口，和我们预期的一样。