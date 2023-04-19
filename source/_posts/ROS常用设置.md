---
title: ROS常用设置
comments: true
date: 2022-08-07 10:42:15
tags:
- ROS
categories:
- ROS
---

# ROS1

## 安装ROS1
如果是在深圳，推荐使用腾讯的官方源，该教程默认安装noetic版本的ROS

```bash
sudo sh -c 'echo "deb https://mirrors.tencent.com/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
sudo apt install curl
curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | sudo apt-key add -
sudo apt update
sudo apt install ros-noetic-ros-base python3-rosdep python3-rosinstall python3-rosinstall-generator python3-wstool build-essential
sudo rosdep init
rosdep update
```

## ROS主从机配置

ros的主从机只需要配置ROS_IP和ROS_MASTER_URI就可以。
主机配置的ROS_IP是主机的局域网的IP， ROS_MASTER_URI是应该是使用局域网的IP加端口，如 ROS_MASTER_URI=“http://192.168.23.10:11311”，  
如果主机端没有配置ROS_IP，那么主机的xmlrpc会直接返回hostname，这时候如果从机并不知道该hostname是对应什么IP，这样从机就无法建立有效链接。
如果不想配置死环境变量，可以使用roslaunch对针对性应用配置临时环境变量，比如：

```xml
<launch>
<node pkg="rviz" type="rviz" name="rviz" >
    <env name="ROS_IP" value="192.168.23.100" />
    <env name="ROS_MASTER_URI" value="http://192.168.23.10:11311" />
</node>
</launch>
```

切记，ROS_MASTER_URI就是局域网内主机的IP，两侧机器是一样的。但是ROS_IP是各自主机的IP。一切配hostname的教程都是大忽悠！！！！

## 常用指令

```bash
rosdep install --from-paths src --ignore-src -r -y
catkin_make -DCMAKE_BUILD_TYPE=Release --only test
catkin_make install -DCATKIN_WHITELIST_PACKAGES="clean_robot_base;ultrasonic;tof_pointcloud" -DCMAKE_BUILD_TYPE=Debug
catkin_create_pkg 包名 依赖1 依赖2 ...
```

有时候为了方便，也可以编辑Makefile，添加一下特定的环境变量

```makefile
ARCH = $(shell arch)
all: 
 @echo CPU ARCH is ${ARCH}
 catkin_make install -DCATKIN_WHITELIST_PACKAGES="clean_robot_base;ultrasonic;tof_pointcloud" \
 -DCMAKE_BUILD_TYPE=Debug \
 -DCATKIN_DEVEL_PREFIX=./devel_${ARCH} \
 -DCMAKE_INSTALL_PREFIX=./install_${ARCH} \
 --build ./build_${ARCH}
```


## CMakeLists.txt的install选项

```bash
# 安装可执行文件
install(TARGETS ${PROJECT_NAME}_node
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

# 安装用户动态库
install(TARGETS ${PROJECT_NAME}
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_GLOBAL_BIN_DESTINATION}
)

# 安装文件
install(FILES
  myfile1
  myfile2
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)

# 安装文件夹
install(DIRECTORY launch
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
  PATTERN ".svn" EXCLUDE)

install(DIRECTORY params
DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
PATTERN ".svn" EXCLUDE)

```


