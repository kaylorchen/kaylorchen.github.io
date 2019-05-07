---
title: How to Deploy your Private NAS with your OpenWrt Router
date: 2015-11-11 16:45:29
tags:
- NAS
- OpenWRT
categories:
- OpenWRT
---
Firstly, we should install USB storage prerequisites.
```
opkg update
opkg install kmod-usb-storage block-mount kmod-fs-ext4 kmod-usb-storage-extras kmod-scsi-core e2fsprogs fdisk
```
If your device’s filesystem isn’t ext4 or ext3, I sugguest to format it.
```commandline
fdisk /dev/sda
```
Delete old partitions and creat new partitions with fdisk.
Format your device and mount it:
```
mkfs.ext4 /dev/sda
mount -t ext4 /dev/sda /mnt/
```
Install samba server:
```
opkg install samba36-server
vi /etc/config/samba
```
```commandline
config samba
        option 'name' 'OpenWrt'
        option 'workgroup' 'WORKGROUP'
        option 'description' 'OpenWrt'
        option 'homes' '1'
config sambashare
        option name 'share'
        option path 'mnt'
        option read_only 'no'
        option guest_ok 'no'
```
Edit /etc/samba/smb.conf.template, and delete invalid users = root. Next:
``smbpasswd -a root``

Enter your samba server’s password.
Now, all the configurations are completed. Start samba server:
```/etc/init.d/samba start```

Finally, Connect our server with PC. “win+R” and enter “\\x.x.x.x”. If you want to use IPv6 address, you can append a line “2001:da8:a800::xxxx(server) samba_server” in your hosts. Use “\\samba_server” instead of “\\x.x.x.x”.
If your device is a Android phone, you can install a APP named “Shared Viewer” from Google Play.
