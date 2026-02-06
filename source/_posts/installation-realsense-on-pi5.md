---
title: installation_realsense_on_pi5
date: 2026-02-06 19:09:18
tags:
---
参考[installation_respbain.md](https://github.com/IntelRealSense/librealsense/blob/development/doc/installation_raspbian.md)和[installation.md](https://github.com/IntelRealSense/librealsense/blob/development/doc/installation.md)

### Install dependencies
缺啥装啥
```
sudo apt update
sudo apt-get install libssl-dev libusb-1.0-0-dev libudev-dev pkg-config libgtk-3-dev
sudo apt-get install git wget cmake build-essential
sudo apt-get install libglfw3-dev libgl1-mesa-dev libglu1-mesa-dev at libtbb-dev
```

### Install librealsense2
```
git clone https://github.com/IntelRealSense/librealsense.git
```

udev规则，两种方法，本质一样，我用了第2种
```
1)
./scripts/setup_udev_rules.sh
```
上面脚本实际就干了这事，--reload-rules跟--reload一样的，我用了reload
```
2)
cd librealsense
sudo cp config/99-realsense-libusb.rules /etc/udev/rules.d/
sudo udevadm control --reload-rules && sudo udevadm trigger
```

```
cd ~/librealsense
mkdir build && cd build
cmake .. -DBUILD_EXAMPLES=true -DCMAKE_BUILD_TYPE=Release -DFORCE_LIBUVC=true
make -j2
sudo make install
```
注意连接线要3.0
```
realsense-viewer
```

### Install Intel® RealSense™ ROS from Sources
```
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws/src/
git clone https://github.com/IntelRealSense/realsense-ros.git
cd realsense-ros/
git checkout `git tag | sort -V | grep -P "^2.\d+\.\d+" | tail -1`
cd ..
```

```
cd ..
catkin_make clean
```

catkin_make clean报错缺ddynamic_reconfigure，在工作空间src下
```
git clone https://github.com/pal-robotics/ddynamic_reconfigure.git
```

catkin_make clean，又缺diagnostic_updater
```
sudo apt-cache search diagnostic_updater
sudo apt install libdiagnostic-updater-dev
```

catkin_make clean不缺东西了
```
catkin_make -DCATKIN_ENABLE_TESTING=False -DCMAKE_BUILD_TYPE=Release
```
报错pluginlib/class_list_macros.h no such file or directory，经查询系统已有pluginlib，只是头文件是class_list_macros.hpp，修改报错位置的头文件引用为
```
# include <pluginlib/class_list_macros.hpp>
```
编译通过

```
catkin_make install
```

### Run realsense-ros
```
roslaunch realsense2_camera rs_camera.launch
```
没报错，但有warning
```
(messenger-libusb.cpp:42) control_transfer returned error, index: 768, error: Success, number: 0
(messenger-libusb.cpp:42) control_transfer returned error, index: 768, error: Resource temporarily unavailable, number: 11
```
查了一下，似乎不影响正常运行
```
rviz
报错rviz::RenderSystem: error creating render window: RenderingAPIException: Invalid parentWindowHandle (wrong server or screen) in GLXWindow::create at ./.obj-aarch64-linux-gnu/ogre_vendor-prefix/src/ogre_vendor/RenderSystems/GLSupport/src/GLX/OgreGLXWindow.cpp (line 246)
```
参考[ROS2安装报错](https://docs.ros.org/en/jazzy/How-To-Guides/Installation-Troubleshooting.html#linux)，这是因为Wayland和rviz不兼容，需要在运行rviz前指定使用x11，再运行rviz即可
```
QT_QPA_PLATFORM=xcb rviz
```