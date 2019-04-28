---
title: Linux 常用配置
date: 2019-04-28 15:37:02
tags:
- Linux
- Ubuntu
categories:
- Linux
---
# SSH
编辑 ***/etc/ssh/ssh_config***
```
ClientAliveInterval 30
ClientAliveCountMax 6
```
ClientAliveInterval表示每隔多少秒，服务器端向客户端发送心跳，是的，你没看错。

下面的ClientAliveInterval表示上述多少次心跳无响应之后，会认为Client已经断开。

所以，总共允许无响应的时间是60*3=180秒。

# VIM
编辑 ***/etc/vim/vimrc***
```
set nu
colorscheme desert
set ts=4
set expandtab
```
