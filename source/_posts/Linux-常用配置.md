---
title: Linux 常用配置
date: 2019-04-28 15:37:02
tags:
- Linux
- Ubuntu
- vim
- git
- ssh
categories:
- Linux
comments: true
---
# Ubuntu 18.04 网络配置
编辑*** /etc/netplan/50-cloud-init.yaml***
```
# This file is generated from information provided by
# the datasource.  Changes to it will not persist across an instance.
# To disable cloud-init's network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
    version: 2
    ethernets:
        eth0:
            dhcp4: true
            match:
                macaddress: b8:27:eb:d2:d7:1e
            set-name: eth0
            optional: true
    wifis:
        wlan0:
            dhcp4: true
            optional: true
            access-points:
                    "Xiaomi_kaylordut_5G":
                            password: "kaylordut.com"

```

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

# 用户管理
- 添加一个组
```
groupadd groupname
```
- 添加现有用户到一个组：
```
usermod -a -G groupname username
```
- 同时将 user 增加到 admins, ftp, www, 和 developers 用户组中，可以输入以下命令：
```
useradd -G admins,ftp,www,developers user
```

# GIT
- 绑定远程仓库
```
git remote add origin git_link
```
- 查看现有远程仓库
```
git remote –v
```
- 取消本地目录关联下的远程库
```
git remote remove origin
```
- 推送主分支
```
git push --set-upstream origin master
```
- 查看分支，并checkout
```
git branch –av
git checkout -t origin/xxx
```
- 删除本地分支
1. 删除merge了的分支
```
git branch -d xxx
```
2. 删除分支（不管它有没有merge）
```
git branch -D xxx
```

# Ubuntu 启动分析
```
systemd-analyze plot > boot.svg
```

# IP
```
ipv4:

$ ip -4 addr
ipv6:

$ ip -6 addr
查詢個別網卡的資訊:

$ ip addr show eth0
或

$ ip addr list eth0
或

$ ip addr show dev eth0
給網卡定義 IP 地址:

$ sudo ip addr add 10.20.0.15/24 dev eth1
從網卡移除 IP 地址:

$ sudo ip addr del 10.20.0.15/24 dev eth1
啟動網卡:

$ sudo ip link set dev eth1 up
停用網卡:

$ sudo ip link set dev eth1 down
顯示 Routing Table

$ip r
或

$ ip route
戓

$ ip route show
或

$ ip route list
```

# SimpleHTTPServer with python
```bash
python -m SimpleHTTPServer
```

