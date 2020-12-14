---
title: Centos7 必备设置和必装软件
date: 2017-12-18
updated: 2017-12-18
tags: [Linux,Centos]
categories: [Linux]
---

# 必备设置

## 安全设置（可选）

* 创建用户并赋予sudo权限
```bash
# id root # 查看 root 用户所属 group
# useradd -g 0 geekspeng  # 新建用户，-g 指明所属group,与root保持一致
# passwd geekspeng # 设置密码
# visudo  # 或者 vim /etc/sudoers 
文件内容改变如下： 
root ALL=(ALL) ALL 已有行 
geekspeng ALL=(ALL) ALL 新增行
```
<!-- more -->

* 限制远程登陆
```bash
# vim /etc/ssh/sshd_config 
PermitRootLogin no # 禁用root用户登录
# service sshd restart # 重启ssh服务以使更改生效
```
* 如果新用户设置了**免密登录（参考下一节）**也可以禁用密码登录（可选）
```bash
# vim /etc/ssh/sshd_config 
PasswordAuthentication no # 禁用密码登录
```
## 基础设置

* 免密登录
```bash
# ssh-copy-id -i ~/.ssh/id_rsa.pub user@host
```
* 修改 CentOS7 默认 yum 源
```bash
# mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup # 备份系统自带yum源配置文件
# curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo # 下载aliyun的yum源
# yum makecache # 生成缓存
```
>centos8
>curl -o /etc/yum.repos.d/CentOS-Base.repo[https://mirrors.aliyun.com/repo/Centos-8.repo](https://mirrors.aliyun.com/repo/Centos-8.repo)
>centos7 arm 架构
>curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-altarch-7.repo -O /etc/yum.repos.d/CentOS-Base.repo

[centos镜像-centos下载地址-centos安装教程-阿里巴巴开源镜像站 (aliyun.com)](https://developer.aliyun.com/mirror/centos?spm=a2c6h.13651102.0.0.3e221b11L5plr8)

* 安装EPEL的yum源
```bash
# yum -y install epel-release # 安装epel的yum源
# cd /etc/yum.repo.d/ # 进入yum源配置文件所在的文件夹
# ls # 安装完成后查看是否生成epel.repo和epel-testing.repo文件
```
epel.repo  #正式版，所有的软件都是稳定可以信赖的
epel-testing.repo #测试版，使用时需要慎重

* 修改EPEL的yum源
```bash
# mv /etc/yum.repos.d/epel.repo /etc/yum.repos.d/epel.repo.backup # 备份EPEL的yum源配置文件
# curl -o /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo # 下载aliyun EPEL的yum源
# yum makecache # 生成缓存
```
>centos8
>sed -i 's|^#baseurl=https://download.fedoraproject.org/pub|baseurl=https://mirrors.aliyun.com|' /etc/yum.repos.d/epel*
>sed -i 's|^metalink|#metalink|' /etc/yum.repos.d/epel*

* 使用豆瓣pip源
```ini
# mkdir -p ~/.pip
# vim ~/.pip/pip.conf
[global]
index-url = https://pypi.doubanio.com/simple
[install]
trusted-host=pypi.doubanio.com
```
# 必装软件

* net-tools、wget、vim、git
```bash
# yum -y install net-tools wget vim git bash-completion
```
* 如果提示没有pip，需要安装pip
```bash
# yum -y install python-pip
```
