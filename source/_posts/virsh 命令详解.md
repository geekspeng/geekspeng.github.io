---
title: virsh 命令详解
date: 2019-04-24
updated: 2019-04-24
tags: [Linux,虚拟化]
categories: [Linux,虚拟化]
---

# 安装

```bash
# yum install qemu-kvm libvirt virt-install virt-manager
```
# 命令列表

## 版本信息

* virsh-v  只显示版本号
* virsh-V  显示版本的详细信息

<!-- more -->

## 查看实例

* virsh list查看活动的虚拟机
* virsh list --all  查看所有的虚拟机（关闭和运行的虚拟机）

虚拟机的状态有（8）种

* runing 是运行状态
* idel 是空闲状态
* pause 暂停状态
* shutdown 关闭状态
* shut off 不运行完全关闭
* crash 虚拟机崩坏状态
* daying 垂死状态
* pmsuspended 客户机被关掉电源中中断
## 连接方式

qemu:///system (本地连接到系统实例)

qemu+unix:///system(本地连接到系统实例)

qemu://example.com/system(远程连接，TLS)

qemu+tcp://example.com/system(远程登录，SASI)

qemu+ssl://example.com/system(远程登录，SSL)

-c - -connect 连接远程的主机

-l - -log 输出日志

-q - -quiet避免额外的信息

-r - - readonly 只读，一般和connect配合使用

-t - - timing 输出消逝的时间

-e - - escape 设置转意序列

## domain命令

### 创建、连接虚拟机

* define（file）从文件定义一个虚拟机（但是不启动）
* undefine （file） 取消定义的虚拟机
* create （file）：从文件创建虚拟机
* console （domain）：连接虚拟机的控制台
### 查看虚拟机状态

* dominfo（domian）显示虚拟机的信息
* domname（idorUUID）显示虚拟机的名字
* domuuid （domian）显示虚拟机的id
* domid（id or name ） 根据名字得到id
* domstate（domian） 显示虚拟机的状态
* domstats（domian） 显示虚拟机的统计信息
* domcontrol(domian) 显示虚拟机的控制接口状态ok or error
* domtime（domian） 显示虚拟机的时间
* dommemstat（domain）获取虚拟机的内存统计信息
* domhostname（domain）显示虚拟机的主机名
* dump （domian file） 把文件配置输出到文件file
* dumpxml（domian）直接显示domian的xml文件配置
* domblklist（domain）显示虚拟机的磁盘
* domblkerror（domian）显示有错的磁盘
* domblkinfo（domaindevice）显示磁盘大小信息
* domblkstat（domaindevice）显示磁盘统计信息
* domiflist（domain）显示网卡接口
* domifaddr（domianinterface） 显示网络接口地址
* domif-getlink（domianinterface） 显示虚拟接口的链接状态
* domifstat（domianinterface） 显示网卡统计信息
* vcpuinfo（domian） 显示cpu的信息
* vcpucount （domian）显示 vcpu个数
* cpu-stats （domian） 显示虚拟机的cpu统计信息
* domdisplay （domian）显示地址和显卡
* vncdisplay（domian） 显示虚拟机的vnc 信息
* ttyconsole （domian） 显示设备用的终端
* domjobinfo （domian） 显示虚拟机的任务
* domjobabort（domian） 终止虚拟机活动的任务
* memtune（domian） 获取或设置虚拟机内存参数
* blkdeviotune（domian） 获取或设置虚拟机的块设备IO参数
* blkiotune（domian） 获取或设置虚拟机的磁盘参数
* domiftune（domian） 获取或设置虚拟机的虚拟接口参数
* metadata（domian） 获取或设置虚拟机的metadata
* schedinfo（domian） 获取或设置虚拟机的调度信息
### 管理虚拟机

* reset（domian）reset虚拟机
* reboot（domian） 重启虚拟机
* start（name or id） 开启虚拟机
* shutdown（domian） 关闭虚拟机(soft shutdown)
* destroy（domain） 销毁虚拟机（Hard poweroff a physical machine）
* suspend （domian） 挂起虚拟机
* resume（domian） 回复虚拟机的suspend状态
* dompmsuspend（<domain> <target>） 挂起虚拟机（电源功能）
* dompmwakeup（<domain>） 回复虚拟机的suspend状态（电源功能）
* save（<domain><file>）保存虚拟机状态到文件中
* restore（<file>）从保存状态的文件还原虚拟机
* managedsave（domian）托管保存虚拟机状态
* managedsave-remove（domian） 移除托管保存虚拟机状态
* managedsave-edit（domian）编辑托管保存状态的XML文件
* managedsave-dumpxml（domian）以XML文件格式显示托管保存状态文件
* managedsave-define（domianxml）用托管保存状态文件替换与之相关联的虚拟机的XML文件
* autostart （domain）：标示自动启动虚拟机
* screenshot （domian） 虚拟机截屏
### 修改虚拟机配置

* set-user-password（domianuser password）设置虚拟机用户密码
* domrename（domiannew-name） 重新命名虚拟机
* setvcpus（domian count）设置虚拟机的虚拟cpu个数
* setmaxmen（domian）设置虚拟机的最大内存
* setmen（domian） size 设置虚拟机的内存
* vcpupin（domian）绑定虚拟机的vcpu与物理CPU
* emulatorpin（domian）绑定虚拟机的仿真器与物理CPU
* edit （domian） 编辑虚拟机的配置文件
* domif-setlink                  set link state of a virtual interface
### 虚拟机文件系统

* domfsfreeze                    Freeze domain's mounted filesystems.
* domfsthaw                      Thaw domain's mounted filesystems.
* domfsinfo                      Get information of domain's mounted filesystems.
* domfstrim                      Invoke fstrim on domain's mounted filesystems.
### 虚拟机迁移

* migrate                        migrate domain to another host
* migrate-setmaxdowntime         set maximum tolerable downtime
* migrate-getmaxdowntime         get maximum tolerable downtime
* migrate-compcache              get/set compression cache size
* migrate-setspeed               Set the maximum migration bandwidth
* migrate-getspeed               Get the maximum migration bandwidth
* migrate-postcopy               Switch running migration from pre-copy to post-copy
### 虚拟机 attach 和 detach

* attach-device                  attach device from an XML file
* attach-disk                    attach disk device
* attach-interface               attach network interface
* detach-device                  detach device from an XML file
* detach-device-alias            detach device from an alias
* detach-disk                    detach disk device
* detach-interface               detach network interface
* update-device                  update device from an XML file
### 虚拟机 image

* save-image-define              redefine the XML for a domain's saved state file
* save-image-dumpxml             saved state domain information in XML
* save-image-edit                edit XML for a domain's saved state file
### 虚拟机 block

* blockcommit                    Start a block commit operation.
* blockcopy                      Start a block copy operation.
* blockjob                       Manage active block operations
* blockpull                      Populate a disk from its backing image.
* blockresize                    Resize block device of domain.
## virtual network 命令

* net-list 显示网络
* net-info（<network>）显示网络信息
* net-uuid （<network>）显示网络的id
* net-name（<id>）显示网络的名字
* net-dumpxml（<network>） 以XML形式显示网络信息
* net-dhcp-leases（<network>）显示 dhcp 租约信息
* net-define （<file>） 从XML文件定义网卡
* net-undefine （<network>） 取消定义的网卡
* net-create （<file>） 从XML文件创建网卡
* net-edit（<network>） 编辑网卡信息
* net-update（<network> <command> <section> <xml>） 更新网卡配置
* net-autostart (<network>) 自动启动网卡
* net-start （<network>）开启网卡
* net-destory ( <network>) 摧毁（停止）网卡
## interface 命令

* iface-list 列出所有的接口
* iface-name （mac） 根据mac得到名字
* iface-mac(interface) 根据名字得到mac
* iface-dumpxml （interface）显示接口的信息
* iface-define（file）从XML文件定义接口
* iface-undefine（interface） 取消定义的接口
* iface-edit（interface） 编辑接口
* iface-start（interface）开启接口 (enable it / "if-up")
* iface-destroy（interface） 摧毁（停止）接口（disable it / "if-down"）
* iface-bridge 创建 bridge 设备并将现有网络设备连接到它
* iface-unbriged 解绑定网桥
* iface-begin（interface）创建当前接口设置的快照，可以稍后提交（iface-commit）或恢复（iface-rollback）
* iface-commit （interface）提交自 iface-begin 和 iface-rollbak以来所做的更改
* iface-rollbak （interface）回滚到通过iface-begin创建的先前保存的配置
## storage pool 命令

* pool-list 的列表
* pool-info 池的信息
* pool-name（id）根据id得到name
* pool-id（name）根据name得到id
* pool-uuid （pool） 返回一个池的uuid
* pool-define（file）定义但是不开启
* pool-create（file）根据文件创建池
* pool-build（pool）建造一个池
* pool-start（poop）开启池
* pool-destory（pool）销毁池，以后能回复
* pool-delete（pool）删除池，以后不能恢复
* pool-auto （pool）标记池自动启动
* pool-dumpxml（pool）查看池的定义文件
* pool-edit（pool）编辑池的定义文件
## volume 命令

* vol-list（pool）列出卷
* vol-info（pool）卷的信息
* vol-name（path）得到卷的名字
* vol-path                       returns the volume path for a given volume name or key
* vol-pool                       returns the storage pool for a given volume key or path
* vol-create                    create a vol from an XML file
* vol-create-as               create a volume from a set of args
* vol-create-from            create a vol, using another volume as input
* vol-delete（<vol>）卷的删除
* vol-upload（pool）      upload file contents to a volume
* vol-download               download volume contents to a file
* vol-resize                     resize a vol
* vol-wipe                       wipe a vol
* vol-clone                      clone a volume.
* vol-dumpxml                vol information in XML
* vol-key                          returns the volume key for a given volume name or path
## nwfilter 命令

* nwfilter-define （file）根据文件生成一个网络过滤器
* nwfilter-undefine（name） 删除网络过滤
* nwfilter-list 列出来网络过滤
* nwfilter-dumpxml（file）生成一个网络过滤的文件
* nwfilter-edit（name） 编辑一个网络过滤器
# 动手实践

## 创建虚拟机

* 准备虚拟机配置文件，复制一份 cirros.xml并命名为cirros_instance.xml
```bash
[root@geekspeng ~]# cd /etc/libvirt/qemu
[root@geekspeng qemu]# ls
autostart  cirros.xml  networks
[root@geekspeng qemu]# cp cirros.xml cirros_instance.xml 
[root@geekspeng qemu]# ls
autostart  cirros_instance.xml  cirros.xml  networks
[root@geekspeng qemu]# 
```
* 修改 cirros_instance.xml 中的 name 为 cirros_instance，并删掉uuid
```bash
[root@geekspeng qemu]# vim cirros_instance.xml 
```
* 创建虚拟机的第一种方式（先定义在启动）
1. 定义一个虚拟机
```bash
[root@geekspeng qemu]# virsh define cirros_instance.xml 
Domain cirros_instance defined from cirros_instance.xml
[root@geekspeng qemu]# virsh list --all
 Id    Name                           State
----------------------------------------------------
 5     cirros                         running
 -     cirros_instance                shut off
```
可以看到此时虚拟机 cirros_instance 处于关机状态
2. 启动虚拟机
```bash
[root@geekspeng qemu]# virsh start cirros_instance 
Domain cirros_instance started[root@geekspeng qemu]# virsh list --all
 Id    Name                           State
----------------------------------------------------
 5     cirros                         running
 6     cirros_instance                running
```
可以看到此时虚拟机 cirros_instance 处于运行状态
* 创建虚拟机的第二种方式（直接启动）
```bash
[root@geekspeng qemu]# virsh list --all
 Id    Name                           State
----------------------------------------------------
 5     cirros                         running[root@geekspeng qemu]# virsh create cirros_instance.xml 
Domain cirros_instance created from cirros_instance.xml[root@geekspeng qemu]# virsh list --all
 Id    Name                           State
----------------------------------------------------
 5     cirros                         running
 7     cirros_instance                running
```
可以看到创建成功后，虚拟机 cirros_instance 直接处于运行状态
## 查看虚拟机

* 列出所有的虚拟机
```bash
[root@geekspeng qemu]# virsh list --all
 Id    Name                           State
----------------------------------------------------
 5     cirros                         running
 7     cirros_instance                running
```
* 列出活动状态的虚拟机（将cirros 虚拟机关机）
```bash
[root@geekspeng qemu]# virsh shutdown cirros
Domain cirros is being shutdown[root@geekspeng qemu]# virsh list --all
 Id    Name                           State
----------------------------------------------------
 7     cirros_instance                running
 -     cirros                         shut off[root@geekspeng qemu]# virsh list
 Id    Name                           State
----------------------------------------------------
 7     cirros_instance                running
```
* 显示虚拟机信息
```bash
[root@geekspeng qemu]# virsh dominfo cirros_instance
Id:             7
Name:           cirros_instance
UUID:           734c231f-0cea-497b-8a24-33368f1c10f5
OS Type:        hvm
State:          running
CPU(s):         1
CPU time:       223.5s
Max memory:     102400 KiB
Used memory:    102400 KiB
Persistent:     no
Autostart:      disable
Managed save:   no
Security model: none
Security DOI:   0
```
* 显示虚拟机的内存统计信息
```bash
[root@geekspeng qemu]# virsh dommemstat cirros_instance 
actual 102400
rss 108380
```
* 显示虚拟机的 cpu信息
```bash
[root@geekspeng qemu]# virsh vcpucount cirros
maximum      config         1
maximum      live           1
current      config         1
current      live           1
[root@geekspeng qemu]# virsh vcpuinfo cirros_instance 
VCPU:           0
CPU:            0
State:          running
CPU time:       219.5s
CPU Affinity:   y
```
上面显示的是VCPU，CPU显示的是编号而不是个数，CPU Affinity为 y 显示 VCPU0绑定到CPU0
* 显示虚拟机的磁盘
```bash
[root@geekspeng qemu]# virsh domblklist cirros_instance 
Target     Source
------------------------------------------------
hda        /var/lib/libvirt/images/cirros-0.3.4-x86_64-disk.img[root@geekspeng qemu]# virsh domblkinfo  cirros_instance hda
Capacity:       41126400
Allocation:     14602240
Physical:       14614528
```
* 显示虚拟机的网卡接口
```bash
[root@geekspeng qemu]# virsh domiflist cirros_instance 
Interface  Type       Source     Model       MAC
-------------------------------------------------------
vnet1      network    default    rtl8139     52:54:00:31:e9:4d[root@geekspeng qemu]# virsh domif-getlink cirros_instance vnet1 
vnet1 up
```
* 查看虚拟机的 xml 配置文件
```bash
[root@geekspeng qemu]# virsh dumpxml cirros_instance 
<domain type='kvm' id='7'>
  <name>cirros_instance</name>
  <uuid>734c231f-0cea-497b-8a24-33368f1c10f5</uuid>
  <memory unit='KiB'>102400</memory>
  <currentMemory unit='KiB'>102400</currentMemory>
  <vcpu placement='static'>1</vcpu>
  <resource>
    <partition>/machine</partition>
  </resource>
  <os>
    <type arch='x86_64' machine='pc-i440fx-rhel7.0.0'>hvm</type>
    <boot dev='hd'/>
  </os>
  <features>
    <acpi/>
    <apic/>
  </features>
  <cpu mode='custom' match='exact' check='full'>
    <model fallback='forbid'>SandyBridge</model>
    <feature policy='require' name='hypervisor'/>
    <feature policy='disable' name='xsaveopt'/>
  </cpu>
  <clock offset='utc'>
    <timer name='rtc' tickpolicy='catchup'/>
    <timer name='pit' tickpolicy='delay'/>
    <timer name='hpet' present='no'/>
  </clock>
  <on_poweroff>destroy</on_poweroff>
  <on_reboot>restart</on_reboot>
  <on_crash>destroy</on_crash>
  <pm>
    <suspend-to-mem enabled='no'/>
    <suspend-to-disk enabled='no'/>
  </pm>
  <devices>
    <emulator>/usr/libexec/qemu-kvm</emulator>
    <disk type='file' device='disk'>
      <driver name='qemu' type='qcow2'/>
      <source file='/var/lib/libvirt/images/cirros-0.3.4-x86_64-disk.img'/>
      <backingStore/>
      <target dev='hda' bus='ide'/>
      <alias name='ide0-0-0'/>
      <address type='drive' controller='0' bus='0' target='0' unit='0'/>
    </disk>
    <controller type='usb' index='0' model='ich9-ehci1'>
      <alias name='usb'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x05' function='0x7'/>
    </controller>
    <controller type='usb' index='0' model='ich9-uhci1'>
      <alias name='usb'/>
      <master startport='0'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x05' function='0x0' multifunction='on'/>
    </controller>
    <controller type='usb' index='0' model='ich9-uhci2'>
      <alias name='usb'/>
      <master startport='2'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x05' function='0x1'/>
    </controller>
    <controller type='usb' index='0' model='ich9-uhci3'>
      <alias name='usb'/>
      <master startport='4'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x05' function='0x2'/>
    </controller>
    <controller type='pci' index='0' model='pci-root'>
      <alias name='pci.0'/>
    </controller>
    <controller type='ide' index='0'>
      <alias name='ide'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x01' function='0x1'/>
    </controller>
    <controller type='virtio-serial' index='0'>
      <alias name='virtio-serial0'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x06' function='0x0'/>
    </controller>
    <interface type='network'>
      <mac address='52:54:00:31:e9:4d'/>
      <source network='default' bridge='virbr0'/>
      <target dev='vnet1'/>
      <model type='rtl8139'/>
      <alias name='net0'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='0x0'/>
    </interface>
    <serial type='pty'>
      <source path='/dev/pts/4'/>
      <target type='isa-serial' port='0'>
        <model name='isa-serial'/>
      </target>
      <alias name='serial0'/>
    </serial>
    <console type='pty' tty='/dev/pts/4'>
      <source path='/dev/pts/4'/>
      <target type='serial' port='0'/>
      <alias name='serial0'/>
    </console>
    <channel type='spicevmc'>
      <target type='virtio' name='com.redhat.spice.0' state='disconnected'/>
      <alias name='channel0'/>
      <address type='virtio-serial' controller='0' bus='0' port='1'/>
    </channel>
    <input type='mouse' bus='ps2'>
      <alias name='input0'/>
    </input>
    <input type='keyboard' bus='ps2'>
      <alias name='input1'/>
    </input>
    <graphics type='spice' port='5901' autoport='yes' listen='127.0.0.1'>
      <listen type='address' address='127.0.0.1'/>
      <image compression='off'/>
    </graphics>
    <sound model='ich6'>
      <alias name='sound0'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x04' function='0x0'/>
    </sound>
    <video>
      <model type='qxl' ram='65536' vram='65536' vgamem='16384' heads='1' primary='yes'/>
      <alias name='video0'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x0'/>
    </video>
    <redirdev bus='usb' type='spicevmc'>
      <alias name='redir0'/>
      <address type='usb' bus='0' port='1'/>
    </redirdev>
    <redirdev bus='usb' type='spicevmc'>
      <alias name='redir1'/>
      <address type='usb' bus='0' port='2'/>
    </redirdev>
    <memballoon model='virtio'>
      <alias name='balloon0'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x07' function='0x0'/>
    </memballoon>
  </devices>
  <seclabel type='dynamic' model='dac' relabel='yes'>
    <label>+107:+107</label>
    <imagelabel>+107:+107</imagelabel>
  </seclabel>
</domain>
```
## 连接虚拟机

* 通过控制窗口登录虚拟机（ctrl + ] 退出登录）
```bash
[root@geekspeng qemu]# virsh console cirros_instance 
Connected to domain cirros_instance
Escape character is ^]login as 'cirros' user. default password: 'cubswin:)'. use 'sudo' for root.
cirros login:
```
* 通过 ssh 连接虚拟机
```bash
[root@geekspeng ~]# ssh cirros@192.168.122.157
cirros@192.168.122.157's password: 
$ 
```
* 远程连接虚拟机
```bash
[root@geekspeng ~]# virt-viewer -c qemu+ssh://10.0.0.200/system cirros
```
## 管理虚拟机

* 关闭虚拟机（shutodwn）
```bash
[root@geekspeng qemu]# virsh list --all
 Id    Name                           State
----------------------------------------------------
 1     cirros                         running
 3     cirros_instance                running[root@geekspeng qemu]# virsh shutdown cirros
Domain cirros is being shutdown[root@geekspeng qemu]# virsh list --all
 Id    Name                           State
----------------------------------------------------
 3     cirros_instance                running
 -     cirros                         shut off
```
* 启动虚拟机（start）
```bash
[root@geekspeng qemu]# virsh list --all
 Id    Name                           State
----------------------------------------------------
 3     cirros_instance                running
 -     cirros                         shut off[root@geekspeng qemu]# virsh start cirros
Domain cirros started[root@geekspeng qemu]# virsh list --all
 Id    Name                           State
----------------------------------------------------
 3     cirros_instance                running
 4     cirros                         running
```
* 挂起虚拟机（suspend ）
```bash
[root@geekspeng qemu]# virsh list --all
 Id    Name                           State
----------------------------------------------------
 3     cirros_instance                running
 4     cirros                         running[root@geekspeng qemu]# virsh suspend cirros
Domain cirros suspended[root@geekspeng qemu]# virsh list --all
 Id    Name                           State
----------------------------------------------------
 3     cirros_instance                running
 4     cirros                         paused
```
* 恢复虚拟机（resume）
```bash
[root@geekspeng qemu]# virsh list --all
 Id    Name                           State
----------------------------------------------------
 3     cirros_instance                running
 4     cirros                         paused[root@geekspeng qemu]# virsh resume cirros
Domain cirros resumed[root@geekspeng qemu]# virsh list --all
 Id    Name                           State
----------------------------------------------------
 3     cirros_instance                running
 4     cirros                         running
```
* 保存虚拟机状态到文件中（save）
```bash
[root@geekspeng qemu]# virsh list --all
 Id    Name                           State
----------------------------------------------------
 3     cirros_instance                running
 5     cirros                         running[root@geekspeng qemu]# virsh save cirros testDomain cirros saved to test[root@geekspeng qemu]# virsh list --all
 Id    Name                           State
----------------------------------------------------
 3     cirros_instance                running
 -     cirros                         shut off
```
* 从保存状态的文件还原虚拟机（restore）
```bash
[root@geekspeng qemu]# virsh list --all
 Id    Name                           State
----------------------------------------------------
 3     cirros_instance                running
 -     cirros                         shut off[root@geekspeng qemu]# virsh restore test
Domain restored from test[root@geekspeng qemu]# virsh list --all
 Id    Name                           State
----------------------------------------------------
 3     cirros_instance                running
 6     cirros                         running
```
* 重置虚拟机（reset ）
```bash
[root@geekspeng qemu]# virsh reset cirros
Domain cirros was reset
```
* 重启虚拟机（reboot）
```bash
[root@geekspeng qemu]# virsh reboot cirros
Domain cirros is being rebooted
```
* 创建删除快照
```bash
[root@geekspeng dnsmasq]# virsh snapshot-create cirros
Domain snapshot 1548304381 created[root@geekspeng dnsmasq]# virsh snapshot-revert cirros 1548304381[root@geekspeng dnsmasq]# virsh snapshot-list cirros
 Name                 Creation Time             State
------------------------------------------------------------
 1548304381           2019-01-23 20:33:01 -0800 running[root@geekspeng dnsmasq]# virsh snapshot-delete cirros 1548304381 
Domain snapshot 1548304381 deleted[root@geekspeng dnsmasq]# virsh snapshot-list cirros
 Name                 Creation Time             State
```
------------------------------------------------------------
* 冷迁移虚拟机（虚拟机需要关机）

源主机（10.0.0.200）

```bash
[root@geekspeng ~]# virsh list --all
 Id    Name                           State
----------------------------------------------------
 -     cirros                         shut off
 -     cirros_instance                shut off
[root@geekspeng ~]# virsh migrate cirros --offline qemu+ssh://10.0.0.201/system  --persistent
root@10.0.0.201's password: 
[root@geekspeng ~]# virsh migrate cirros_instance --offline qemu+ssh://10.0.0.201/system  --persistent
root@10.0.0.201's password: [root@geekspeng ~]# virsh list --all
 Id    Name                           State
----------------------------------------------------
 -     cirros                         shut off
 -     cirros_instance                shut off
```

目标主机（10.0.0.201）

```bash
[root@geekspeng1 ~]# virsh list --all
 Id    Name                           State
----------------------------------------------------
 -     cirros                         shut off
 -     cirros_instance                shut off
```
迁移完成后，虚拟机同时在源主机和目标主机存在

* 热迁移虚拟机（需要共享存储）
## 修改虚拟机配置

* 重新命名虚拟机（domrename）
```bash
[root@geekspeng qemu]# virsh shutdown cirros
Domain cirros is being shutdown[root@geekspeng qemu]# virsh domrename cirros cirros1
Domain successfully renamed[root@geekspeng qemu]# virsh list --all
 Id    Name                           State
----------------------------------------------------
 13    cirros_instance                running
 -     cirros1                        shut off[root@geekspeng qemu]# virsh domrename cirros1 cirros
Domain successfully renamed[root@geekspeng qemu]# virsh start cirros
Domain cirros started[root@geekspeng qemu]# virsh list --all
 Id    Name                           State
----------------------------------------------------
 13    cirros_instance                running
 14    cirros                         running
```
* 编辑虚拟机的配置文件（edit）

将vcpu 个数从1个改成两个 <vcpu placement='static'>2</vcpu>

```bash
[root@geekspeng qemu]# virsh vcpucount cirros
maximum      config         1
maximum      live           1
current      config         1
current      live           1[root@geekspeng qemu]# virsh shutdown cirros
Domain cirros is being shutdown[root@geekspeng qemu]# virsh edit cirros
Domain cirros XML configuration edited.[root@geekspeng qemu]# virsh create /etc/libvirt/qemu/cirros.xml 
Domain cirros created from /etc/libvirt/qemu/cirros.xml
[root@geekspeng qemu]# virsh vcpucount cirros
maximum      config         2
maximum      live           2
current      config         2
```
current      live           2
* 设置虚拟机的内存大小（setmaxmem，setmem）
```bash
[root@geekspeng qemu]# virsh shutdown cirros
Domain cirros is being shutdown[root@geekspeng qemu]# virsh dominfo cirros | grep memory
Max memory:     131072 KiB
Used memory:    131072 KiB
[root@geekspeng qemu]# virsh setmaxmem cirros 524288 # 需要关机[root@geekspeng qemu]# virsh dominfo cirros | grep memory
Max memory:     524288 KiB
Used memory:    131072 KiB[root@geekspeng qemu]# virsh start cirros
Domain cirros started[root@geekspeng qemu]# virsh setmem cirros 524288 # 需要开机[root@geekspeng qemu]# virsh dominfo cirros | grep memory
Max memory:     524288 KiB
Used memory:    524288 KiB
```
* 设置虚拟机的vcpu个数（setvcpus）
```bash
[root@geekspeng ~]# virsh vcpucount cirros_instance 
maximum      config         2
maximum      live           2
current      config         1
current      live           1[root@geekspeng ~]# virsh setvcpus cirros_instance 2  --current [root@geekspeng ~]# virsh setvcpus cirros_instance 2  --config
[root@geekspeng ~]# virsh vcpucount cirros_instance 
maximum      config         2
maximum      live           2
current      config         2
current      live           2
```
* 关闭或打开某个网口（domif-setlink）
```bash
[root@geekspeng qemu]# virsh domiflist cirros
Interface  Type       Source     Model       MAC
-------------------------------------------------------
vnet0      bridge     br0        rtl8139     52:54:00:b9:bf:4f[root@geekspeng qemu]# virsh domif-setlink cirros vnet0 down
Device updated successfully[root@geekspeng qemu]# virsh domif-getlink cirros vnet0
vnet0 down[root@geekspeng qemu]# virsh domif-setlink cirros vnet0 up
Device updated successfully[root@geekspeng qemu]# virsh domif-getlink cirros vnet0
vnet0 up
```
## 挂载卸载硬盘

* 创建虚拟硬盘
```bash
[root@geekspeng ~]# virsh vol-create-as default cirros.qcow2 100M --format qcow2
Vol cirros.qcow2 created
[root@geekspeng ~]# virsh vol-list default
 Name                 Path                                    
------------------------------------------------------------------------------
 cirros-0.3.4-x86_64-disk.img /var/lib/libvirt/images/cirros-0.3.4-x86_64-disk.img
 cirros.qcow2         /var/lib/libvirt/images/cirros.qcow2
```
* 挂载虚拟硬盘
```bash
[root@geekspeng ~]# virsh domblklist cirros
Target     Source
------------------------------------------------
vda        /var/lib/libvirt/images/cirros-0.3.4-x86_64-disk.img
[root@geekspeng ~]# virsh attach-disk cirros /var/lib/libvirt/images/cirros.qcow2 vdb
Disk attached successfully[root@geekspeng ~]# virsh domblklist cirros
Target     Source
------------------------------------------------
vda        /var/lib/libvirt/images/cirros-0.3.4-x86_64-disk.img
vdb        /var/lib/libvirt/images/cirros.qcow2
```
重启后仍然生效
* 卸载虚拟硬盘
```bash
[root@geekspeng ~]# virsh domblklist cirros
Target     Source
------------------------------------------------
vda        /var/lib/libvirt/images/cirros-0.3.4-x86_64-disk.img
vdb        /var/lib/libvirt/images/cirros.qcow2[root@geekspeng ~]# virsh detach-disk cirros vdb
Disk detached successfully[root@geekspeng ~]# virsh domblklist cirros
Target     Source
------------------------------------------------
vda        /var/lib/libvirt/images/cirros-0.3.4-x86_64-disk.img
```
* 删除虚拟硬盘
```bash
[root@geekspeng ~]# virsh vol-list default 
 Name                 Path                                    
------------------------------------------------------------------------------
 cirros-0.3.4-x86_64-disk.img /var/lib/libvirt/images/cirros-0.3.4-x86_64-disk.img
 cirros.qcow2         /var/lib/libvirt/images/cirros.qcow2    [root@geekspeng ~]# virsh vol-delete cirros.qcow2 --pool default 
Vol cirros.qcow2 deleted[root@geekspeng ~]# virsh vol-list default 
 Name                 Path                                    
------------------------------------------------------------------------------
 cirros-0.3.4-x86_64-disk.img /var/lib/libvirt/images/cirros-0.3.4-x86_64-disk.img
```
## 挂载卸载网卡

临时挂载虚拟网卡，修改保存在内存中，重启就失效

```bash
[root@geekspeng dnsmasq]# virsh domiflist cirros
Interface  Type       Source     Model       MAC
-------------------------------------------------------
vnet1      network    default    virtio      52:54:00:f2:14:b0[root@geekspeng dnsmasq]# virsh attach-interface cirros network default --model virtioInterface attached successfully[root@geekspeng dnsmasq]# virsh domiflist cirros
Interface  Type       Source     Model       MAC
-------------------------------------------------------
vnet1      network    default    virtio      52:54:00:f2:14:b0
vnet2      network    default    virtio      52:54:00:12:9c:2b
```
虽然 virsh 看绑定了vnet2，但是虚拟机内部并没有增加一张网卡，手动增加并启动
永久挂载虚拟网卡，修改会保存到配置文件中（也可以直接修改配置文件）

```bash
[root@geekspeng dnsmasq]# virsh attach-interface cirros bridge br0 --model virtio --persistent --config
[root@geekspeng dnsmasq]# virsh dumpxml cirros >/etc/libvirt/qemu/cirros.xml
[root@geekspeng dnsmasq]# virsh define /etc/libvirt/qemu/cirros.xml
```
卸载虚拟网卡

```bash
[root@geekspeng dnsmasq]# virsh domiflist cirros
Interface  Type       Source     Model       MAC
-------------------------------------------------------
vnet1      network    default    virtio      52:54:00:f2:14:b0
vnet2      network    default    virtio      52:54:00:12:9c:2b[root@geekspeng dnsmasq]# virsh detach-interface cirros --type network --mac 52:54:00:12:9c:2b
Interface detached successfully[root@geekspeng dnsmasq]# virsh domiflist cirros
Interface  Type       Source     Model       MAC
-------------------------------------------------------
vnet1      network    default    virtio      52:54:00:f2:14:b0
```
## 删除虚拟机

```bash
[root@geekspeng ~]# virsh shutdown cirros_instance  # 关机
Domain cirros_instance is being shutdown[root@geekspeng ~]# virsh list --all
 Id    Name                           State
----------------------------------------------------
 23    cirros                         running
 -     cirros_instance                shut off[root@geekspeng ~]# virsh undefine cirros_instance # 删除定义虚拟机的xml文件  
Domain cirros_instance has been undefined[root@geekspeng ~]# virsh list --all
 Id    Name                           State
----------------------------------------------------
 23    cirros                         running
```
# 遇到的问题

* virsh shutdown 不起作用

如果使用 Xen HVM or QEMU/KVM，需要安装 ACPI  并且启动 ACPI ，同时检查虚拟机的配置文件是否配置了 ACPI

```bash
# virsh dumpxml $your-vm-name | grep acpi
<features><acpi/></features>
```
如果 shutdown 还不起作用需要确定虚拟机此时已经完全启动起来，否则虽然virsh list 显示虚拟机是运行状态，但是还是无法执行virsh shutdown命令

* 如果我更改了正在运行的虚拟机的XML，那么更改是否会立即生效？

不会. 重新定义正在运行的虚拟机的XML不会更改任何内容，更改将在下一次VM启动后生效.。Libvirt 有一组命令用于对正在运行的虚拟机进行实时更改，这些命令具有不同的支持，具体取决于hypervisor，例如 virsh attach- *， virsh detach- *，virsh setmem，virsh setvcpus


