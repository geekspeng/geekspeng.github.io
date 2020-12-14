---
title: 通过 libvirt 远程管理虚拟机
date: 2019-04-16
updated: 2019-04-16
tags: [Linux,虚拟化]
categories: [虚拟化]
---

# 通过qemu+ssh方式

通过qemu+ssh连接方式比较简单，只要能通过ssh远程访问，命令如下：

```bash
# virsh -c qemu+ssh://root@192.168.1.166/system
```
如果2个节点设置了互信，免密钥登录，可直接执行virsh相关命令，

```bash
# virsh -c qemu+ssh://root@192.168.1.166/system list
 Id    名称                         状态
----------------------------------------------------
 3     vm01                           running
```
<!-- more -->

# 通过qemu+tcp方式

被控端上：

修改/etc/sysconfig/libvirtd,开启以下2个配置项：

```bash
# egrep -v "^#|^$" /etc/sysconfig/libvirtd
LIBVIRTD_CONFIG=/etc/libvirt/libvirtd.conf
LIBVIRTD_ARGS="--listen
```
修改配置文件
```bash
# vim /etc/libvirt/libvirtd.conf
listen_tls = 0              #禁用tls登录
listen_tcp = 1              #启用tcp方式登录
tcp_port = "16509"          #tcp端口16509
listen_addr = "0.0.0.0"     #监听地址
auth_tcp = "none"           #TCP不使用认证
```
重启libvirtd并查看监听的端口

```bash
# systemctl restart libvirtd
# netstat -anltp|grep 16509
tcp   0      0 0.0.0.0:16509    0.0.0.0:*      LISTEN      28843/libvirtd
```
主控端上远程访问（需要确保可以访问被控端的16509 tcp端口）

```bash
# virsh -c qemu+tcp://192.168.1.166/system list
 Id    名称                         状态
----------------------------------------------------
 3     vm01                           running
```