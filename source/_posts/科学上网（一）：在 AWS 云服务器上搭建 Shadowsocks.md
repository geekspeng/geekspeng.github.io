---
title: 科学上网（一）：在 AWS 云服务器上搭建 Shadowsocks
tags: [科学上网,Shadowsocks,AWS云服务器]
categories: [科学上网]
---
身为程序员平时遇到问题当然需要上网查找资料，普通人首先想到的肯定是百度，但是百度上面给的答案良莠不齐质量不高，这个时候就要借助Goolge，但是由于某些我们都懂的原因在国内我们却访问不了Goolge，这时候我就需要自备梯子。网上虽然有很多免费的服务器可以使用，但是通常质量不会很高，极不稳定，速度也非常慢，所谓一分钱一分货，想不花钱就能用上优质的服务，几乎是不可能。[xhay1122](http://bwg.xhay1122.com/)在他的博客中分享了自己利用廉价的vps搭建的shadowsocks服务器，质量还是不错的比较稳定，但是毕竟是分享给大家用的而且流量有限不敢敞开用，所以最后我选择自己动手，丰衣足食，利用AWS 云服务器搭建Shadowsocks服务器。
<!-- more -->


# Shadowsocks 原理

![](https://i.imgur.com/EsxWsBA.png)

Shadowsocks(ss) 是由 Clowwindy 开发的一款软件，其作用本来是加密传输资料。当然，也正因为它加密传输资料的特性，使得 GFW 没法将由它传输的资料和其他普通资料区分开来（上图），也就不能干扰我们访问那些「不存在」的网站了。

# 创建 AWS 云服务器
先去亚马逊AWS上面注册一个账号：https://amazonaws-china.com/cn/， 只要有信用卡可以免费使用一年AWS的部分服务

## 注册AWS
1. 点击注册进入注册页面
![](https://i.imgur.com/3BrSqCr.png)

2. 填写邮件地址，密码，账户名称
![](https://i.imgur.com/QMm5QIG.png)

3. 这里选择个人就好了，然后填写个人信息
![](https://i.imgur.com/Xnot9Jj.png)

4. 填写信用卡信息

## 创建 AWS 实例

登录后进入AWS的控制台，点击构建解决方案中的 
**启动虚拟机——> 开始使用 ——> 更多选项——> 高级 EC2 启动实例向导 ——>AWS Marketplace ——> 搜索”centos6”** 


# 部署 Shadowsocks
Shadowsocks 需要同时具备客户端和服务器端，所以它的部署也需要分两步

##  部署 Shadowsocks 服务器
这里使用 teddysun 的一键安装脚本。先切换到root用户

```
su root
```
可使用 sudo passwd root 先修改root密码，然后执行以下是3条命令，每次输入一行、回车，等待屏幕上的操作完成后再输入下一条。

```
wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks.sh
chmod +x shadowsocks.sh
./shadowsocks.sh 2>&1 | tee shadowsocks.log
```

最后一步输完，你应该会看到下图中内容──是要你为 Shadowsocks 服务设置一个个人密码
![](https://i.imgur.com/0Mh4IN6.jpg)

输好回车后会让你选择一个端口，输入 1-65535 间的数字都行
![](https://i.imgur.com/3pN3qBj.jpg)

遵照上图指示，按任意键开始部署 Shadowsocks。这时你什么都不用做，只需要静静地等它运行完就好。结束后就会看到你所部署的 Shadowsocks 的配置信息。

![](https://i.imgur.com/4RDAaJ3.jpg)

复制其中黄框中的内容，也就是服务器 IP、服务器端口、你设的密码和加密方式。你就可以在自己任意的设备上进行登录使用了。

## 安装 Shadowsocks 客户端
根据操作系统下载相应的客户端。
[Mac 版客户端下载](https://github.com/shadowsocks/shadowsocks-iOS/releases)
[Win 版客户端下载](https://github.com/shadowsocks/shadowsocks-windows/releases)
打开客户端，在「服务器设定」里新增服务器。然后依次填入服务器 IP、服务器端口、你设的密码和加密方式。

Mac 版客户端
![](https://i.imgur.com/Q9ii2BM.png)

Win 版客户端
![](https://i.imgur.com/BH5674r.png)

然后启用代理，就可以实现科学上网了

Mac 版客户端，点击打开Shadowsocks
![](https://i.imgur.com/fvuRZ2o.png)

Win 版客户端，点击”启用系统代理”，选择PAC模式，在PAC中选择从xxx更新本地PAC
![](https://i.imgur.com/kuM3l6p.png)


## 提升Shadowsocks服务器速度
实际上你已经可以在自己的任意设备上进行登录使用了。但是为了更好的连接速度，你还需要多做几步。

### TCP Fast Open
首先是打开 TCP Fast Open，vi /etc/rc.local ，在最后增加如下内容：

```
echo 3 > /proc/sys/net/ipv4/tcp_fastopen
```

然后用同样的方法修改 /etc/sysctl.conf，在最后增加如下内容：

```
net.ipv4.tcp_fastopen = 3
```

再打开一个 Shadowsocks 配置文件，编辑 /etc/shadowsocks.json，把其中 "fast_open" 一项的 false 替换成 true 修改如下：

```
"fast_open":true
```
最后，输入以下命令重启 Shadowsocks：

```
/etc/init.d/shadowsocks restart
```

### 开启锐速
[锐速 ServerSpeeder](https://github.com/91yun/serverspeeder) 是一个 TCP 加速软件，对 Shadowsocks 客户端和服务器端间的传输速度有显著提升。

不同于 FinalSpeed 或 Kcptun 等需要客户端的工具，「锐速」的一大优势是只需要在服务器端单边部署就行了。另外，「锐速」虽然已经停止注册和安装了，不过网上还是有不少「破解版」可用

锐速破解版一键安装：

```
wget -N --no-check-certificate https://github.com/91yun/serverspeeder/raw/master/serverspeeder.sh && bash serverspeeder.sh
```

安装上面官网的的安装步骤执行一键安装脚本会出现如下的错误信息：

```
前面的省略...
Complete!
=================================================
操作系统：CentOS 
发行版本：7.4 
内核版本：3.10.0-693.el7.x86_64 
位数：x64 
锐速版本：3.10.61.0 
=================================================
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 96179  100 96179    0     0  77305      0  0:00:01  0:00:01 --:--:-- 77314


锐速暂不支持该内核，程序退出.自动安装判断比较严格，你可以到http://www.91yun.org/serverspeeder91yun手动下载安装文件尝试不同版本

```

**监测VPS架构**

```
wget -N --no-check-certificate https://raw.githubusercontent.com/91yun/code/master/vm_check.sh && bash vm_check.sh
```
如果是kvm还是xen或者vmare则可以装锐速，如果是Openvz，则不可装锐速

**改核适配锐速**
**Centos7安装内核（不太推荐）**

CentOS 7支持安装锐速的内核：3.10.0-327.el7.x86_64

```
yum install -y linux-firmware
rpm -ivh http://xz.wn789.com/CentOSkernel/kernel-3.10.0-229.1.2.el7.x86_64.rpm --force
rpm -qa | grep kernel #查看内核是否安装成功
reboot #重启VPS
uname -r #当前使用内核版本
```
锐速针对Centos 7的版本较少，推荐在Centos 6中安装

成功界面如下，看到license信息过期时间为”2034-12-31”就没问题了。
![](https://i.imgur.com/7tbRVYX.jpg)

部署锐速
依然使用一键安装脚本，输入以下命令：

```
wget -N --no-check-certificate https://raw.githubusercontent.com/91yun/serverspeeder/master/serverspeeder-all.sh && bash serverspeeder-all.sh
```
安装需要一段时间，等待一会。

至此，整个搭建过程就大功告成了！接下来，尽情地享受起飞的速度吧😄









