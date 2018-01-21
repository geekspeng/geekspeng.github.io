---
title: Python入门07 — 函数
date: 2018-01-21
updated: 2018-01-21
tags: [Python]
categories: [Python]
---

函数（Functions）是指可重复使用的程序片段。它是有名字的代码块，接受输入、提供输出并可存储在文件中供以后使用。你通过这一特殊的名字在你的程序任何地方来运行代码块，并可重复任何次数。

<!-- more -->



# 定义函数

Python 定义函数使用 def 关键字，一般格式如下：

```
def 函数名 (参数列表):
    函数体
```



**函数头**

第1行（以`def`打头的那行）被称为函数头。

- 函数头总是以关键字`def`（definition的缩写）打头，接下来是一个空格，然后是函数名。
- 函数名后面是一对圆括号，其中可以包括参数列表，也可以不包括参数列表。
- 与循环和`if`语句一样，函数头也以冒号`:`结尾。



> **给函数命名**
>
> 与变量名一样，函数名也只能包含字母、数字和下划线`_`，且不能以数字打头。



**函数体**

函数头后面所有缩进的代码被称为函数体。

- 可选的文档字符串，用三引号标识文档字符串的开始和结束位置。
- 你需要完成的工作的代码块。在这个代码块中，可使用函数头中的变量。
- 最后，函数应使用关键字`return`返回一个值。



> **文档字符串一种格式约定**
>
> Python文档字符串通常遵循一种标准格式约定：用三引号标识文档字符串的开始和结束位置；第1行是函数的简要描述，对程序员很有帮助；接下来是详情和示例。
>
> **文档字符串的其他好处**
>
> 与内置函数一样，你也可轻松地查看自己编写的函数的文档字符串。如查看下面我们定义的计算圆面积的函数 area：
>
> ```python
> >>> print(area.__doc__)
> Returns the area of a circle with the given radius.
> For example:
> <>    >>> area(5.5)
>     95.033177771091246
> ```
>
> Python还有一个很有用的工具——doctest，可用于自动运行文档字符串中的Python示例代码。这是一种不错的代码测试方式，还可帮助确保文档准确地描绘了函数。更详细的信息请参阅[http://docs.python.org/3/ library/ doctest.html](http://docs.python.org/3/%20library/%20doctest.html)。

> **提示**    函数并非必须包含return语句，如果函数没有包含return语句，Python将认为它以return None结束。这很常见：函数常被用于执行返回值无关紧要的任务，如在屏幕上打印输出。



下面是一个计算圆面积的函数：

```python
# area.py
def area(radius):
    """ Returns the area of a circle
    with the given radius.
    For example:
    >>> area(5.5)
    95.033177771091246
    """
    return 3.14 * radius ** 2
```

上面这个函数是带有参数的，下面我们再举一个不带参数的函数：

```python
# say_hello.py
def say_hello():
    print('hello world')
```



# 调用函数

文章开头我们在说明什么是函数的时候说过，函数是有名字的代码块，允许你通过函数名在你的程序任何地方来运行函数，这就是所谓的*调用（Calling）*函数。



请看内置函数`pow(x, y)`，它计算`x ** y`，即`x`的`y`次方：

```python
>>> pow(2, 5)
32
```

其中，`pow`是**函数名**，2和5是传递给`pow`的实参，32是返回值。当你在表达式中调用函数时，Python将函数调用替换为其返回值，例如，表达式`pow(2, 5) + 8`与`32 + 8`等价，结果为40。



> **计算幂**
>
> `pow(x, y)`与`x ** y`等价。在Python中，`pow(0, 0)`（还有`0 ** 0`）的值为1。这一点一直存在争议，有些数学家认为，`pow(0, 0)`的值应该不确定或未定义；但其他一些数学家认为，将`pow(0, 0)`定义为1更合乎逻辑。Python显然支持后一种观点。



即便函数不接受任何输入（即没有参数），也必须在函数名后添加圆括号`()`：

```python
>>> dir()
['__builtins__', '__doc__', '__name__', '__package__']
```

`()`让Python执行指定的函数，如果省略`()`，输出将如下：

```python
>>> dir
<built-in function dir>
```

省略了`()`时，Python不执行函数，而告诉你dir指向一个函数。



**不返回值的函数**

有些函数（如`print`）不返回值。请看下述代码：

```python
>>> x = print('hello')
hello
>>> x
>>> print(x)
None
```

这里将特殊值`None`赋给了变量`x`。`None`表明“无返回值”：它既不是字符串，也不是数字，因此你不能用它来做任何有意义的计算。



**给函数名赋值**

你必须小心，以避免无意间让内置函数名指向其他函数或值。不幸的是，Python并不会阻止你编写类似下面的代码：

```python
>>> dir = 3
>>> dir
3
>>> dir()
Traceback (most recent call last):
    File "<pyshell#28>", line 1, in <module>
    dir()
TypeError: 'int' object is not callable
```

这里让`dir`指向了数字3，导致你再也无法访问`dir`原来指向的函数！要恢复原样，需要重启Python。



# 函数参数

函数中的参数通过将其放置在用以定义函数的一对圆括号中指定，并通过逗号予以分隔。当我们调用函数时，我们以同样的形式提供需要的值。要注意在此使用的术语——在定义函数时给定的名称称作*“形参”（Parameters）*，在调用函数时你所提供给函数的值称作*“实参”（Arguments）*。

## 参数传递

向函数传递参数时，Python采用**按引用传递**的方式。这意味着当你传递参数时，函数将使用新变量名来引用原始值。



案例（保存为 `reference.py`）：

```python
# reference.py
def add(a, b):
	return a + b

x, y = 3, 4
print(add(x, y))
```

输出：

```bash
$ python reference.py
7
```

在设置`x`和`y`后，内存类似于下图1所示。当调用`add(x,y)`时，Python创建两个新变量——`a`和`b`，它们分别指向`x`和`y`的值，如图2所示。注意到没有复制实参的值，而只是给它们指定新名称，而函数将使用这些新名称来引用它们。将`a`和`b`相加后，函数返回，而`a`和`b`被自动删除。在整个函数调用过程中，`x`和`y`未受影响。

![图1](http://p15d1hccg.bkt.clouddn.com/blog/180121/Id7e7AlKJ9.jpg)

图1 将x和y分别设置为3和4后的内存状态



![mark](http://p15d1hccg.bkt.clouddn.com/blog/180121/j2K59ccDHG.jpg)

图2  刚调用add(x,y)后的内存状态：a和b分别指向x和y指向的值



> **按值传递**
>
> 有些编程语言（如C++）可按值传递参数。按值传递参数时，将创建其拷贝，并将该拷贝传递给函数。如果传递的值很大，复制可能消耗大量时间和内存。Python不支持按值传递。



按引用传递简单而高效，但有些事情它做不了。例如，请看下面这个名不副实的函数：

```python
# reference.py
def set1(x):
    x = 1

m = 5
set1(m)
print(m) # 输出 5
```

输出：

```bash
$ python reference.py
5
```

函数`set1`想将传入的变量的值设置为1，但如果你尝试运行它，结果并不符合预期：

`m`的值根本没变，太令人意外了。这都是按引用传递导致的。为帮助理解，将这个示例分解成下面几步：

1. 将5赋给`m`。
2. 调用`set1(m)`：将`m`的值赋给`x`，这样`m`和`x`都指向5。
3. 将1赋给`x`，结果如下图所示。
4. 函数set1结束后，x被删除。

![mark](http://p15d1hccg.bkt.clouddn.com/blog/180121/3c4093j0dC.jpg)



## 参数类型

**必选参数**

在函数调用的时候必需传入与函数定义时数量相同的参数，并且参数顺序也要一致。



案例（保存为 `function_require.py`）：

```python
# function_require.py
def add(x, y):        # x, y 是必选参数
	return x + y

print(add(1, 2))             # 数量一致，通过
print(add())                 # 一个参数都不传，报错
# print(add(1))                # 只传了一个参数，报错
```

输出：

```bash
$ python function_require.py
3
Traceback (most recent call last):
  File "reference.py", line 6, in <module>
    print(add())                 # 一个参数都不传，报错
TypeError: add() missing 2 required positional arguments: 'x' and 'y'
```



**默认参数**

默认参数是指在定义函数的时候提供一些默认值，如果在调用函数的时候没有传递该参数，则自动使用默认值，否则使用传递时该参数的值。你可以通过在函数定义时附加一个赋值运算符（`=`）来为参数指定默认参数值。

> **提示**
>
> - 默认参数要放在所有必选参数的后面。这是因为值是按参数所处的位置依次分配的。
>
>
> - 默认参数应该使用不可变对象。



案例（保存为 `function_default.py`）：

```python
# function_require.py
def say(message, times=1):
    print(message * times)

say('Hello')
say('World', 5)
```

输出：

```bash
$ python function_default.py
Hello
WorldWorldWorldWorldWorld
```



**可变参数**

在某些情况下，我们在定义函数的时候，无法预估函数应该有多少个参数，也就是参数数量是可变的。为了能让一个函数接受任意数量的位置参数，可以使用一个* 参数。



案例（保存为 `function_varargs.py`）：

```python
def avg(first, *rest):
	return (first + sum(rest)) / (1 + len(rest))
  
print(avg(1, 2)) 
print(avg(1, 2, 3, 4))
```

输出：

```bash
$ python function_varargs.py
1.5
2.5
```



**关键字参数**

关键字参数和函数调用关系紧密，函数调用使用关键字参数来确定传入的参数值。



案例（保存为 `function_keyword.py`）：

```python
# function_keyword.py
def func(a, b):
    print('a is', a, 'and b is', b)

func(3, 7)
func(67, b=12)
func(a=15, b=42)
func(b=50, a=100)
```

输出：

```bash
$ python function_keyword.py
a is 3 and b is 7
a is 67 and b is 12
a is 15 and b is 42
a is 100 and b is 50
```



> **关键字参数有两大好处**
>
> 首先，它们清晰地指出了参数值，有助于提高程序的可读性；其次，关键字参数的顺序无关紧要。

> **提示**
>
> 关键参数要放在所有必选参数的后面。



# 变量作用域

函数带来的一个重要问题是作用域。变量的作用域指的是它在程序的哪些地方可访问或可见。

**局部变量**

首次赋值发生在函数内的变量被称为局部变量，同时函数的参数也被视为局部变量。

下面就是一个局部变量的例子：

```python
# function_local.py
def add(x, y):
    sum = x + y
    return sum
```

函数`add`有3个局部变量——`x`、`y`、`sum`。



**全局变量**

在函数外面声明的变量称为**全局变量**，程序中的任何函数或代码都可读取它。要访问全局变量，必须使用关键字`global` ，因为在不使用 `global` 语句的情况下，不可能为一个定义于函数之外的变量赋值。

案例（保存为 `function_global.py`）：

```python
# function_global.py
x = 50

def func():
    global x

    print('x is', x)
    x = 2
    print('Changed global x to', x)

func()
print('Value of x is', x)
```

输出：

```bash
$ python function_global.py
x is 50
Changed global x to 2
Value of x is 2
```

要在函数`func`中范围全局变量`x`必须使用关键字`global` 。



如果不使用关键字`global` ，访问的将会是局部变量。

案例（保存为 `function_local.py`）：

```python
# function_local.py
x = 50

def func(x):
    print('local x is', x)
    x = 2
    print('Changed local x to', x)

func(x)
print('global x is still', x)
```

输出：

```bash
$ python function_local.py
local x is 50
Changed local x to 2
global x is still 50
```