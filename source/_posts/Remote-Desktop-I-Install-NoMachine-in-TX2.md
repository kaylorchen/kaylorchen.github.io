---
title: 'Remote Desktop I: Install NoMachine in TX2'
comments: true
date: 2019-04-30 14:03:31
tags:
- Remote Desktop
- Ubuntu
- TX2
categories:
- Remote Desktop 
---
Most remote desktops do not support the ARM platform, but NoMachine is an exception. NoMachine has comparable performance to teamviewer.
# Install NoMachine in TX2
## Download NoMachine
### Download NoMachine by Browser
Download NoMachine from https://www.nomachine.com/download/linux&id=30&s=ARM. 
Select ***NoMachine for ARM ARMv8 DEB***
### Download NoMachine by Terminal
```
cd /tmp
wget https://download.nomachine.com/download/6.6/Arm/nomachine_6.6.8_7_arm64.deb

```
If the NoMachine version is updated, the above link will be invalid,
## Install NoMachine
```
cd /tmp
sudo dpkg -i nomachine*arm64.deb
```
# Configure TX2 Resolution
***sudo vi /etc/X11/xorg.conf*** as follow:
```
# Copyright (c) 2015, NVIDIA CORPORATION.  All Rights Reserved.
#
# This is the minimal configuration necessary to use the Tegra driver.
# Please refer to the xorg.conf man page for more configuration
# options provided by the X server, including display-related options
# provided by RandR 1.2 and higher.

# Disable extensions not useful on Tegra.
Section "Module"
    Disable     "dri"
    SubSection  "extmod"
    Option  "omit xfree86-dga"
    EndSubSection
EndSection

Section "Device"
    Identifier  "Tegra0"
    Driver      "nvidia"
    Option      "AllowEmptyInitialConfiguration" "true"
EndSection

Section "Monitor"
   Identifier "DSI-0"
   Option    "Ignore"
EndSection

Section "Screen"
        Identifier "Default Screen"
        Monitor "Configured Monitor"
        Device "Configured Video Device"
        SubSection "Display"
                   Depth 24
                   Virtual 1920 1080
        EndSubSection
EndSection
```
PS: You can modify "**Virtual 1920 1080**".
