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
  curl -fsSL https://mirrors.huaweicloud.com/docker-ce/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
  ```
  
  - 设置仓库源
  
  ```bash
  echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://mirrors.huaweicloud.com/docker-ce/linux/ubuntu \
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
newgrp docker # 及时生效
reboot
```

# 一些基本指令

## 合并不同平台容器到同一个tag

```bash
docker manifest create image:tag image1:tag1 image2:tag2
docker manifest push image:tag
```

## 拉取镜像

```bash
docker pull ubuntu:20.04
```

从远程拉取名仓库为ubuntu的，tag为20.04的镜像到本地

## 运行容器

我们前面已经拉取了一个ubuntu 20.04的基础容器，现在我们可以试着运行这个容器。进入容器并执行bash命令。

```bash
docker run --name=network_test --net=host -it ubuntu:20.04 bash
```

- --name 给即将运行的容器命名一个别名
- --net 这个参数默认是使用doker的内置网桥，这里使用host说明容器直接使用主机网络
- -it 这里-i表示标准输出，-t表示生成tty
- bash 是表示进入容器之后需要运行的指令

## 重新运行容器

实际操作里面，我们时常想重新进入容器。比如

```bash
╰─ docker ps -l           
CONTAINER ID   IMAGE          COMMAND   CREATED          STATUS                     PORTS     NAMES
e977e612b806   ubuntu:20.04   "bash"    41 minutes ago   Exited (0) 2 minutes ago             network_test
```

从上面的信息看到，这个容器执行指令bash，在2分钟之前退出了。可是使用以下指令重启容器

```bash
docker start network_test
或者
docker start e977e612b806
```

容器启动之后，我们可以使用docker attach指令重新进入终端,如：

```bash
docker attach network_test 
```

但是，有时候可能需要加入多个终端，所以可以使用docker exec来实现

```bash
docker exec -it network_test bash
```

## 文件夹映射

在实际的开发过程中，我们经常需要共享主机和docker的文件。所以我们可以把主机某些文件夹映射到docker的目录下

```bash
docker run -it -v /opt/kaylor/:/root/data/ --name=map_test ubuntu:20.04 bash
```

-v 主机文件夹路径：docker的文件夹映射路径

## 保存与加载

```bash
docker save -o rk3588_docker.tar kaylor/rk3588_dev_env:latest
```

将本机的名为kaylor/rk3588_dev_env，tag为latest的镜像保存输出为rk3588_docker.tar

```bash
docker load --input rk3588_docker.tar
```

这里注意，加载的tar包需要时通过docker save保存的，通过docker save的镜像会带有原始的提交信息。

## 构建镜像

通常我们会在容器中添加了我们的一些自定义的操作，我们可以通过构建镜像的方式，把容器保存成镜像。
比如：

```bash
docker container ps -a ##查看所有容器的信息
CONTAINER ID   IMAGE                         COMMAND       CREATED        STATUS                    PORTS     NAMES
05cd5820058b   deb7cc07afe8                  "/bin/bash"   2 months ago   Exited (0) 2 weeks ago              musing_aryabhata
```

上面的命令我们知道容器的ID是05cd5820058b，下面的命令是将这个容器提交生成image  
-a 是添加作者信息  
-m 是添加commit的信息  
kaylor/rk3588_dev_env:v1 镜像名为kaylor/rk3588_dev_env tag为v1

```bash
docker commit -a "kaylorchen" -m "add libsystemd-dev" 05cd5820058b kaylor/rk3588_dev_env:v1
```

## 使用主机GPU

- 查看 -gpus 参数是否安装成功
  
  ```bash
  docker run --help | grep -i gpus
  ```

- 运行容器
  
  ```bash
  docker run --gpus all --name container_name -d -t image_id
  docker run --gpus="1" images_id
  ```

## 创建自定义子网使用固定IP

```bash
docker network create --subnet=192.168.77.0/24 kaylor_bridge
docker run -d --name kaylor --net kaylor_bridge --ip 192.168.77.2 ubuntu:22.04
```

# 跨平台使用

- 使用特权容器

windows和mac使用的是桌面版，默认已经启动了binfmt_misc, 我们使用的是Linux，可以手动启动该功能，但是我们有更好的方法，就是启动一个特权容器，指令如下：

```bash
docker run --rm --privileged docker/binfmt:a7996909642ee92942dcd6cff44b9b95f08dad64
```

使用指令**ls -al /proc/sys/fs/binfmt_misc**，看看是否有qemu字样的输出。
使用 **docker buildx ls**查看构建器支持的CPU架构。

- 安装qemu包
  
  ```bash
  sudo apt install qemu-user-static
  ```

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
