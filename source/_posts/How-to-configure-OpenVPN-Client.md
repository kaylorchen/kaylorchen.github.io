---
title: How to configure OpenVPN Client
date: 2015-11-07 15:57:58
tags:
- OpenVPN
- OpenWRT
- Ubuntu
- Linux
- Windows
categories:
- Network
---
### Generate client certifictes with OpenVPN server
Before configuring client, I must generate some client files in the server.
```bash
cd /etc/openvpn/easy-rsa
source vars
./build-key client
```
If you have some different devices, you can :
```bash
./build-key client1
./build-key client2
./build-key clientx
```
After generating keys, we need to copy ca.crt , client.crt and client.key from server to client.
### Configure OpenVPN client
#### Generate client configuration
Details are as follows:
```bash
client
dev tun # the same as server
proto udp #the same as server
remote vpn.server.hostname 7000
resolv-retry infinite
nobind
persist-key
persist-tun
ns-cert-type server
verb 3
ca ca.crt
cert client.crt
key client.key
```

#### Ubuntu Client
Save the above configuration as "client.conf" in the path: ***/etc/openvpn***, and Copy ca.crt , client.crt and client.key to ***/etc/openvpn/***. Finally,
```bash
/etc/init.d/openvpn start
```

#### Windows Client
Download OpenVPN Client software via the [link](https://openvpn.net/community-downloads/). There is an old version client in my [github](https://github.com/kaylorchen/OpenVPN_installation_for_windows_and_Android).
After the client software installation, we can copy ca.crt , client.crt and client.key to  ***installation_diretory/config***. Start your OpenVPN, Right-click the OpenVPN icon on the taskbar and choose correct configuration to connect your server.

#### Android device
If you use a portable device, you should modify your configuration as following the [link](https://community.openvpn.net/openvpn/wiki/IOSinline).
You can download the client software from Google Play. Also, you can download an old version client from my [github](https://github.com/kaylorchen/OpenVPN_installation_for_windows_and_Android).
After installing the client, you can import configuration file to client and enjoy it.

#### OpenWRT router
you can copy your client.conf to ***/etc/openvpn***. ou also must edit ***/etc/config/openvpn***, the content is as follows:
```bash
config openvpn custom_config
        option enable 1
        option config /etc/openvpn/client.conf
        ......
```
start your openvpn:
```bash
/etc/init.d/openvpn start
```
If you want to realize the self-starting function when your router is on:
```bash
/etc/init.d/openvpn enable
```