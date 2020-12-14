---
title: ab 进行压力测试
date: 2019-03-11
updated: 2019-03-11
tags: [压力测试,高性能]
categories: [高性能]
---

# 安装 ab（apache bench）

通过安装 httpd 的方式，顺带安装ab

```bash
# yum -y install httpd
```
如果不想安装 httpd 但是又想使用ab命令的话，可以只安装 httpd-tools

```bash
# yum -y install httpd-tools
```
<!-- more -->

# 使用 ab 进行压力测试

## ab 的使用入门

ab的命令参数比较多，我们经常使用的是-c （并发用户数） 和 -n（请求总数），这里以[https://www.baidu.com/](https://www.baidu.com/)为例进行说明

```bash
# ab -c 10 -n 100 'https://www.baidu.com'
This is ApacheBench, Version 2.3 <$Revision: 1430300 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/
Benchmarking www.baidu.com (be patient).....done
Server Software:        BWS/1.1
Server Hostname:        www.baidu.com
Server Port:            443
SSL/TLS Protocol:       TLSv1.2,ECDHE-RSA-AES128-GCM-SHA256,2048,128
Document Path:          /
Document Length:        227 bytes
Concurrency Level:      10
Time taken for tests:   1.777 seconds
Complete requests:      100
Failed requests:        0
Write errors:           0
Total transferred:      89300 bytes
HTML transferred:       22700 bytes
Requests per second:    56.29 [#/sec] (mean)
Time per request:       177.650 [ms] (mean)
Time per request:       17.765 [ms] (mean, across all concurrent requests)
Transfer rate:          49.09 [Kbytes/sec] received
Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:       88  114  47.8    105     355
Processing:    29   40  46.6     34     370
Waiting:       29   39  36.8     34     300
Total:        118  155  64.8    141     467
Percentage of the requests served within a certain time (ms)
  50%    141
  66%    144
  75%    146
  80%    147
  90%    163
  95%    366
  98%    452
  99%    467
 100%    467 (longest request)
```
主要看以下几组数据

* 吞吐率（Requests per second）

服务器并发处理能力的量化描述，单位是reqs/s，指的是在某个并发用户数下单位时间内处理的请求数。某个并发用户数下单位时间内能处理的最大请求数，称之为最大吞吐率

记住：吞吐率是基于并发用户数的。这句话代表了两个含义：

a、吞吐率和并发用户数相关

b、不同的并发用户数下，吞吐率一般是不同的

计算公式：

总请求数/处理完成这些请求数所花费的时间，即

Request per second = Complete requests / Time taken for tests

* 用户平均请求等待时间（Time per request）

计算公式：

处理完成所有请求数所花费的时间 /（总请求数/并发用户数），即

Time per request = Time taken for tests /（ Complete requests / Concurrency Level）

* 服务器平均请求等待时间（Time per request:across all concurrent requests）

计算公式：

处理完成所有请求数所花费的时间/总请求数，即

Time taken for / testsComplete requests

可以看到，它是吞吐率的倒数，也等于用户平均请求等待时间/并发用户数，即

Time per request / Concurrency Level

* 并发连接数（The number of concurrent connections）

并发连接数指的是某个时刻服务器所接受的请求数目，简单的讲，就是一个会话

* 并发用户数（Concurrency Level）

要注意区分这个概念和并发连接数之间的区别，一个用户可能同时会产生多个会话，也即连接数

## ab 的使用进阶

* 发送 get 请求
```bash
# ab -c 10 -n 100 'https://www.baidu.com/s?wd=ab'
This is ApacheBench, Version 2.3 <$Revision: 1430300 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/Benchmarking www.baidu.com (be patient).....doneServer Software:        BWS/1.1
Server Hostname:        www.baidu.com
Server Port:            443
SSL/TLS Protocol:       TLSv1.2,ECDHE-RSA-AES128-GCM-SHA256,2048,128Document Path:          /s?wd=ab
Document Length:        227 bytesConcurrency Level:      10
Time taken for tests:   1.853 seconds
Complete requests:      100
Failed requests:        0
Write errors:           0
Total transferred:      89300 bytes
HTML transferred:       22700 bytes
Requests per second:    53.97 [#/sec] (mean)
Time per request:       185.288 [ms] (mean)
Time per request:       18.529 [ms] (mean, across all concurrent requests)
Transfer rate:          47.07 [Kbytes/sec] receivedConnection Times (ms)
              min  mean[+/-sd] median   max
Connect:       87  103   9.1    104     147
Processing:    34   55  27.2     40     114
Waiting:       34   54  27.2     39     113
Total:        122  158  30.8    147     252Percentage of the requests served within a certain time (ms)
  50%    147
  66%    151
  75%    188
  80%    196
  90%    204
  95%    216
  98%    233
  99%    252
 100%    252 (longest request)
```
* 发送 post 请求
```bash
# ab -c 10 -n 100 -v 4 -p 'userlogin.txt' 'http://test.com/'
```
userlogin.txt 内容如下：

```bash
user_name=test&password=test
```
* 发送带 Cookie 的请求
```bash
# ab -c 10 -n 100 －C key1=value1;key2=value2 http://test.com/
```
* 发送带 Header 的请求
```bash
# ab -c 10 -n 100 -H 'Host: 10.0.40.11' 'http://test.com'
```
```bash
ab -c1 -n1 -v4 -p project.txt -H 'Cookie: user_id=f3cebf17a6b14c6482ea8b618aed715c; user_name=administrator; project_id=154c5031aa3748d7a7deada6d5fce1f6; project_name=admin; acl_type=admin; hyhive_token=AjcBLwinKbn3uIpmkNsJQxmkveiZYFL46yj_csHR9TWNwm_erc77aL3fpBi7yF2Hq-zJRDvdJ9ns_7asAYWtb1ac3oAx68UiRpKw0utk3AaCdwf3Xqr56ExbJLzdqrKfuOeVWbQjpgoSLGAElUO3JQgduVIVzysPss3KiOyCo1j6NHlcBAAAAAg' 'http://10.0.40.11/api/hyhive/vm/list'
```
# 附录 ab 参数解释

```bash
# ab --help
-n 即requests，用于指定压力测试总共的执行次数。
-c 即concurrency，用于指定的并发数。
-t 即timelimit，等待响应的最大时间(单位：秒)。
-b 即windowsize，TCP发送/接收的缓冲大小(单位：字节)。
-p 即postfile，发送POST请求时需要上传的文件，此外还必须设置-T参数。
-u 即putfile，发送PUT请求时需要上传的文件，此外还必须设置-T参数。
-T 即content-type，用于设置Content-Type请求头信息，例如：application/x-www-form-urlencoded，默认值为text/plain。
-v 即verbosity，指定打印帮助信息的冗余级别。
-w 以HTML表格形式打印结果。
-i 使用HEAD请求代替GET请求。
-x 插入字符串作为table标签的属性。
-y 插入字符串作为tr标签的属性。
-z 插入字符串作为td标签的属性。
-C 添加cookie信息，例如："Apache=1234"(可以重复该参数选项以添加多个)。
-H 添加任意的请求头，例如："Accept-Encoding: gzip"，请求头将会添加在现有的多个请求头之后(可以重复该参数选项以添加多个)。
-A 添加一个基本的网络认证信息，用户名和密码之间用英文冒号隔开。
-P 添加一个基本的代理认证信息，用户名和密码之间用英文冒号隔开。
-X 指定使用的host和端口号，例如:"126.10.10.3:88"。
-V 打印版本号并退出。
-k 使用HTTP的KeepAlive特性。
-d 不显示百分比。
-S 不显示预估和警告信息。
-g 输出结果信息到gnuplot格式的文件中。
-e 输出结果信息到CSV格式的文件中。
-r 指定接收到错误信息时不退出程序。
-h 显示用法信息，其实就是ab -help。
```
