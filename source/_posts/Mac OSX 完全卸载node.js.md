---
title: Mac OSX 完全卸载node.js
date: 2017-09-01
updated: 2017-09-01
tags: [node,mac]
categories: [node]
---

## 删除/usr/local/lib中的所有node和node_modules的文件夹

```
$ cd /usr/local/lib
$ sudo rm -rf node
$ sudo rm -rf node_modules
```

<!-- more --> 

## 如果是从brew安装的, 运行brew uninstall node
## 检查~/中所有的local，lib或者include文件夹, 删除里面所有node和node_modules

```
$ cd ~/local
$ sudo rm -rf node
$ sudo rm -rf node_modules

$ cd ~/lib
$ sudo rm -rf node
$ sudo rm -rf node_modules

$ cd ~/include
$ sudo rm -rf node
$ sudo rm -rf node_modules
```
## 在/usr/local/bin中, 删除所有node的可执行文件

```
$ cd /usr/local/bin
$ sudo rm -rf node
$ sudo rm -rf node_modules
```
## 最后运行以下代码:

```
$ sudo rm /usr/local/bin/npm
$ sudo rm /usr/local/share/man/man1/node.1
$ sudo rm /usr/local/lib/dtrace/node.d
$ sudo rm -rf ~/.npm
$ sudo rm -rf ~/.node-gyp
$ sudo rm /opt/local/bin/node
$ sudo rm /opt/local/include/node
$ sudo rm -rf /opt/local/lib/node_modules
```



