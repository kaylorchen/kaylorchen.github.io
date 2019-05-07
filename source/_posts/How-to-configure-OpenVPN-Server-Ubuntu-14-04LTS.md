---
title: How to configure OpenVPN Server (Ubuntu 14.04LTS)
date: 2015-11-06 15:30:57
tags: 
- OpenVPN 
- Ubuntu
categories:
- Network
---
### Install Software
```bash
apt-get update
apt-get install openvpn
apt-get install easy-rsa
```
### Generate Certificates
```bash
cp -r  /usr/share/easy-rsa/ etc/openvpn/
cd /etc/openvpn/easy-rsa
```
you can edit vars flie, I wanted 1024bit DH parms, so I changed KEY_SIZE from 2048 to 1024.  Some options may be changed with yourself informations. For example, the KEY_EMAIL is modified by me with my private e-mail.

### Build ca.crt, dh1024.pem, server.key and server.crt
```bash
source var
./clean-all
./build-ca
./build-key-server server
./buid-dh
```
After finishing the above steps, ca.crt, dh1024.pem, server.key and server.crt had been generated in path /etc/openvpn/easy-rsa/keys.

### Edit OpenVPN server’s configuration file
```bash
cd /etc/openvpn
touch server.conf
```
Edit server.conf with gedit or vim, the .conf file content is as follows:
```bash
port 7000 #listening port
proto udp #TCP or UDP
dev tun #TAP or TUN
ca /etc/openvpn/easy-rsa/keys/ca.crt
cert /etc/openvpn/easy-rsa/keys/server.crt
key /etc/openvpn/easy-rsa/keys/server.key # This file should be kept secret
dh /etc/openvpn/easy-rsa/keys/dh1024.pem
server 10.8.0.0 255.255.255.0
ifconfig-pool-persist ipp.txt #record of client &lt;-&gt; virtual IP address associations
push "redirect-gateway def1 bypass-dhcp"
push "dhcp-option DNS 8.8.8.8"
client-to-client #clients will see other clients, by default, clients will only see the server
keepalive 10 120
comp-lzo
persist-key
persist-tun
status openvpn-status.log # output a short log file showing current connections
log openvpn.log
log-append openvpn.log
verb 3
```

### Start OpenVPN server and configure Forward
```bash
/etc/init.d/openvpn start
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
```
If your OpenVPN is running on the VPS:
```bash

iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o venet0 -j SNAT --to (venet0 ip)```
