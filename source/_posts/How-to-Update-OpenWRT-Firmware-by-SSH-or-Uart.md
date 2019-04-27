---
title: How to Update OpenWRT Firmware by SSH or Uart
date: 2015-11-05 15:08:23
tags: 
- OpenWRT
---
If you want to have a diy router, I suggest you buy a openwrt router.

Firstly, you can find [“Table fo Hardware’](http://wiki.openwrt.org/toh/start). this is the main table of hardware, listing all devices that are suppprted by Openwrt.

I had bought a GL.inet router, and its hardware was the same as the TL-WR720N, so I updated the router with WR720N’s firmware.

There are some different versions [firmares](https://downloads.openwrt.org/).  Binary releases and historic releases are relatively stable, but it may not have some new features. Development Snapshots was the newest firmware.

If you are not a developer, I suggest to use a stable firmware. Some softwares are incompatible across multiple releases. Development snapshots are updated fast, so we have to update the newest firmware when we want to install some sofrwares.  However, I can lost some important configuration files if the firmware is updated.

I can download my router’s [firmware](https://downloads.openwrt.org/chaos_calmer/15.05/ar71xx/generic/). there are usually 2 kind of firmware: sysupgrade and factory. Some configuration files (network, firewall, etc. ) will be reserved if the router is updated with sysupgrade firmware. All the configuration files will be cleared with factory firmware.

### Let’s update firmware:
```
wget https://downloads.openwrt.org/chaos_calmer/15.05/ar71xx/generic/xxx.bin
```
If you download a sydupgrade version:

```bash 
sysupgrade -v xxx.bin
```
Also, if you download factory version:

```bash 
mtd -r write xxx.bin firmware
```


 