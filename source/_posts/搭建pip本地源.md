---
title: 搭建pip本地源
date: 2018-06-23
updated: 2018-06-23
tags: [python,pip]
categories: [python]
---

# 准备工作

* 安装 pip
```bash
# yum -y install python-pip
```
* 使用豆瓣pip源加快python包安装速度
```ini
# mkdir -p ~/.pip
# vim ~/.pip/pip.conf
[global]
index-url = https://pypi.doubanio.com/simple
[install]
trusted-host=pypi.doubanio.com
```
<!-- more -->

# 下载Python包并生成索引

## 创建软件包存放目录

```bash
# mkdir -p /home/pypi/packages
```
## 下载软件包

* 第一种方式：通过 pip 下载
```bash
# pip download simplejson -d /home/pypi/packages # 单个python包下载
# pip download -r requirements.txt -d /home/pypi/packages #批量下载
```
* 第二种方式：通过pip2pi下载软件包

安装 pip2pi

```bash
# pip install pip2pi
```
下载软件包

```bash
# pip2tgz /home/pypi/packages simplejson
# pip2tgz /home/pypi/packages -r pip_requirements.txt
```
* 生成软件包索引

软件包下载到本地文件系统后，需要为全部软件包生成索引（Index），这样pip在安装查询时可以快速判断指定的依赖软件包是否存在于本地pip源中

```bash
#  dir2pi --normalize-package-names /home/pypi/packages
```
dir2pi命令将会在 /home/pypi/packages 目录生成simple子目录，每个软件包在simple目录中都会生成对应子目录，目录名称为标准化后的软件包名。simple中每个以软件包名称命名的子目录下都会生成一个index.html文件

# 使用 pypiserver 搭建pip源

## pip 安装 pypiserver

```bash
# pip install pypiserver
```
## 启动 pypiserver

```bash
 # pypi-server -p 8080 /home/pypi/packages
```
注意，请确保端口号8080没有被占用，如被占用改用其他端口即可
## 升级目录下的所有包

```bash
# pypi-server -U /home/pypi/packages/
```
# 使用本地pip源安装软件

## 指定本地pip源安装

```bash
# pip install -i http://10.0.0.101/simple -r requirements.txt
```
## 配置本地pip源安装

* 配置本地pip源
```ini
# vim ~/.pip/pip.conf
[global]
index-url = http://10.0.0.101/simple
[install]
trusted-host=10.0.0.101
```
* 安装软件包
```bash
# pip install -r requirements.txt
```
