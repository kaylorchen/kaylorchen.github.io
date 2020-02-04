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

# Linux 内核编译指令
配置config文件
```bash
make menuconfig
```
编译内核和模块
```bash
make
```
单独编译模块和安装（不要轻易安装内核模块，如果不指定安装路径，容易损坏系统）
```bash
make modules
make modules_install INSTALL_MOD_PATH=安装路径
```

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
- 打补丁
```bash
git am *.patch
```
- 修改commit的message
```bash
git commit --amend -m "I miss you"
```
3. 代理
- 加代理
```bash
git config --global http.proxy 'socks5://127.0.0.1:1080'
git config --global https.proxy 'socks5://127.0.0.1:1080'
git config http.proxy 'socks5://127.0.0.1:1080'
git config https.proxy 'socks5://127.0.0.1:1080'
#只对github.com
git config --global http.https://github.com.proxy socks5://127.0.0.1:1080
```
- 取消代理
```bash
git config --global --unset http.https://github.com.proxy
git config --global --unset http.proxy
git config --global --unset https.proxy
git config --unset http.proxy
git config --unset https.proxy

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
python -m SimpleHTTPServer 8080

python3 -m http.server 8080
```

# Udev 检查指令
查看某个设备的详细信息，如/dev/video0
```
udevadm info -a /dev/video0
```
编辑匹配规则（/etc/udev/rules.d/10-local.rules）:
```
#map cameras to clear names
KERNEL=="video*", ATTRS{manufacturer}=="RoboteX", ATTRS{product}=="Drive Camera", SYMLINK+="drive"
KERNEL=="video*", ATTRS{manufacturer}=="RoboteX", ATTRS{product}=="PTZ Camera", SYMLINK+="ptz"
KERNEL=="video*", ATTRS{manufacturer}=="RoboteX", ATTRS{product}=="Arm Camera", SYMLINK+="armCam"
KERNEL=="video*", ATTRS{manufacturer}=="RoboteX", ATTRS{product}=="USB2.0 Camera", SYMLINK+="radCam"
#map storage device to clear name
KERNEL=="mmcblk?p1", ATTRS{vendor}=="0x8086", ATTRS{device}=="0x0f16", SYMLINK+="storage"
#KERNEL=="mmcblk?p2", SYMLINK+="storage"
KERNEL=="sd?1", KERNELS=="1-4.4:1.0", SYMLINK+="storage"
```
触发规则
```
udevadm trigger
```

# 查看文件夹大小
```bash
du
du --max-depth=0 ./
du -k ./
du -m ./
du -h ./
```

