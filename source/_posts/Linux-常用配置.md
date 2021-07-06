---
title: Linux 常用配置
date: 2020-05-25 15:37:02
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
追踪日志
```
journalctl -f
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

# Ubuntu 18.04 网络配置

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

# SSH

编辑 **_/etc/ssh/ssh_config_**

```
ClientAliveInterval 30
ClientAliveCountMax 6
```

ClientAliveInterval 表示每隔多少秒，服务器端向客户端发送心跳，是的，你没看错。

下面的 ClientAliveInterval 表示上述多少次心跳无响应之后，会认为 Client 已经断开。

所以，总共允许无响应的时间是 60\*3=180 秒。

# VIM

编辑 **_/etc/vim/vimrc_**

```
set nu
colorscheme desert
set ts=4
set expandtab
```
# GRUB
- 让grub记住你上一次的启动项
编辑 **_/etc/default/grub_**
```commandline
GRUB_DEFAULT=saved
GRUB_SAVEDEFAULT=true
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
vim ~/.git-credentials
```

内容如下：

```
https://{username}:{passwd}@github.com
```

添加全局配置

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

- ipv4:

```bash
ip -4 addr
```

- ipv6:

```
ip -6 addr
```

- 查詢個別網卡的資訊:

```
ip addr show eth0
ip addr list eth0
ip addr show dev eth0
```

- 給網卡定義 IP 地址:

```
sudo ip addr add 10.20.0.15/24 dev eth1
```

- 從網卡移除 IP 地址:

```
sudo ip addr del 10.20.0.15/24 dev eth1
```

- 啟動網卡:

```
sudo ip link set dev eth1 up
```

- 停用網卡:

```
sudo ip link set dev eth1 down
```

- 顯示 Routing Table

```
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

- 无线网络配置

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
auto wlan0
iface wlan0 inet dhcp
wpa-conf /etc/wpa_supplicant/wpa.conf
```
# iw指令

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
