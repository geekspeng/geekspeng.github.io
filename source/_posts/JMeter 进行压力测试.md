---
title: JMeter 进行压力测试
date: 2019-03-24
updated: 2019-03-24
tags: [压力测试,高性能]
categories: [高性能]
---

# 下载JMeter 5.1.1（Requires Java 8+）并设置中文界面

1. 下载后解压到任意位置

[http://mirrors.shu.edu.cn/apache//jmeter/binaries/apache-jmeter-5.1.1.zip](http://mirrors.shu.edu.cn/apache//jmeter/binaries/apache-jmeter-5.1.1.zip)

2. 设置中文界面

修改启动文件 jmeter.batapache-jmeter-5.1.1\bin\jmeter.bat，把默认language 改为zh_CN

```
改动前：
set JMETER_LANGUAGE=-Duser.language="en" -Duser.region="EN"
改动后：
set JMETER_LANGUAGE=-Duser.language="zh" -Duser.region="CN"
```
3. 启动JMeter

进入JMeter的bin目录下，windows系统双击jmeter.bat文件即可启动

<!-- more -->

# 使用JMeter录制脚本

## 浏览器设置代理

1. 打开火狐浏览器，找到选项，然后点击

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/zfCVimoPrq0bAoBt.png!thumbnail)

2. 拖到最下方，点击设置

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/oRJun1Ne62wcFvfy.png!thumbnail)

3. 设置填写 localhost 和端口8082（注意不要和系统其它程序的端口冲突）

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/efmUIS4IP4kCOA51.png!thumbnail)

**注意：**

**只要你确定不再使用JMeter进行脚本录制，那么你要记得把火狐浏览器的网络代理给设置回来，点系统默认代理**

## 设置JMeter 并录制脚本

1. 在测试计划上点击右键，添加线程组

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/xIaFn8warNoM86e5.png!thumbnail)

2. 添加录制控制器

点击 “线程组”，然后右键，根据如下图步骤，添加一个录制控制器

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/xP5HfWuFXN0uCG7X.png!thumbnail)

3. 添加代理服务器

点击 “测试计划”，然后右键，根据如下图步骤，添加一个代理服务器

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/nOyaAEVZe7AJcmbO.png!thumbnail)

添加之后，修改端口（这里端口和浏览器代理端口保持一致）和目标控制器

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/jFTlCHHtbtwRup7c.png!thumbnail)

4. 录制脚本

点击代理服务器右侧里面的启动录制按钮，弹出一个根证书的弹窗，点击确定

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/MiMwdIu7dEELgRp6.png!thumbnail)

在火狐浏览器地址栏手动输入[www.baidu.com](http://www.baidu.com)，等页面加载完成，我们点击“新闻”这个链接，页面加载完成，我们选择停止录制，然后点击展开录制控制器，可以看到以下这些请求。

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/Q8bjnFj1VsoHNqFE.png!thumbnail)

# 使用JMeter进行性能测试

1. 添加聚合报告、查看结果树、用表格查看结果、图形结果，右键点击线程组，添加监听器

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/N6USIQfkHtkjnJ1l.png!thumbnail)

2. 修改线程组参数并启动测试

选择线程组，然后修改参数，修改完成后点击开始按钮

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/Fqti7gG9rNgOZd0H.png!thumbnail)

线程数：模仿用户并发的数量

Ramp-up:运行线程的总时间，单位是秒

循环次数：每个线程循环次数

# JMeter 性能测试结果分析

## 查看结果树

* 请求结果（其中红色的是出错的请求，绿色的为通过）

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/JxYERYElAs0KKa4c.png!thumbnail)

* 取样器结果

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/OiEJ4QNhkGMbXWEY.png!thumbnail)

>**相关名词解释**
>Thread Name：线程组名称
>Sample Start:：启动开始时间
>Load time：加载时长
>Latency：等待时长
>Size in bytes：发送的数据总大小
>Headers size in bytes：发送数据的其余部分大小
>Sample Count：发送统计
>Error Count：交互错误统计
>Response code：返回码
>Response message：返回信息
>Response headers：返回的头部信息
* 发送的请求和响应数据

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/jLfIVIzhDWwCiFmb.png!thumbnail)

## 聚合报告

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/HQrOHVZqljINrXcK.png!thumbnail)


>**相关名词****解释**
>Sample（样本）：本次测试场景共运行多少线程；
>Average（平均值）：平均响应时间，单位ms； 
>Median（中位数）：统计意义上的响应时间中值，单位ms；
>90% line（90%百分位）：所有线程中90%的线程响应时间都小于xx的值;
>Min（最小值）：响应最小时间，单位ms；
>Max（最大值）：响应最大时间，单位ms；
>Error（异常%）：出错率；
>Throughput（吞吐量）：以“requests/second、requests/minute、 requests /hour”来衡量
>Kb/sec（接收/发送 Kb/sec）：以Kb/seond来衡量的吞吐量
## 用表格查看结果

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/0T2UHCcA2AghD0Zt.png!thumbnail)


>相关名词解释
>Sample：每个请求的序号
>Start Time：每个请求开始时间
>Thread Name：每个线程的名称
>Label：Http请求名称
>Sample Time：每个请求所花时间，单位毫秒
>Status：请求状态，如果为勾则表示成功，如果为叉表示失败。
>Bytes：请求的字节数
>>样本数目：也就是上面所说的请求个数，成功的情况下等于你设定的并发数目乘以循环次数
>平均：每个线程请求的平均时间
>最新样本：表示服务器响应最后一个请求的时间
>偏离：服务器响应时间变化、离散程度测量值的大小，或者，换句话说，就是数据的分布。
## 图形结果

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/Yzqa4jmFm5IvDeaI.png!thumbnail)

## 清除测试数据

* 清除部分数据

点击左边要清除的选项，比如，清除聚合报告，点击聚合报，然后点击工具栏的小扫把图标即可

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/DXKyQIWhq5MWKILo.png!thumbnail)

* 清除全部数据

点击工具栏的大扫把图标即可

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/MLtPaxJfPMUyj9LL.png!thumbnail)

# JMeter 使用cookie信息

## 浏览器登录后保存cookie信息

1. 打开火狐浏览器，登录要保存cookie信息的网页
2. 打开调试模式切换到存储，菜单->Web 开发者-> 存储探查器

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/jjs4AfRw1uIPZQ8R.png!thumbnail)

3. 打开cookie，然后右侧红框区域内的所有数据就是cookie信息

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/f0QYB7T8M1cg7DQm.png!thumbnail)

## JMeter 设置 HTTP Cookie管理器

1. 在线程组上点击右键，然后按下图添加 cookie 管理器

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/zsXJoN3ch94ucOPN.png!thumbnail)

2. 把火狐浏览器里的 cookie 的名称、域名、路径、值填写到cookie管理器里

![图片](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/12snIWyF4SAjn2bv.png!thumbnail)

可以点击添加一项一项填写，也可以直接载入文件，将cookie 信息保存到文件方便下次使用

3. 现在整个线程组的所有请求就都会使用这个cookie，如果只有部分请求需要使用，可以拖到对应的请求下面
