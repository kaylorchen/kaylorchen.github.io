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

# ZSH 配置和 FZF 配置

## zsh 基本配置

```bash
apt install zsh
git clone https://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh --depth=1
cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
chsh -s /bin/zsh
```

编辑~/.zshrc，修改一下on-my-zsh的默认插件和禁止其每次启动都启动更新, 请找到相应的行
```bash
zstyle ':omz:update' mode disabled
plugins=(git sudo extract z cp safe-paste colored-man-pages)
```
sudo插件是当你忘记加sudo的时候，连续按两次esc，就会在指令上添加sudo  
extract是解压指令，无脑解压各种压缩包，再也不需要查询参数了  
z是一个目录跳转指令，使用它替代cd的话，可以记录你的常用目录  
cp的使用指令是cpv，是一个带进度条的拷贝指令


## 添加插件

在.zshrc 上添加如下内容：

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
bindkey '^U' backward-kill-line

zstyle ':completion:*' rehash true
unsetopt no_match

alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'
```

## 安装配置 fzf

### 安装与升级
- Install

```bash
git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
~/.fzf/install
```

- Upgrade

```bash
cd ~/.fzf && git pull && ./install
```
### 基础使用

- 基础操作

```bash
fzf # 输入fzf直接搜索
fzf -m # 多选模式
```
使用CTRL-J/CTRL-K(或者CTRL-N/CTRL-P)进行上下选择  
使用Enter选中条目, CTRL-C/ESC进行退出操作  
在多选择模式(-m), 使用TAB和Shift-TAB标记多个条目  
Emacs 风格按键绑定  
支持鼠标操作  

- 语法匹配

| 标记       | 匹配类型       | 描述                |
|----------|------------|-------------------|
| sbtrkt   | 	模糊匹配      | 	内容匹配sbtrkt(字符匹配) |
| ‘wild    | 	精确匹配(单引号) | 	内容包含单词wild(单词匹配) |
| ^music   | 	前缀精确匹配    | 	以music开头         |
| .mp3$	   | 后缀精确匹配	    | 以.mp3结尾           |
| !fire	   | 反转匹配	      | 内容不包含fire         |
| !^music	 | 前缀反转匹配	    | 不以music开头         |
| !.mp3$	  | 后缀反转匹配	    | 不以.mp3结尾          |

fzf的过滤默认是与操作，每个与操作之间使用空格间隔，如果需要使用或操作，参考下面的例子：
```bash
^core go$ | rb$ | py$ # 表示以`core`开头, 且以`go`或`rb`或`py`结尾
```

- 一般快捷键和环境变量

| 按键      | 	描述                                  |
|---------|--------------------------------------|
| CTRL-T  | 	命令行打印选中内容                           |
| CTRL-R	 | 命令行历史记录搜索, 并打印输出                     |
| ALT-C	  | 模糊搜索目录, 并进入(cd)，macos可以使用 cd **[TAB] |

| name	               | description example                                           |
|---------------------|---------------------------------------------------------------|
| FZF_DEFAULT_COMMAND | 	输入为 tty 时的默认命令 export FZF_DEFAULT_COMMAND=’fd –type f’       |
| FZF_DEFAULT_OPTS	   | 设置默认选项 export FZF_DEFAULT_OPTS=”–layout=reverse –inline-info” |
| FZF_CTRL_T_COMMAND	 | 按键映射行为设置                                                      |
| FZF_CTRL_T_OPTS	    | 按键映射选项设置                                                      |
| FZF_CTRL_R_OPTS	    | 按键映射选项设置                                                      |
| FZF_ALT_C_COMMAND	  | 按键映射行为设置                                                      |
| FZF_ALT_C_OPTS	     | 按键映射选项设置                                                      |

### 个性化配置

- 更换搜索引擎(可选)
```bash
brew install fd # Mac os 
apt install fd-find # Ubuntu Linux
```
- 设置环境FZF的环境变量

编辑 ~/.bashrc 或者 ~/.zshrc, 添加如下内容
```bash
export FZF_CTRL_T_COMMAND="fd --exclude={.git,.idea,.vscode,.sass-cache,node_modules,build} --type f --follow --hidden  --color=always"
export FZF_DEFAULT_COMMAND="fd --exclude={.git,.idea,.vscode,.sass-cache,node_modules,build} --type f --follow --hidden  --color=always"
export FZF_DEFAULT_OPTS="--height 60% --layout=reverse --preview '(highlight -O ansi {} || cat {}) 2> /dev/null | head -500' --ansi"
```
FZF_DEFAULT_COMMAND 依赖于安装了fd搜索指令，如果没有安装，可以不设置。
FZF_DEFAULT_OPTS 依赖于highlight指令，如果没有则会调用cat指令。Ubuntu可以通过**apt install highlight**安装该指令，MacOS可以通过**brew install highlight**安装该指令

- shell命令补全

fzf默认使用 ** + [TAB] 进行补全，常用的补全指令有：
```bash
kill -9 **
vim **
cd **
ssh **
export **
```


# v4l2-ctl

## 获取支持的分辨率和编码格式

```bash
v4l2-ctl --list-formats-ext -d /dev/video0
```

## 显示 Camera 所有信息(分辨率:Width/Height)

```bash
v4l2-ctl -d /dev/video0 --all
```

## 获取支持的编码格式

```bash
v4l2-ctl  --list-formats -d /dev/video0
```

## 获取支持的 camera 设备

```bash
v4l2-ctl --list-devices -d /dev/video0
```

## 获取摄像头控制参数

```bash
v4l2-ctl -d /dev/video0 --list-ctrls
```

# Samba 相关

## 挂载

vers 参数要指定

```bash
sudo mount  //192.168.100.1/share/ share/ -o vers=2.0
sudo mount  //192.168.100.1/share/ share/ -o vers=2.0,uid=ubuntu,gid=ubuntu
```

# NFS 配置

## 安装 NFS

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

  - 指定 ip 地址的主机：192.168.0.200
  - 指定子网中的所有主机：192.168.0.0/24 192.168.0.0/255.255.255.0
  - 指定域名的主机：david.bsmart.cn
  - 指定域中的所有主机：\*.bsmart.cn
  - 所有主机：\*

- 选项：
  - 访问权限
    - 只读 ro
    - 读写 rw
  - 用户映射选项
    - all_squash：将远程访问的所有普通用户及所属组都映射为匿名用户或用户组（nfsnobody）；
    - no_all_squash：与 all_squash 取反（默认设置）；
    - root_squash：将 root 用户及所属组都映射为匿名用户或用户组（默认设置）；
    - no_root_squash：与 rootsquash 取反；
    - anonuid=xxx：将远程访问的所有用户都映射为匿名用户，并指定该用户为本地用户（UID=xxx）；
    - anongid=xxx：将远程访问的所有用户组都映射为匿名用户组账户，并指定该匿名用户组账户为本地用户组账户（GID=xxx）；
- 其他选项
  - secure：限制客户端只能从小于 1024 的 tcp/ip 端口连接 nfs 服务器（默认设置）；
  - insecure：允许客户端从大于 1024 的 tcp/ip 端口连接服务器；
  - sync：将数据同步写入内存缓冲区与磁盘中，效率低，但可以保证数据的一致性；
  - async：将数据先保存在内存缓冲区中，必要时才写入磁盘；
  - wdelay：检查是否有相关的写操作，如果有则将这些写操作一起执行，这样可以提高效率（默认设置）；
  - no_wdelay：若有写操作则立即执行，应与 sync 配合使用；
  - subtree_check：若输出目录是一个子目录，则 nfs 服务器将检查其父目录的权限(默认设置)；
  - no_subtree_check：即使输出目录是一个子目录，nfs 服务器也不检查其父目录的权限，这样可以提高效率；

配置完成之后，重启 nfs service.

```bash
sudo /etc/init.d/nfs-kernel-server restart
```

## 客户端 mount

假设服务 IP 为: 192.168.20.200,

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

这是 chain 的前端，是用来改变目标地址的

### FORWARD

如果目标地址不是本机，机会进行转发。注意 linux 内核默认是禁止转发的，所以需要开启上面步骤的内核转发

### POSTROUTING(SNAT)

它的作用是修改数据包的源地址，比如内网 IP 访问公网 IP，出口路由会将数据包的源地址改为自己的公网 IP，否者数据包有去无回。

### 链操作

操作参数

| 参数 | 描述     |
| ---- | -------- |
| -I   | 插入     |
| -A   | 追加     |
| -R   | 替换     |
| -D   | 删除     |
| -L   | 列表显示 |

过滤参数

| 参数             | 描述                |
| ---------------- | ------------------- |
| -s               | 匹配源地址          |
| -d               | 匹配目的地址        |
| -p               | 协议匹配            |
| -i               | 入接口匹配          |
| -o               | 出接口匹配          |
| --sport，--dport | 源和目的端口匹配    |
| -j               | 跳转,也就是包的方向 |
| !                | 取反                |

- 列出 nat 表的所有规则

```bash
iptables -t nat -n -L
```

使用 -n 选项是因为避免长时间的反向 DNS 查询

- 添加一个规则到 filter 表的 FORWARD 链

```bash
iptables -t filter -A FORWARD -s 10.1.1.11 -d 202.1.1.1 -j ACCEPT
```

在 iptables 中，默认的表名就是 filter，所以这里可以省略-t filter 直接写成: iptables -A FORWARD -s 10.1.1.11 -d 202.1.1.1 -j ACCEPT

- 允许 eth3 接口过来的包通过 FORWARD 链

```bash
iptables -A FORWARD -i eth3 -j ACCEPT
```

## 简单 nat 路由器

- 环境介绍

linux 2.4 +
2 个网络接口
Lan 口:10.1.1.254/24 eth0
Wan 口:60.1.1.1/24 eth1
目的：实现内网中的节点（10.1.1.0/24）可控的访问 internet。
首先将 Lan 的节点 pc 的网关指向 10.1.1.254。

确定你的 linux 的 ip 配置无误，可以正确的 ping 通内外的地址。同时用 route 命令查看 linux 的本地路由表，确认指定了可用的 ISP 提供的默认网关。

- 操作

  - 打开 linux 的转发功能：sysctl net.ipv4.ip_forward=1

  - 将 FORWARD 链的策略设置为 DROP，这样做的目的是做到对内网 ip 的控制，你允许哪一个访问 internet 就可以增加一个规则，不在规则中的 ip 将无法访问 internet.

```bash
iptables -P FORWARD DROP
```

- 这条规则规定允许任何地址到任何地址的确认包和关联包通过。一定要加这一条，否则你只允许 lan IP 访问没有用，至于为什么，下面我们再详细说。

```bash
iptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT
```

- 这条规则做了一个 SNAT，也就是源地址转换，将来自 10.1.1.0/24 的地址转换为 60.1.1.1

(因为是让内网上网，因此对于代理服务器而言 POSTROUTING（经过路由之后的包应该要把源地址改变为 60.1.1.1，否则包无法返回）)

```bash
iptables -t nat -A POSTROUTING -s 10.1.1.0/24 -j SNAT --to 60.1.1.1
```

- 有这几条规则，一个简单的 nat 路由器就实现了。这时你可以将允许访问的 ip 添加至 FORWARD 链，他们就能访问 internet 了。

比如我想让 10.1.1.9 这个地址访问 internet,那么你就加如下的命令就可以了。

```bash
iptables -A FORWARD -s 10.1.1.9 -j ACCEPT
```

也可以精确控制他的访问地址,比如我就允许 10.1.1.99 访问 3.3.3.3 这个 ip

```bash
iptables -A FORWARD -s 10.1.1.99 -d 3.3.3.3 -j ACCEPT
```

或者只允许他们访问 80 端口

```bash
iptables -A FORWARD -s 10.1.1.0/24 -p tcp --dport http -j ACCEPT
```

## 端口转发

借鉴[此链接](http://xstarcd.github.io/wiki/Linux/iptables_forward_internetshare.html)

## 保存 iptables 指令

- 使用 iptables-restore
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

- 使用 iptables-persistent

```bash
sudo apt install iptables-persistent
```

每当设置了新的 iptables 规则后，使用如下命令保存规则即可，规则会根据 ipv4 和 ipv6 分别保存在了/etc/iptables/rules.v4 和/etc/iptables/rules.v6 文件中。

```bash
netfilter-persistent  save
```

# 日志分析

## 命令相关

显示全部最近一次重新引导后收集到的 journal 条目

```
journalctl -b
```

要查看上次引导的 journal 记录，则可使用-1 相对指针配合-b 标记

```
journalctl -b -1
```

通过以下命令查看全部 2015 年 1 月 10 日下午 5：15 之后的条目

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
journalctl /path/to/executable_file
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

查看和删除系统日志

```
journalctl --disk-usage
journalctl --vacuum-size=10M # 删除系统日志，只剩10M
journalctl --vacuum-time=1min #删除一分钟之前的系统日志
```

查看 journald 进程信息

```
sudo systemctl status systemd-journald.service
```

## 编码中使用系统 log

安装依赖库

```
apt install -y libsystemd-dev
```

相关函数

```
#include <systemd/sd-journal.h>
int sd_journal_print(int priority, const char *format, ...);
int sd_journal_printv(int priority, const char *format, va_list ap);
int sd_journal_send(const char *format, ...);
int sd_journal_sendv(const struct iovec *iov, int n);
int sd_journal_perror(const char *message);
```
The priority value is one of LOG_EMERG, LOG_ALERT, LOG_CRIT, LOG_ERR, LOG_WARNING, LOG_NOTICE, LOG_INFO, LOG_DEBUG, as defined in syslog.h

# find 指令

指定目录搜索文件

```
find dir -name "*pppoe*"
find dir -perm /u+x -type f #查找dir下的可执行文件
```

# Linux 内核编译指令

配置 config 文件

```bash
make menuconfig
# 跨平台配置的注意事项，比如
export CROSS_COMPILE=/home/kaylor/rk3588/rk3588_sdk/prebuilts/gcc/linux-x86/aarch64/gcc-arm-10.3-2021.07-x86_64-aarch64-none-linux-gnu/bin/aarch64-none-linux-gnu-
make ARCH=arm64 menuconfig
```

根据已有的.config 文件，补充缺失的配置

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

# nohup 指令

nohup 即不挂起，不会因为终端退出而终结
比如编译 Openwrt

```
nohup make -j1 V=s >& make.log & 2>&1
```

# 客户定制 USB 设备号驱动加载

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

# TMUX

在 home 目录下创建 .tmux.conf 文件

```bash
setw -g mode-keys vi
set -g default-terminal "screen-256color"
```

# VIM

编辑 **_/etc/vim/vimrc_**

```
set relativenumber
set number
colorscheme desert
set ts=4
set expandtab
set paste
set encoding=utf8
set laststatus=2            " 设置状态栏在倒数第2行
set statusline=%F%m%r%h%w\ [FORMAT=%{&ff}]\ [TYPE=%Y]\ [POS=%04l,%04v][%p%%]\ [LEN=%L]
```

# GRUB

让 grub 记住你上一次的启动项
编辑 **_/etc/default/grub_**

```commandline
GRUB_DEFAULT=saved
GRUB_SAVEDEFAULT=true
```

设置网卡名字为 eth\*的形式

```bash
GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0"
```

执行指令更新 grub

```bash
update-grub
```

# 用户管理

- 添加一个组

```
groupadd groupname
```

- 添加一个新用户

```bash
adduser new_user
```

- 更改用户名

使用 root 用户登录

```bash
killall -u old_username
usermode -l new_username old_username
groupmod -n new_groupname old_groupname
usermod -d /home/new_username new_username -m
```

- 添加现有用户到一个组：

```
usermod -a -G groupname username
```

- 同时将 user 增加到 admins, ftp, www, 和 developers 用户组中，可以输入以下命令：

```
useradd -G admins,ftp,www,developers user
```

- 查看用户的 ID 信息

```bash
id username
```

# 文件传输

- rsync

拷贝文件不改变文件的权限和属性，记得使用sudo:  
```bash
sudo rsync -avz /path/to/source/ /path/to/destination/ # 同步source文件夹下的所有文件 
sudo rsync -avz /path/to/source /path/to/destination/  # 同步source文件夹
```  
以下命令将从源目录拷贝所有文件和子目录到目标目录，但不拷贝符号链接:
```bash
rsync -av --no-links /path/to/source/ /path/to/destination/
```   
如果您想要拷贝符号链接的目标文件，可以使用“-L”选项，如下所示：
```bash
rsync -avL /path/to/source/ /path/to/destination/
```  
请注意，“-L”选项会将符号链接的目标文件拷贝到目标目录中，而不是链接本身。如果您想要拷贝链接本身，可以使用“-a”选项和“--copy-links”选项，如下所示：
```bash
rsync -a --copy-links /path/to/source/ /path/to/destination/
```
这将拷贝符号链接本身，而不是链接的目标文件。

# 网络抓包
## 基础

下面是一些用来配置 **tcpdump** 的选项，它们非常容易被遗忘，也容易和其它类型的过滤器比如Wireshark等混淆。

### 选项

- **-i any** 监听所有的网卡接口，用来查看是否有网络流量
- **-i eth0** 只监听eth0网卡接口
- **-D** 显示可用的接口列表
- **-n** 不要解析主机名
- **-nn** 不要解析主机名或者端口名
- **-q** 显示更少的输出(更加quiet)
- **-t** 输出可读的时间戳
- **-tttt** 输出最大程度可读的时间戳
- **-X** 以hex和ASCII两种形式显示包的内容
- **-XX** 与 **-X**类似，增加以太网header的显示
- **-v, -vv, -vvv** 显示更加多的包信息
- **-c** 只读取x个包，然后停止
- **-s** 指定每一个包捕获的长度，单位是byte，使用`-s0`可以捕获整个包的内容
- **-S** 输出绝对的序列号
- **-e** 获取以太网header
- **-E** 使用提供的秘钥解密IPSEC流量

### 表达式

在**tcpdump**中，可以使用表达式过滤指定类型的流量。有三种主要的表达式类型：**type**，**dir**，**proto**。

- 类型（type）选项包含：**host**，**net**，**port**
- 方向（dir）选项包含：**src**，**dst**
- 协议（proto）选项包含：**tcp**，**udp**，**icmp**，**ah**等

## 示例

### 捕获所有流量

查看所有网卡接口上发生了什么

    tcpdump -i any

### 指定网卡接口

查看指定网卡上发生了什么

    tcpdump -i eth0

### 原生输出

查看更多的信息，不解析主机名和端口号，显示绝对序列号，可读的时间戳

    tcpdump -ttttnnvvS
    
### 查看指定IP的流量

这是最常见的方式，这里只查看来自或者发送到IP地址1.2.3.4的流量。

    tcpdump host 1.2.3.4

### 查看更多的包信息，输出HEX

当你需要查看包中的内容时，使用hex格式输出是非常有用的。

    # tcpdump -nnvXSs 0 -c1 icmp

    tcpdump: data link type PKTAP
    tcpdump: listening on pktap, link-type PKTAP (Apple DLT_PKTAP), capture size 262144 bytes
    16:08:16.791604 IP (tos 0x0, ttl 64, id 34318, offset 0, flags [none], proto ICMP (1), length 56)
        192.168.102.35 > 114.114.114.114: ICMP 192.168.102.35 udp port 50694 unreachable, length 36
    	IP (tos 0x0, ttl 152, id 0, offset 0, flags [none], proto UDP (17), length 112)
        114.114.114.114.53 > 192.168.102.35.50694: [|domain]
    	0x0000:  5869 6c88 7f64 784f 4392 ed7e 0800 4500  Xil..dxOC..~..E.
    	0x0010:  0038 860e 0000 4001 e906 c0a8 6623 7272  .8....@.....f#rr
    	0x0020:  7272 0303 3665 0000 0000 4500 0070 0000  rr..6e....E..p..
    	0x0030:  0000 9811 16cd 7272 7272 c0a8 6623 0035  ......rrrr..f#.5
    	0x0040:  c606 005c 0000                           ...\..
    1 packet captured
    357 packets received by filter
    0 packets dropped by kernel

### 使用源和目的地址过滤

    tcpdump src 2.3.4.6
    tcpdump dst 3.4.5.6

### 过滤某个子网的数据包

    tcpdump net 1.2.3.0/24
    
### 过滤指定端口相关的流量

    tcpdump port 3389
    tcpdump src port 1025

### 过滤指定协议的流量

    tcpdump icmp

### 只显示IPV6流量

    tcpdump ip6

### 使用端口范围过滤

    tcpdump portrange 21-23

### 基于包的大小过滤流量

    tcpdump less 32
    tcpdump greater 64
    tcpdump <=128

### 将捕获的内容写入文件

使用`-w`选项可以将捕获的数据包信息写入文件以供以后分析，这些文件就是著名的PCAP(PEE-cap)文件，很多应用都可以处理它。

    tcpdump port 80 -w capture_file

使用tcpdump加载之前保存的文件进行分析

    tcpdump -r capture_file

## 高级

使用组合语句可以完成更多高级的过滤。

- AND: **and** or **&&**
- OR: **or** or **||**
- EXCEPT: **not** or **!**

### 过滤指定源IP和目的端口

    tcpdump -nnvvS src 10.5.2.3 and dst port 3389

### 过滤指定网络到另一个网络

比如下面这个，查看来自192.168.x.x的，并且目的为10.x或者172.16.x.x的所有流量，这里使用了hex输出，同时不解析主机名

    tcpdump -nvX src net 192.168.0.0/16 and dst net 10.0.0.0/8 or 172.16.0.0/16

### 过滤到指定IP的非ICMP报文

    tcpdump dst 192.168.0.2 and src net and not icmp

### 过滤来自非指定端口的指定主机的流量

下面这个过滤出所有来自某个主机的非ssh流量

    tcpdump -vv src mars and not dst port 22

### 复杂分组和特殊字符

当构建复杂的过滤规则的时候，使用单引号将规则放到一起是个很好的选择。特别是在包含**()**的规则中。比如下面的规则就是错误的，因为括号在shell中会被错误的解析，可以对括号使用`\`进行转义或者使用单引号

    tcpdump src 10.0.2.3 and (dst port 3389 or 22)
    
应该修改为

    tcpdump 'src 10.0.2.3 and (dst port 3389 or 22)'

### 隔离指定的TCP标识

可以基于指定的TCP标识（flag）来过滤流量。

> 下面的过滤规则中，**tcp[13]** 表示在TCP header中的偏移位置13开始，后面的数字代表了匹配的byte数。

#### 显示所有的URGENT (URG)包

    tcpdump 'tcp[13] & 32!=0'

#### 显示所有的ACKNOWLEDGE (ACK)包

    tcpdump 'tcp[13] & 16!=0'

#### 显示所有的PUSH(PSH)包

    tcpdump 'tcp[13] & 8!=0'

#### 显示所有的RESET(RST)包

    tcpdump 'tcp[13] & 4!=0'

#### 显示所有的SYNCHRONIZE (SYN) 包

    tcpdump 'tcp[13] & 2!=0'

#### 显示所有的FINISH(FIN)包

    tcpdump 'tcp[13] & 1!=0'

#### 显示说有的SYNCHRONIZE/ACKNOWLEDGE (SYNACK)包

    tcpdump 'tcp[13]=18'

#### 其它方式
    
与大多数工具一样，也可以使用下面这种方式来捕获指定TCP标识的流量

    tcpdump 'tcp[tcpflags] == tcp-syn'
    tcpdump 'tcp[tcpflags] == tcp-rst'
    tcpdump 'tcp[tcpflags] == tcp-fin'

### 识别重要流量

最后，这里有一些重要的代码片段你可能需要，它们用于过滤指定的流量，例如畸形的或者恶意的流量。

#### 过滤同时设置SYN和RST标识的包（这在正常情况下不应该发生）

    tcpdump 'tcp[13] = 6'

#### 过滤明文的HTTP GET请求

    tcpdump 'tcp[32:4] = 0x47455420'

#### 通过横幅文本过滤任意端口的SSH连接

    tcpdump 'tcp[(tcp[12]>>2):4] = 0x5353482D'

#### 过滤TTL小于10的包（通常情况下是存在问题或者在使用traceroute）

    tcpdump 'ip[8] < 10'

#### 过滤恶意的包

    tcpdump 'ip[6] & 128 != 0'


# GIT

- 变基

```bash
git checkout experiment
git rebase master
git checkout master
git merge experiment
```

- 一般的合并

把 dev 合并到 master

```bash
git checkout master
git merge dev
```

- 分支重命名

```bash
git branch -m new_name
```

- clone 指定分支

```bash
 git clone -b dev_branch [url]
```

--depth=1 参数，克隆最近一次的 commit 的，体积会变小

```bash
git clone https://github.com/xxx/xxx.git --depth=1
```

如果需要间该分支所有的 commit 克隆下来，需要

```bash
git fetch --unshallow
```

但会产生另外一个问题，他只会把默认分支 clone 下来，其他远程分支并不在本地，所以这种情况下，需要用如下方法拉取其他分支：

```bash
git clone --depth 1 https://github.com/dogescript/xxxxxxx.git
git remote set-branches origin 'remote_branch_name'
git fetch --depth 1 origin remote_branch_name
git checkout remote_branch_name
```

- git bash 显示中文

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

- Linux 配置文件

编辑全局配置文件 **_~/.gitconfig_**

```
[user]
	email = you@example.com
[https]
    proxy = socks://192.168.15.1:10808
[http]
    proxy = socks://192.168.15.1:10808
[core]
	editor = vim
```

- tag 使用

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

# 分区自动挂载和格式化

格式化分区的时候使用 -E root_owner=1000:1000 选项
```bash
sudo mkfs.ext4 -E root_owner=1000:1000 /dev/sda1
```

确认系统有autofs4内核模块的支持，如果没有将不能使用x-systemd.automount选项，编辑/etc/fstab，添加
```bash
/dev/sda1 /opt/storage ext4 nofail,user,exec,rw,x-systemd.automount  0 0
```
x-systemd.automount选项是使用是挂载，没有使用的时候不挂载。


# 网络配置

## netplan 网络配置

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

## IP 指令

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

- 添加和删除路由和默认网关

```bash
ip route add default via 192.168.1.254
ip route add 192.168.0.0/24 via 192.168.1.1 dev eth0
ip route del 192.168.0.0/24 via 192.168.1.1 dev eth0
route add default gw 192.168.1.254
```
- 特殊路由

路由查找遵循“最长匹配原则”，当路由器收到一个IP数据包时，会将数据包的目的IP地址与自己本地路由表中的表项进行bit by bit的逐位查找，直到找到匹配度最长的条目，这叫最长匹配原则。  
最简单的理解方式就是，当路由都匹配的时候，看看谁的子网掩码位数比较多。

```bash
ip route add 0.0.0.0/1 via 192.168.23.1 dev eth0
ip route add 128.0.0.0/1 via 192.168.23.1 dev eth0
```


## 静态 IP 和 DNS 设置

- 设置静态 IP

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

为了支持 DNS，需要安装 apt install resolvconf ifupdown  
如果为了保证有一个固定的 DNS，可以编辑 **_/etc/resolvconf/resolv.conf.d/tail_**, 添加一个默认 DNS

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

- CAN 接口配置

```bash
edge@host-63b5d7:/etc/network/interfaces.d$ cat can0
auto can0
iface can0 inet manual
pre-up ip link set can0 type can bitrate 500000
pre-up ip link set can0 type can restart-ms 1
edge@host-63b5d7:/etc/network/interfaces.d$ ip -d -s link show can0
5: can0: <NOARP,UP,LOWER_UP,ECHO> mtu 16 qdisc pfifo_fast state UP mode DEFAULT group default qlen 10
    link/can  promiscuity 0
    can state ERROR-ACTIVE (berr-counter tx 0 rx 0) restart-ms 1
	  bitrate 498701 sample-point 0.870
	  tq 26 prop-seg 33 phase-seg1 33 phase-seg2 10 sjw 1
	  mttcan: tseg1 2..255 tseg2 0..127 sjw 1..127 brp 1..511 brp-inc 1
	  mttcan: dtseg1 1..31 dtseg2 0..15 dsjw 1..15 dbrp 1..15 dbrp-inc 1
	  clock 38400000numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535
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

如果不使用指定的配置文件，也可以如下：

```
auto wlan0
allow-hotplug wlan0
iface wlan0 inet dhcp
wpa-ssid "GUEST"
wpa-psk "sangfor123"
```
## 网络启动等待的问题

有时候设置了接口的DHCP服务，网络会一直在等待，最后只有网络服务启动超时了，才能启动系统。可以通过修改启动超时参数达到目的。

```bash
╰─ cat /lib/systemd/system/networking.service 
[Unit]
Description=Raise network interfaces
Documentation=man:interfaces(5)
DefaultDependencies=no
Requires=ifupdown-pre.service
Wants=network.target
After=local-fs.target network-pre.target apparmor.service systemd-sysctl.service systemd-modules-load.service ifupdown-pre.service
Before=network.target shutdown.target network-online.target
Conflicts=shutdown.target

[Install]
WantedBy=multi-user.target
WantedBy=network-online.target

[Service]
Type=oneshot
EnvironmentFile=-/etc/default/networking
ExecStart=/sbin/ifup -a --read-environment
ExecStop=/sbin/ifdown -a --read-environment --exclude=lo
RemainAfterExit=true
TimeoutStartSec=5s
```

为再缩短启动时间，我们可以设置系统的systemd的默认启动超时时间，编辑 **/etc/systemd/system.conf**, 添加以下内容：   
```bash
DefaultTimeoutStartSec=10s
DefaultTimeoutStopSec=10s

```

## NetworkManager 管理相关

### 永久不管理接口

- 查看网卡状态

```bash
nmcli device status
```

- 编辑文件 /etc/NetworkManager/conf.d/99-unmanaged-devices.conf  
  NetworkManager 的系统配置文件可能的位置还有 **_/lib/NetworkManager/conf.d/_** 和 **_/usr/lib/NetworkManager/conf.d/_**

```
[keyfile]
unmanaged-devices=interface-name:interface_1;interface-name:interface_2;...
```

- 重启服务

```bash
systemctl reload NetworkManager
```

### 临时更改接口管理状态

```bash
nmcli device set enp1s0 managed no/yes
```

## NAT 转发设置

使用主机作为网关给其他设备上网

```bash
echo "Forward setting"
sudo iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE
sudo iptables -F
sudo iptables -P INPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -P OUTPUT ACCEPT
sudo sysctl -w net.ipv4.ip_forward=1
```

## iw 指令 (无线网络)

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

# Python 的简易 httpserver

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
sudo udevadm trigger
```

# 查看文件夹大小

```bash
du
du --max-depth=0 ./
du -k ./
du -m ./
du -h ./
```

# dd 指令

- 生成空的 image 和扩容

```bash
dd if=/dev/zero of=disk.img bs=1M count=256 #生成256MBytes的image
cat blank.img >> old.img #将blank.img追加到old.img之后
```

追加之后的扩容操作，需要参考接下来的“**_使用 parted 扩容分区_**”，对 image 进行分区，请参考我的另一个文章[**_制作一个空的 image_**](https://blog.kaylordut.com/2021/08/07/%E5%88%B6%E4%BD%9C%E4%B8%80%E4%B8%AA%E7%A9%BA%E7%9A%84image/#more)

- 烧写 image

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

## 跨平台使用 docker

运行特权容器

```bash
docker run --rm --privileged docker/binfmt:66f9012c56a8316f9244ffd7622d7c21c1f6f28d
```

查看支持的 CPU 信息

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

编辑 **_/etc/docker/daemon.json_**

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

## Doker 的 C/S 模式

- Romote API

unix:///var/run/docker.sock
tcp://host:port
fd://socketfd

## Docker 的守护进程

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
Registry 相关：
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

## Docker 环境变量

export DOCKER_HOST="tcp://x.x.x.x:2375"

## 镜像保存与加载

```bash
docker save -o 要保存的文件名 要保存的镜像
$ docker save -o test.tar ubuntu
docker load --input 加载的文件名
```

## dockerfile 指令

Dockerfile 例子

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

必须是 dockerfile 的第一条指令

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

虽然我们在镜像构建中指定了暴露的端口号，但在容器运行时，我们仍需要手动的指定容器的端口映射，就像我们之前曾经使用的这个 docker run 命令。也就是说，在 dockerfile 中使用 expose 指令来指定的端口，只是告诉 Docker，该容器内的应用程序会使用特定的端口儿，但是出于安全的考虑，Docker 并不会自动地打开端口，是需要在使用时再 run 命令中添加对端口的映射指令。

- CMD

```bash
CMD ["executalbe", "param1", "param2"]
CMD command param1 param2
CMD ["param1", "param2"] #作为ENTRYPOINT指令的默认参数
```

cmd 是指定容器运行时的默认指令，如果 docker run 带指令，会覆盖 cmd 的指令

- ENTRYPOINT

```bash
ENTRYPOINT ["executalbe", "param1", "param2"]
ENTRYPOINT command param1 param2
```

docker run 的指令不能覆盖该指令。

- ADD and COPY

```bash
ADD/COPY src dest
ADD/COPY ["src"..."dest"] #路径中有空格也可以使用
```

add 包含 tar 的解压缩功能，复制推荐使用 copy

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

# PVE 指令

```bash
qm stop ID #关机
```

# exec， source 和 fork 的区别

fork (/directory/script.sh) ：如果 shell 中包含执行命令，那么子命令并不影响父级的命令，在子命令执行完后再执行父级命令。子级的环境变量不会影响到父级。

fork 是最普通的, 就是直接在脚本里面用 /directory/script.sh 来调用 script.sh 这个脚本. 运行的时候开一个 sub-shell 执行调用的脚本， sub-shell 执行的时候, parent-shell 还在。sub-shell 执行完毕后返回 parent-shell 。 sub-shell 从 parent-shell 继承环境变量.但是 sub-shell 中的环境变量不会带回 parent-shell 。整个执行的过程是 fork + exec + waitpid (此三者均为系统调用)。

exec (exec /directory/script.sh) ：执行子级的命令后，不再执行父级命令。

exec 与 fork 不同，不需要新开一个 sub-shell 来执行被调用的脚本. 被调用的脚本与父脚本在同一个 shell 内执行。但是使用 exec 调用一个新脚本以后, 父脚本中 exec 行之后的内容就不会再执行了。这是 exec 和 source 的区别。

source (source /directory/script.sh) ：执行子级命令后继续执行父级命令，同时子级设置的环境变量会影响到父级的环境变量。
