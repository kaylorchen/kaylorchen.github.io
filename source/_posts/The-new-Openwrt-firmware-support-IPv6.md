---
title: The new Openwrt firmware support IPv6
date: 2015-11-07 15:52:10
tags:
- OpenVPN
categories:
- Network
---
The old version firmware may have some bugs about IPv6 after my experiment.
In order to support IPv6, I chose the newest [firmware](https://downloads.openwrt.org/snapshots/trunk/ar71xx/generic/).
After upgrading firmware, some configuration file must be modified.
```bash
vi /etc/config/dhcp
```
Add or modify some contents as following:
```bash
config dhcp 'lan'
option interface 'lan'
option start '100'
option limit '150'
option leasetime '12h'
option dhcpv6 'relay'
option ra 'relay'
option ndp 'relay'

config dhcp 'wan6'
option interface 'wan'
option dhcpv6 'relay'
option ra 'relay'
option ndp 'relay'
option master 1
```

Restart our router~~