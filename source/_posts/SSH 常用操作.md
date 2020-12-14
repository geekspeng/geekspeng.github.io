---
title: SSH 常用操作
date: 2018-03-01
updated: 2018-03-01
tags: [Linux]
categories: [Linux]
---

CentOS的版本号信息一般存放在配置文件当中，在CentOS中，与其版本相关的配置文件中都有centos关键字，该文件一般存放在/etc/目录下，所以说我们可以直接在该文件夹下搜索相关的文件

<!-- more -->

* 口令登录
```bash
# ssh user@host  如：ssh pika@192.168.0.111
```
SSH的publish key和private key都是自己生成的，没法公证。只能通过Client端自己对公钥进行确认。通常在第一次登录的时候，系统会出现下面提示信息：

```bash
Are you sure you want to continue connecting (yes/no)? yes
```
如果输入yes后，会出现下面信息：

```bash
Warning: Permanently added 'ssh-server.example.com,12.18.429.21' (RSA) to the list of known hosts. 
Password: (enter password) 
```
该host已被确认，并被追加到文件known_hosts中，然后就需要输入密码进行登录

* 免密登录
```bash
# ssh-keygen -t rsa
# ssh-copy-id -i ~/.ssh/id_rsa.pub user@host
```
然后输入密码即可，其实就是将本地的公钥写入到远程主机的~/.ssh/authorized_keys文件中
替代方法1

```bash
# cat ~/.ssh/id_rsa.pub  # 本地主机查看公钥
# echo <paste-your-key-here> >> ~/.ssh/authorized_keys # 远程主机写入公钥
```
替代方法2

```bash
# scp ~/.ssh/id_rsa.pub user@host:/root # 本地主机拷贝公钥搭到远程主机
# cat id_rsa.pub >> ~/.ssh/authorized_keys # 远程主机写入公钥
```

* 远程主机重装系统后，需要删除known_hosts保存的host key
```bash
# vim ~/.ssh/known_hosts
# 找到对应的记录，然后按dd
```
known_hosts中存储是已认证的远程主机host key，每个SSH Server都有一个secret, unique ID, called a host key。

* 复制文件到远程主机
```bash
# scp  ./ubuntu/text.txt user@host:/home/
```
-r 递归复制整个目录


* 参考

图解SSH原理

[https://www.jianshu.com/p/33461b619d53](https://www.jianshu.com/p/33461b619d53)

