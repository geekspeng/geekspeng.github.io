---
title: 科学上网（二）：极路由配置Shadowsocks
date: 2017-12-15
updated: 2017-12-15
tags: [科学上网,Shadowsocks,极路由]
categories: [科学上网]
---

上一篇讲了如何在 AWS 云服务器上搭建 Shadowsocks服务器从而实现科学上网，这篇我们主要讲怎么在极路由上配置Shadowsocks，从而使连上这台路由器的设备都可以轻松地访问goolge等网站而不需在所有需要科学上网的设备上进行配置。
<!-- more -->

# Shadowsocks客户端安装
通过[hiwifi-ss](https://github.com/qiwihui/hiwifi-ss)开源项目我们可以轻松的在极路由安装Shadowsocks客户端。

首先电脑WiFi连接到极路由，由于需要从网上下载Shadowsocks客户端，所以请确保极路由能连上网（插上网线或者作为无线中继）。

然后我们通过SSH登录到极路由的控制台，我使用的MAC默认带有SSH，如果没有SSH请自行百度安装，端口号默认使用的是1022，这里需要输入WiFi密码

```
xuepengdeMBP:~ geekspeng$ ssh root@hiwifi.com -p 1022
root@hiwifi.com's password:
```

登录成功后会显示下面的信息

```
BusyBox v1.22.1 (2017-08-10 17:53:48 CST) built-in shell (ash)
Enter 'help' for a list of built-in commands.

***********************************************************
              __  __  _              _   ____  _   TM
             / / / / (_) _      __  (_) / __/ (_)
            / /_/ / / / | | /| / / / / / /_  / /
           / __  / / /  | |/ |/ / / / / __/ / /
          /_/ /_/ /_/   |__/|__/ /_/ /_/   /_/
                  http://www.hiwifi.com/
***********************************************************
root@Hiwifi:~#
```
最后安装Shadowsocks客户端

**1.新版hiwifi** => 使用项目根目录下的 shadow.sh 脚本进行安装, 建议使用以下一键命令:

```
cd /tmp && curl -k -o shadow.sh https://raw.githubusercontent.com/qiwihui/hiwifi-ss/master/shadow.sh && sh shadow.sh && rm shadow.sh
```

**2.hiwifi 1.2.5.15805s**

```
cd /tmp && curl -k -o shadow.sh https://raw.githubusercontent.com/qiwihui/hiwifi-ss/master/shadow.sh && sh shadow.sh 12515805s && rm shadow.sh
```

# Shadowsocks账号设置

在浏览器中登录极路由后台，完成配置后，点击开关开启即可，如果一切正常，下方状态会显示连接正常
![](http://p15d1hccg.bkt.clouddn.com/15132641329660.jpg)

