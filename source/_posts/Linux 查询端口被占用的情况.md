---
title: Linux 查询端口被占用的情况
date: 2018-04-02
updated: 2018-04-02
tags: [Linux]
categories: [Linux]
---

1. lsof -i:端口号 用于查看某一端口的占用情况
```bash
[root@node1 ~]# lsof -i:22
COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
sshd     6622 root    3u  IPv4  36215      0t0  TCP *:ssh (LISTEN)
sshd     6622 root    4u  IPv6  36224      0t0  TCP *:ssh (LISTEN)
sshd    31358 root    3u  IPv4 122192      0t0  TCP node1:ssh->192.168.46.1:64212 (ESTABLISHED)
sshd    31914 root    3u  IPv4 126124      0t0  TCP node1:ssh->192.168.46.1:62861 (ESTABLISHED)
```
<!-- more -->

如果提示 bash: lsof: command not found  则通过yum 安装后重试即可

```bash
# yum install lsof -y
```
2. lsof -i  查看所有被TCP和UDP使用的端口
```bash
[root@node1 ~]# lsof -i
COMMAND    PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
sshd      6622 root    3u  IPv4  36215      0t0  TCP *:ssh (LISTEN)
sshd      6622 root    4u  IPv6  36224      0t0  TCP *:ssh (LISTEN)
master    6965 root   13u  IPv4  36605      0t0  TCP localhost:smtp (LISTEN)
master    6965 root   14u  IPv6  36606      0t0  TCP localhost:smtp (LISTEN)
dhclient 10738 root    6u  IPv4  44971      0t0  UDP *:bootpc
sshd     31358 root    3u  IPv4 122192      0t0  TCP node1:ssh->192.168.46.1:64212 (ESTABLISHED)
sshd     31914 root    3u  IPv4 126124      0t0  TCP node1:ssh->192.168.46.1:62861 (ESTABLISHED)
```
3. netstat -tuln/tuan 查看所有被TCP和UDP使用的端口
```bash
[root@node1 ~]# netstat -tuln
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN
tcp6       0      0 :::22                   :::*                    LISTEN
tcp6       0      0 ::1:25                  :::*                    LISTEN
udp        0      0 0.0.0.0:68              0.0.0.0:*[root@node1 ~]# netstat -tuan
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN
tcp        0      0 192.168.46.133:22       192.168.46.1:64212      ESTABLISHED
tcp        0      0 192.168.46.133:22       192.168.46.1:62861      ESTABLISHED
tcp6       0      0 :::22                   :::*                    LISTEN
tcp6       0      0 ::1:25                  :::*                    LISTEN
udp        0      0 0.0.0.0:68              0.0.0.0:*
```
4. netstat -tuln/tuan | grep 端口号，查看指定端口是否被TCP和UDP使用
```bash
[root@node1 ~]# netstat -tuln | grep 22
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN
tcp6       0      0 :::22                   :::*                    LISTEN
[root@node1 ~]# netstat -tuan | grep 22
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN
tcp        0      0 192.168.46.133:22       192.168.46.1:64212      ESTABLISHED
tcp        0      0 192.168.46.133:22       192.168.46.1:62861      ESTABLISHED
```
tcp6       0      0 :::22                   :::*                    LISTEN
**附录**

lsof -i 常用参数介绍

```bash
-i 4  　　　#ipv4地址
-i 6  　　　#ipv6地址
-i tcp  　 #tcp连接
-i udp  　 #udp连接
-i :3306 　#端口
-i @ip  　 #查看与某个ip地址建立的连接
```
netstat 常用的两组命令

```bash
netstat -tuln
netstat -tuan
```
netstat 常用参数介绍

```bash
-t (tcp) 仅显示tcp相关选项
-u (udp)仅显示udp相关选项
-n 拒绝显示别名，能显示数字的全部转化为数字
-l 仅列出在Listen(监听)的服务状态
-p 显示建立相关链接的程序名
```
