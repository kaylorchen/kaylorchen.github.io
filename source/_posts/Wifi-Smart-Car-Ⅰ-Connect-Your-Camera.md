---
title: 'Wifi Smart Car Ⅰ: Connect Your Camera'
date: 2015-11-15 21:55:38
tags:
- Wifi Smart Car
- OpenWRT
categories:
- Wifi Smart Car
---
# Hardware
- GL.inet AR150 or GL.inet 6416/6408
- Logitech C270 camera

I update the router with TP-WR720N‘s firmware.
## Install usb driver and mjpg server
```
opkg update
opkg install kmod-usb2 kmod-usb-ohci kmod-video-gspca-core kmod-video-core kmod-video-uvc  kmod-i2c-core kmod-input-core mjpg-streamer
```
## Configurate mjpg server
```
vi /etc/config/mjpg-streamer
```
the details are as follow:
```
config mjpg-streamer 'core'
        option enabled '1'
        option input 'uvc'
        option output 'http'
        option device '/dev/video0'
        option resolution '640x480'
        option fps '60'
        option www '/www/webcam'
        option port '8080'
        option yuv '0'
```
## Test
Restart your router, and type router_ip:8080 in your address bar of browser. you will see:

![](/image/Cam_Capture.jpg)

**PS**: some router need to enable USB device. For instance, AR150′s USB device is enabled by GPIO6.
```
echo 6 > /sys/class/gpio/export # Load GPIO6
echo out >  /sys/class/gpio/gpio6/direction # Set direction as out
echo 1 > /sys/class/gpio/gpio6/value  # Set output level as high, enable USB device
echo 6 > /sys/class/gpio/unexport # Unload GPIO, but output level keep the current voltage
```

