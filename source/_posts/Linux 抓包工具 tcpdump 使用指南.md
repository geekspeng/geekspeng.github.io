---
title: Linux下查看网卡驱动和驱动信息
date: 2018-11-27
updated: 2018-11-27
tags: [Linux,tcpdump]
categories: [Linux]
---

# tcpdump简介

**tcpdump**是一款 Linux 平台的抓包工具。它可以抓取涵盖整个 TCP/IP 协议族的数据包，支持针对网络层、协议、主机、端口的过滤，并提供 and、or、not 等逻辑语句来过滤无用的信息

安装tcpdump：

```bash
# yum -y install tcpdump
```

<!-- more -->

# tcpdump命令格式

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/A4u40kTQQ2stkARg.png!thumbnail)

## tcpdump 常用选项

```bash
-D 列出操作系统所有可以用于抓包的接口
-c 指定要抓取包的数量
-i 指定监听的网卡， -i any 显示所有网卡
-n 表示不解析主机名，直接用 IP 显示，默认是用 hostname 显示
-nn 表示不解析主机名和端口，直接用端口号显示，默认显示是端口号对应的服务名
-P 指定抓取的包是流入的包还是流出的，可以指定参数 in, out, inout 等，默认是 inout
-q 快速打印输出，即只输出少量的协议相关信息
-s len 设置要抓取数据包长度为 len，默认只会截取前 96bytes 的内容， -s0 的话，会截取全部内容。
-S 将 TCP 的序列号以绝对值形式输出，而不是相对值
-t 不要打印时间戳
-vv 输出详细信息（比如 tos、ttl、checksum等）
-w：将抓包数据输出到文件中而不是标准输出
-r：从给定的数据包文件中读取数据，使用"-"表示从标准输入中读取。
```
## tcpdump 过滤器

```bash
proto：可选有 ip、arp、rarp、tcp、udp、icmp、ether 等，默认是所有协议的包
dir：可选有 src、dst、src or dst、src and dst，默认为 src or dst
type：可选有 host、net、port、portrange（端口范围，比如 21-42），默认为 host
```
表达式单元之间可以使用操作符" and（&&） 、 or（||）、not（!） "进行连接，使用括号"()"可以改变表达式的优先级，但需要注意的是括号会被shell解释，所以应该使用反斜线""转义为"()"

特别要记住在使用 && 的时候，要用单引号或者双引号包住表达式

# 使用 tcpdump 的常用选项

* 不转换主机名、端口号等
```bash
# tcpdump -nn
```
* 显示详细信息
```bash
# tcpdump -v
# tcpdump -vv
# tcpdump -vvv
```
每增加一个 -v 标记，输出会包含更多信息
* 指定网络接口
```bash
# tcpdump -i ens33
# tcpdump -i any
```
如果不指定网络接口， tcpdump 在运行时会选择编号最低的网络接口
any 这一特定的网络接口名用来让 tcpdump 监听所有的接口

* 指定抓包数量
```bash
# tcpdump -c 10
```
* 指定抓包大小
```bash
# tcpdump -s 100
```
* 写入文件
```bash
# tcpdump -w /var/tmp/tcpdata.pcap
```
* 读取文件
```bash
# tcpdump -r /var/tmp/tcpdata.pcap
```
也可以利用 wireshark 来读取 tcpdump 保存的文件
* 组合使用
```bash
# tcpdump -nnvvv -i any -c 100 -s 100
```
# 使用 tcpdump 的过滤器

tcpdump 可以通过各式各样的表达式，来过滤所截取或者输出的数据

* 查找特定主机的数据包
```bash
# tcpdump -nvvv -i any -c 3 host 10.0.3.1
```
只会显示源 IP 或者目的 IP 地址是 10.0.3.1 的数据包
* 只显示源地址为特定主机的数据包
```bash
# tcpdump -nvvv -i any -c 3 src host 10.0.3.1
```
* 过滤源和目的端口
```bash
# tcpdump -nvvv -i any -c 3 port 22 and port 60738
```
tcpdump 只输出端口号是 22 和 60738 的数据包
* 查找两个端口号的数据包
```bash
# tcpdump -nvvv -i any -c 20 'port 80 or port 443'
```
端口号 80 表示 http 连接，443 表示 https
* 查找两个特定端口和来自特定主机的数据包
```bash
# tcpdump -nvvv -i any -c 20 '(port 80 or port 443) and host 10.0.3.169'
```
* 查找某协议的数据包
```
# tcpdump -nnvvv -i any tcp
```
# tcpdump 的输出格式

一般格式

```bash
系统时间  源主机.端口 > 目标主机.端口  数据包参数
```
示例

```bash
# tcpdump -n -i any -c 1  host 10.0.0.210
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on any, link-type LINUX_SLL (Linux cooked), capture size 262144 bytes
22:59:04.971954 IP 10.0.0.210.ssh > 10.0.0.1.49663: Flags [P.], seq 1081757770:1081757982, ack 3080549455, win 274, length 212
1 packet captured
2 packets received by filter
0 packets dropped by kernel
```
可以按照 src-ip.src-port > dest-ip.dest-port: Flags[S] 格式来分析。源地址位于 > 前面，后面则是目的地址

通过 Flags[S] 可以判断数据包类型

* [S] – SYN (开始连接)
* [.] – 没有标记
* [P] – PSH (数据推送)
* [F] – FIN (结束连接)
* [R] – RST (重启连接)
# 参考文档

[https://mp.weixin.qq.com/s/NTkocGensdIPYL5qi5E14w](https://mp.weixin.qq.com/s/NTkocGensdIPYL5qi5E14w)

[https://mp.weixin.qq.com/s/2frB2Chaw2H1UitPNTYzhA](https://mp.weixin.qq.com/s/2frB2Chaw2H1UitPNTYzhA)


