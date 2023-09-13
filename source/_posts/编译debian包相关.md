---
title: 编译debian包相关
comments: true
date: 2022-08-07 10:42:15
tags:
- Debian
categories:
- Debian
---

# 开启deb-src选项

比如需要再focal上编译jammy的源码，可以假如jammy的源码选项
```
$ cat /etc/apt/sources.list.d/jammy-source.list                             ─╯
deb-src https://mirrors.cloud.tencent.com/ubuntu/ jammy universe main restricted multiverse #Added by software-properties
deb-src https://mirrors.cloud.tencent.com/ubuntu/ jammy-updates universe main restricted multiverse #Added by software-properties
deb-src https://mirrors.cloud.tencent.com/ubuntu/ jammy-backports main restricted universe multiverse #Added by software-properties
deb-src https://mirrors.cloud.tencent.com/ubuntu/ jammy-security universe main restricted multiverse #Added by software-properties
```

# 获取源码安装编译依赖

假设需要编译vim
```
apt showsrc vim
apt source vim
apt build-dep vim
```

# 编译相关包

```
fakeroot debian/rules binary
fakeroot debian/rules clean
DEB_BUILD_OPTIONS="parallel=7 nocheck" fakeroot debian/rules binary # 使用7个线程编译，编译过程不测试代码
```