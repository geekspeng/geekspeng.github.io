---
title: 搭建 Node.js 开发环境
date: 2017-09-01
updated: 2017-09-01
tags: [node,nvm]
categories: [node]
---

* 如果你想长期做 node 开发, 或者想快速更新 node 版本, 或者想快速切换 node 版本, 那么在非 Windows(如 osx, linux) 环境下, 请使用 nvm 来安装你的 node 开发环境, 保持系统的干净。如果你使用 Windows 做开发, 那么你可以使用 nvmw 来替代 nvm

* nvm 的全称是 Node Version Manager，之所以需要这个工具，是因为 Node.js 的各种特性都没有稳定下来，所以我们经常由于老项目或尝新的原因，需要切换各种版本

<!-- more --> 

## 安装nvm

```
$ curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.30.2/install.sh | bash
```

## 查看nvm安装是否成功，输入nvm


```
$ nvm

Node Version Manager

Note: <version> refers to any version-like string nvm understands. This includes:
  - full or partial version numbers, starting with an optional "v" (0.10, v0.1.2, v1)
  - default (built-in) aliases: node, stable, unstable, iojs, system
  - custom aliases you define with `nvm alias foo`

Usage:
  nvm help                                  Show this message
  nvm --version                             Print out the latest released version of nvm
  nvm install [-s] <version>                Download and install a <version>, [-s] from source. Uses .nvmrc if available
    --reinstall-packages-from=<version>     When installing, reinstall packages installed in <node|iojs|node version number>
  nvm uninstall <version>                   Uninstall a version
  nvm use [--silent] <version>              Modify PATH to use <version>. Uses .nvmrc if available
  nvm exec [--silent] <version> [<command>] Run <command> on <version>. Uses .nvmrc if available
  nvm run [--silent] <version> [<args>]     Run `node` on <version> with <args> as arguments. Uses .nvmrc if available
  nvm current                               Display currently activated version
  nvm ls                                    List installed versions
  nvm ls <version>                          List versions matching a given description
  nvm ls-remote                             List remote versions available for install
  nvm version <version>                     Resolve the given description to a single local version
  nvm version-remote <version>              Resolve the given description to a single remote version
  nvm deactivate                            Undo effects of `nvm` on current shell
  nvm alias [<pattern>]                     Show all aliases beginning with <pattern>
  nvm alias <name> <version>                Set an alias named <name> pointing to <version>
  nvm unalias <name>                        Deletes the alias named <name>
  nvm reinstall-packages <version>          Reinstall global `npm` packages contained in <version> to current version
  nvm unload                                Unload `nvm` from shell
  nvm which [<version>]                     Display path to installed node version. Uses .nvmrc if available

Example:
  nvm install v0.10.32                  Install a specific version number
  nvm use 0.10                          Use the latest available 0.10.x release
  nvm run 0.10.32 app.js                Run app.js using node v0.10.32
  nvm exec 0.10.32 node app.js          Run `node app.js` with the PATH pointing to node v0.10.32
  nvm alias default 0.10.32             Set default node version on a shell

Note:
  to remove, delete, or uninstall nvm - just remove the `$NVM_DIR` folder (usually `~/.nvm`)
```

## 通过 nvm 安装任意版本的 node，可以指定版本号，或者用stable(稳定版)代替


```
$ nvm install stable
Downloading https://nodejs.org/dist/v8.4.0/node-v8.4.0-darwin-x64.tar.xz...
######################################################################## 100.0%
WARNING: checksums are currently disabled for node.js v4.0 and later
nvm is not compatible with the npm config "prefix" option: currently set to "/Users/geekspeng/npm-global"
Run `nvm use --delete-prefix v8.4.0` to unset it.
```

## 通过 nvm 安装任意版本的 iojs

```
$ nvm install iojs
Downloading https://iojs.org/dist/v3.3.1/iojs-v3.3.1-darwin-x64.tar.xz...
######################################################################## 100.0%
WARNING: checksums are currently disabled for io.js
Now using io.js v3.3.1 (npm v2.14.3)
```

## 查看安装的node，箭头指向的就是当前使用的node版本

```
$ nvm ls
->       v8.4.0
default -> stable (-> v8.4.0)
node -> stable (-> v8.4.0) (default)
stable -> 8.4 (-> v8.4.0) (default)
iojs -> N/A (default)
```
## nvm常用命令

```
nvm install v0.10.32                  Install a specific version number
nvm use 0.10                          Use the latest available 0.10.x release
nvm run 0.10.32 app.js                Run app.js using node v0.10.32
nvm exec 0.10.32 node app.js          Run `node app.js` with the PATH pointing to node v0.10.32
nvm alias default 0.10.32             Set default node version on a shell
```


## 使用 cnpm 加速 npm
同理 nvm , npm 默认是从国外的源获取和下载包信息, 所以可能会比较慢. 可以通过简单的 ---registry 参数, 使用国内的镜像 http://registry.npm.taobao.org


```
$ npm install koa --registry=http://registry.npm.taobao.org
```

毕竟镜像跟官方的 npm 源还是会有一个同步时间差异, 目前 cnpm 的默认同步时间间隔是 10 分钟. 如果你是模块发布者, 或者你想马上同步一个模块, 那么推荐你安装 cnpm cli

```
$ npm install cnpm -g --registry=http://registry.npm.taobao.org
```
通过 cnpm 命令行, 你可以快速同步任意模块

```
$ cnpm sync koa connect mocha
```
例如我想马上同步 koa, 直接打开浏览器: http://npm.taobao.org/sync/koa
或者在命令行中通过 open 命令同步

```
$ open http://npm.taobao.org/sync/koa
```

## 当开启一个新的 shell 窗口时，找不到 node 命令的情况

### 这种情况一般来自两个原因
1、shell 不知道 nvm 的存在

2、nvm 已经存在，但是没有 default 的 Node.js 版本可用。

### 解决方式

1、检查 ~/.profile 或者 ~/.bash_profile 中有没有下面的语句，没有的话就通过vim添加进入

```
export NVM_DIR="/Users/geekspeng/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads 
nvm bash_completion
```

**注意：**

* ~/.bashrc虽然有上面的语句但是每次新开命令行都要source ~/.bashrc，按理来说修改后source一次后面就不需要souce了
* ~/.bashrc包含专用于你的bash shell的bash信息,当登录时以及每次打开新的shell时,该文件被读取）


2、 调用nvm ls 查看default 的指向

```
$ nvm ls 
->       v8.4.0
default -> stable (-> v8.4.0)
node -> stable (-> v8.4.0) (default)
stable -> 8.4 (-> v8.4.0) (default)
iojs -> N/A (default)
```
如果default没有指向的话，执行nvm alias default stable指定版本，执行完后再查看下

```
$ nvm alias default stable
```



