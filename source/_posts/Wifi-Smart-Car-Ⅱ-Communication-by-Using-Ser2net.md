---
title: 'Wifi Smart Car Ⅱ: Communication by Using Ser2net'
date: 2015-11-17 11:16:57
tags:
- Wifi Smart Car
- OpenWRT
categories:
- Wifi Smart Car
---
Ser2net is a software that can convert data between net and serial port.
```
opkg update 
opkg install ser2net
```

Use command ``ls /dev/tty*``, and find the router’s serial port is called as ttyATH0.
Edit the ***/etc/ser2net.conf***, find a line by searching for “2001” within this file and change it as following:

```
2001:raw:600:/dev/ttyATH0:115200 NONE 1STOPBIT 8DATABITS XONXOFF LOCAL -RTSCTS
```
Start ser2net server with command: ``ser2netConnect`` router’s uart to PC, and open serial port debugging assistant.
Open your command prompt or terminal, type ``telnet router_ip 2001``. My router ip is 192.168.1.110, so I enter ``telnet 192.168.1.110 2001``.

Now, test this server. I can type some characters in terminal, they are shown in serial port debugging assistant and vice versa.
![](/image/ser2net.jpg)