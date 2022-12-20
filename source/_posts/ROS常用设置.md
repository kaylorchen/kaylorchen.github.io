---
title: ROS常用设置
comments: true
date: 2021-08-07 10:42:15
tags:
- ROS
categories:
- ROS
---

# ROS1

## 安装ROS1
如果是在深圳，推荐使用腾讯的官方源

```bash
sudo sh -c 'echo "deb https://mirrors.tencent.com/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
sudo apt install curl
curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | sudo apt-key add -
sudo apt update
sudo apt install ros-noetic-desktop
sudo apt install python3-rosdep python3-rosinstall python3-rosinstall-generator python3-wstool build-essential
sudo rosdep init
rosdep update
```

## ROS主从机配置

ros的主从机只需要配置ROS_IP和ROS_MASTER_URI就可以。
主机配置的ROS_IP是主机的局域网的IP， ROS_MASTER_URI是应该是使用局域网的IP加端口，如 ROS_MASTER_URI=“http://192.168.23.10:11311”，如果不想配置死环境变量，可以使用roslaunch对针对性应用配置临时环境变量，比如：

```xml
<launch>
<node pkg="rviz" type="rviz" name="rviz" >
    <env name="ROS_IP" value="192.168.23.100" />
    <env name="ROS_MASTER_URI" value="http://192.168.23.10:11311" />
</node>
</launch>
```

