---
title: How to Realize QoS in your Openwrt Router
date: 2015-11-07 16:27:52
tags:
- OpenWRT
---
I realize IP QoS(Quality of Service) in my Openwrt with iptables and tc(Traffic Control).

Firstly , we should installs the related software packages.
```bash
opkg update
opkg install tc iptables
```
If your firmware is trunk version, you have to install kmod-sched. this software has been installed in stable version. If openwrt lacks the software, “RTNETLINK answers: No such file or directory” will be present when you enter tc command. So:
```bash
opkg install kmod-sched
```
After the above steps, let’s edit Qos shell scripts.
```bash
mkdir /etc/qos
vi /etc/qos/start.sh
```
Details of the start.sh are as follows:
```bash
#!/bin/sh

IDEV="br-lan"
ODEV="tun0"

UP="800kbit"
DOWN="1200kbit"

UPLOAD="80kbit"
DOWNLOAD="120kbit"

MUPLOAD="160kbit"
MDOWNLOAD="160kbit"

INET="192.168.254."

IPS="240"
IPE="249"

tc qdisc del dev $ODEV root 2>/dev/null
tc qdisc del dev $IDEV root 2>/dev/null

tc qdisc add dev $ODEV root handle 10: htb default 256
tc qdisc add dev $IDEV root handle 10: htb default 256

tc class add dev $ODEV parent 10: classid 10:1 htb rate $UP ceil $UP
tc class add dev $IDEV parent 10: classid 10:1 htb rate $DOWN ceil $DOWN

i=$IPS;
while [ $i -le $IPE ]
do
tc class add dev $ODEV parent 10:1 classid 10:2$i htb rate $UPLOAD ceil $MUPLOAD prio 1
tc qdisc add dev $ODEV parent 10:2$i handle 100$i: pfifo
tc filter add dev $ODEV parent 10: protocol ip prio 100 handle 2$i fw classid 10:2$i
tc class add dev $IDEV parent 10:1 classid 10:2$i htb rate $DOWNLOAD ceil $MDOWNLOAD prio 1
tc qdisc add dev $IDEV parent 10:2$i handle 100$i: pfifo
tc filter add dev $IDEV parent 10: protocol ip prio 100 handle 2$i fw classid 10:2$i
iptables -t mangle -A PREROUTING -s $INET$i -j MARK --set-mark 2$i
iptables -t mangle -A PREROUTING -s $INET$i -j RETURN
iptables -t mangle -A POSTROUTING -d $INET$i -j MARK --set-mark 2$i
iptables -t mangle -A POSTROUTING -d $INET$i -j RETURN
i=`expr $i + 1`
done
```
Generate *stop.sh*.
```bash
#!/bin/sh

INET="192.168.254."

IPS="240"
IPE="249"

tc qdisc del dev $ODEV root 2>/dev/null
tc qdisc del dev $IDEV root 2>/dev/null

p=$IPS;
while [ $p -le $IPE ]
do
iptables -t mangle -D PREROUTING -s $INET$p -j MARK --set-mark 2$p
iptables -t mangle -D PREROUTING -s $INET$p -j RETURN
iptables -t mangle -D POSTROUTING -d $INET$p -j MARK --set-mark 2$p
iptables -t mangle -D POSTROUTING -d $INET$p -j RETURN
p=`expr $p + 1`
done
```

```bash
chmod +x st*
./start.sh
```
