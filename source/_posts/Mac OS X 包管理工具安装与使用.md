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



# 使用 homebrew-bundle 备份软件列表

**备份软件列表**

```bash
$ brew bundle dump --describe --force --file="~/Desktop/Brewfile"
```

参数说明：
- `--describe`：为列表中的命令行工具加上说明性文字。
- `--force`：直接覆盖之前生成的`Brewfile`文件。如果没有该参数，则询问你是否覆盖。
- `--file="~/Desktop/Brewfile"`：在指定位置生成文件。如果没有该参数，则在当前目录生成 `Brewfile` 文件。


**批量安装软件**

```bash
$ brew bundle --file="~/Desktop/Brewfile"
```

# 替换 Homebrew 源

默认官方的更新源都是存放在[GitHub](https://github.com/)上的，这也是中国大陆用户访问缓慢的原因，一般来说我们会更倾向选择国内提供的更新源，在此推荐[中国科大](https://mirrors.ustc.edu.cn/)以及[清华大学](https://mirrors.tuna.tsinghua.edu.cn/)提供的更新源。
Homebrew的更新源由三部分组成：本体（brew.git）、核心（homebrew-core.git）以及二进制预编译包（homebrew-bottles）。
从.git的后缀名可以看出，Homebrew的更新源是以Git仓库的形式存在的，所以需要用到Git。也正是如此，使得可以对其进行克隆，成为新源。

**配置镜像源**

```bash
# 替换brew.git:
$ cd "$(brew --repo)"
# 中国科大:
$ git remote set-url origin https://mirrors.ustc.edu.cn/brew.git
# 清华大学:
$ git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git

# 替换homebrew-core.git:
$ cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
# 中国科大:
$ git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git
# 清华大学:
$ git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git

# 替换homebrew-bottles:
# 中国科大:
$ echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles' >> ~/.bash_profile
$ source ~/.bash_profile
# 清华大学:
$ echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles' >> ~/.bash_profile
$ source ~/.bash_profile

# 应用生效:
$ brew update
```

# 重置 Homebrew 源

```bash
# 重置brew.git:
$ cd "$(brew --repo)"
$ git remote set-url origin https://github.com/Homebrew/brew.git

# 重置homebrew-core.git:
$ cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
$ git remote set-url origin https://github.com/Homebrew/homebrew-core.git
```

至于homebrew-bottles，推荐直接去修改`.bash_profile`文件删除 HOMEBREW_BOTTLE_DOMAIN 那一行。

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