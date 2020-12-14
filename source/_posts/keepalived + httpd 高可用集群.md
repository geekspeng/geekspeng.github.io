---
title: keepalived + httpd 高可用集群
date: 2019-09-12
updated: 2019-09-12
tags: [keepalived,高可用]
categories: [高可用]
---

# 实验环境

```bash
操作系统：centos7.5
keepalived MASTER:  192.168.46.133
keepalived BACKUP1: 192.168.46.134
keepalived BACKUP2: 192.168.46.135
http 服务器1: 192.168.46.133﻿
http 服务器2: 192.168.46.134
http 服务器3: 192.168.46.135
VIP: 192.168.46.100
```
注：keepalived 和 http 服务既可以放到同一个节点也可以放到不同的节点

<!-- more -->

# 安装 httpd 并启动服务

1. 在所有节点上安装 httpd
```bash
# yum install httpd -y
```
2. 三台节点分别设置httpd首页显示信息，这里设置显示服务器 ip 地址，方便我们判断访问的是哪台服务器

http 服务器1

```bash
# echo "192.168.46.133" >/var/www/html/index.html
```
http 服务器2

```bash
# echo "192.168.46.134" >/var/www/html/index.html
```
http 服务器3

```bash
# echo "192.168.46.135" >/var/www/html/index.html
```
3. 关闭所有节点的防火墙
```bash
# systemctl stop firewalld
# systemctl disable firewalld
```
4. 启动所有节点的 httpd 服务，并确定可以通过浏览器访问所有节点的 http 服务器
```bash
# systemctl start httpd
# systemctl enable httpd
```
http 服务器1

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/JHFz3Cy876YvvLZ1.png!thumbnail)

http 服务器2

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/08DgGd4ioiQ5SHLb.png!thumbnail)

http 服务器3

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/Q1jCGmpqPGopL2fW.png!thumbnail)

# keepalived 实现 http 服务的高可用

1. 在所有节点上安装 keepalived
```bash
# yum install keepalived -y
```
2. 配置 keepalived 实现 httpd 的高可用

MASTER 节点配置如下

```bash
# vim /etc/keepalived/keepalived.conf
vrrp_instance VI_1 {
    state MASTER #指定该节点为主节点，备用节点设置为BACKUP
    interface ens33 #绑定虚拟IP的服务器的网卡
    virtual_router_id 51 #虚拟路由编号，主备要一致
    priority 50 #主节点的优先级，数值在1~254，注意从节点必须比主节点的优先级别低
    advert_int 1 #组播信息发送间隔，主备要一致
    #设置验证信息，主备要一致
    authentication {
      auth_type PASS
      auth_pass 1111
    }
    virtual_ipaddress {
      192.168.46.100 #指定虚拟IP，主备要设置一样，可多设，每行一个
    }
 }
```

BACKUP 节点配置基本同 MASTER 节点一样，不同之处如下

```ini
# vim /etc/keepalived/keepalived.conf
state BACKUP
priority 30
 
#其它配置跟keepalived主机相同
```
3. 启动 keepalived
```bash
# systemctl start keepalived
# systemctl enable keepalived
```
4. 在MASTER 节点查看是否生成 VIP
```bash
# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:f9:43:90 brd ff:ff:ff:ff:ff:ff
    inet 192.168.46.133/24 brd 192.168.46.255 scope global dynamic ens33
       valid_lft 1204sec preferred_lft 1204sec
    inet 192.168.46.100/32 scope global ens33
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fef9:4390/64 scope link
```
       valid_lft forever preferred_lft forever可以看到网卡ens33 多了一个 192.168.46.100 的 IP 地址
5. 测试keepalived 是否生效
* 通过 VIP 访问http 服务器，此时访问的是master 节点的http 服务器

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/09AvAQDiw2sI3HyK.png!thumbnail)

* 关闭 master 节点的 keepalived 服务，再通过 VIP访问http服务器，此时访问的是keepalivedbackup节点的 http 服务器

keepalived master 节点

```bash
# systemctl stop keepalived.service
# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:f9:43:90 brd ff:ff:ff:ff:ff:ff
    inet 192.168.46.133/24 brd 192.168.46.255 scope global dynamic ens33
       valid_lft 1279sec preferred_lft 1279sec
    inet6 fe80::20c:29ff:fef9:4390/64 scope link
```
       valid_lft forever preferred_lft forever此时网卡ens33 已经没有 192.168.46.100 的 IP 地址

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/Eng96uUPIAIYiTbh.png!thumbnail)

此时访问的是 keepalived backup2 节点上的 http 服务器，查看 keepalived backup2 节点查看是否生成 VIP

```bash
# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:ea:c4:b2 brd ff:ff:ff:ff:ff:ff
    inet 192.168.46.135/24 brd 192.168.46.255 scope global noprefixroute dynamic ens33
       valid_lft 1289sec preferred_lft 1289sec
    inet 192.168.46.100/32 scope global ens33
       valid_lft forever preferred_lft forever
    inet6 fe80::96ad:543:7d7d:2ea1/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```
可以看到网卡ens33 多了一个 192.168.46.100 的 IP 地址
* 启动 master 节点的 keepalived 服务，通过 VIP访问http服务器，查看VIP 是否恢复到 master

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/vnTuqZLb61QahdTi.png!thumbnail)

keepalived master 节点

```bash
# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:f9:43:90 brd ff:ff:ff:ff:ff:ff
    inet 192.168.46.133/24 brd 192.168.46.255 scope global dynamic ens33
       valid_lft 1452sec preferred_lft 1452sec
    inet 192.168.46.100/32 scope global ens33
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fef9:4390/64 scope link
```
       valid_lft forever preferred_lft forever
keepalived backup2 节点

```bash
# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:ea:c4:b2 brd ff:ff:ff:ff:ff:ff
    inet 192.168.46.135/24 brd 192.168.46.255 scope global noprefixroute dynamic ens33
       valid_lft 1745sec preferred_lft 1745sec
    inet6 fe80::96ad:543:7d7d:2ea1/64 scope link noprefixroute
```
       valid_lft forever preferred_lft forever
测试结果说明，当keepalived backup 节点在keepalived master 节点宕机的情况会自动接管了资源。但待keepalived主机恢复正常的时候，主机会重新接管资源

存在的问题，如果只是 master 节点 的 http 服务挂了，但是keepalived 正常运行，此时backup 节点并不会接管 master 节点的工作，导致 http 服务不可用，所以需要keepalived 定时检测 http是否可用，如果不可用可以考虑恢复 master 节点的 http 服务或者关闭 master 节点的 keepalived 服务（此时backup 节点并接管 master 节点的工作）

# keepalived 负载均衡配置

LVS的IP负载均衡技术是通过IPVS模块实现的。IPVS模块是LVS集群的核心软件模块，它安装在LVS集群作为负载均衡的主节点上，虚拟出一个IP地址和端口对外提供服务。用户通过访问这个虚拟服务（VS），然后访问请求由负载均衡器（LB）调度到后端真实服务器（RS）中，由RS实际处理用户的请求给返回响应。

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/WOT7suLJyVEf8IxI.png!thumbnail)

1. 修改所有节点的 keepalived 配置文件，添加以下内容
```bash
#虚拟IP服务
virtual_server 192.168.46.100 80 {
    delay_loop 6 #设置检查间隔
    lb_algo rr #设置LVS调度算法, rr|wrr|lc|wlc|lblc 等调度算法
    lb_kind DR #设置LVS实现负载的机制，有NAT、TUN、DR三个模式
    nat_mask 255.255.255.0
   persistence_timeout 50 #持久连接设置，会话保持时间
   protocol TCP #转发协议为TCP
  #后端实际TCP服务配置
  real_server 192.168.46.133 80 {  # 指定real server1的IP地址
     weight 3  # 配置节点权值，数字越大权重越高
     TCP_CHECK {  
        connect_timeout 10         
        nb_get_retry 3  
        delay_before_retry 3  
        connect_port 80  
        }
  }
  real_server 192.168.46.134 80 {  # 指定real server1的IP地址
     weight 3  # 配置节点权值，数字越大权重越高
  TCP_CHECK {  
        connect_timeout 10         
        nb_get_retry 3  
        delay_before_retry 3  
        connect_port 80  
        }
  }
  real_server 192.168.46.135 80 {  # 指定real server1的IP地址
     weight 3  # 配置节点权值，数字越大权重越高
     TCP_CHECK {  
        connect_timeout 10         
        nb_get_retry 3  
        delay_before_retry 3  
        connect_port 80  
        }
  }
}
```
2. **I**PVS的三种转发模式需要进行的操作

**DR模式（Direct Routing）**

DR模式下，客户端的请求包到达负载均衡器的虚拟服务IP端口后，负载均衡器不会改写请求包的IP和端口，但是会改写请求包的MAC地址为后端RS的MAC地址，然后将数据包转发；真实服务器处理请求后，响应包直接回给客户端，不再经过负载均衡器。所以DR模式的转发效率是最高的，特别适合下行流量较大的业务场景，比如请求视频等大文件。

DR模式的特点

* 数据包在LB转发过程中，源/目的IP端口都不会变化

LB只是将数据包的MAC地址改写为RS的MAC地址，然后转发给相应的RS。

* **每台RS上都必须在环回网卡上绑定LB的虚拟服务IP**

因为LB转发时并不会改写数据包的目的IP，所以RS收到的数据包的目的IP仍是LB的虚拟服务IP。为了保证RS能够正确处理该数据包，而不是丢弃，必须在RS的环回网卡上绑定LB的虚拟服务IP。这样RS会认为这个虚拟服务IP是自己的IP，自己是能够处理这个数据包的。否则RS会直接丢弃该数据包！

* RS上的业务进程必须监听在环回网卡的虚拟服务IP上，且端口必须和LB上的虚拟服务端口一致

因为LB不会改写数据包的目的端口，所以RS服务的监听端口必须和虚拟服务端口一致，否则RS会直接拒绝该数据包。

* RS处理完请求后，响应直接回给客户端，不再经过LB因为RS

收到的请求数据包的源IP是客户端的IP，所以理所当然RS的响应会直接回给客户端，而不会再经过LB。这时候要求RS和客户端之间的网络是可达的。

* LB和RS须位于同一个子网

因为LB在转发过程中需要改写数据包的MAC为RS的MAC地址，所以要能够查询到RS的MAC。而要获取到RS的MAC，则需要保证二者位于一个子网，否则LB只能获取到RS网关的MAC地址。

所有 real_server 的环回网卡上绑定LB的 virtual_server 的 ip

```bash
# vim /etc/init.d/realserver
VIP=192.168.46.100
case "$1" in
start)
       ifconfig lo:0 $VIP netmask 255.255.255.255 broadcast $VIP
       route add -host $VIP dev lo:0
       echo "1" >/proc/sys/net/ipv4/conf/lo/arp_ignore
       echo "2" >/proc/sys/net/ipv4/conf/lo/arp_announce
       echo "1" >/proc/sys/net/ipv4/conf/all/arp_ignore
       echo "2" >/proc/sys/net/ipv4/conf/all/arp_announce
       sysctl -p >/dev/null 2>&1
       echo "RealServer Start OK"
       ;;
stop)
       ifconfig lo:0 down
       route del $VIP >/dev/null 2>&1
       echo "0" >/proc/sys/net/ipv4/conf/lo/arp_ignore
       echo "0" >/proc/sys/net/ipv4/conf/lo/arp_announce
       echo "0" >/proc/sys/net/ipv4/conf/all/arp_ignore
       echo "0" >/proc/sys/net/ipv4/conf/all/arp_announce
       echo "RealServer Stoped"
       ;;
*)
       echo "Usage: $0 {start|stop}"
       exit 1
esac
exit 0
```
保存脚本文件后更改该文件权限，然后开启realserver服务

```bash
# chmod 755 /etc/init.d/realserver
# service realserver start
```
* NAT模式（Network Address Translation）

NAT模式下，请求包和响应包都需要经过LB处理。当客户端的请求到达虚拟服务后，LB会对请求包做目的地址转换（DNAT），将请求包的目的IP改写为RS的IP。当收到RS的响应后，LB会对响应包做源地址转换（SNAT），将响应包的源IP改写为LB的IP。

**NAT模式的特点**

* LB会修改数据包的地址

对于请求包，会进行DNAT；对于响应包，会进行SNAT。

* LB会透传客户端IP到RS（DR模式也会透传）

虽然LB在转发过程中做了NAT转换，但是因为只是做了部分地址转发，所以RS收到的请求包里是能看到客户端IP的。

* **需要将RS的默认网关地址配置为LB的浮动IP地址**

因为RS收到的请求包源IP是客户端的IP，为了保证响应包在返回时能走到LB上面，所以需要将RS的默认网关地址配置为LB的虚拟服务IP地址。当然，如果客户端的IP是固定的，也可以在RS上添加明细路由指向LB的虚拟服务IP，不用改默认网关。

* LB和RS须位于同一个子网，并且客户端不能和LB/RS位于同一子网

因为需要将RS的默认网关配置为LB的虚拟服务IP地址，所以需要保证LB和RS位于同一子网。

又因为需要保证RS的响应包能走回到LB上，则客户端不能和RS位于同一子网。否则RS直接就能获取到客户端的MAC，响应包就直接回给客户端了，不会走网关，也就走不到LB上面了。这时候由于没有LB做SNAT，客户端收到的响应包源IP是RS的IP，而客户端的请求包目的IP是LB的虚拟服务IP，这时候客户端无法识别响应包，会直接丢弃。

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/rxm8qF63R10NZShc.png!thumbnail)

* FULLNAT模式

FULLNAT模式下，LB会对请求包和响应包都做SNAT+DNAT。

**FULLNAT模式的特点**

* LB完全作为一个代理服务器

FULLNAT下，客户端感知不到RS，RS也感知不到客户端，它们都只能看到LB。此种模式和七层负载均衡有点相似，只不过不会去解析应用层协议，而是在TCP层将消息转发。

* LB和RS对于组网结构没有要求

不同于NAT和DR要求LB和RS位于一个子网，FULLNAT对于组网结构没有要求。只需要保证客户端和LB、LB和RS之间网络互通即可。

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/DoBkMJD1bGUjdZaC.png!thumbnail)

三种转发模式性能从高到低：DR > NAT >FULLNAT

虽然FULLNAT模式的性能比不上DR和NAT，但是FULLNAT模式没有组网要求，允许LB和RS部署在不同的子网中，这给运维带来了便利。并且 FULLNAT模式具有更好的可拓展性，可以通过增加更多的LB节点，提升系统整体的负载均衡能力。

3. 重启 keepalived 服务，并验证负载均衡是否生效

重启 keepalived 服务

```bash
# service keepalived restart
```
验证负载均衡是否生效

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/za6qbM9lGq8Z2EI2.png!thumbnail)

此时访问的是  http 服务器3

关闭 http 服务器3上的 http 服务

```bash
# service httpd stop
```
![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/q85wcJIh838EOorP.png!thumbnail)

此时访问的是  http 服务器2

# IPVS支持的调度算法

对于后端的RS集群，LB是如何决策应该把消息调度到哪个RS节点呢？这是由负载均衡调度算法决定的。IPVS常用的调度算法有：

* 轮询（Round Robin）

LB认为集群内每台RS都是相同的，会轮流进行调度分发。从数据统计上看，RR模式是调度最均衡的。

* 加权轮询（Weighted Round Robin）

LB会根据RS上配置的权重，将消息按权重比分发到不同的RS上。可以给性能更好的RS节点配置更高的权重，提升集群整体的性能。

* 最小连接数（Least Connections）

LB会根据和集群内每台RS的连接数统计情况，将消息调度到连接数最少的RS节点上。在长连接业务场景下，LC算法对于系统整体负载均衡的情况较好；但是在短连接业务场景下，由于连接会迅速释放，可能会导致消息每次都调度到同一个RS节点，造成严重的负载不均衡。

* 加权最小连接数（Weighted Least Connections）

最小连接数算法的加权版~

* 地址哈希（Address Hash）

LB上会保存一张哈希表，通过哈希映射将客户端和RS节点关联起来。


