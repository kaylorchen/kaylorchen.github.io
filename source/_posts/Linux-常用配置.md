---
title: Linux 常用配置
date: 2022-11-15 15:37:02
tags:
  - Linux
  - Ubuntu
  - vim
  - git
  - ssh
  - docker
categories:
  - Linux
comments: true
---

# ZSH配置和FZF配置

## zsh基本配置

```bash
apt install zsh
git clone https://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh --depth=1
cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
chsh -s /bin/zsh
```

## 添加插件
在.zshrc上添加如下内容：

```
# zplug configruation
if [[ ! -d "${ZPLUG_HOME}" ]]; then
    if [[ ! -d ~/.zplug ]]; then
        git clone https://github.com/zplug/zplug ~/.zplug
        # If we can't get zplug, it'll be a very sobering shell experience. To at
        # least complete the sourcing of this file, we'll define an always-false
        # returning zplug function.
        if [[ $? != 0 ]]; then
            function zplug() {
                return 1
            }
        fi
    fi
    export ZPLUG_HOME=~/.zplug
fi
if [[ -d "${ZPLUG_HOME}" ]]; then
    source "${ZPLUG_HOME}/init.zsh"
fi
zplug 'plugins/git', from:oh-my-zsh, if:'which git'
zplug 'romkatv/powerlevel10k', use:powerlevel10k.zsh-theme
zplug "plugins/vi-mode", from:oh-my-zsh
zplug 'zsh-users/zsh-autosuggestions'
zplug 'zsh-users/zsh-completions', defer:2
zplug 'zsh-users/zsh-history-substring-search'
zplug 'zsh-users/zsh-syntax-highlighting', defer:2

if ! zplug check; then
    zplug install
fi

zplug load

bindkey '^K' kill-line
bindkey '^B' backward-char
bindkey '^F' forward-char

zstyle ':completion:*' rehash true
unsetopt no_match
```
## 安装配置fzf

- Install
```bash
git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
~/.fzf/install
```

- upgrade
```bash
cd ~/.fzf && git pull && ./install
```


# v4l2-ctl

## 获取支持的分辨率和编码格式

```bash
v4l2-ctl --list-formats-ext -d /dev/video0
```

## 显示Camera所有信息(分辨率:Width/Height)

```bash
v4l2-ctl -d /dev/video0 --all
```

## 获取支持的编码格式

```bash
v4l2-ctl  --list-formats -d /dev/video0
```

## 获取支持的camera设备

```bash
v4l2-ctl --list-devices -d /dev/video0
```

## 获取摄像头控制参数

```bash
v4l2-ctl -d /dev/video0 --list-ctrls
```


# Samba 相关

## 挂载

vers参数要指定
```bash
sudo mount  //192.168.100.1/share/ share/ -o vers=2.0
sudo mount  //192.168.100.1/share/ share/ -o vers=2.0,uid=ubuntu,gid=ubuntu
```

# NFS配置

## 安装NFS
```bash
sudo apt install nfs-kernel-server
```
## 配置共享目录
服务端的共享目录配置文件为　**_/etc/exports_**
```bash
# /etc/exports: the access control list for filesystems which may be exported
#		to NFS clients.  See exports(5).
#
# Example for NFSv2 and NFSv3:
# /srv/homes       hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)
#
# Example for NFSv4:
# /srv/nfs4        gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)
# /srv/nfs4/homes  gss/krb5i(rw,sync,no_subtree_check)
#
/home/ubuntu *(rw,anonuid=1000,anongid=1000,sync,no_subtree_check)
```
```bash
<输出目录> [客户端1 选项（访问权限,用户映射,其他）] [客户端2 选项（访问权限,用户映射,其他）]
```
- 输出目录：
  - 服务器共享的绝对目录地址

- 客户端：
  - 指定ip地址的主机：192.168.0.200
  - 指定子网中的所有主机：192.168.0.0/24 192.168.0.0/255.255.255.0
  - 指定域名的主机：david.bsmart.cn
  - 指定域中的所有主机：*.bsmart.cn
  - 所有主机：*

- 选项：
  - 访问权限
    - 只读 ro
    - 读写 rw
  - 用户映射选项
    -  all_squash：将远程访问的所有普通用户及所属组都映射为匿名用户或用户组（nfsnobody）；
    - no_all_squash：与all_squash取反（默认设置）；
    - root_squash：将root用户及所属组都映射为匿名用户或用户组（默认设置）；
    - no_root_squash：与rootsquash取反；
    - anonuid=xxx：将远程访问的所有用户都映射为匿名用户，并指定该用户为本地用户（UID=xxx）；
    - anongid=xxx：将远程访问的所有用户组都映射为匿名用户组账户，并指定该匿名用户组账户为本地用户组账户（GID=xxx）；
- 其他选项
  - secure：限制客户端只能从小于1024的tcp/ip端口连接nfs服务器（默认设置）；
  - insecure：允许客户端从大于1024的tcp/ip端口连接服务器；
  - sync：将数据同步写入内存缓冲区与磁盘中，效率低，但可以保证数据的一致性；
  - async：将数据先保存在内存缓冲区中，必要时才写入磁盘；
  - wdelay：检查是否有相关的写操作，如果有则将这些写操作一起执行，这样可以提高效率（默认设置）；
  - no_wdelay：若有写操作则立即执行，应与sync配合使用；
  - subtree_check：若输出目录是一个子目录，则nfs服务器将检查其父目录的权限(默认设置)；
  - no_subtree_check：即使输出目录是一个子目录，nfs服务器也不检查其父目录的权限，这样可以提高效率；

配置完成之后，重启nfs service.
```bash
sudo /etc/init.d/nfs-kernel-server restart
```

## 客户端mount
假设服务IP为: 192.168.20.200,
```bash
kaylor@kaylor-ThinkPad-T460:~$ showmount -e 192.168.20.200 #显示服务器共享的目录
Export list for 192.168.20.200:
/home/ubuntu *
kaylor@kaylor-ThinkPad-T460:~$ sudo mount 192.168.20.200:/home/ubuntu nfs
#mac mount需要加参数
sudo  mount -o vers=4,resvport 192.168.51.200:/home/ubuntu nfs
```



# iptables 网络转发设置
## 设置内核转发
- 即时开启内核转发
```bash
echo "1" > /proc/sys/net/ipv4/ip_forward
```
- 永久开启内核转发
```bash
vim  /etc/sysctl.conf
=========================================================================
net.ipv4.ip_forward = 1

sysctl -p
```
## iptables 指令说明
![](/image/iptables_netfilter_chains.png)
### PREROUTING(DNT) 
这是chain的前端，是用来改变目标地址的

### FORWARD
如果目标地址不是本机，机会进行转发。注意linux内核默认是禁止转发的，所以需要开启上面步骤的内核转发

### POSTROUTING(SNAT)
它的作用是修改数据包的源地址，比如内网IP访问公网IP，出口路由会将数据包的源地址改为自己的公网IP，否者数据包有去无回。

### 链操作

操作参数

|参数|描述|
|--|--|
|-I|插入|
|-A|追加|
|-R|替换|
|-D|删除|
|-L|列表显示|

过滤参数

| 参数 | 描述 |
| -- | -- |
| -s	| 匹配源地址 |
|-d	|匹配目的地址|
|-p	|协议匹配|
|-i	|入接口匹配|
|-o	|出接口匹配|
|--sport，--dport	|源和目的端口匹配|
|-j	|跳转,也就是包的方向|
|!	|取反|

- 列出nat表的所有规则
```bash
iptables -t nat -n -L
```
使用 -n 选项是因为避免长时间的反向DNS查询
- 添加一个规则到filter表的FORWARD链
```bash
iptables -t filter -A FORWARD -s 10.1.1.11 -d 202.1.1.1 -j ACCEPT
```
在iptables中，默认的表名就是filter，所以这里可以省略-t filter直接写成: iptables -A FORWARD -s 10.1.1.11 -d 202.1.1.1 -j ACCEPT

- 允许eth3接口过来的包通过FORWARD链
```bash
iptables -A FORWARD -i eth3 -j ACCEPT
```
## 简单nat路由器
- 环境介绍
  
linux 2.4 +
2个网络接口
Lan口:10.1.1.254/24 eth0
Wan口:60.1.1.1/24 eth1
目的：实现内网中的节点（10.1.1.0/24）可控的访问internet。
首先将Lan的节点pc的网关指向10.1.1.254。

确定你的linux的ip配置无误，可以正确的ping通内外的地址。同时用route命令查看linux的本地路由表，确认指定了可用的ISP提供的默认网关。
- 操作

   - 打开linux的转发功能：sysctl net.ipv4.ip_forward=1

   - 将FORWARD链的策略设置为DROP，这样做的目的是做到对内网ip的控制，你允许哪一个访问internet就可以增加一个规则，不在规则中的ip将无法访问internet.
```bash
iptables -P FORWARD DROP
```

   - 这条规则规定允许任何地址到任何地址的确认包和关联包通过。一定要加这一条，否则你只允许lan IP访问没有用，至于为什么，下面我们再详细说。
```bash
iptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT
```
   - 这条规则做了一个SNAT，也就是源地址转换，将来自10.1.1.0/24的地址转换为60.1.1.1

(因为是让内网上网，因此对于代理服务器而言POSTROUTING（经过路由之后的包应该要把源地址改变为60.1.1.1，否则包无法返回）)
```bash
iptables -t nat -A POSTROUTING -s 10.1.1.0/24 -j SNAT --to 60.1.1.1
```
   - 有这几条规则，一个简单的nat路由器就实现了。这时你可以将允许访问的ip添加至FORWARD链，他们就能访问internet了。

  比如我想让10.1.1.9这个地址访问internet,那么你就加如下的命令就可以了。
```bash
iptables -A FORWARD -s 10.1.1.9 -j ACCEPT
```
  也可以精确控制他的访问地址,比如我就允许10.1.1.99访问3.3.3.3这个ip
  
```bash
iptables -A FORWARD -s 10.1.1.99 -d 3.3.3.3 -j ACCEPT
```
  或者只允许他们访问80端口
```bash
iptables -A FORWARD -s 10.1.1.0/24 -p tcp --dport http -j ACCEPT
```

## 端口转发
借鉴[此链接](http://xstarcd.github.io/wiki/Linux/iptables_forward_internetshare.html)
 
## 保存 iptables指令
- 使用iptables-restore
设置了相关规则之后，保存到文件中

```bash
iptables-save > /etc/iptables-rules
ip6tables-save > /etc/ip6tables-rules
```

然后新建一个脚本文件，保存到**_/etc/network/if-pre-up.d/_**目录下，记得修改脚本的权限：
```bash
#!/bin/bash
iptables-restore < /etc/iptables.rules
```

- 使用iptables-persistent

```bash
sudo apt install iptables-persistent
```

每当设置了新的iptables规则后，使用如下命令保存规则即可，规则会根据ipv4和ipv6分别保存在了/etc/iptables/rules.v4和/etc/iptables/rules.v6文件中。
```bash
netfilter-persistent  save
```

# 日志分析

查看日志
```
journalctl
```
显示全部最近一次重新引导后收集到的journal条目
```
journalctl -b
```
要查看上次引导的journal记录，则可使用-1相对指针配合-b标记
```
journalctl -b -1
```
通过以下命令查看全部2015年1月10日下午5：15之后的条目

```
journalctl --since "2015-01-10 17:15:00"
journalctl --since "2015-01-10" --until "2015-01-11 03:00"
journalctl --since 09:00 --until "1 hour ago"
journalctl –-since yesterday
```
过滤
```
journalctl -u nginx.service
journalctl -u nginx.service --since today
journalctl _PID=8088
journalctl _UID=33 --since today
```
显示内核信息
```
journalctl -k
```
根据优先级显示
0: emerg
1: alert
2: crit
3: err
4: warning
5: notice
6: info
7: debug
```
journalctl -p err -b
```
显示近期日志
```
journalctl -n 20
```
追踪日志/实时滚动日志
```
journalctl -f
journalctl  -u nginx.service  -f
```



# find指令

指定目录搜索文件
```
find dir -name "*pppoe*"
```

# Linux 内核编译指令

配置 config 文件

```bash
make menuconfig
```

根据已有的.config文件，补充缺失的配置
```
make defconfig
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
指定内核源码和模块源码所在的绝对路径
```bash
make -C 内核源码绝对路径 M=模块源码文件所在的绝对路径 modules  
make ARCH=arm CROSS_COMPILE= -C /lib/modules/5.10.17-v7l+/build M=/home/pi/rtl8812au  modules
```

安装模块具体操作
```
install -p -m 644 88XXau.ko  /lib/modules/5.10.17-v7l+/kernel/drivers/net/wireless/
/sbin/depmod -a 5.10.17-v7l+
```


安装内核头文件
```bash
make headers_install INSTALL_HDR_PATH=安装路径
```
# nohup指令

nohup 即不挂起，不会因为终端退出而终结
比如编译Openwrt
```
nohup make -j1 V=s >& make.log & 2>&1
```

# 客户定制USB设备号驱动加载
```bash
sudo modprobe -r em28xx
echo "modprobe finished"
sudo modprobe em28xx card=65
echo "em28xx configured"
sudo su -c "echo '2694 0008' > /sys/bus/usb/drivers/em28xx/new_id"
echo "Custom VID and PID enabled"
```

# nload 分析网卡流量

```
-a period       Sets the length in seconds of the time window for average
                calculation.
                Default is 300.
-i max_scaling  Specifies the 100% mark in kBit/s of the graph indicating the
                incoming bandwidth usage. Ignored if max_scaling is 0 or the
                switch -m is given.
                Default is 10240.
-m              Show multiple devices at a time; no traffic graphs.
-o max_scaling  Same as -i but for the graph indicating the outgoing bandwidth
                usage.
                Default is 10240.
-t interval     Determines the refresh interval of the display in milliseconds.
                Default is 500.
-u h|b|k|m|g    Sets the type of unit used for the display of traffic numbers.
   H|B|K|M|G    h: auto, b: Bit/s, k: kBit/s, m: MBit/s etc.
                H: auto, B: Byte/s, K: kByte/s, M: MByte/s etc.
                Default is h.
-U h|b|k|m|g    Same as -u, but for a total amount of data (without "/s").
   H|B|K|M|G    Default is H.
devices         Network devices to use.
                Default is to use all auto-detected devices.
--help
-h              Print this help.

example: nload -t 200 -i 1024 -o 128 -U M

```

# SSH

## 服务器端

编辑 **/etc/ssh/sshd_config**

```
ClientAliveInterval 30
ClientAliveCountMax 6
```

ClientAliveInterval 表示每隔多少秒，服务器端向客户端发送心跳，是的，你没看错。

下面的 ClientAliveInterval 表示上述多少次心跳无响应之后，会认为 Client 已经断开。

所以，总共允许无响应的时间是 60\*3=180 秒。

## 客户端

编辑 **/etc/ssh/ssh_config**

```
ServerAliveInterval 30
ServerAliveCountMax 100
```

# VIM

编辑 **_/etc/vim/vimrc_**

```
set nu
colorscheme desert
set ts=4
set expandtab
set paste
```
# GRUB

让grub记住你上一次的启动项
编辑 **_/etc/default/grub_**
```commandline
GRUB_DEFAULT=saved
GRUB_SAVEDEFAULT=true
```

设置网卡名字为eth*的形式
```bash
GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0"
```

执行指令更新grub
```bash
update-grub
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

- 查看用户的ID信息

```bash
id username
```

# GIT

- 变基

```bash
git checkout experiment
git rebase master
git checkout master
git merge experiment
```

- clone 指定分支

```bash
 git clone -b dev_branch [url]
```

--depth=1 参数，克隆最近一次的commit的，体积会变小

```bash
git clone https://github.com/xxx/xxx.git --depth=1
```

如果需要间该分支所有的commit克隆下来，需要

```bash
git fetch --unshallow
```
但会产生另外一个问题，他只会把默认分支clone下来，其他远程分支并不在本地，所以这种情况下，需要用如下方法拉取其他分支：
```bash
git clone --depth 1 https://github.com/dogescript/xxxxxxx.git
git remote set-branches origin 'remote_branch_name'
git fetch --depth 1 origin remote_branch_name
git checkout remote_branch_name
```


- Windows git bash 显示中文
  
```
git config --global core.quotepath false
```

- 取消文件跟踪

```bash
git rm -r --cached dir #不跟踪，但保留文件
git rm -r --f dir #删除文件
git rm --cached readme.txt #不跟踪，但保留文件
git rm --f readme.txt #删除文件
```

- 远程仓库(remote)

```
git remote add origin [url]
git remote set-url origin [url] //更换URL
git remote –v //查看remote信息
git remote remove origin //取消远程关联
```

- 推送分支

```
git push --set-upstream origin master
git push --all origin
```
- 拉取远程分支

```bash
git pull <远程库名> <远程分支名>:<本地分支名> 
git pull origin develop:develop #拉取远程分支中的develop分支和本地develop分支merge
git pull origin develop #拉取远程分支中的develop与当前分支merge
git pull --all
```

- 查看分支，并 checkout

```
git branch –av
git checkout -t origin/xxx
```

- 删除本地分支

删除 merge 了的分支

```
git branch -d xxx
```

删除分支（不管它有没有 merge）

```
git branch -D xxx
```

删除远程分支

```
git push origin --delete branch_name
```

- 补丁

```bash
git format-patch xxxxxxxxxx
git am *.patch
```

- 修改 commit 的 message

```bash
git commit --amend -m "I miss you"
```

- 代理

加代理

```bash
git config --global http.proxy 'socks5://127.0.0.1:1080'
git config --global https.proxy 'socks5://127.0.0.1:1080'
git config http.proxy 'socks5://127.0.0.1:1080'
git config https.proxy 'socks5://127.0.0.1:1080'
#只对github.com
git config --global http.https://github.com.proxy socks5://127.0.0.1:1080
```

取消代理

```bash
git config --global --unset http.https://github.com.proxy
git config --global --unset http.proxy
git config --global --unset https.proxy
git config --unset http.proxy
git config --unset https.proxy

```

- Linux 设置免密

```bash
git config --global credential.helper store
```

编辑全局配置文件 **_~/.gitconfig_**

```
[credential]
	helper = store
[user]
	email = you@example.com
[https]
    proxy = socks://192.168.15.1:10808
[http]
    proxy = socks://192.168.15.1:10808
[core]
	editor = vim
```

- tag使用

```bash
git tag v1.0.0
git push --tag
git tag -d v1.0.0
```


# ROS 常用指令

- 配置 ROS 环境变量（临时）

```bash
export ROS_IP=192.168.15.147
export ROS_MASTER_URI="http://192.168.15.3:11311"
```

- 设置 ROS Log 等级

```bash
rqt_logger_level
```

# Ubuntu 启动分析

```
systemd-analyze plot > boot.svg
```

# 网络配置

## Ubuntu 18.04 网络配置

编辑 **_/etc/netplan/50-cloud-init.yaml_**

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

## IP指令

- ipv4:

```bash
ip -4 addr
```

- ipv6:

```
ip -6 addr
```

- 基本网卡操作:

```
ip addr show eth0
ip addr list eth0
ip addr show dev eth0
sudo ip addr add 10.20.0.15/24 dev eth1
sudo ip addr del 10.20.0.15/24 dev eth1
sudo ip link set dev eth1 up
sudo ip link set dev eth1 down
ip r
ip route
ip route show
ip route list
```

- 添加路由和默认网关

```bash
ip route add default via 192.168.1.254
route add default gw 192.168.1.254
```

## 静态IP和DNS设置 

- 设置静态IP

```bash
hunter@hunter-Drone:~$ cat /etc/network/interfaces
# interfaces(5) file used by ifup(8) and ifdown(8)
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 192.168.111.205
netmask 255.255.255.0
gateway 192.168.111.1
dns-nameservers 192.168.111.1
```
为了支持DNS，需要安装 apt install resolvconf ifupdown  
如果为了保证有一个固定的DNS，可以编辑 **_/etc/resolvconf/resolv.conf.d/tail_**, 添加一个默认DNS
```bash
nameserver 114.114.114.114
nameserver 119.29.29.29
```

- 桥接多网口
  
  安装桥接工具 **_sudo apt install bridge-utils_** , 生成网桥配置文件 **/etc/network/interfaces.d/br0** ，你会看到如下内容：

```bash
auto br0
iface br0 inet static
bridge_ports eth0 eth1
address 192.168.23.10
netmask 255.255.255.0
gateway 192.168.23.100 

auto br0:0
iface br0:0 inet static
address 192.168.100.101
netmask 255.255.255.0
```

## 无线网络配置

```bash
kaylor@kaylor-ThinkPad-T460:~$ wpa_passphrase ssid password
network={
	ssid="ssid"
	#psk="password"
	psk=44116ea881531996d8a23af58b376d70f196057429c258f529577a26e727ec1b
}
```

使用以上指令生成配置文件，然后在 **_/etc/network/interfaces.d/_** 下建立 wlan0 文件，内容如下：

```
allow-hotplug wlan0
iface wlan0 inet manual
wpa-roam /etc/wpa_supplicant/wpa.conf
iface default inet static
address 192.168.20.200
netmask 255.255.255.0
gateway 192.168.20.1
dns-nameservers 192.168.20.1
```
## NetworkManager管理相关

### 永久不管理接口
-  查看网卡状态
```bash
nmcli device status
```
- 编辑文件 /etc/NetworkManager/conf.d/99-unmanaged-devices.conf  
NetworkManager的系统配置文件可能的位置还有 **_/lib/NetworkManager/conf.d/_** 和 **_/usr/lib/NetworkManager/conf.d/_**

```
[keyfile]
unmanaged-devices=interface-name:interface_1;interface-name:interface_2;...
```
- 重启服务
```bash
systemctl reload NetworkManager
```

### 临时不管理接口
```bash
nmcli device set enp1s0 managed no
```


## iw指令 (无线网络)

```
iw help    # 帮助
iw list    # 获得所有设备的功能，如带宽信息（2.4GHz，和5GHz），和802.11n的信息
iw dev wlan0 scan    # 扫描
iw event    # 监听事件 
iw dev wlan0 link    # 获得链路状态 
iw wlan0 connect foo    # 连接到已禁用加密的AP，这里它的SSID是foo 
iw wlan0 connect foo 2432  # 假设你有两个AP SSID 都是 foo ，你知道你要连接的是在 2432 频道
iw wlan0 connect foo keys 0:abcde d:1:0011223344    # 连接到使用WEP的AP
iw dev wlan1 station dump    # 获取station 的统计信息
iw dev wlan1 station get     # 获得station对应的peer统计信息
iw wlan0 set bitrates legacy-2.4 12 18 24    # 修改传输比特率 
iw dev wlan0 set bitrates mcs-5 4    # 修改tx HT MCS的比特率 
iw dev wlan0 set bitrates mcs-2.4 10  
iw dev wlan0 set bitrates mcs-5    # 清除所有 tx 比特率和设置的东西来恢复正常
iw dev  set txpower  <auto|fixed|limit> [<tx power in mBm>]   #设置传输功率
iw phy  set txpower  <auto|fixed|limit> [<tx power in mBm>]   #设置传输功率
iw dev wlan0 set power_save on  #设置省电模式
iw dev wlan0 get power_save  #查询当前的节电设定
iw phy phy0 interface add moni0 type monitor  #添加一个 monitor 接口
iw wlan0 info #查看当前网络状态
iw phy0 info #查看phy0物理网卡信息
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
规则中只能包含单个父设备

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

KERNEL=="wlan*", SUBSYSTEMS=="usb", NAME="wfb"
```

重命名网卡

```
KERNEL=="wlan*", SUBSYSTEMS=="sdio", NAME="hostapd"
KERNEL=="wlan*", SUBSYSTEMS=="usb", NAME="wfb"
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

# dd指令
- 生成空的image和扩容

```bash
dd if=/dev/zero of=disk.img bs=1M count=256 #生成256MBytes的image
cat blank.img >> old.img #将blank.img追加到old.img之后
```
追加之后的扩容操作，需要参考接下来的“***使用parted扩容分区***”，对image进行分区，请参考我的另一个文章[***制作一个空的image***](https://blog.kaylordut.com/2021/08/07/%E5%88%B6%E4%BD%9C%E4%B8%80%E4%B8%AA%E7%A9%BA%E7%9A%84image/#more)

- 烧写image
  
```bash
gunzip -c kaylor.img.gz | dd of=/dev/xxx bs=20M
```

# 使用 parted 扩容分区

- 修改分区表

```bash
sudo parted /dev/sdb
```

打印信息如下：

```
GNU Parted 3.2
Using /dev/sdb
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) p
Model: Multiple Card Reader (scsi)
Disk /dev/sdb: 15.8GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:

Number  Start  End     Size    Type     File system  Flags
 1      154kB  52.4MB  52.3MB  primary  ext2         boot

(parted) resizepart 1
Warning: Partition /dev/sdb1 is being used. Are you sure you want to continue?
Yes/No? Yes
End?  [52.4MB]? 15.8G
(parted) p
Model: Multiple Card Reader (scsi)
Disk /dev/sdb: 15.8GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:

Number  Start  End     Size    Type     File system  Flags
 1      154kB  15.8GB  15.8GB  primary  ext2         boot

(parted) q
Information: You may need to update /etc/fstab.
```

上述的指令可以复合成：

```bash
sudo parted -s /dev/sdb resizepart 1 15.8G
```

- 恢复文件系统

```bash
sudo e2fsck -f /dev/sdb1
```

- 扩容

```bash
sudo resize2fs /dev/sdb1
```

# APT

- 列出软件包可用版本

```
apt-cache madison package-name
```

# docker

## 基本指令

```bash
docker ps -l #查看上一次运行的容器
docker ps -a #查看历史运行的容器
docker run -i -t ubuntu bash #运行ubuntu， -i表示标准输出， -t表示生成tty， bash是运行的指令
docker inspecet 唯一ID或者容器名字 #返回容器的信息
docker run --name=your_name -it ubuntu bash #定义容器的名字
docker start -i container_name #重新启动停止的容器
docker rm 容器的ID/容器名 #删除已经停止的容器
```

## 守护式容器

```bash
docker run -i -t ubuntu /bin/bash #启动之后使用Ctrl+P 和 Ctrl+Q
docker attach 容器ID/容器名 #重新进入退出的容器

docker run -d CONTAINER [COMMAND] [argv ...] #后台启动容器，返回Docker守护进程分配的ID

docker logs [-f] [-t] [--tail] CONTAINER
# -f --follow=true|false default:false
# -t --timestamps=true|false default:false
# --tail="all"

docker top #查看运行情况
docker exec [-d] [-t] [-t] CONTAINER_ID/NAME [COMMAND] [argv ...] #在运行的容器中启动新的进程
docker stop CONTAINER_NAME/ID # 发送一个停止信号
docker kill CONTAINER_NAME_ID # 直接停止容器
```

## 容器端口映射

```bash
dodker run -P -i -t ubuntu /bin/bash #-P --publish-all=true|false default: false
docker run -p 80 -i -t ubuntu /bin/bash  # -p --publish=[] 指定端口
docker run -p 8080:80 -i -t ubuntu /bin/bash
docker run -p 0.0.0.0:80 -i -t ubuntu /bin/bash
docker run -p 0.0.0.0:8080:80 -i -t ubuntu /bin/bash
docker port CONTAINER_NAME/ID #查看端口映射情况
```

## 跨平台使用docker
运行特权容器
```bash
docker run --rm --privileged docker/binfmt:66f9012c56a8316f9244ffd7622d7c21c1f6f28d
```
查看支持的CPU信息
```bash
ls -al /proc/sys/fs/binfmt_misc/
```

## 主机文件夹映射

```bash
docker run -it -v /home/kaylor/wifibroadcast/:/data --name rpi_env kaylor/rpi-env:20210507 /bin/bash
``` 

## 镜像

```bash
docker images [options] [repository]
-a, --all=false #显示所有镜像，包括中间镜像
-f, --filter=[]
--no-trunc=false #不截断ID
-q, --quiet=false #只显示ID

docker rmi [options] IMAGE
-f, --force=false #强制删除
--no-prune=false #保留未打标签的父镜像

docker rmi ${docker images -q ubuntu} #删除ubuntu的所有镜像
```

## 查找 Images
- 从网站查找
https://registry.hub.docker.com
- 使用命令

```bash
docker search [options] term #最多返回25个结果
--automated=false
--no-trunc-false
-s, --stars=0 
```
## 拉取 Images
```bash
docker pull [options] NAME[:TAG]
-a, -all-tags=false
```

## 推送镜像


```bash
docker push repository
```

## 配置国内镜像地址
编辑 ***/etc/docker/daemon.json***
```config
{
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
}
```
## 构建镜像
```bash
docker commit #通过容器构建
docker commit [options] CONTAINER [REPOSITORY[:TAG]]
# -a, --author="" 
# -m, --message="" Commit message
# -p, --pause=true Pause container during commit
docker build #通过Dockerfile文件构造
docker build [options] PATH | URL | -
# --force-rm=false
# --no-cache=false
# --pull=false
# -q, -quiet=false
# --rm=true
# -t,--tag=""
```

## Doker的C/S模式
- Romote API

unix:///var/run/docker.sock
tcp://host:port
fd://socketfd

## Docker的守护进程
docker -d [options]
-D, --debug=false
-e, --exec-driver="native"
-g, --graph="var/lib/docker"
--icc=true
-l, --log-level="info"
--lable=[]
-p, --pidfile=""/var/run/docker.pid
-G, --group="docker"
-H, --host=[]
--tls=false
--tlscacert="/home/kaylor/.docker/ca.pem"
--tlscert="/home/kaylor/.docker/cert.pem"
--tlskey="/home/kaylor/.docker/key.pem"
--tlsverify=false
Registry相关：
--insecure-registry=[]
--registry-mirror=[]
网络相关:
-b, --bridge=""
--bip=""
--fixed-cidr=""
--fixed-cidr-v6=""
--dns=[]
--dns-search=[]
--ip=0.0.0.0
--ip-forward=true
--ip-masq=true
--iptables=true
--ipv6=false
--mtu=0

## Docker环境变量
export DOCKER_HOST="tcp://x.x.x.x:2375"

## 镜像保存与加载

```bash
docker save -o 要保存的文件名 要保存的镜像
$ docker save -o test.tar ubuntu
docker load --input 加载的文件名
```

## dockerfile 指令
Dockerfile例子

```bash
FROM ubuntu:14.04
MAINTAINER kaylor "kaylor@kaylordut.com"
RUN apt update
RUN apt install -y vim
EXPOSE 80
```
- FROM
```bash
FROM <image> 
FROM <image>:<tag>
```
必须是dockerfile的第一条指令
- MAINTAINER
```bash
MAINTAINER <name>
```
包含作者和联系信息
- RUN
```bash
RUN <command> 
shell 模式： /bin/sh -c command
RUN echo hello
```
```bash
RUN ["executalbe", "param1", "param2"]
exec 模式：
RUN ["/bin/bash", "-c", "echo hello"]
```
- EXPOSE
```bash
EXPOSE <port> [<port>...]
```
虽然我们在镜像构建中指定了暴露的端口号，但在容器运行时，我们仍需要手动的指定容器的端口映射，就像我们之前曾经使用的这个docker run命令。也就是说，在dockerfile中使用expose指令来指定的端口，只是告诉Docker，该容器内的应用程序会使用特定的端口儿，但是出于安全的考虑，Docker并不会自动地打开端口，是需要在使用时再run命令中添加对端口的映射指令。
- CMD
```bash
CMD ["executalbe", "param1", "param2"]
CMD command param1 param2
CMD ["param1", "param2"] #作为ENTRYPOINT指令的默认参数
```
cmd是指定容器运行时的默认指令，如果docker run带指令，会覆盖cmd的指令
- ENTRYPOINT
```bash
ENTRYPOINT ["executalbe", "param1", "param2"]
ENTRYPOINT command param1 param2
```
docker run的指令不能覆盖该指令。
- ADD and COPY
```bash
ADD/COPY src dest
ADD/COPY ["src"..."dest"] #路径中有空格也可以使用
```
add包含tar的解压缩功能，复制推荐使用copy

- VOLUME
- WORKDIR
- ENV
- USER
- ONBUILD

# fswebcam 命令行拍照

```bash
fswebcam -S 10 -r 640x480 --no-banner 1.jpg

-S 跳过N帧
-r 设置分辨率
--no-banner 隐藏banner
```

# PVE指令

```bash
qm stop ID #关机
```

# exec， source 和 fork 的区别

fork (/directory/script.sh) ：如果 shell 中包含执行命令，那么子命令并不影响父级的命令，在子命令执行完后再执行父级命令。子级的环境变量不会影响到父级。

fork是最普通的, 就是直接在脚本里面用 /directory/script.sh 来调用 script.sh 这个脚本. 运行的时候开一个 sub-shell 执行调用的脚本， sub-shell 执行的时候, parent-shell 还在。sub-shell执行完毕后返回 parent-shell 。 sub-shell 从 parent-shell 继承环境变量.但是 sub-shell 中的环境变量不会带回 parent-shell 。整个执行的过程是 fork + exec + waitpid (此三者均为系统调用)。

exec (exec /directory/script.sh) ：执行子级的命令后，不再执行父级命令。

exec 与 fork 不同，不需要新开一个 sub-shell 来执行被调用的脚本. 被调用的脚本与父脚本在同一个 shell 内执行。但是使用 exec 调用一个新脚本以后, 父脚本中 exec 行之后的内容就不会再执行了。这是 exec 和 source 的区别。

source (source /directory/script.sh) ：执行子级命令后继续执行父级命令，同时子级设置的环境变量会影响到父级的环境变量。
