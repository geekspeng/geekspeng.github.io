---
title: npm 模块全局安装的权限问题
date: 2017-09-01
updated: 2017-09-01
tags: [node,npm]
categories: [node]
---

安装全局 npm 模块报 EACCES 错误的问题，例如：

```
$ npm install -g coffee-script
```
因为缺省的 npm 全局安装目录(/usr/local/node_modules)没有给当前登录用户以写权限。
当然可以在前面加上 sudo 来提升用户权限，但其实还有更好的方法

你可以通过以下三种方式的任意一种解决这个问题:

1. 修改npm默认安装目录的权限
2. 修改npm默认安装目录
3. 借助第三方工具安装node，比如brew

<!-- more --> 

## 修改npm默认目录的权限

```
$ sudo chown -R $(whoami) $(npm config get prefix)/{lib/node_modules,bin,share}
```
## 修改npm默认安装目录

### 创建一个用于全局安装的目录

```
mkdir ~/.npm-global
```
### 修改npm默认安装目录


```
npm config set prefix '~/.npm-global'
```

### 打开或者创建 ~/.profile 文件并且添加下面的语句:

```
export PATH=~/.npm-global/bin:$PATH
```
### 更新系统变量

```
source ~/.profile
```

### 测试

```
npm install -g jshint
```

从此以后 npm install -g 安装的模块就都会到该用户名字下面的 ~/.npm-global 目录中，这样就做到了用户隔离。

## 借助第三方工具安装node
如果是Mac OS系统，则可以使用Homebrew软件包管理器完全避免此问题

```
brew install node
```

>  引用：https://docs.npmjs.com/getting-started/fixing-npm-permissions 



