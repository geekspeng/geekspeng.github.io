---
title: nginx + httpd负载均衡和缓存
date: 2019-09-18
updated: 2019-09-18
tags: [nginx,负载均衡]
categories: [负载均衡]
---

# 实验环境

```bash
操作系统：centos7.5
httpd服务器1: 10.0.0.101
httpd服务器2: 10.0.0.102
nginx服务器: 192.168.46.103
```
<!-- more -->

# 安装并启动httpd

1. 在需要部署httpd 的节点上安装 httpd
```bash
# yum install httpd -y
```
2. 设置httpd首页显示信息，这里设置显示服务器 ip 地址，方便我们判断访问的是哪台服务器

http 服务器1

```bash
# echo "10.0.0.101" >/var/www/html/index.html
```
http 服务器2

```bash
# echo "10.0.0.102" >/var/www/html/index.html
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

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/DyGqYnDrOmQ7Tkyc.png!thumbnail)

http 服务器2

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/wLoIS462ZyEdaaMq.png!thumbnail)

# Nginx 实现 httpd 负载均衡

* 安装 nginx
```bash
# yum -y install epel-release #默认Centos7没有nginx源，需要安装epel的yum源
# yum install -y nginx
```
* 配置 nginx 实现httpd 的负载均衡

默认的配置文件为

```bash
/etc/nginx/nginx.conf
```
我们可以改默认的配置文件，也可以在/etc/nginx/conf.d/ 新增配置文件，默认配置会include /etc/nginx/conf.d/ 目录下的配置

在/etc/nginx/conf.d/ 目录下新建webservers.conf 文件，配置如下：

```ini
# 负载均衡
upstream webservers { 
  server 10.0.0.101; # httpd 服务器1
  server 10.0.0.102; # httpd 服务器2
}
server {
  listen 80;
  server_name upstream.geekspeng.com; # 域名
  location / {
      proxy_pass http://webservers;  # 反向代理
      proxy_set_header  X-Real-IP  $remote_addr;
  }
}
```
* 测试配置是否正确并启动 nginx
```bash
[root@node3 nginx]# nginx -t # 测试配置是否正确
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@node3 ~]# systemctl start nginx
[root@node3 ~]# systemctl enable nginx
```
* 访问[http://upstream.geekspeng.com/](http://upstream.geekspeng.com/)

由于nginx 默认配置文件默认配置80 端口指向nginx 测试页面（如下图），所以这里我们配置通过域名 upstream.geekspeng.com 访问 nginx服务器（可以通过配置不同的域名指向不同的应用程序）

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/j4NQqznDgNgwIMAy.png!thumbnail)

为了能通过域名访问 nginx服务器，我们需要在 hosts 文件追加添加一条记录

win7 的host 文件在 C:\Windows\System32\drivers\etc\hosts

```bash
10.0.0.103 upstream.geekspeng.com
```
通过域名访问，此时请求会交替的转发给httpd 服务器

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/421tJCYBcMUbmNJT.gif)

# Nginx 实现服务器静态缓存

* 配置 nginx 实现缓存

在/etc/nginx/conf.d/ 目录下新建cache.conf 文件，配置如下：

```ini
proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=cache_zone:10m inactive=1d max_size=100m;
server  {
listen 80;
server_name cache.geekspeng.com; # 域名
	location / {
		proxy_cache cache_zone; #keys_zone的名字
		proxy_cache_key $host$uri$is_args$args; #缓存规则
		proxy_cache_valid any 1m;
		proxy_pass http://10.0.0.101;
	}
}
```
首先需要在http中加入proxy_cache_path，用来制定缓存的目录以及缓存目录深度制定等。它的格式如下：

```ini
proxy_cache_path path [levels=number] keys_zone=zone_name:zone_size [inactive=time] [max_size=size];
```
>path是用来指定缓存在磁盘的路径地址。比如：/data/nginx/cache。那以后生存的缓存文件就会存在这个目录下。
>levels用来指定缓存文件夹的级数，可以是：levels=1, levels=1:1, levels=1:2, levels=1:2:3 可以使用任意的1位或2位数字作为目录结构分割符，如 X, X:X,或 X:X:X 例如: 2, 2:2, 1:1:2，但是最多只能是三级目录。

那这个里面的数字是什么意思呢。表示取hash值的个数。比如：

>现在根据请求地址localhost/index.php?a=4用md5进行哈希，得到e0bd86606797639426a92306b1b98ad9
>levels=1:2 表示建立2级目录，把hash最后1位(9)拿出建一个目录，然后再把9前面的2位(ad)拿来建一个目录, 那么缓存文件的路径就是/data/nginx/cache/9/ad/e0bd86606797639426a92306b1b98ad9
>以此类推：levels=1:1:2表示建立3级目录，把hash最后1位(9)拿出建一个目录，然后再把9前面的1位(d)建一个目录, 最后把d前面的2位(8a)拿出来建一个目录 那么缓存文件的路径就是/data/nginx/cache/9/d/8a/e0bd86606797639426a92306b1b98ad9

keys_zone 所有活动的key和元数据存储在共享的内存池中，这个区域用keys_zone参数指定。zone_name指的是共享池的名称， zone_size指的是共享池的大小。注意每一个定义的内存池必须是不重复的路径，例如：

```
proxy_cache_path /data/nginx/cache/one levels=1 keys_zone=one:10m;
proxy_cache_path /data/nginx/cache/two levels=2 keys_zone=two:100m;
proxy_cache_path /data/nginx/cache/three levels=1 keys_zone=three:10m;
```
>inactive 表示指定的时间内缓存的数据没有被请求则被删除，默认inactive为10分钟。inactive=1d 1小时。inactive=30m 30分钟。
>max_size 表示单个文件最大不超过的大小。它被用来删除不活动的缓存和控制缓存大小，当目前缓存的值超出max_size指定的值之后， 超过其大小后最少使用数据（LRU替换算法）将被删除。max_size=10g表示当缓存池超过10g就会清除不常用的缓存文件。
>clean_time 表示每间隔自动清除的时间。clean_time=1m 1分钟清除一次缓存。

说完了这个很重要的参数。我们再来说在server模块里的几个配置参数：

>proxy_cache用来指定用哪个keys_zone的名字，也就是用哪个目录下的缓存。 上面我们指定了三个one,two,three。比如，我现在想用one这个缓存目录: proxy_cache one
>proxy_cache_key 这个其实蛮重要的，它用来指定生成hash的url地址的格式。根据这个key映射成一个hash值， 然后存入到本地文件。proxy_cache_key $host$uri表示无论后面跟的什么参数，都会访问一个文件，不会再生成新的文件。 而如果proxy_cache_key $is_args$args，那么传入的参数localhost/index.php?a=4与localhost/index.php?a=44将映射成两个不同hash值的文件。
>proxy_cache_key 默认是 “$scheme$host$request_uri”。但是一般我们会把它设置成：$host$uri$is_args$args一个完整的url路径。
>proxy_cache_valid 它是用来为不同的http响应状态码设置不同的缓存时间。
```
proxy_cache_valid  200 302 10m;
proxy_cache_valid  301 1h;
proxy_cache_valid  any 1h; #所有的状态都缓存1小时
```
表示为http status code 为200和302的设置缓存时间为10分钟，404代码缓存1分钟。
* 测试配置是否正确并重新加载 nginx
```bash
[root@node3 nginx]# nginx -t # 测试配置是否正确
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful[root@node3 ~]# systemctl reload nginx
```
* 访问[http://cache.geekspeng.com/](http://upstream.geekspeng.com/)

为了能通过域名访问 nginx服务器，我们需要在 hosts 文件追加添加一条记录

win7 的host 文件在 C:\Windows\System32\drivers\etc\hosts

```
10.0.0.103 cache.geekspeng.com
```
* 查看 nginx 缓存目录
```bash
[root@node3 ~]# tree /var/cache/nginx/
/var/cache/nginx/
└── 5
    └── a7
        └── 2b21765c7b4c13abf90a75280fd43a752 directories, 1 file
```
* 查看缓存文件内容
```bash
[root@node3 ~]# cat /var/cache/nginx/5/a7/2b21765c7b4c13abf90a75280fd43a75`kn"b-5846a255896c3"
KEY: cache.geekspeng.com/
HTTP/1.1 200 OK
Date: Wed, 20 Mar 2019 08:41:53 GMT
Server: Apache/2.4.6 (CentOS)
Last-Modified: Tue, 19 Mar 2019 03:31:51 GMT
ETag: "b-5846a255896c3"
Accept-Ranges: bytes
Content-Length: 11
Connection: close
Content-Type: text/html; charset=UTF-810.0.0.101
```
* 刷新网页，再次访问查看浏览器 和 httpd 服务器情况

打开Chrome开发者工具，并且切换到Network，刷新页面

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/ePZ2mHMlFr8iaB3H.png!thumbnail)


在 Response Headers 中我们可以看到：

```
X-Cache: HIT
X-Via: 10.0.0.103
```
>MISS 未命中，请求被传送到后端
>HIT 缓存命中
>EXPIRED 缓存已经过期请求被传送到后端
>UPDATING 正在更新缓存，将使用旧的应答
>STALE 后端将得到过期的应答
>BYPASS 缓存被绕过了

查看httpd 服务器1 的访问日志，如果缓存没有过期，此时不会新增访问日志

```bash
[root@node1 ~]# tailf /var/log/httpd/access_log tailf /var/log/httpd/access_log
```
* 更新httpd 服务器1 的内容后，再次访问查看浏览器 和 httpd 服务器情况

更新 httpd 服务器1 的内容

```bash
[root@node1 ~]# echo "10.0.0.101 \n 10.0.0.101" >/var/www/html/index.html
```
打开Chrome开发者工具，并且切换到Network，刷新页面

在 Response Headers 中我们可以看到 cache 已经过期：

```
X-Cache: EXPIRED
X-Via: 10.0.0.103
```
查看httpd 服务器1 的访问日志，此时会看到访问信息

```bash
[root@node1 ~]# tailf /var/log/httpd/access_log tailf /var/log/httpd/access_log
10.0.0.103 - - [20/Mar/2019:04:51:46 -0400] "GET / HTTP/1.0" 200 14 "-" "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36"
```
查看缓存文件内容，此时缓存已经更新

```bash
[root@node3 ~]# cat /var/cache/nginx/5/a7/2b21765c7b4c13abf90a75280fd43a75
`ko"ֽ19-58482a752aad1"
KEY: cache.geekspeng.com/
HTTP/1.1 200 OK
Date: Wed, 20 Mar 2019 08:46:13 GMT
Server: Apache/2.4.6 (CentOS)
Last-Modified: Wed, 20 Mar 2019 08:46:11 GMT
ETag: "19-58482a752aad1"
Accept-Ranges: bytes
Content-Length: 25
Connection: close
Content-Type: text/html; charset=UTF-8
10.0.0.101 \n 10.0.0.101
```
# Nginx 实现浏览器静态缓存

```ini
# 静态文件，nginx自己处理
location ~ ^/(images|javascript|js|css|flash|media|static)/ {
	root /var/www/virtual/htdocs;
	expires 30d;
    # 过期30天，静态文件不怎么更新，过期可以设大一点，如果频繁更新，则可以设置得小一点
}
```
# 反向代理示例

```ini
## 下面配置反向代理的参数
server {
    listen    80;
    ## 1. 用户访问 http://ip:port，则反向代理到 https://github.com
    location / {
        proxy_pass  https://github.com;
        proxy_redirect     off;
        proxy_set_header   Host            $host;
        proxy_set_header   X-Real-IP       $remote_addr;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```
# 遗留问题

1. 动态缓存
2. 缓存更新策略
