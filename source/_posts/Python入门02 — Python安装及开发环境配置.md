---
title: Python入门02 — Python安装及开发环境配置
date: 2018-01-10
updated: 2018-01-10
tags: [Python,Mac,Windows,Linux]
categories: [Python]
---

Python 是一款易于学习且功能强大的编程语言。 它具有高效率的数据结构，能够简单又有效地实现面向对象编程。Python 简洁的语法与动态输入之特性，加之其解释性语言的本质，使得它成为一种在多种领域与绝大多数平台都能进行脚本编写与应用快速开发工作的理想语言。这篇文章主要介绍Python在Mac OS X、Windows和Linux系统的安装，为以后python的学习做准备。

<!-- more -->

# 在Mac OS X 系统中安装

Mac OS X自带并安装了一个Python版本，但该版本没有IDLE编辑器，通常也不是最新版本，需要安装最新版本

- 对于 Mac OS X 用户，你可以使用 [Homebrew](http://brew.sh/) 并通过命令 `brew install python3` 进行安装，如果你还没有安装Homebrew，可以参考[Mac OS X 包管理工具 Homebrew 安装与使用](http://geekspeng.cn/2018/01/09/Mac%20OS%20X%20%E5%8C%85%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7%E5%AE%89%E8%A3%85%E4%B8%8E%E4%BD%BF%E7%94%A8/)
- 也可以下载一个安装程序并运行它

要想验证安装是否成功，你可以通过按键 `[Command + Space]`（以启动 Spotlight 搜索），输入 `Terminal` 并按下 `[enter]`键来启动终端程序。现在，试着运行 `python3` 来确保其没有任何错误

```bash
xuepengdeMacBook-Pro:~ geekspeng$ python3
Python 3.6.4 (v3.6.4:d48ecebad5, Dec 18 2017, 21:07:28)
[GCC 4.2.1 (Apple Inc. build 5666) (dot 3)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>>
```



# 在Windows系统中安装

1. 访问Python下载页面[www.python.org/download](http://www.python.org/download)并下载最新的版本python（32位选择[Windows x86 executable installer](https://www.python.org/ftp/python/3.6.4/python-3.6.4.exe)，64位选择 [Windows x86-64 executable installer](https://www.python.org/ftp/python/3.6.4/python-3.6.4-amd64.exe)），目前最新版本为3.6.4
2. 双击安装程序安装python，其安装过程与其它 Windows 平台的软件的安装过程无异，请务必确认勾选了`Add Python 3.5 to PATH` 选项




要想验证安装是否成功，你可以通过点击开始并点击 `运行` 。在对话中输入 `cmd` 并按下回车键。然后，输入 `python` 以确保其没有任何错误。如果输出与Mac一样，说明Python运行正常



**补充说明**

* 如未勾选相关选项，你可以点击 `Add Python to environment variables` 。它和安装程序第一屏的 `Add Python 3.5 to PATH`能起到相同效果。如果这个也忘记勾选也不要紧，只需要参考下面步骤手动添加环境变量即可
* 若要想改变安装位置，勾选 `Customize installation` 选项，点击 `Next` 后在安装位置中输入 `C:\python36`



**手动添加环境变量**

对于 Windos 7 与 8：

1. 在桌面右击计算机并选择 `属性` 或点击 `开始` 并选择 `控制面板` → `系统与安全` → `系统` 。点击左侧的 `高级系统设置` 并选择 `高级` 标签。点击底部 `系统变量` 下的 `环境变量` ，找到 `PATH` 属性，将其选中并点击 `编辑` 

2. 在最后一行并添加Python安装的路径，比如`;C:\Python36` 

3. 点击 `确定` 以完成操作。你不需要进行重启，不过你可能需要关闭并重启命令提示符

   ​

# 在Linux系统中安装

如果你使用的是Linux，很可能已经安装了Python。要确认这一点，可打开命令行窗口并输入python验证。对于 GNU/Linux 用户，你可以使用发行版的包管理器来安装 Python 3，例如在 Debian 与 Ubuntu 平台下，你可以输入命令：`sudo apt-get update && sudo apt-get install python3` 

要想验证安装是否成功，你可以通过打开 `Terminal` 应用或通过按下 `Alt + F2` 组合键并输入 `gnome-terminal` 来启动终端程序。如果这不起作用，请查阅你所使用的的 GNU/Linux 发行版的文档。现在，运行 `python3` 命令来确保其没有任何错误。如果输出与Mac一样，说明Python运行正常