---
title: 科学上网（一）：在 AWS 云服务器上搭建 Shadowsocks
date: 2017-12-14
updated: 2017-12-14
tags: [科学上网,Shadowsocks,AWS云服务器]
categories: [科学上网]
---
身为程序员平时遇到问题当然需要上网查找资料，普通人首先想到的肯定是百度，但是百度上面给的答案良莠不齐质量不高，这个时候就要借助Goolge，但是由于某些我们都懂的原因在国内我们却访问不了Goolge，这时候我就需要自备梯子。网上虽然有很多免费的服务器可以使用，但是通常质量不会很高，极不稳定，速度也非常慢，所谓一分钱一分货，想不花钱就能用上优质的服务，几乎是不可能。[xhay1122](http://bwg.xhay1122.com/)在他的博客中分享了自己利用廉价的vps搭建的shadowsocks服务器，质量还是不错的比较稳定，但是毕竟是分享给大家用的而且流量有限不敢敞开用，所以最后我选择自己动手，丰衣足食，利用AWS 云服务器搭建Shadowsocks服务器。
<!-- more -->


# Shadowsocks 原理

![](http://p15d1hccg.bkt.clouddn.com/15132555841950.png)

Shadowsocks(ss) 是由 Clowwindy 开发的一款软件，其作用本来是加密传输资料。当然，也正因为它加密传输资料的特性，使得 GFW 没法将由它传输的资料和其他普通资料区分开来（上图），也就不能干扰我们访问那些「不存在」的网站了。

# 创建 AWS 云服务器
先去亚马逊AWS上面注册一个账号：https://amazonaws-china.com/cn/， 只要有信用卡可以免费使用一年AWS的部分服务

## 注册AWS
1. 点击注册进入注册页面
![](http://p15d1hccg.bkt.clouddn.com/15132560890311.png)

2. 填写邮件地址，密码，账户名称
![](http://p15d1hccg.bkt.clouddn.com/15132563312499.png)

3. 这里选择个人就好了，然后填写个人信息
![](http://p15d1hccg.bkt.clouddn.com/15132568272500.png)

4. 填写信用卡信息
![](http://p15d1hccg.bkt.clouddn.com/15136977189629.png)
绑定完信用卡之后，信用卡会扣取1美元的费用，网上看的教程说后面会退还.

5. 电话验证
填写完信息后让系统拨打你的电话，然后页面上会显示出一个PIN码，在电话上输入即可


## 创建 AWS 实例
1. 点击右上角切换服务器机房，建议选择亚太地区的服务器，因为亚太地区的服务器相对于北美的服务器延迟要低一些，这里我选择 **东京**
![](http://p15d1hccg.bkt.clouddn.com/15136024142885.png)

2. 点击左上角的服务选择 **EC2**
![](http://p15d1hccg.bkt.clouddn.com/15136026702644.png)

3. 点击 **启动实例**
![](http://p15d1hccg.bkt.clouddn.com/15136027931278.png)

4. 在 **AWS Marketplace** 搜索 **centos6** ,然后点击 **选择**
![](http://p15d1hccg.bkt.clouddn.com/15136029683293.png)

5. 点击 **continue**
![](http://p15d1hccg.bkt.clouddn.com/15136030558122.png)

6. 这里我们就直接选用免费的就可以了，然后点击 **下一步**
![](http://p15d1hccg.bkt.clouddn.com/15136032510119.png)

7. 后面我们都使用默认配置，都直接点击下一步，直到 **配置安全组** 的时候，我们将类型改成 **所有流量**，然后点击 **审核和启动**
![](http://p15d1hccg.bkt.clouddn.com/15136034503025.png)

8. 点击右下角的 **启动** 会弹出一个密钥窗口选择 **创建新密钥对** ，接着填写密钥名称，点击 **下载密钥** ，最后点击 **启动实例**
![](http://p15d1hccg.bkt.clouddn.com/15136042148471.png)

9. 接着初始化主机，初始化完成后出现下面的这个界面，点击右下角的 **查看实例**
![](http://p15d1hccg.bkt.clouddn.com/15136045186657.png)

## 连接 AWS 实例
1. 点击 **连接**
![](http://p15d1hccg.bkt.clouddn.com/15136047734339.png)

2. 按照提示我们直接通过ssh连接，首先打开SSH客户端，我用的MAC自带SSH，所以直接打开终端（Windows可以根据提示使用PuTTY连接），并将路径切换到之前保存密钥的路径下，然后根据提示修改密钥的权限，最后复制下面的示例并将root改为centos（我们安装的centos系统的用户名是centos）
![](http://p15d1hccg.bkt.clouddn.com/15136451779598.png)

```
Last login: Tue Dec 19 09:01:19 on ttys002
xuepengdeMacBook-Pro:~ geekspeng$ cd Downloads/
xuepengdeMacBook-Pro:Downloads geekspeng$ ls
shadowsock.pem
xuepengdeMacBook-Pro:Downloads geekspeng$ chmod 400 shadowsock.pem
xuepengdeMacBook-Pro:Downloads geekspeng$ ssh -i "shadowsock.pem" centos@ec2-13-115-236-100.ap-northeast-1.compute.amazonaws.com
[centos@ip-172-31-22-183 ~]$
```

# 部署 Shadowsocks
Shadowsocks 需要同时具备客户端和服务器端，所以它的部署也需要分两步

##  部署 Shadowsocks 服务器
这里使用 teddysun 的一键安装脚本。

可使用 sudo passwd root 先修改root密码，然后切换到root用户

```
[centos@ip-172-31-22-183 ~]$ sudo passwd root
Changing password for user root.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
[centos@ip-172-31-22-183 ~]$ su root
Password:
[root@ip-172-31-22-183 centos]#
```
这里需要安装 wget，后面需要用到，需要确认的地方都输入y就可以了

```
[root@ip-172-31-22-183 centos]# yum install wget
```

然后执行以下是3条命令，每次输入一行、回车，等待屏幕上的操作完成后再输入下一条。

```
wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks.sh
chmod +x shadowsocks.sh
./shadowsocks.sh 2>&1 | tee shadowsocks.log
```

上面最后一步输完，按照提示输入进行设置，设置完过后按任意键开始部署 Shadowsocks。这时你什么都不用做，只需要静静地等它运行完就好。结束后就会看到你所部署的 Shadowsocks 的配置信息。

```
Congratulations, Shadowsocks-python server install completed!
Your Server IP        :  13.115.236.100
Your Server Port      :  8989
Your Password         :  ********
Your Encryption Method:  aes-256-cfb

Welcome to visit:https://teddysun.com/342.html
Enjoy it!
```

复制服务器 IP、服务器端口、你设的密码和加密方式。你就可以在自己任意的设备上进行登录使用了。

## 安装 Shadowsocks 客户端

根据操作系统下载相应的客户端。
[Mac 版客户端下载](https://github.com/shadowsocks/shadowsocks-iOS/releases)
[Win 版客户端下载](https://github.com/shadowsocks/shadowsocks-windows/releases)

**打开客户端，在「服务器设定」里新增服务器。然后依次填入服务器 IP、服务器端口、你设的密码和加密方式。**

Mac 版客户端
![](http://p15d1hccg.bkt.clouddn.com/15132596983585.png)

Win 版客户端
![](http://p15d1hccg.bkt.clouddn.com/15132591141053.png)

**然后启用代理，就可以实现科学上网了**

Mac 版客户端，点击打开Shadowsocks
![](http://p15d1hccg.bkt.clouddn.com/15132597970585.png)

Win 版客户端，点击”启用系统代理”，选择PAC模式，在PAC中选择从xxx更新本地PAC
![](http://p15d1hccg.bkt.clouddn.com/15132598862289.png)


## 提升Shadowsocks服务器速度
实际上你已经可以在自己的任意设备上进行使用了。但是为了更好的连接速度，你还需要多做几步。

### TCP Fast Open

**编辑 /etc/rc.local 文件，按照下面的步骤操作**

1） 首先打开rc.local文件

```
vi /etc/rc.local
```
2） 然后按 **i键** 进入编辑模式，通过 ↑  ↓ ← →按键移动光标，在最后增加如下内容：echo 3 > /proc/sys/net/ipv4/tcp_fastopen

```
#!/bin/sh
#
# This script will be executed *after* all the other init scripts.
# You can put your own initialization stuff in here if you don't
# want to do the full Sys V style init stuff.

touch /var/lock/subsys/local

# set a random pass on first boot
if [ -f /root/firstrun ]; then
  dd if=/dev/urandom count=50|md5sum|passwd --stdin root
  passwd -l root
  rm /root/firstrun
fi

if [ ! -d /root/.ssh ]; then
  mkdir -m 0700 -p /root/.ssh
  restorecon /root/.ssh
fi
echo 3 > /proc/sys/net/ipv4/tcp_fastopen
~
~
~
~

```

3）编辑完过后首先按**ESC键**，再输入**:wq**即可以保存退出了

**然后按照同样的方法修改 /etc/sysctl.conf，在最后增加如下内容：**

```
net.ipv4.tcp_fastopen = 3
```

**再打开一个 Shadowsocks 配置文件，编辑 /etc/shadowsocks.json，把其中 "fast_open" 一项的 false 替换成 true 修改如下：**

```
"fast_open":true
```

**最后，输入以下命令重启 Shadowsocks：**

```
/etc/init.d/shadowsocks restart
```

### 开启锐速
[锐速 ServerSpeeder](https://github.com/91yun/serverspeeder) 是一个 TCP 加速软件，对 Shadowsocks 客户端和服务器端间的传输速度有显著提升。

不同于 FinalSpeed 或 Kcptun 等需要客户端的工具，「锐速」的一大优势是只需要在服务器端单边部署就行了。另外，「锐速」虽然已经停止注册和安装了，不过网上还是有不少「破解版」可用

**锐速破解版一键安装：**

```
wget -N --no-check-certificate https://github.com/91yun/serverspeeder/raw/master/serverspeeder.sh && bash serverspeeder.sh
```

安装上面官网的的安装步骤执行一键安装脚本会出现如下的错误信息：

```
前面的省略...
===============System Info=======================
CentOS
2.6.32-696.1.1.el6.x86_64
x64
=================================================


--2017-12-19 14:14:49--  https://raw.githubusercontent.com/91yun/serverspeeder/test/serverspeederbin.txt
Resolving raw.githubusercontent.com... 151.101.72.133
Connecting to raw.githubusercontent.com|151.101.72.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 96179 (94K) [text/plain]
Saving to: “serverspeederbin.txt”

100%[======================================>] 96,179      --.-K/s   in 0.006s

2017-12-19 14:14:49 (16.5 MB/s) - “serverspeederbin.txt” saved [96179/96179]

>>>This kernel is not supported. Trying fuzzy matching...




Serverspeeder is not supported on this kernel! View all supported systems and kernels here: https://www.91yun.org/serverspeeder91yun

```

**监测VPS架构**

```
wget -N --no-check-certificate https://raw.githubusercontent.com/91yun/code/master/vm_check.sh && bash vm_check.sh
```
如果是kvm还是xen或者vmare则可以装锐速，如果是Openvz，则不可装锐速

**改核适配锐速**

CentOS 6支持安装锐速的内核：2.6.32–504.3.3.el6.x86_64

```
uname -r #查看当前内核版本
rpm -ivh http://xz.wn789.com/CentOSkernel/kernel-firmware-2.6.32-504.3.3.el6.noarch.rpm
rpm -ivh http://xz.wn789.com/CentOSkernel/kernel-2.6.32-504.3.3.el6.x86_64.rpm --force
rpm -qa | grep kernel #查看是否安装成功
reboot #重启VPS
uname -r #当前使用内核版本
```

**部署锐速**
依然使用一键安装脚本，输入以下命令：

```
wget -N --no-check-certificate https://raw.githubusercontent.com/91yun/serverspeeder/master/serverspeeder-all.sh && bash serverspeeder-all.sh
```
安装需要一段时间，等待一会。成功界面如下，看到license信息过期时间为”2034-12-31”就没问题了。

```
[Running Status]
ServerSpeeder is running!
version              3.10.61.0

[License Information]
License              763D1329B3D1824A (valid on current device)
MaxSession           unlimited
MaxTcpAccSession     unlimited
MaxBandwidth(kbps)   unlimited
ExpireDate           2034-12-31

[Connection Information]
TotalFlow            1
NumOfTcpFlows        1
TotalAccTcpFlow      0
TotalActiveTcpFlow   0

[Running Configuration]
accif                eth0
acc                  1
advacc               1
advinacc             1
wankbps              10000000
waninkbps            10000000
csvmode              0
subnetAcc            0
maxmode              1
pcapEnable           0
```

至此，整个搭建过程就大功告成了！接下来，尽情地享受起飞的速度吧😄

