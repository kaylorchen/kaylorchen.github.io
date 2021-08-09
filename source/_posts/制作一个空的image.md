---
title: Create a Blank Image
comments: true
date: 2021-08-07 10:41:15
tags:
- parted
- fdisk
categories:
- Linux
---

# Shell script
```shell
#!/bin/bash
start=8192
middle=532479 # start ~ middle is 256MBytes, this is boot partition
# end=6291456 
end=614400
total_size=$(expr $end \/ 2048) #Unit is MBytes
echo total size is $total_size MBytes

echo "Deleting old image"
sudo rm ${total_size}MB -r

echo "Unmounting everything that could be mounted"
sudo umount * > /dev/null 2>&1
sudo losetup -d /dev/loop101 > /dev/null 2>&1

echo "Recreating mount points"
mkdir rootfs
mkdir boot

echo "Creating empty file"
dd if=/dev/zero of=disk.img bs=1M count=$total_size

echo "Associating loopback device to image"
sudo losetup /dev/loop101 disk.img

echo "Partitioning image"
sudo parted -s /dev/loop101 mklabel msdos
sudo parted -a none -s /dev/loop101 unit s mkpart primary fat32 $start $middle
sudo parted -a none -s /dev/loop101 unit s mkpart primary ext4 $(expr $middle \+ 1) $( expr $end \- 1)

echo "Setting partition as bootable"
sudo parted /dev/loop101 set 1 boot on
sudo mkfs.vfat -F 32 /dev/loop101p1  # "-F 32" is fat32
sudo mkfs.ext4 /dev/loop101p2

echo "Adding label to partition :"
sudo fatlabel /dev/loop101p1 boot # this partition is fat32 fs
sudo e2label /dev/loop101p2 rootfs

echo "Mounting disk image"
sudo mount /dev/loop101p1 boot
sudo mount /dev/loop101p2 rootfs
sudo blkid  | grep loop101

# ROOT=root=PARTUUID=$(sudo blkid | grep loop100p1 | sed  -e 's/ /\n/g' |grep PARTUUID | awk -F '\"' '{print $2}')
# echo ROOT=$ROOT
# sudo sed -i -e 's/root=\/dev\/mmcblk0p1/'${ROOT}'/g' mnt/boot/grub/grub.cfg

echo "Unmounting files"
sudo umount rootfs
sudo umount boot
sudo losetup -d /dev/loop101

echo "gzip disk.img"
ls -l |grep disk.img
fdisk -l disk.img
parted disk.img print
sudo gzip disk.img
ls -l |grep disk.img

echo "rename disk.img.gz to blank_image.img.gz"
mv disk.img.gz blank_image.img.gz
mkdir ${total_size}MB
mv blank_image.img.gz ${total_size}MB

echo "Deleting temporary files"
sudo rm -rf rootfs
sudo rm -rf boot
```
