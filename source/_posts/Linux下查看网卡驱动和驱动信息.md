---
title: Linux下查看网卡驱动和驱动信息
date: 2018-11-18
updated: 2018-11-18
tags: [Linux]
categories: [Linux]
---

# 必备设置

* 查看网卡驱动类型
```
ethtool -i 网卡名
```
示例：

```bash
# ethtool -i enp0s3
driver: e1000
version: 7.3.21-k8-NAPI
firmware-version:
bus-info: 0000:00:03.0
supports-statistics: yes
supports-test: yes
supports-eeprom-access: yes
supports-register-dump: yes
supports-priv-flags: no
```
driver: e1000就是网卡类型

<!-- more -->

* 查查看网卡驱动信息

modinfo 网卡类型

示例：

```bash
modinfo e1000
结果如下：
filename: /lib/modules/3.10.0-327.13.1.el7.x86_64/kernel/drivers/net/ethernet/intel/e1000/e1000.ko
version: 7.3.21-k8-NAPI
license: GPL
description: Intel(R) PRO/1000 Network Driver
author: Intel Corporation, linux.nics@intel.com
rhelversion: 7.2
srcversion: 4354B761874070384453524
```

