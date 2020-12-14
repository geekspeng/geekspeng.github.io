---
title: 查看 CentOS 的版本号
date: 2018-09-01
updated: 2019-09-01
tags: [Linux,CentOS]
categories: [Linux]
---

# 查看 CentOS 的版本号

CentOS的版本号信息一般存放在配置文件当中，在CentOS中，与其版本相关的配置文件中都有centos关键字，该文件一般存放在/etc/目录下，所以说我们可以直接在该文件夹下搜索相关的文件

<!-- more -->

```bash
[root@node1 ~]# ll /etc/*centos*
-rw-r--r--. 1 root root 38 Apr 28  2018 /etc/centos-release
-rw-r--r--. 1 root root 51 Apr 28  2018 /etc/centos-release-upstream
```
其中存放其版本配置信息的文件为“centos-release”，翻译过来就是“CentOS的发行版”，所以说我们可以在这里查看CentOS相应的版本信息

```bash
[root@node1 ~]# cat /etc/centos-release
CentOS Linux release 7.5.1804 (Core)
```
# 查看CentOS内核版本

* 查看内核版本
```bash
[root@node1 ~]# uname -r
3.10.0-862.el7.x86_64
```
* 查看硬件架构
```bash
[root@node1 ~]# uname -i
x86_64
```
* 查看操作系统位数
```bash
[root@node1 ~]# getconf LONG_BIT
64
```

