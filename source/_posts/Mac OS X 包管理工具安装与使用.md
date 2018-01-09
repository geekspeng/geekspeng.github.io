---
title: Mac OS X 包管理工具 Homebrew 安装与使用
date: 2018-01-09
updated: 2018-01-09
tags: [Mac,包管理工具,Homebrew]
categories: [Mac]
---

Homebrew 是Mac OS 下的包管理工具，类似于Ubuntu下的apt-get命令，通过这个工具我们可以快速获取所需要的软件而不需要像在Windows系统中那样打开浏览器，找到需要下载的安装包，然后才能进行下载。Homebrew拥有安装、卸载、更新、查看、搜索等很多实用的功能。通过一条简单的指令，就可以实现包管理，而不用你关心各种依赖和文件路径的情况，十分方便快捷。
<!-- more -->

# Homebrew 能干什么?

1. 使用 Homebrew 安装 Apple 没有预装但 [你需要的东西](https://github.com/Homebrew/homebrew-core/tree/master/Formula)
2. Homebrew 会将软件包安装到独立目录，并将其文件软链接至 `/usr/local`
3. Homebrew 不会将文件安装到它本身目录之外，所以您可将 Homebrew 安装到任意位置
4. 轻松创建你自己的 Homebrew 包
5. 完全基于 git 和 ruby，所以自由修改的同时你仍可以轻松撤销你的变更或与上游更新合并
6. Homebrew 的配方都是简单的 Ruby 脚本
7. Homebrew 使 macOS 更完整。使用 `gem` 来安装 gems、用 `brew` 来安装那些依赖包



# Homebrew 安装

```ruby
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

**安装homebrew后报错-bash: brew: command not found 的解决方法**

其实解决这个问题真的很简单。如下：

```bash
vim ~/.bash_profile
```

添加下面一行

```bash
export PATH=/usr/local/bin:$PATH
```

保存，执行下面命令使配置生效

```bash
source ~/.bash_profile
```

重新打开命令行工具，再次使用brew 命令就ok了



# Homebrew 基本使用

**安装任意包**

```bash
$ brew install <package_name>  
```

**卸载任意包**

```bash
$ brew uninstall <packageName>
```

**更新 Homebrew 在服务器端上的包目录**

```bash
$ brew update  
```

**查看你的包是否需要更新**

```bash
$ brew outdated  
```

**更新包**

```bash
$ brew upgrade <package_name>  
```

**查询可用的包**

```bash
$ brew search <packageName>
```

**查看你安装过的包列表（包括版本号）**

```bash
$ brew list --versions  
```

**查看任意包信息**

```bash
$ brew info <packageName>
```

**查看帮助信息**

```bash
$ brew -h
```

**Homebrew 将会把老版本的包缓存下来，以便当你想回滚至旧版本时使用。但这是比较少使用的情况，当你想清理旧版本的包缓存时，可以运行：**

```bash
$ brew cleanup  
```



# Homebrew 镜像

由于Homebrew 下载源基本在国外，因此在中国的开发者下载速度可能会比较慢，针对这个问题，有一些人为国内的开发者做了一个Homebrew 镜像，比如http://ban.ninja/

**配置镜像源**

设置环境变量 HOMEBREW_BOTTLE_DOMAIN 即可使用本镜像源加速下载 Homebrew 资源。

**bash**

打开 `~/.bashrc`

```bash
sudo vim ~/.bashrc
```

在 `~/.bashrc` 中加入

```bash
export HOMEBREW_BOTTLE_DOMAIN=http://7xkcej.dl1.z0.glb.clouddn.com
```

**说明**

本镜像源只镜像了 Homebrew 托管在 Bintray 上的二进制预编译包，所以只对这些二进制包有加速功能（Homebrew 大部分情况下使用该渠道下载安装软件）。



#  Homebrew Cask

Homebrew Cask 由社区进行维护，因此它有更多，更丰富的软件，我们可以通过Homebrew Cask 优雅、简单、快速的安装和管理 OS X 图形界面程序，比如 Google Chrome 和 Evernote。

**安装**

安装 Homebrew-cask 是如此的简单直接，运行以下命令即可完成：

```bash
$ brew tap caskroom/cask  // 添加 Github 上的 caskroom/cask 库
$ brew install brew-cask  // 安装 brew-cask
$ brew update && brew upgrade brew-cask && brew cleanup // 更新  
$ brew cask install google-chrome // 安装 Google 浏览器
```

**搜索**

如果你想查看 cask 上是否存在你需要的 app，可以到 [caskroom.io](http://caskroom.io/)进行搜索。

**文件预览插件**

有些 [插件](https://github.com/sindresorhus/quick-look-plugins) 可以让 Mac 上的文件预览更有效，比如语法高亮、markdown 渲染、json 预览等等。

```bash
$ brew cask install qlcolorcode
$ brew cask install qlstephen
$ brew cask install qlmarkdown
$ brew cask install quicklook-json
$ brew cask install qlprettypatch
$ brew cask install quicklook-csv
$ brew cask install betterzipql
$ brew cask install webp-quicklook
$ brew cask install suspicious-package   
```

**OS X 图形界面程序**

```bash
$ brew cask install alfred
$ brew cask install appcleaner
$ brew cask install cheatsheet
$ brew cask install dropbox
$ brew cask install google-chrome
$ brew cask install onepassword
$ brew cask install sublime-text
$ brew cask install totalfinder
...  
```



#  Cakebrew

Mac下Homebrew的图形化界面工具[Cakebrew](https://www.cakebrew.com/)

**安装**

```bash
$ brew cask install cakebrew
```

如果不能下载直接上[官网](https://www.cakebrew.com/)下载dmg包进行安装