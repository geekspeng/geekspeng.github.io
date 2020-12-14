---
title: Centos7 CPU 隔离 和 绑定
date: 2019-05-11
updated: 2019-05-11
tags: [Linux,虚拟化]
categories: [Linux,虚拟化]
---

# 隔离CPU核心

从一般内核 SMP 平衡和调度算法中删除指定的 cpu (由*cpu_number*定义)。 将进程移动到或移出“独立” CPU 的唯一方法是通过 CPU 亲和系统调用。 cpu 数量从0开始，因此最大值比系统上的 cpu 数量少1

此选项是隔离 cpu 的首选方法。 另一种方法是手动设置系统中所有任务的 CPU 掩码，这可能会导致问题和次优的负载均衡器性能

<!-- more -->

* 查看 CPU 情况，共有4个CPU
```bash
# lscpu
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                4
On-line CPU(s) list:   0-3
Thread(s) per core:    1
Core(s) per socket:    1
Socket(s):             4
NUMA node(s):          1
Vendor ID:             GenuineIntel
CPU family:            6
Model:                 42
Model name:            Intel(R) Core(TM) i5-2520M CPU @ 2.50GHz
Stepping:              7
CPU MHz:               2491.949
BogoMIPS:              4983.89
Hypervisor vendor:     VMware
Virtualization type:   full
L1d cache:             32K
L1i cache:             32K
L2 cache:              256K
L3 cache:              3072K
NUMA node0 CPU(s):     0-3
```
* 修改/etc/default/grub 文件GRUB_CMDLINE_LINUX  行末尾添加 isolcpus=1,2，隔离出第2个和第3个CPU
```bash
GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=cl/root rd.lvm.lv=cl/swap  rhgb quiet  isolcpus=2,3"
```
* 重新编译image
```bash
# grub2-mkconfig -o /boot/grub2/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-3.10.0-514.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-514.el7.x86_64.img
Found linux image: /boot/vmlinuz-0-rescue-120896e1b2924a618de2776af043d4dc
Found initrd image: /boot/initramfs-0-rescue-120896e1b2924a618de2776af043d4dc.img
```
* 重启
```bash
# reboot
```
* 检查启动项是否设置成功
```bash
[root@localhost ~]# cat /proc/cmdline
BOOT_IMAGE=/vmlinuz-3.10.0-862.el7.x86_64 root=/dev/mapper/centos-root ro crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet LANG=en_US.UTF-8 isolcpus=1,2
```
* 检查隔离是否生效，如果设置成功，则落在该核心的线程会很少，而其他核心的线程比较多
```bash
# ps -eLo pid,user,lwp,psr  | awk '{if($4==1) print $0}'  #检查核心1
    12 root         12   1
    13 root         13   1
    14 root         14   1
    15 root         15   1
    16 root         16   1
# ps -eLo pid,user,lwp,psr  | awk '{if($4==2) print $0}'  #检查核心2
    17 root         17   2
    18 root         18   2
    19 root         19   2
    20 root         20   2
    21 root         21   2
```
* 可以用 top 命令来查看
```bash
# 执行top，按数字1，就可以调出每个核心的使用状态，然后按下f按键，向下找到"P= Last Used Cpu (SMP)"这一行，按下空格或者'd'，再按q按键返回，就可以看到每个线程具体落在哪个核心上面（最后一项P）
```
![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/kbaHP99nZvYr4bEk.png!thumbnail)

# CPU 绑定

## taskset

* 查看进程的 CPU 关联信息（十六进制形式）
```bash
# taskset -p 2915
pid 2915's current affinity mask: ff
```
对应于二进制格式的“11111111” ，这意味着进程可以在8个不同的 CPU 核心(从0到7)中的任何一个上运行。最低位对应于核心 ID 0，从右到核心 ID 1的第二个最低位，从第三个最低位到核心 ID 2，等等。
* 查看进程的 CPU 关联信息（列表形式）
```bash
# taskset -cp 2915
pid 2915's current affinity list: 0-7
```
* 固定一个正在运行的进程到特定的 CPU 核心
```bash
$ taskset -p <COREMASK> <PID>
$ taskset -cp <CORE-LIST> <PID>
```
* 启动一个固定在特定 CPU 内核上的新程序
```bash
# taskset <COREMASK> <EXECUTABLE>
$ taskset -c <CORE-LIST> <EXECUTABLE>
```
* 查看进程使用的哪个CPU
```bash
# ps -o psr -p <PID>
```
注：如果 taskset 命令不可用，需要安装 util-linux

```bash
# sudo yum install util-linux
```
## Cgroup

* 创建一个控制组
```bash
# mkdir /sys/fs/cgroup/cpuset/tiger # 创建一个控制组，删除用 rmdir 命令
```
* 限制 tiger 控制组下所有进程只能使用逻辑核 1 和 2
```bash
# echo "1-2" > /sys/fs/cgroup/cpuset/tiger/cpuset.cpus
# echo "0" > /sys/fs/cgroup/cpuset/tiger/cpuset.mems
```
对于 cpuset.mems 参数而言，每个内存节点和 NUMA 节点一一对应。如果进程的内存需求量较大，可以把所有的 NUMA 节点都配置进去。出于性能的考虑，配置的逻辑核和内存节点一般属于同一个 NUMA 节点，可用“numactl --hardware”命令获知它们的映射关系。

* 验证效果

将当前会话加入刚刚创建的控制组里

```bash
# echo $$ > /sys/fs/cgroup/cpuset/tiger/cgroup.procs # 写入当前进程编号
```
进程在加入一个控制组后，控制组所对应的限制会即时生效。启动一个计算密集型的任务，申请用 4 个逻辑核。

```bash
# stress -c 4
```
注：如果 stress 命令不可用，需要安装 stress

```bash
# yum install -y stress
```
观察 CPU 的使用情况来验证效果，只有编号为 1 和 2 的两个逻辑核在工作，用户态的比例高达 100%

```bash
# top
top - 05:54:12 up  1:44,  7 users,  load average: 3.14, 2.66, 4.67
Tasks: 139 total,   9 running, 130 sleeping,   0 stopped,   0 zombie
%Cpu0  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu1  :100.0 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu2  :100.0 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu3  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :   997980 total,   511464 free,   152984 used,   333532 buff/cache
```
KiB Swap:  2097148 total,  2097148 free,        0 used.   632004 avail Mem
