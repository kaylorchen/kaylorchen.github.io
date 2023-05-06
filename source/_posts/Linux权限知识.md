---
title: Linux权限知识
mathjax: true
comments: true
date: 2023-05-06 10:41:15
tags:
  - Linux
categories:
  - Linux
---

# 权限的数值表示

Linux的权限多数时候是3个数字表示  
r --- read --- 4  
w --- write --- 2   
x --- executed --- 1  
有时候会用4位数字表示，跟下面介绍的suid，sgid，sbit有关  
SUID->4  
SGID->2  
SBIT->1  

# UID 和 GID

UID属于user id，一般来说Linux的第一个非root用户的UID是1000  
GID属于组id，一般来说Linux的第一个非root用户组的GID是1000  

# 特殊ID  

## SUID 

SUID是用来作用户提权的，比如说：
```bash 
╰─ ll /usr/bin/sudo                                                        ─╯
-rwsr-xr-x 1 root root 166056 Apr  4 19:56 /usr/bin/sudo*
```
这里表示sudo文件属于root用户，s（小写）表示sudo有可执行权限，其他用户执行  
这个sudo的时候会暂时获得root权限。

## SGID 

SGID针对于文件的功能跟SUID的功能一样  
当SGID作用于目录的时候，意义就非常重大。**当用户对某一目录有写和执行权限时，该用户就可以在该目录下建立文件，如果该目录用 SGID 修饰，则该用户在这个目录下建立的文件都是属于这个目录所属的组**

## SBIT
```bash
╰─ ll -d /tmp                                                              ─╯
drwxrwxrwt 25 root root 266240 May  6 21:18 /tmp/
```
权限信息中最后一位 t 表明该目录被设置了 SBIT 权限。SBIT 对目录的作用是：当用户在该目录下创建新文件或目录时，仅有自己和 root 才有权力删除。

## 一些细节
小写的s和t代表了文件或者文件夹可执行的权限，如果是大写的S和T的话，那就是这个文件或者文件夹没有可执行权限。
