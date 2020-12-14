---
title: 查看 Linux 文件打开情况
date: 2018-04-02
updated: 2018-04-02
tags: [Linux]
categories: [Linux]
---

# Linux 下有哪些文件

在介绍lsof命令之前，先简单说一下，linux主要有哪些文件：

* 普通文件
* 目录
* 符号链接
* 面向块的设备文件
* 面向字符的设备文件
* 管道和命名管道
* 套接字

<!-- more -->

# lsof 命令实用用法介绍

lsof，是list open files的简称

lsof输出各列信息的意义如下

```
COMMAND：进程的名称 
PID：进程标识符
USER：进程所有者
FD：文件描述符，应用程序通过文件描述符识别该文件。cwd 值表示应用程序的当前工作目录，这是该应用程序启动的目录，除非它本身对这个目录进行更改,txt 类型的文件是程序代码，如应用程序二进制文件本身或共享库，如上列表中显示的 /sbin/init 程序。其次数值表示应用程序的文件描述符，这是打开该文件时返回的一个整数。u表示该文件被打开并处于读写模式，r表示该文件被打开并处于读模式，w表示该文件被打开并处于写模式，同时还有大写的W表示该应用程序具有对整个文件的写锁。初始打开每个应用程序时，都具有三个文件描述符，从 0 到 2，分别表示标准输入、输出和错误流。所以大多数应用程序所打开的文件的 FD 都是从 3 开始。
TYPE：文件类型，文件和目录分别称为 REG 和 DIR。而CHR 和 BLK，分别表示字符和块设备；或者 UNIX、FIFO 和 IPv4，分别表示 UNIX 域套接字、先进先出 (FIFO) 队列和网际协议 (IP) 套接字。
DEVICE：指定磁盘的名称
SIZE：文件的大小
NODE：索引节点（文件在磁盘上的标识）
NAME：打开文件的确切名称
```
## 查看当前打开的所有文件

一般来说，直接输入lsof命令产生的结果实在是太多，可能很难找到我们需要的信息。不过借此说明一下一条记录都有哪些信息。

```bash
$ lsof（这里选取一条记录显示）
COMMAND   PID                      USER   FD             TYPE        DEVICE SIZE/OFF   NODE   NAME
vi        27940                    hyb    7u      REG               8,15     16384     137573 /home/hyb/.1.txt.swp
```
从左往右分别代表：打开该文件的程序名，进程id，用户，文件描述符，文件类型，设备，大小，iNode号，文件名。
这条记录，表明进程id为27940的vi程序，打开了文件描述值为7，且处于读写状态的，在/home/hyb目录下的普通文件(REG regular file).1.txt.swap，当前大小16384字节。

## 列出被删除但占用空间的文件

在生产环境中，我们可能会使用df命令看到磁盘空间占满了，然而实际上又很难找到占满空间的文件，这常常是由于某个大文件被删除了，但是它却被某个进程打开，导致通过普通的方式找不到它的踪迹，最常见的就是日志文件。我们可以通过lsof来发现这样的文件：

```bash
$ lsof | grep deleted
tuned      6621         root    8u      REG              253,0      4096   17581562 /tmp/ffiLIRq9l (deleted)
gmain      6621 7082    root    8u      REG              253,0      4096   17581562 /tmp/ffiLIRq9l (deleted)
tuned      6621 7083    root    8u      REG              253,0      4096   17581562 /tmp/ffiLIRq9l (deleted)
tuned      6621 7084    root    8u      REG              253,0      4096   17581562 /tmp/ffiLIRq9l (deleted)
tuned      6621 7097    root    8u      REG              253,0      4096   17581562 /tmp/ffiLIRq9l (deleted)
```
可以看到这些被删除的但仍然被打开文件。这个时候就可以根据实际情况分析，到底哪些文件可能过大但是却被删除了，导致空间仍然占满。
## 恢复打开但被删除的文件

我们可以找到被删除但是仍然被打开的文件，实际上文件并没有真正的消失，如果是意外被删除的，我们还有手段恢复它。以/var/log/cat文件为例，我们先删除它

```bash
$　rm /var/log/syslog
```
使用lsof查看那个进程打开了该文件：

```bash
$ lsof | grep syslog
rs:main    993 1119           syslog    5w      REG               8,10     78419     528470 /var/log/syslog (deleted)
```
可以找到进程id为993的进程打开了该文件，我们知道每个进程在/proc下都有文件描述符打开的记录：

```bash
$ ll /proc/993/fd
lr-x------ 1 root   root   64 3月   5 18:30 0 -> /dev/null
l-wx------ 1 root   root   64 3月   5 18:30 1 -> /dev/null
l-wx------ 1 root   root   64 3月   5 18:30 2 -> /dev/null
lrwx------ 1 root   root   64 3月   5 18:30 3 -> socket:[15032]
lr-x------ 1 root   root   64 3月   5 18:30 4 -> /proc/kmsg
l-wx------ 1 root   root   64 3月   5 18:30 5 -> /var/log/syslog (deleted)
l-wx------ 1 root   root   64 3月   5 18:30 6 -> /var/log/auth.log
```
这里就找到了被删除的syslog文件，文件描述符是５，我们把它重定向出来：

```bash
$ cat /proc/993/fd/5 > syslog
$ ls -al /var/log/syslog
-rw-r--r-- 1 root root 78493 3月   5 19:22 /var/log/syslog
```
这样我们就恢复了syslog文件。
## 查看当前文件被哪些进程打开

Windows下经常遇到要删除某个文件，然后告诉你某个程序正在使用，然而不告诉你具体是哪个程序。我们可以在资源管理器-性能-资源监视器-cpu-关联的句柄处搜索文件，即可找到打开该文件的程序，但是搜索速度感人。

linux就比较容易了，使用lsof命令就可以了，例如要查看当前哪些程序打开了hello.c:

```bash
$ lsof hello.c
COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF   NODE NAME
tail    28731  hyb    3r   REG   8,15      228 138441 hello.c
```
但是我们会发现，使用vi打开的hello.c并没有找出来，这是因为vi打开的是一个临时副本。我们换一种方式查找：

```bash
$ lsof | grep hello.c
tail      28906                    hyb    3r      REG               8,15       228     138441 /home/hyb/workspaces/c/hello.c
vi        28933                    hyb    9u      REG               8,15     12288     137573 /home/hyb/workspaces/c/.hello.c.swp
```
这样我们就找到了两个程序和hello.c文件相关。
这里grep的作用是从所有结果中只列出符合条件的结果。

## 查看某个目录文件被打开情况

```bash
$ lsof +D ./
```
## 查看当前进程打开了哪些文件

使用方法：lsof -c 进程名

通常用于程序定位问题，例如用于查看当前进程使用了哪些库，打开了哪些文件等等。假设有一个循环打印字符的hello程序：

```bash
$ lsof -c hello
COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF   NODE NAME
hello   29190  hyb  cwd    DIR   8,15     4096 134538 /home/hyb/workspaces/c
hello   29190  hyb  rtd    DIR   8,10     4096      2 /
hello   29190  hyb  txt    REG   8,15     9816 138314 /home/hyb/workspaces/c/hello
hello   29190  hyb  mem    REG   8,10  1868984 939763 /lib/x86_64-linux-gnu/libc-2.23.so
hello   29190  hyb  mem    REG   8,10   162632 926913 /lib/x86_64-linux-gnu/ld-2.23.so
hello   29190  hyb    0u   CHR 136,20      0t0     23 /dev/pts/20
hello   29190  hyb    1u   CHR 136,20      0t0     23 /dev/pts/20
hello   29190  hyb    2u   CHR 136,20      0t0     23 /dev/pts/20
```
我们可以从中看到，至少它用到了/lib/x86_64-linux-gnu/libc-2.23.so以及hello文件。
也可以通过进程id查看，可跟多个进程id，使用逗号隔开：

```bash
$ lsof -p 29190
COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF   NODE NAME
hello   29190  hyb  cwd    DIR   8,15     4096 134538 /home/hyb/workspaces/c
hello   29190  hyb  rtd    DIR   8,10     4096      2 /
hello   29190  hyb  txt    REG   8,15     9816 138314 /home/hyb/workspaces/c/hello
hello   29190  hyb  mem    REG   8,10  1868984 939763 /lib/x86_64-linux-gnu/libc-2.23.so
hello   29190  hyb  mem    REG   8,10   162632 926913 /lib/x86_64-linux-gnu/ld-2.23.so
hello   29190  hyb    0u   CHR 136,20      0t0     23 /dev/pts/20
hello   29190  hyb    1u   CHR 136,20      0t0     23 /dev/pts/20
hello   29190  hyb    2u   CHR 136,20      0t0     23 /dev/pts/20
```
当然这里还有一种方式，就是利用proc文件系统，首先找到hello进程的进程id:

```bash
$ ps -ef | grep hello
hyb      29190 27929  0 21:14 pts/20   00:00:00 ./hello 2
hyb      29296 28848  0 21:18 pts/22   00:00:00 grep --color=auto hello
```
可以看到进程id为29190，查看该进程文件描述记录目录：
```bash
$ ls -l /proc/29190/fd
lrwx------ 1 hyb hyb 64 3月   2 21:14 0 -> /dev/pts/20
lrwx------ 1 hyb hyb 64 3月   2 21:14 1 -> /dev/pts/20
lrwx------ 1 hyb hyb 64 3月   2 21:14 2 -> /dev/pts/20
```
这种方式能够过滤很多信息，因为它只列出了该进程实际打开的，这里它只打开了1,2,3，即标准输入，标准输出和标准错误。
# 查看某个用户打开了哪些文件

linux是一个多用户操作系统，怎么知道其他普通用户打开了哪些文件呢？可使用-u参数

```bash
$ lsof -u hyb
（内容太多，省略）
```
# 列出除了某个进程或某个用户打开的文件

实际上和前面使用方法类似，只不过，在进程id前面或者用户名前面加^，例如：

```bash
$ lsof -p ^1     #列出除进程id为１的进程以外打开的文件
$ lsof -u ^root  #列出除root用户以外打开的文件
```
