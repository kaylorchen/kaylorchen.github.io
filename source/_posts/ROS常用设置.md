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

## 编译相关

```bash
rosdep install --from-paths src --ignore-src -r -y #根据package.xml安装依赖
catkcatkin_make install -DCATKIN_WHITELIST_PACKAGES="clean_robot_base;ultrasonic;tof_pointcloud" -DCMAKE_BUILD_TYPE=Debug
catkin_create_pkg 包名 依赖1 依赖2 ...
```

有时候为了方便，也可以编辑Makefile，添加一下特定的环境变量

```makefile
ARCH = $(shell arch)
all: 
 @echo CPU ARCH is ${ARCH}
 catkin_make install -DCATKIN_WHITELIST_PACKAGES="clean_robot_base;ultrasonic;tof_pointcloud" \
 -DCMAKE_BUILD_TYPE=Release \
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

## 安装部署
ROS提供了一个robot_upstart的特殊节点作为安装服务。我们先看一下robot_upstart的用法：
```bash
> rosrun robot_upstart install -h
usage: install [-h] [--job JOB] [--interface ethN] [--user NAME] [--setup path/to/setup.bash] [--rosdistro DISTRO] [--master http://MASTER:11311]
               [--logdir path/to/logs] [--augment] [--provider [upstart|systemd]] [--symlink] [--wait] [--systemd-after After=]
               pkg/path [pkg/path ...]

Use this tool to quickly and easily create system startup jobs which run one or more ROS launch files as a daemonized background process on your computer.
More advanced users will prefer to access the Python API from their own setup scripts, but this exists as a simple helper, an example, and a compatibility
shim for previous versions of robot_upstart which were bash-based.

positional arguments:
  pkg/path             包名/需要安装的launch文件路劲，比如包名是test，launch文件路径是launch/test.launch，那么这个选项就是test/launch/test.launch 
optional arguments:
  -h, --help            show this help message and exit
  --job JOB             给服务取一个指定的名字，可以不设置
  --interface ethN      Specify network interface name to associate job with.
  --user NAME           Specify user to launch job as.
  --setup path/to/setup.bash 这里是指向一个包的setup.bash路径，为了避免不必要的麻烦，使用绝对路径
  --rosdistro DISTRO    Specify ROS distro this is for.
  --master http://MASTER:11311
                        Specify an alternative ROS_MASTER_URI for the job launch context.
  --logdir path/to/logs
                        Specify an a value for ROS_LOG_DIR in the job launch context.
  --augment             Bypass creating the job, and only copy user files. Assumes the job was previously created.
  --provider [upstart|systemd]
                        Specify provider if the autodetect fails to identify the correct provider
  --symlink             不复制launch文件，使用软链接指向包的launch文件 
  --wait                Pass a wait flag to roslaunch.
  --systemd-after test.service 设置这个服务在test.service之后启动
```

参考指令
```bash
 rosrun robot_upstart install --job kaylor --setup $(pwd)/install/setup.bash --systemd-after "test.service network.target" test/launch/talker.launch
```
上面的指令，注意setup.bash一定是bash！！！如果需要加入环境变量，可以自己添加一个.bash文件，设置环境变量之后，再source这个ros的setup.bash。当然，也可以通过修改service文件引入环境变量


# ROS2

## 编译相关

### colcon build
- 使用symlink选项，install文件夹的文件生成软连接，否则生成的是拷贝  
```
colcon build --symlink-install
```
- 编译指定的包  
```
colcon build --packages-select package_name
```
- 编译指定的包和它的依赖  
```
colcon build --packages-up-to package_name
```
- 编译时忽略的包  
```
colcon build --packages-ignore package_name
```
- 编译的时候添加cmake参数选项   
```
colcon build --cmake-args -DCMAKE_BUILD_TYPE=Debug/Release
```
- 编译一个package 并且把log 显示在屏幕上  
```
colcon build --packages-select rmw_coredds_shared_cpp --symlink-install --cmake-args -DCMAKE_BUILD_TYPE=Debug --event-handlers=console_direct+
```
- 编译之前构建失败的包，不包括先前已经中止的包  
```
colcon build --packages-select-build-failed
```
- 跳过之前之前已经完成构建的包
```
colcon build --packages-skip-build-finished
```
- 指定编译的build base  
```
colcon build --build-base path
```
- 指定安装目录
```
colcon build --install-base path
```
- 并行编译参数
```
colcon build --parallel-worker Number
```

