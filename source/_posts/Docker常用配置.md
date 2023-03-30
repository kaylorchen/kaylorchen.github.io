---
title: Docker常用配置
comments: true
date: 2021-08-07 10:41:15
tags:
- docker
categories:
- docker
---

# 安装docker

## 卸载旧版本

```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
```

## 安装新版本

- 设置仓库源
  
  - 更新apt包索引，安装依赖

  ```bash
  sudo apt-get update
  sudo apt-get install ca-certificates curl gnupg lsb-release
  ```

  - 添加docker官方GPG key
  
  ```bash
  sudo mkdir -p /etc/apt/keyrings
  curl -fsSL https://mirrors.cloud.tencent.com/docker-ce/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
  ```

  - 设置仓库源

  ```bash
  echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://mirrors.cloud.tencent.com/docker-ce/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  ```

- 安装docker引擎
  
    - 安装最新版本
  
    ```bash
    sudo apt update
    sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin
    ```
    至此，你已经完成了docker最新稳定版本的安装！！！

    - 选择安装指定版本
  
    ```bash
    1. 列出可用版本
    $ apt-cache madison docker-ce | awk '{ print $3 }'
    5:20.10.16~3-0~ubuntu-jammy
    5:20.10.15~3-0~ubuntu-jammy
    5:20.10.14~3-0~ubuntu-jammy
    5:20.10.13~3-0~ubuntu-jammy
  
    2. 安装指定版本
    $ VERSION_STRING=5:20.10.13~3-0~ubuntu-jammy
    $ sudo apt-get install docker-ce=$VERSION_STRING docker-ce-cli=$VERSION_STRING containerd.io docker-compose-plugin
    ```
            
# 设置非root用户不使用sudo

```bash
usermod -a -G docker username
reboot
```


# 跨平台使用

windows和mac使用的是桌面版，默认已经启动了binfmt_misc, 我们使用的是Linux，可以手动启动该功能，但是我们有更好的方法，就是启动一个特权容器，指令如下：
```bash
docker run --rm --privileged docker/binfmt:a7996909642ee92942dcd6cff44b9b95f08dad64
```
使用指令***ls -al /proc/sys/fs/binfmt_misc***，看看是否有qemu字样的输出。
使用***docker buildx ls***查看构建器支持的CPU架构。

# 构建树莓派交叉编译镜像
```bash
#下载官方镜像
wget https://downloads.raspberrypi.org/raspios_lite_armhf/images/raspios_lite_armhf-2021-05-28/2021-05-07-raspios-buster-armhf-lite.zip
unzip 2021-05-07-raspios-buster-armhf-lite.zip
fdisk -l 2021-05-07-raspios-buster-armhf-lite.img #这里是使用fdisk查看分区信息，mount指令需要使用
mkdir image
sudo mount -o loop,offset=$((512*532480)) 2021-05-07-raspios-buster-armhf-lite.img image/
sudo apt install qemu-user-static -y
sudo mv image/etc/ld.so.preload image/etc/ld.so.preload.bak
sudo cp /usr/bin/qemu-arm-static image/usr/bin/
cd image/
sudo tar cvf ../rpi-env-2021-05-07.tar .
cd ..
umount image
docker import rpi-env-2021-05-07.tar rpi-env:20210507
docker run -it rpi-env:20210507 /bin/bash
```
退出容器之后，可以使用***docker start ID*** 和 ***docker attach ID*** 重新进入
我已经设置好了基础仓库，可以直接使用***docker pull kaylor/rpi-env***拉取

# 共享宿主机文件夹

/home/kaylor/aaa --> /share
```bash
docker run -it -v /home/kaylor/aaa:/share kaylor/rpi-env:20210507 /bin/bash
```
# 从根文件系统构建docker镜像

参考链接 https://docs.docker.com/engine/reference/commandline/import/

```bash
cd rootfs_path
sudo tar -c . | docker import - docker_name
```



