---
title: Linux虚拟网络基础
date: 2019-04-06
updated: 2019-04-06
tags: [Linux,网络]
categories: [Linux]
---

TAP/TUN 是 Linux 内核实现的一对虚拟网络设备，TAP 工作在二层虚拟以太设备，TUN 工作在三层

基于 TAP 驱动，即可实现虚拟机 vNIC 的功能，虚拟机的每个 vNIC 都与一个 TAP 设备相连，vNIC 之于 TAP 就如同 NIC 之于 eth

<!-- more -->

# TAP/TUN

TAP/TUN 是 Linux 内核实现的一对虚拟网络设备，TAP 工作在二层虚拟以太设备，TUN 工作在三层

基于 TAP 驱动，即可实现虚拟机 vNIC 的功能，虚拟机的每个 vNIC 都与一个 TAP 设备相连，vNIC 之于 TAP 就如同 NIC 之于 eth

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/spYkMDTpsJUD0VI8.png!thumbnail)

甚至连数据结构，tap与tun的定义都是同一个，两者仅仅是通过一个Flag来区分

当一个 TAP 设备被创建时，在 Linux 设备文件目录下会生成一个对应的字符设备文件，用户程序可以像打开一个普通文件一样对这个文件进行读写

Linux得有tun模块（Linux使用tun模块实现了tun/tap）

```bash
# modinfo tun
filename:       /lib/modules/4.14.37-4.el7.x86_64/kernel/drivers/net/tun.ko.xz
alias:          devname:net/tun
alias:          char-major-10-200
license:        GPL
author:         (C) 1999-2004 Max Krasnyansky <maxk@qualcomm.com>
description:    Universal TUN/TAP device driver
srcversion:     768D398B7DC8B10F73D9182
depends:
intree:         Y
name:           tun
vermagic:       4.14.37-4.el7.x86_64 SMP mod_unload modversions
```
当Linux版本具有tun模块时，还得看看其是否已经加载

```bash
# lsmod | grep tun
tun                    32768  21 vhost_net
```
如果没有加载，则使用如下命令进行加载：

```bash
# modprobe tun
```
创建 tap/tun 设备：

```bash
ip tuntap add dev tap0 mod tap # 创建 tap 
ip tuntap add dev tun0 mod tun # 创建 tun
```
删除 tap/tun 设备：

```bash
ip tuntap del dev tap0 mod tap # 删除 tap 
ip tuntap del dev tun0 mod tun # 删除 tun
```
VETH 设备总是成对出现，一端连着内核协议栈，另一端连着另一个设备，一个设备收到内核发送的数据后，会发送到另一个设备上去，这种设备通常用于容器中两个 namespace 之间的通信。
