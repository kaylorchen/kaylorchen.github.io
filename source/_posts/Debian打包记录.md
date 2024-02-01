---
title: Debian打包记录 
date: 2022-11-15 15:37:02
tags:
  - Linux
categories:
  - Linux
comments: true
---
 
# 安装编译依赖
 安装必要软件
```bash
apt install devscripts equivs
```
进入源码目录
```bash
mk-build-deps --install --remove
```

