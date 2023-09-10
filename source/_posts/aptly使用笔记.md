---
title: aptly使用笔记
date: 2023-09-10 11:45:41
tags:
- aptly
categories:
- aptly
comments: true
---

# aptly配置

## 建立一个repo

```
aptly repo create -distribution="focal" -component="main" -comment="kaylor ros2 humble repository" repository_name
```
- distribution 是list中url之后的第一个参数
- component 是list中url之后的第二个参数

## 往仓库里面添加deb包

```
aptly repo add  repository_name packages_path
```
## 查看仓库里的包信息

```
aptly repo show -with-packages repository_name
```
## 删除软件包
```
$ aptly repo remove stable percona-server-client-5.5
Loading packages...
[-] percona-server-client-5.5_5.5.35-rel33.0-611.squeeze_i386 removed
[-] percona-server-client-5.5_5.5.35-rel33.0-611.squeeze_amd64 removed
```

## 发布软件包

```
aptly publish repo repository_name prefix
```
- repository_name 从repo（官方建议从snapshot发布）发布版本
- prefix 可选项，这是url后面的文件夹路径名，最好还是加一个
  
## 更新发布

```
 aptly publish update --force-overwrite -batch  -passphrase="xxxxxxxx" distribution prefix
```

## drop发布

```
aptly publish drop <distribution> [<prefix>]
```

# 使用Nginx假设服务器

直接上配置文件

```
$ cat /etc/nginx/conf.d/aptly.conf
server {
    listen 60000;
    listen [::]:60000;
    server_name yourdomain.com;
    root /home/ubuntu/.aptly/public;
    index index.html
    allow all;
    autoindex on;
}
```