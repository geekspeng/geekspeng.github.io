---
title: Python入门08 — 模块
date: 2018-01-22
updated: 2018-01-22
tags: [Python]
categories: [Python]
---

模块是一个包含所有你定义的函数和变量的文件，其后缀名是.py。模块可以被别的程序引入，以使用该模块中的函数等功能。

<!-- more -->

# 创建模块

要创建模块，可创建一个.py文件，在其中包含用于完成任务的函数。



案例（保存为 `mymodule.py`）：

```python
def say_hi():
    print('Hi, this is mymodule speaking.')

__version__ = '0.1
```

正如你所看见的，与我们一般所使用的 Python 的程序相比其实并没有什么特殊的区别。



# 导入模块

## import 语句

想使用 Python 源文件，只需在另一个源文件里执行 import 语句，语法如下：

```python
import module1[, module2[,... moduleN]
```



另一个模块（保存为 `mymodule_demo.py`）：

```python
import mymodule

mymodule.say_hi()
print('Version', mymodule.__version__)
```

输出：

```bash
$ python mymodule_demo.py
Hi, this is mymodule speaking.
Version 0.1
```



**Python解释器是怎样找到对应的文件的呢？**

Python 解释器将从它的 `sys.path` 变量所提供的目录中进行搜索。如果找到了对应模块，则该模块中的语句将在开始运行，并*能够*为你所使用。

`sys.path` 内包含了导入模块的字典名称列表。你能观察到 `sys.path` 的第一段字符串是空的——这一空字符串代表当前目录也是 `sys.path` 的一部分，它与 `PYTHONPATH` 环境变量等同。这意味着你可以直接导入位于当前目录的模块。否则，你必须将你的模块放置在 `sys.path` 内所列出的目录中。

另外要注意的是当前目录指的是程序启动的目录。你可以通过运行 `import os; print(os.getcwd())` 来查看你的程序目前所处在的目录。



## from…import 语句

Python的from语句让你从模块中导入一个指定的部分到当前命名空间中，语法如下：

```python
from modname import name1[, name2[, ... nameN]]
```



下面是一个使用 `from...import` 语法的范本（保存为 `mymodule_demo2.py`）：

```python
from mymodule import say_hi, __version__

say_hi()
print('Version', __version__)
```

`mymodule_demo2.py` 所输出的内容与 `mymodule_demo.py` 所输出的内容是一样的。



在这里需要注意的是，如果导入到 mymodule 中的模块里已经存在了 `__version__` 这一名称，那将产生冲突。这可能是因为每个模块通常都会使用这一名称来声明它们各自的版本号。因此，我们大都推荐最好去使用 `import` 语句，尽管这会使你的程序变得稍微长一些。



> **警告：**一般来说，你应该尽量*避免*使用 `from...import` 语句，而去使用 `import` 语句。这是为了避免在你的程序中出现名称冲突，同时也为了使程序更加易读。



你还可以使用：

```python
from mymodule import *
```

这将导入诸如 `say_hi` 等所有公共名称，但不会导入 `__version__` 名称，因为后者以双下划线开头。



> **警告：**要记住你应该避免使用 import-star 这种形式，即 `from mymodule import *`。

> **Python 之禅**
>
> Python 的一大指导原则是“明了胜过晦涩”。你可以通过在 Python 中运行 `import this` 来了解更多内容。



# 按字节码编译的 .pyc 文件 

导入一个模块是一件代价高昂的事情，因此 Python 引入了一些技巧使其能够更快速的完成。其中一种方式便是创建*按字节码编译的（Byte-Compiled）*文件，这一文件以 `.pyc` 为其扩展名，是将 Python 转换成中间形式的文件。这一 `.pyc` 文件在你下一次从其它不同的程序导入模块时非常有用——它将更加快速，因为导入模块时所需要的一部分处理工作已经完成了。同时，这些按字节码编译的文件是独立于运行平台的。

注意：这些 `.pyc` 文件通常会创建在与对应的 `.py` 文件所处的目录中。如果 Python 没有相应的权限对这一目录进行写入文件的操作，那么 `.pyc` 文件将*不会*被创建。



# 模块的 `__name__`

模块除了函数和和变量外，还可以包括可执行的代码，一般用来初始化这个模块。当模块第一次被导入时，它所包含的代码将被执行。

每个模块都有一个名称，而模块中的语句可以找到它们所处的模块的名称。默认情况下当模块第一次被导入时，它所包含的代码将被执行。如果我们想在模块被引入时，模块中的某一程序块不执行，我们可以用`__name__`属性来使该程序块仅在该模块自身运行时执行。



案例（保存为 `module_using_name.py`）：

```python
if __name__ == '__main__':
    print('This program is being run by itself')
else:
    print('I am being imported from another module')
```

输出：

```bash
$ python module_using_name.py
This program is being run by itself

$ python
>>> import module_using_name
I am being imported from another module
>>>
```

每一个 Python 模块都定义了它的 `__name__` 属性。如果它与 `__main__` 属性相同则代表这一模块是由用户独立运行的，因此我们便可以采取适当的行动。



# `dir` 函数

内置的 `dir()` 函数能够返回由对象所定义的名称列表。 如果这一对象是一个模块，则该列表会包括函数内所定义的函数、类与变量。

如果参数是模块名称，函数将返回这一指定模块的名称列表。 如果没有提供参数，函数将返回当前模块的名称列表。



案例：

```bash
$ python
>>> import sys

# 给出 sys 模块中的属性名称
>>> dir(sys)
['__displayhook__', '__doc__', '__excepthook__', '__interactivehook__', '__loader__', '__name__', '__package__', '__spec__', '__stderr__', '__stdin__', '__stdout__', '_clear_type_cache', '_current_frames', '_debugmallocstats', '_enablelegacywindowsfsencoding', '_getframe', '_git', '_home', '_xoptions', 'api_version', 'argv', 'base_exec_prefix', 'base_prefix', 'builtin_module_names', 'byteorder', 'call_tracing', 'callstats', 'copyright', 'displayhook', 'dllhandle', 'dont_write_bytecode', 'exc_info', 'excepthook', 'exec_prefix', 'executable', 'exit', 'flags', 'float_info', 'float_repr_style', 'get_asyncgen_hooks', 'get_coroutine_wrapper', 'getallocatedblocks', 'getcheckinterval', 'getdefaultencoding', 'getfilesystemencodeerrors', 'getfilesystemencoding', 'getprofile', 'getrecursionlimit', 'getrefcount', 'getsizeof', 'getswitchinterval', 'gettrace', 'getwindowsversion', 'hash_info', 'hexversion', 'implementation', 'int_info', 'intern', 'is_finalizing', 'maxsize', 'maxunicode', 'meta_path', 'modules', 'path', 'path_hooks', 'path_importer_cache', 'platform', 'prefix', 'ps1', 'ps2', 'set_asyncgen_hooks', 'set_coroutine_wrapper', 'setcheckinterval', 'setprofile', 'setrecursionlimit', 'setswitchinterval', 'settrace', 'stderr', 'stdin', 'stdout', 'thread_info', 'version', 'version_info', 'warnoptions', 'winver']
# only few entries shown here

# 给出当前模块的属性名称
>>> dir()
['__annotations__', '__builtins__', '__doc__', '__loader__', '__name__', '__package__', '__spec__', 'sys']

# 创建一个新的变量 'a'
>>> a = 5

>>> dir()
['__annotations__', '__builtins__', '__doc__', '__loader__', '__name__', '__package__', '__spec__', 'a', 'sys']

# 删除或移除一个名称
>>> del a

>>> dir()
['__annotations__', '__builtins__', '__doc__', '__loader__', '__name__', '__package__', '__spec__', 'sys']
```

要注意被导入进来的模块所能生成的列表也会是这一列表的一部分。

关于 `del` 的一个小小提示——这一语句用于*删除*一个变量或名称，当这一语句运行后，在本例中即 `del a`，你便不再能访问变量 `a`——它将如同从未存在过一般。



# 包

现在，你必须开始遵守用以组织你的程序的层次结构。变量通常位于函数内部，函数与全局变量通常位于模块内部。如果你希望组织起这些模块的话，应该怎么办？这便是包（Packages）应当登场的时刻。

包是指一个包含模块与一个特殊的 `__init__.py` 文件的文件夹，后者向 Python 表明这一文件夹是特别的，因为其包含了 Python 模块。

建设你想创建一个名为“world”的包，其中还包含着 ”asia“、”africa“等其它子包，同时这些子包都包含了诸如”india“、”madagascar“等模块。

下面是你会构建出的文件夹的结构：

```bash
- <some folder present in the sys.path>/
    - world/
        - __init__.py
        - asia/
            - __init__.py
            - india/
                - __init__.py
                - foo.py
        - africa/
            - __init__.py
            - madagascar/
                - __init__.py
                - bar.py

```

包是一种能够方便地分层组织模块的方式。你将在 标准库中看到许多有关于此的实例。