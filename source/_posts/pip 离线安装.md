---
title: pip 离线安装
date: 2018-06-23
updated: 2018-06-23
tags: [python,pip]
categories: [python]
---

* 安装 pip

# 指定包文件路径进行离线安装

```bash
# pip install --no-index /home/pypi/packages/simplejson-3.16.0.tar.gz 
```
--no-index：取消索引

# 以本地文件为pip源进行离线安装

```bash
# pip install package_name --no-index -f file:///home/pypi/packages/
# pip install -r requirements.txt --no-index -f file:///home/pypi/packages/
```
--no-index：取消索引