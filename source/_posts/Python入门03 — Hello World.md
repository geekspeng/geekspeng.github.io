---
title: Python入门03 — Hello World
date: 2018-01-11
updated: 2018-01-11
tags: [Python,Mac,Windows,Linux]
categories: [Python]
---

现在让我们开始学习如何运行一个传统的“Hello World”程序，这基本上是学习任何编程语言的需要做的第一步。下面将会告诉你如何编写、保存与运行 Python 程序。

通过 Python 来运行的你的程序有两种方法

- 使用交互式解释器提示符
- 直接运行一个源代码文件

<!-- more -->

#  使用解释器提示符

在你的操作系统中打开终端（Terminal）程序，然后通过输入 `python3` 并按下 `[enter]` 键来打开 Python 提示符（Python Prompt）。

当你启动 Python 后，你会看见在你能开始输入内容的地方出现了 `>>>` 。这个被称作 *Python 解释器提示符（Python Interpreter Prompt）* 。

在 Python 解释器提示符，输入`print("Hello World")`，在输入完成后按下 `[enter]` 键。你将会看到屏幕上打印出 `Hello World` 字样。

```python
xuepengdeMacBook-Pro:~ geekspeng$ python3
Python 3.6.4 (v3.6.4:d48ecebad5, Dec 18 2017, 21:07:28)
[GCC 4.2.1 (Apple Inc. build 5666) (dot 3)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> print("Hello World")
Hello World
>>>
```



**如何退出解释器提示符**

如果你正在使用一款 GNU/Linux 或 OS X 上的 Shell 程序，你可以通过按下 `[ctrl + d]` 组合键或是输入 `exit()` （注意：要记住要包含括号 `()`）并敲下 `[enter]` 来退出解释器提示符。

如果你使用的是 Windows 命令提示符，可以按下 `[ctrl + z]`组合键并敲击 `[enter]` 键来退出。



# 直接运行一个源代码文件

## 选择一款编辑器

当我们希望运行某些程序时，总不能每次都在解释器提示符中输入我们的程序。因此我们需要将它们保存为文件，从而我们便可以多次地运行这些程序。要想创建我们的 Python 源代码文件，我们需要一款能够让你输入并保存代码的编辑器软件。



**对编辑器的两项基本要求**

- 语法高亮

  通过标以不同颜色来帮助你区分 Python 程序中的不同部分，从而能够让你更好 *看清* 你的程序，并使它的运行模式更加形象化。

- 文本自动缩进

  Python的一个与众不同之处是，使用缩进来 *标识代码块*。要在Python中标识代码块，必须以同样程度缩进代码块的每一行。在其他大多数编程语言中，缩进只用于让代码更美观；但在Python中，必须使用缩进来指出语句所属的代码块。所以编辑器如果支持文本自动缩进的话，能减少大量我们花在手动缩进上的时间。



**编辑器推荐**

- 初学者

  推荐你使用 [PyCharm 教育版](https://www.jetbrains.com/pycharm-edu/) 软件，它在 Windows、Mac OS X、GNU/Linux 上都可以运行，从而你只需要专注于学习 Python 而不是编辑器。

- 熟悉[Vim](http://www.vim.org/) 或 [Emacs](http://www.gnu.org/software/emacs/)的程序员

  那你一定在用 [Vim](http://www.vim.org/) 或 [Emacs](http://www.gnu.org/software/emacs/) 了。无需多言，它们都是最强大的编辑器之一，用它们来编写你的 Python 程序自是受益颇多。



**1. PyCharm**

[PyCharm 教育版](https://www.jetbrains.com/pycharm-edu/)是一款能够对你编写 Python 程序的工作有所帮助的免费编辑器。

当你打开 PyCharm 时，你会看见如下界面，点击 `Create New Project` ：

![](http://p15d1hccg.bkt.clouddn.com/153731.jpg)



选择 `Pure Python` ：

![PyCharm 新项目](http://p15d1hccg.bkt.clouddn.com/153923.jpg)



将你的项目路径位置中的 `untitled` 更改为 `helloworld` ，你所看到的界面细节应该类似于下方这番：

![](http://p15d1hccg.bkt.clouddn.com/154212.jpg)



点击 `Create` 按钮。

对侧边栏中的 `helloworld` 右击选中，并选择 `New` -> `Python File` ：

![](http://p15d1hccg.bkt.clouddn.com/154351.jpg)



你会被要求输入名字，现在输入 `hello` ：

![](http://p15d1hccg.bkt.clouddn.com/154454.jpg)



现在你便可以看见一个新的文件已为你开启：

![](http://p15d1hccg.bkt.clouddn.com/155020.jpg)



删除那些已存在的内容，现在由你自己输入以下代码：

```python
print("hello world")
```



现在右击你所输入的内容（无需选中文本），然后点击 `Run 'hello'` 。

![](http://p15d1hccg.bkt.clouddn.com/154836.jpg)



此刻你将会看到你的程序所输出的内容（它所打印出来的内容）：

![](http://p15d1hccg.bkt.clouddn.com/154943.jpg)

虽然只是刚开始的几个步骤，但从今以后，每当我们要求你创建一个新的文件时，记住只需在 `helloworld` 上右击并选择 -> `New` -> `Python File` 并继续如上所述步骤一般输入内容并运行即可。



**2. Vim**

1. 安装 Vim
   - Mac OS X 应该通过 [HomeBrew](http://brew.sh/) 来安装 `macvim` 包。
   - Windows 用户应该通过 [Vim 官方网站](http://www.vim.org/download.php) 下载“自安装可执行文件”。
   - GNU/Linux 用户应该通过他们使用的发行版的软件仓库获取 Vim。例如 Debian 与 Ubuntu 用户可以安装 `vim` 包。
2. 安装 [jedi-vim](https://github.com/davidhalter/jedi-vim) 插件为 Vim 增添自动完成功能。
3. 安装与之相应的 `jedi` Python 包：`pip install -U jedi`



**3. Emacs**

1. 安装Emacs 24+
   - Mac OS X 用户应该从 [http://emacsformacosx.com](http://emacsformacosx.com/) 获取 Emacs。
   - Windows 用户应该从 <http://ftp.gnu.org/gnu/emacs/windows/> 获取 Emacs。
   - GNU/Linux 用户应该从他们使用的发行版的软件仓库获取 Emacs。如 Debian 和 Ubuntu 用户可以安装 `emacs24` 包。
2. 安装 [ELPY](https://github.com/jorgenschaefer/elpy/wiki)。



## 使用终端运行源代码文件

要想运行你的 Python 程序：

1. 打开终端窗口
2. 使用 `cd` 命令来**改**变**目**录到你保存文件的地方。
3. 通过输入命令 `python3 hello.py` 来运行程序。程序的输出结果应如下方所示：

```
$ python3 hello.py
hello world
```