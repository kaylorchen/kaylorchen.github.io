---
title: Deploy a Transparent Proxy with Squid
date: 2016-01-06 11:33:04
tags:
- Proxy
- Squid
categories:
- Network
---
Sometimes, we want to have a local transparent proxy, and it will provide http, https and e-mail’s proxy.
Why do we need a local proxy? As we know, most of routers don’t support Ipv6. If there is a router with ipv6 in a local area network, and it also is proxy server, so other device will browse a ipv6 homepage via proxy.
OK, let’s begin:
```
apt-get update
apt-get install squid
```
Edit squid’s configuration file, it’s path is ***/etc/squid3/squid.conf***. there is a reference file:
```
acl SSL_ports port 443
acl Safe_ports port 80          # http
acl Safe_ports port 21          # ftp
acl Safe_ports port 443         # https
acl Safe_ports port 70          # gopher
acl Safe_ports port 210         # wais
acl Safe_ports port 1025-65535  # unregistered ports
acl Safe_ports port 280         # http-mgmt
acl Safe_ports port 488         # gss-http
acl Safe_ports port 591         # filemaker
acl Safe_ports port 777         # multiling http
acl CONNECT method CONNECT
http_access deny !Safe_ports
http_access deny CONNECT !SSL_ports
http_access allow localhost manager
http_access deny manager
http_access allow localhost
acl openvpn6 src 10.8.6.0/24
http_access allow openvpn6
http_access deny all
http_port 3128
coredump_dir /var/spool/squid3
refresh_pattern ^ftp:           1440    20%     10080
refresh_pattern ^gopher:        1440    0%      1440
refresh_pattern -i (/cgi-bin/|\?) 0     0%      0
refresh_pattern (Release|Packages(.gz)*)$      0       20%     2880
refresh_pattern .               0       20%     4320
forwarded_for transparent
```
After all the above steps, enter a command and restart service.
```
/etc/init.d/squid3 restart
```
Finally, configure your browser with “server’s ip” and “port”(3128).