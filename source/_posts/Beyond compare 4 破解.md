---
title: Beyond compare 4 破解
date: 2019-02-14
updated: 2019-02-14
tags: [工具,破解,Beyond compare]
categories: [工具]
---

# 密钥
```
w4G-in5u3SH75RoB3VZIX8htiZgw4ELilwvPcHAIQWfwfXv5n0IHDp5hv 1BM3+H1XygMtiE0-JBgacjE9tz33sIh542EmsGs1yg638UxVfmWqNLqu- Zw91XxNEiZF7DC7-iV1XbSfsgxI8Tvqr-ZMTxlGCJU+2YLveAc-YXs8ci RTtssts7leEbJ979H5v+G0sw-FwP9bjvE4GCJ8oj+jtlp7wFmpVdzovEh v5Vg3dMqhqTiQHKfmHjYbb0o5OUxq0jOWxg5NKim9dhCVF+avO6mDeRNc OYpl7BatIcd6tsiwdhHKRnyGshyVEjSgRCRY11IgyvdRPnbW8UOVULuTE
```
<!-- more -->

# Beyond Compare许可证密钥已被撤销解决办法

## windows

1. 打开如下文件夹的目录：C:\Users\用户文件夹\AppData\Roaming\Scooter Software\Beyond Compare 4
1. 然后删掉该文件夹下 BCState.xml 和 BCState.xml.bak

![image-20190216173135520](/images/image-20190216173135520.png)

3. 然后重新打开Beyond Compare**（无需重新安装，还双击原来的图标）**

>提示：
>如果不行就删除所有文件，重新试用Beyond Compare 4
>双击后，它会提示你重新安装，然后点击下一个，下一个下一个，ok，搞定

## mac

1. 打开如下文件夹的目录：
```
$cd "/Users/$(whoami)/Library/Application Support/Beyond Compare/"
```
2. 然后删掉该文件夹下 BCState.xml、BCState.xml.bak 和 registry.dat

![image-20190216173223415](/images/image-20190216173223415.png)

也可以直接用命令直接删除
```
$ rm "/Users/$(whoami)/Library/Application Support/Beyond Compare/BCState.xml"
$ rm "/Users/$(whoami)/Library/Application Support/Beyond Compare/BCState.xml.bak"
$ rm "/Users/$(whoami)/Library/Application Support/Beyond Compare/registry.dat"
```
3. 然后重新打开Beyond Compare**（无需重新安装，还双击原来的图标）**

>提示：
>如果不行就删除所有文件，重新试用Beyond Compare 4
>双击后，它会提示你重新安装，然后点击下一个，下一个下一个，ok，搞定

**为了一劳永逸可以直接修改****Beyond Compare 4 启动程序，在每次启动的时候自动删除**
**BCState.xml 和 BCState.xml.bak**

1. 进入Mac应用程序目录下，找到刚刚安装好的Beyond Compare，路径如下/Applications/Beyond Compare.app/Contents/MacOS
2. 修改启动程序文件BCompare为BCompare.real
3. 在当前目录下新建一个文件BCompare，文件内容如下：
```
#!/bin/bash
rm "/Users/$(whoami)/Library/Application Support/Beyond Compare/BCState.xml"
rm "/Users/$(whoami)/Library/Application Support/Beyond Compare/BCState.xml.bak"
rm "/Users/$(whoami)/Library/Application Support/Beyond Compare/registry.dat"
"`dirname "$0"`"/BCompare.real $@
```
4. 修改文件的权限
```
$ chmod a+x "/Applications/Beyond Compare.app/Contents/MacOS/BCompare"
```

