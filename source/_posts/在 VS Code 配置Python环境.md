---
title: 在 VS Code 配置Python环境
date: 2018-04-12
updated: 2018-04-12
tags: [vscode,python]
categories: [python]
---

默认情况下，Python 扩展寻找并使用它在系统路径中找到的第一个 Python 解释器。 如果它没有找到解释器，它就会发出警告。 在 macOS 上，如果您使用的是安装在 os 上的 Python 解释器，扩展还会发出警告，因为您通常希望使用直接安装的解释器。 无论哪种情况，都可以通过在用户设置中将 python.disableInstallationCheck 设置为 true 来禁用这些警告。

<!-- more -->

# 配置Python环境

>默认情况下，Python 扩展寻找并使用它在系统路径中找到的第一个 Python 解释器。 如果它没有找到解释器，它就会发出警告。 在 macOS 上，如果您使用的是安装在 os 上的 Python 解释器，扩展还会发出警告，因为您通常希望使用直接安装的解释器。 无论哪种情况，都可以通过在用户设置中将 python.disableInstallationCheck 设置为 true 来禁用这些警告。
1. 要选择的解释器，请从命令选项板（⇧⌘P）调用Python：Select Interpreter命令

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/bMCUA556OYUMoXXt.png!thumbnail)

也可以单击状态栏选择解释器

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/M4xFEdaCXBE3OW30.png!thumbnail)

2. Python: Select Interpreter 命令显示可用的全局环境、 conda 环境和虚拟环境的列表

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/b88B7rOfjgoXgb5m.png!thumbnail)

>注意: 在 Windows 上，VS 代码需要一点时间来检测可用的 conda 环境。 在此过程中，您可能会在环境的路径之前看到"(缓存)"。 该标签表明 VS Code 目前正在处理该环境的缓存信息。
3. 从列表中选择的解释器后会自动添加 python.pythonPath （解释器的路径）条目到工作区设置中。

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/yCxCyWYBU50EjygS.png!thumbnail)

4. 状态栏始终显示当前解释器

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/aOQ1pb08l0c6jlWL.png!thumbnail)

## 激活终端中的Python环境

在编辑器中打开一个 .py 文件，并用 Terminal: Create New Integrated Terminal 命令打开终端，VS Code 会自动激活选定的Python环境。

>提示: 为了防止自动激活选定的环境，可以在 settings.json 文件中添加"python.terminal.activateEnvironment": false

使用 Python: Select Interpreter 命令更改解释器不会影响已经打开的终端面板。

>提示: 在某个 Python 环境被激活的 shell 中启动 VS Code 并不会自动激活默认集成终端中选定的环境。 使用终端: 在运行 VS Code后创建新的集成终端命令即可自动激活选定的Python环境
## 选择Python调试环境

默认情况下，Python.pythonPath 设置指定用于调试的 Python 解释器。 但是，如果在 launch.json 的调试配置中有 pythonPath 属性，则使用该解释器。

在决定使用哪个解释器进行调试时，VS Code 应用以下优先顺序:

* launch.json 中选定的调试配置 pythonPath 属性
* 工作区 settings.json 中的 python.pythonPath 属性
* 用户 settings.json 中的 python.pythonPath 属性
# 查找Python环境的位置

Python扩展程序会自动在以下位置查找解释器

* 标准安装路径，例如/usr/local/bin，/usr/sbin，/sbin，c:\\python27，c:\\python36，等
* 工作空间（项目）文件夹下的虚拟环境
* 工作空间（项目）文件夹下的 .direnv 中的解释器
* 工作空间（项目）文件夹下的 pipenv 环境。如果找到一个解释器，那么就不会搜索其他解释器，因为pipenv希望管理环境的所有方面
* pyenv 安装的解释器
* conda环境中的Python解释器
* python.venvPath设置的文件夹中的虚拟环境，可以包含多个虚拟环境。 该扩展程序在venvPath的第一级子文件夹中查找虚拟环境

如果VS Code未自动找到要使用的解释器，则可以在工作区 settings.json文件中手动设置路径

1. 打开设置，选择工作区
2. 添加或修改python.pythonPath的条目，填写Python可执行文件的完整路径（如果直接编辑settings.json，请将以下行添加到设置中）：
```
"python.pythonPath": "/home/python36/python",
```
可以使用 ${ env: VARIABLE }的语法在路径设置中使用环境变量

例如，如果你创建了一个名为 PYTHON_INSTALL_LOC 的变量，它有一个解释器的路径，你可以使用下面的设置值:

```
"python.pythonPath": "${env:PYTHON_INSTALL_LOC}",
```
通过使用环境变量，你可以很容易地使用不同路径在操作系统之间转移一个项目，只是在操作系统上设置环境变量即可。
# 环境变量定义文件

环境变量定义文件是一个简单的文本文件，定义环境变量采用键值对的形式environment_variable=value，不支持多行值，并用 # 注释

扩展会加载由 python.envFile 设置的环境变量定义文件。 此设置的默认值为 ${workspaceFolder}/.env，即当前工作空间（项目）中的.env文件

>可以修改用户设置settings.json 中的 python.envFile 配置
>

调试配置还包含一个envFile属性，也默认为当前工作空间中的.env文件

在开发 web 应用程序时，可能在开发服务器和生产服务器之间切换

* 可以将 python.envFile 设置为 ${workspaceolder}/prod.env
* 在调试配置中将 envFile 属性设置为 ${workspaceolder}/dev.env
## 变量替换

在定义文件中定义一个环境变量时，你可以使用任何现有环境变量的值，使用以下一般语法:

```
<VARIABLE>=...${EXISTING_VARIABLE}...
```
其中... 表示值中的包含的其他文本
适用以下规则

* 使用前必须先定义变量
* 单引号或双引号不会影响替换值，并包含在定义的值中
* $字符可以使用反斜杠进行转义，如\ $
* 只能使用简单的替换; 不支持嵌套
    * 例如${_${OTHERVAR}_EX}
* 可以使用递归替换
    * 例如PYTHONPATH=${PROJ_DIR}:${PYTHONPATH}（其中PROJ_DIR是任何其他环境变量）
* 语法不支持的条目保持原样
# PYTHONPATH 变量的使用

PYTHONPATH  环境变量指定了 Python 解释器应该寻找模块的其他位置。 PYTHONPATH 的值可以包含由 os.pathsep (Windows 上的分号，linux / macos 上的冒号)分隔的多个路径值，并且会忽略无效路径

>注意: 必须通过操作系统设置 PYTHONPATH 变量，因为 VS Code 没有提供直接设置环境变量的方法。

建议将 PYTHONPATH 变量设置在环境变量定义文件中

>注意: PYTHONPATH 不指定 Python 解释器本身的路径，因此不能在 Python.PYTHONPATH 设置中使用它
