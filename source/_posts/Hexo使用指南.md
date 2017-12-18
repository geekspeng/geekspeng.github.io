---
title: Hexo使用指南
date: 2017-09-01
updated: 2017-09-01
tags: [hexo]
categories: [hexo]
---

## 安装 Hexo

``` bash
$ npm install -g hexo-cli
```

## 建站
安装 Hexo 完成后，请执行下列命令，Hexo 将会在指定文件夹中新建所需要的文件。

``` bash
$ hexo init <folder>
$ cd <folder>
$ npm install
```
<!-- more --> 

新建完成后，指定文件夹的目录如下：

```
.
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
```

**scaffolds**
模版 文件夹。当您新建文章时，Hexo 会根据 scaffold 来建立文件。

Hexo的模板是指在新建的markdown文件中默认填充的内容。例如，如果您修改scaffold/post.md中的Front-matter内容，那么每次新建一篇文章时都会包含这个修改。

**source**
资源文件夹是存放用户资源的地方。除 _posts 文件夹之外，开头命名为 _ (下划线)的文件 / 文件夹和隐藏的文件将会被忽略。Markdown 和 HTML 文件会被解析并放到 public 文件夹，而其他文件会被拷贝过去。

**themes**
主题 文件夹。Hexo 会根据主题来生成静态页面。

**public**
hexo generate 生成的静态网页

## 常用命令

三部曲

``` bash
$ hexo clean
$ hexo generate
$ hexo deploy
```

### init

``` bash
$ hexo init [folder]
```
新建一个网站。如果没有设置 folder ，Hexo 默认在目前的文件夹建立网站。

### new

```
$ hexo new [layout] <title>
```
新建一篇文章。如果没有设置 layout 的话，默认使用 _config.yml 中的 default_layout 参数代替。如果标题包含空格的话，请使用引号括起来。

### generate

```
$ hexo generate
```
生成静态文件。

```
选项	描述
-d, --deploy	文件生成后立即部署网站
-w, --watch	监视文件变动
```

该命令可以简写为

``` bash
$ hexo g
```
### publish

```
$ hexo publish [layout] <filename>
```
发表草稿。

### server

```
$ hexo server
```
启动服务器。默认情况下，访问网址为： http://localhost:4000/。

```
选项	描述
-p, --port	重设端口
-s, --static	只使用静态文件
-l, --log	启动日记记录，使用覆盖记录格式
```
### deploy

```
$ hexo deploy
```
部署网站。

```
参数	描述
-g, --generate	部署之前预先生成静态文件
```
该命令可以简写为：

``` bash
$ hexo d
```
### clean

``` bash
$ hexo clean
```
清除缓存文件 (db.json) 和已生成的静态文件 (public)。

在某些情况（尤其是更换主题后），如果发现您对站点的更改无论如何也不生效，您可能需要运行该命令。

### render

``` bash
$ hexo render <file1> [file2] ...
```
渲染文件。

```
参数	描述
-o, --output	设置输出路径
```
### migrate

``` bash
$ hexo migrate <type>
```
从其他博客系统 迁移内容。

### list

```
$ hexo list <type>
```
列出网站资料。

### version

```
$ hexo version
```
显示 Hexo 版本。

## 主题
创建 Hexo 主题非常容易，您只要在 themes 文件夹内，新增一个任意名称的文件夹，并修改 _config.yml 内的 theme 设定，即可切换主题，当然也可以直接下载主题放到themes 文件夹内。一个主题可能会有以下的结构：


```
.
├── _config.yml
├── languages
├── layout
├── scripts
└── source
```

打开 站点配置文件， 找到 theme 字段，并将其值更改为 next


```
theme: next
```




