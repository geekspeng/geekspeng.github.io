---
title: 使用virt-install安装虚拟机
date: 2019-04-18
updated: 2019-04-18
tags: [Linux,虚拟化]
categories: [Linux,虚拟化]
---

* 安装 virt-install

  ```bash
  # yum install -y virt-install
  ```

<!-- more -->

# 准备工作

安装 virt-install

```bash
# yum install -y virt-install
```
# ISO镜像安装虚拟机

* 在官方网站下载CentOS-7ISO镜像
* 使用qemu-img工具创建一个虚拟硬盘
```bash
qemu-img create -f qcow2 /tmp/centos7.qcow2 10G
```
注意qemu 用户需要有/tmp/centos7.qcow2 访问权限
* 以ISO文件作为cdrom，qcow2文件作为第一块虚拟硬盘，启动虚拟机
```bash
virt-install \
  --name centos7 \
  --ram 1024 \
  --vcpus 1 \
  --disk /tmp/centos7.qcow2,format=qcow2 \
  --network bridge=virbr0 \ 
  --graphics none \
  --os-type linux \
  --os-variant centos7.0 \
  --cdrom=CentOS-7-x86_64-Minimal-1804.iso
```
```bash
* 进入安装界面，进行相关配置，比如时区、键盘映射、语言等。
* 安装程序会识别虚拟机的虚拟硬盘，即qcow2文件，映射为/dev/vda，并进入分区引导界面。
* 分区完成后，开始拷贝操作系统所需要的文件。
* 用户自定义设置，包括创建用户、预装程序。
* 安装grub引导程序，退出重启，此时操作系统已经安装到qcow2虚拟硬盘中。
* 从硬盘启动，进入虚拟机，安装cloud-init、growpart、qemu-guest-agent等工具。
* 删除虚拟机，只需要保留qcow2虚拟硬盘文件，镜像制作完成。
```
# 从安装源安装

* 使用qemu-img工具创建一个虚拟硬盘
```bash
qemu-img create -f qcow2 /tmp/centos7.qcow2 10G
```
注意qemu 用户需要有/tmp/centos7.qcow2 访问权限
* 用 --location 代替 --cdrom 参数，qcow2文件作为第一块虚拟硬盘，启动虚拟机
```bash
virt-install \
--name centos7 \
--ram 1024 \
--vcpus 1 \
--disk /tmp/centos7.qcow2,format=qcow2 \
--os-type linux \
--os-variant centos7.0 \
--network bridge=virbr0 \
--graphics none \
--location 'http://mirror.i3d.net/pub/centos/7/os/x86_64/' 
```
# 直接启动 qow2 镜像

* 修改Cloud image的密码

image 下载地址[http://cloud.centos.org/centos/7/images/](http://cloud.centos.org/centos/7/images/)

安装软件 libguestfs-tools

```bash
# yum install libguestfs-tools
```
设置 root 密码

```bash
# virt-customize -a centos7.qcow2 --root-password password:123456
```
```bash
设置一个随机密码
# virt-customize -a centos7.qcow2 --root-password random
```
* 启动 qow2 镜像
```bash
virt-install \
--name centos7 \
--ram 1024 \
--vcpus 1 \
--disk ./centos7.qcow2,format=qcow2 \
--os-type linux \
--os-variant centos7.0 \
--network bridge=virbr0 \
--graphics none \
--boot hd
```
注意 qemu 用户需要有./centos7.qcow2 访问权限
--boot hd，从磁盘启动

* cloud image默认是不允许用 root以及密码进行登录，放开限制并重启sshd
```bash
# vi /etc/ssh/sshd_config
PermitRootLogin yes
PasswordAuthentication yes
# systemctl restart sshd
```
# 附录

* network

--network bridge=virbr0  # 桥接模式

--network network=default # 默认方式

* --graphics

--graphics none # 无界面

--graphics vnc,port=5999 # vnc控制台

* --os-variant 操作系统变异

使用 osinfo-query os 命令获得支持的操作系统变种的列表

```bash
osinfo-query os
 Short ID             | Name                                               | Version  | ID
----------------------+----------------------------------------------------+----------+----------------------------------------- .................. centos6.0            | CentOS 6.0                                         | 6.0      | http://centos.org/centos/6.0
 centos6.1            | CentOS 6.1                                         | 6.1      | http://centos.org/centos/6.1
 centos6.2            | CentOS 6.2                                         | 6.2      | http://centos.org/centos/6.2
 centos6.3            | CentOS 6.3                                         | 6.3      | http://centos.org/centos/6.3
 centos6.4            | CentOS 6.4                                         | 6.4      | http://centos.org/centos/6.4
 centos6.5            | CentOS 6.5                                         | 6.5      | http://centos.org/centos/6.5
 centos6.6            | CentOS 6.6                                         | 6.6      | http://centos.org/centos/6.6
 centos6.7            | CentOS 6.7                                         | 6.7      | http://centos.org/centos/6.7
 centos6.8            | CentOS 6.8                                         | 6.8      | http://centos.org/centos/6.8
 centos6.9            | CentOS 6.9                                         | 6.9      | http://centos.org/centos/6.9
 centos7.0            | CentOS 7.0                                         | 7.0      | http://centos.org/centos/7.0
 
 .................. ubuntu10.04          | Ubuntu 10.04 LTS                                   | 10.04    | http://ubuntu.com/ubuntu/10.04
 ubuntu10.10          | Ubuntu 10.10                                       | 10.10    | http://ubuntu.com/ubuntu/10.10
 ubuntu11.04          | Ubuntu 11.04                                       | 11.04    | http://ubuntu.com/ubuntu/11.04
 ubuntu11.10          | Ubuntu 11.10                                       | 11.10    | http://ubuntu.com/ubuntu/11.10
 ubuntu12.04          | Ubuntu 12.04 LTS                                   | 12.04    | http://ubuntu.com/ubuntu/12.04
 ubuntu12.10          | Ubuntu 12.10                                       | 12.10    | http://ubuntu.com/ubuntu/12.10
 ubuntu13.04          | Ubuntu 13.04                                       | 13.04    | http://ubuntu.com/ubuntu/13.04
 ubuntu13.10          | Ubuntu 13.10                                       | 13.10    | http://ubuntu.com/ubuntu/13.10
 ubuntu14.04          | Ubuntu 14.04 LTS                                   | 14.04    | http://ubuntu.com/ubuntu/14.04
 ubuntu14.10          | Ubuntu 14.10                                       | 14.10    | http://ubuntu.com/ubuntu/14.10
 ubuntu15.04          | Ubuntu 15.04                                       | 15.04    | http://ubuntu.com/ubuntu/15.04
 ubuntu15.10          | Ubuntu 15.10                                       | 15.10    | http://ubuntu.com/ubuntu/15.10
 ubuntu16.04          | Ubuntu 16.04                                       | 16.04    | http://ubuntu.com/ubuntu/16.04
 ubuntu16.10          | Ubuntu 16.10                                       | 16.10    | http://ubuntu.com/ubuntu/16.10
 ubuntu17.04          | Ubuntu 17.04                                       | 17.04    | http://ubuntu.com/ubuntu/17.04
 ubuntu17.10          | Ubuntu 17.10                                       | 17.10    | http://ubuntu.com/ubuntu/17.10 ..................
```
* virt-customize 常用命令

[https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/virtualization_deployment_and_administration_guide/sect-guest_virtual_machine_disk_access_with_offline_tools-using_virt_customize](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/virtualization_deployment_and_administration_guide/sect-guest_virtual_machine_disk_access_with_offline_tools-using_virt_customize)

安装 virt-customize

```bash
# yum install libguestfs-tools
```
安装或删除软件包

```bash
# virt-customize -a centos7.qcow2 --install epel-release
```
设置自己的SSH key

```bash
# virt-customize -a centos7.qcow2 --ssh-inject centos:string:"ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBKCqX6EZIrGHoGaMII4QAqr0QC72t+Kg/c5ZIRNTMb6Q+BwzejQgjhBTXeyPnp0rfE9XI4pTxkZqAUOGSK9Bfqg= smiller@bruckner"
```

设置 root 用户的密码为 daemon

```bash
virt-customize -acentos7.qcow2 --root-password password:daemon
```
报错

```bash
virt-customize: error: libguestfs error: could not create appliance through
libvirt.

Try running qemu directly without libvirt using this environment variable:
export LIBGUESTFS_BACKEND=direct

Original error from libvirt: Cannot access storage file
'/root/CentOS-7-aarch64-GenericCloud-2003.qcow2' (as uid:107, gid:107):
Permission denied [code=38 int1=13]

If reporting bugs, run virt-customize with debugging enabled and include
the complete output:

virt-customize -v -x [...]
```
解决方法

设置环境变量

```bash
export LIBGUESTFS_BACKEND=direct
```
