---
title: How to Build the Cross-Compilation Environment
date: 2024-03-07 15:57:58
tags:
- Linux
- Ubuntu
categories:
- Tools
---

# Backgrounds

In most cases, we now use QEMU technology to run ARM containers on X86 platforms, employing these containers as cross-compilation tools. However, due to the different instruction set architectures between the host environment and the Docker environment, compilation speeds are slow. To address this issue, we can directly use an ARM cross-compiler toolchain on the X86 host. During the development of large projects, when you need to link a large number of third-party libraries, the compiler will alert you of link errors and other issues.

# How

My development environment requires Ubuntu 22.04, so the rootfs I am using is Ubuntu 22.04. If you want to use a different version, you can modify the script according to your own situation.

## Install the cross-compilation toolchain
```bash
sudo apt update
sudo apt install gcc-aarch64-linux-gnu g++-aarch64-linux-gnu qemu-user-static
```

## Build a cross-compilation environment rootfs

```bash
git clone https://github.com/kaylorchen/rk3588_dev_rootfs.git
cd rk3588_dev_rootfs
bash ./build-rootfs.sh
```

## Compile your code using the cross-compilation environment
Following the steps above, a file named "toolchain-aarch64.cmake" will be generated in the current directory. You will need to use it in your CMake project.
For example:
```bash
mkdir build
cd build
cmake -DCMAKE_TOOLCHAIN_FILE=/opt/data/orangepi/rk3588_dev_rootfs/toolchain-aarch64.cmake -DCMAKE_EXPORT_COMPILE_COMMANDS=ON ..
make 
```
> /opt/data/orangepi/rk3588_dev_rootfs/toolchain-aarch64.cmake is an absolute path


## Update your rootfs
When we compile code, sometimes we find that some dependency packages are missing. We need to update our rootfs.  
1. edit ***_config.sh_***, add a new package named "aaa":
```bash
#!/bin/bash
apt update
apt install -y --no-install-recommends vim libopencv-dev g++ gcc \
fakeroot devscripts libspdlog-dev libsystemd-dev libcap-dev liblz4-dev \
libgcrypt-dev libzstd-dev debhelper rknpu2-dev librockchip-mpp-dev librga-dev \
libstb-dev libturbojpeg0-dev libjpeg-turbo8-dev kaylordut-dev aaa
```
2. update rootfs
```bash
./update.sh
```
