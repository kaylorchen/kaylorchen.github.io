---
title: Setup a SoftAP with Raspberry Pi
comments: true
date: 2019-05-22 18:08:34
tags:
- Raspberry Pi
- hostapd
- dnsmasq
categories:
- [Raspberry Pi]
- [Network]
---
# Install hostapd and dnsmasq
```bash
sudo apt install hostapd dnsmasq
```
# Configuration
## Edit dhcpcd.conf
Edit ***/etc/dhcpcd.conf*** and append a new line: **denyinterfaces wlan0**.

## Edit interfaces
Edit ***/etc/network/interfaces.d/wlan0***, and the content is as follows:
```bash
allow-hotplug wlan0
iface wlan0 inet static
address 192.168.77.77
netmask 255.255.255.0
```

## Restart dhcpcd service
```bash
sudo service dhcpcd restart
sudo ifdown wlan0
sudo ifup wlan0
```

## Configure hostapd
Edit ***/etc/hostapd/hostapd.conf***, and the contend is as follows:
```bash
# This is the name of the WiFi interface we configured above
interface=wlan0
# Use the nl80211 driver with the brcmfmac driver
driver=nl80211
# This is the name of the network
ssid=Pi3-AP
# Use the 2.4GHz band
hw_mode=g
# Use channel 6
channel=6
# Enable 802.11n
ieee80211n=1
# Enable WMM
wmm_enabled=1
# Enable 40MHz channels with 20ns guard interval
ht_capab=[HT40][SHORT-GI-20][DSSS_CCK-40]
# Accept all MAC addresses
macaddr_acl=0
# Use WPA authentication
auth_algs=1
# Require clients to know the network name
ignore_broadcast_ssid=0
# Use WPA2
wpa=2
# Use a pre-shared key
wpa_key_mgmt=WPA-PSK
# The network passphrase
wpa_passphrase=raspberry
# Use AES, instead of TKIP
rsn_pairwise=CCMP
```
Edit ***/etc/default/hostapd***, and append a new line: **DAEMON_CONF="/etc/hostapd/hostapd.conf"**
Test: ``sudo hostapd /etc/hostapd/hostapd.conf``