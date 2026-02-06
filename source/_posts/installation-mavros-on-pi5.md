---
title: installation_mavros_on_pi5
date: 2026-02-06 18:16:31
tags:
---
### 从源码安装MAVROS

#### 参考其[Github链接](https://github.com/mavlink/mavros/tree/master/mavros)

#### 1. 软件环境说明
* 树莓派5，4GB版本
* Linux raspberrypi 6.6.20+rpt-rpi-2712
* Debian 12 bookworm
* ros debian 1.15.15
* gcc 12.2.0, g++ 12.2.0, cmake 3.25.1, python 3.11.2, boost 1.74

#### 2. 配置swap
* `free -h` 查看当前是否有swap，有的话是多大，不是4GB的需要修改
* `swapon --show`查看当前swap文件，如果太小，需要增加
* `sudo nano /etc/dphys-swapfile` 设置CONF_SWAPSIZE=4096, CONF_MAXSWAP=4096，保存并退出
* `sudo /etc/init.d/dphys-swapfile restart` 重启swap服务
* `free -h` 查看是否生效

#### 3. 终端命令
* `sudo apt update` 更新最新源
* `sudo apt install python3-catkin python3-rosinstall-generator catkin-tools -y` 安装catkin工具
* `mkdir -p ~/catkin_ws_swarm_path_planning/src` 创建新工作空间
* `cd ~/catkin_ws_swarm_path_planning`
* `catkin_init_workspace` 初始化工作空间
* `wstool init src` 初始化rosinstall工具
* `rosinstall_generator --rosdistro noetic mavlink | tee /tmp/mavros.rosinstall` 将mavlink的github链接添加至暂存文件，使用noetic版本，网络问题多运行几次
* `rosinstall_generator --rosdistro noetic --upstream mavros | tee -a /tmp/mavros.rosinstall` 将mavros的github链接添加至暂存文件，同上
* `rosinstall_generator --rosdistro noetic control_toolbox | tee -a /tmp/mavros.rosinstall` 同上，后续几个为mavros的依赖库
* `rosinstall_generator --rosdistro noetic geographic_msgs | tee -a /tmp/mavros.rosinstall`
* `rosinstall_generator --rosdistro noetic realtime_tools | tee -a /tmp/mavros.rosinstall`
* `rosinstall_generator --rosdistro noetic uuid_msgs | tee -a /tmp/mavros.rosinstall`
* `rosinstall_generator --rosdistro noetic control_msgs | tee -a /tmp/mavros.rosinstall`
* `wstool merge -t src /tmp/mavros.rosinstall` 将暂存文件里的所有链接合并至当前工作空间src下的.rosinstall文件
* `wstool update -t src -j4` 根据.rosinstall文件下载所需软件库
* `rosdep install --from-paths src --ignore-src -y` 根据rosdep依赖关系补充安装依赖项，可能报错，需要先`rosdep update`，网络问题多运行几次
* `ls /usr/share/GeographicLib` 检查是否有Geographiclib所需数据集，没有需下载或拷贝
* `catkin build` 编译工作空间下所有软件
* `source devel/setup.bash` 将可执行文件添加至当前终端命令
* `roslaunch mavros px4.launch` 测试安装是否成功