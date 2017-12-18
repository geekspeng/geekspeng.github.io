---
title: 通过 nrm 自由切换 npm 源
date: 2017-09-01
updated: 2017-09-01
tags: [node,nrm]
categories: [node]
---

nrm可以快速地切换不同的npm registries，包括：npm，cnpm，taobao，nj（nodejitsu），rednpm

## 安装

```
$ npm install -g nrm
```
<!-- more -->

## 例子

### - nrm ls
星号表示当前使用的 registry

```
$ nrm ls

* npm ---- https://registry.npmjs.org/
  cnpm --- http://r.cnpmjs.org/
  taobao - https://registry.npm.taobao.org/
  nj ----- https://registry.nodejitsu.com/
  rednpm - http://registry.mirror.cqupt.edu.cn/
  npmMirror  https://skimdb.npmjs.com/registry/
  edunpm - http://registry.enpmjs.org/
```

### - nrm use

```
$ nrm use cnpm  //switch registry to cnpm
 
    Registry has been set to: http://r.cnpmjs.org/
```

## 用法


```
Usage: nrm [options] [command]
 
  Commands:
 
    ls                           List all the registries
    use <registry>               Change registry to registry
    add <registry> <url> [home]  Add one custom registry
    del <registry>               Delete one custom registry
    home <registry> [browser]    Open the homepage of registry with optional browser
    test [registry]              Show the response time for one or all registries
    help                         Print this help
 
  Options:
 
    -h, --help     output usage information
    -V, --version  output the version number
```

## npm registries
* npm
* cnpm
* nodejitsu
* taobao
* rednpm

## 注意
当您使用其他registry时，不能使用publish命令

 



