---
title: How to setup your private MQTT server
comments: true
date: 2019-05-02 21:56:35
tags:
- IoT
- MQTT
categories:
- IoT
---

# Install MQTT Software Package
```
apt update
apt install mosquitto
```
# Configure and Start MQTT Service
Edit ***/etc/mosquitto/mosquitto.conf***, and append the following contents:
```
#change default port
port 11111 
#deny anonymous
allow_anonymous false 
#select password file
password_file /etc/mosquitto/passwd 
```
Then, generate password file for MQTT service.
```
mosquitto_passwd -c /etc/mosquitto/passwd username
```
If you want to add one more user, you can:
```
mosquitto_passwd /etc/mosquitto/passwd another_username
```
Finally, start MQTT service:
```
systemctl restart mosquitto
```

# Test MQTT Service
## MAC OS Client
Download **MQTTBox** software from Apple Store.
![](/image/Mqtt_client2.png)
![](/image/Mqtt_client.png)
## Chrome Plugin
Search **MQTTLens** in Chrome APP Store and install it.
![](/image/Mqtt_client3.png)
![](/image/Mqtt_client4.png)
