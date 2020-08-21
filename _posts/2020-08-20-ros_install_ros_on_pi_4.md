---
layout: post
title: 简记ROS环境配置 ROS Melodic on 树莓派4（Raspbian | Debian9.8）
categories:  ROS
description: none
keywords: pool, unity
---

第一次尝试Raspbian，感觉比ubuntu清爽好用一些，自带了很多的IDE，不过安装ros的时候还是有几个坑，简单记录下。

------


### 主要参考安装过程链接：

[Installing ROS Melodic on the Raspberry Pi](http://wiki.ros.org/ROSberryPi/Installing%20ROS%20Melodic%20on%20the%20Raspberry%20Pi)

安装前，进行了如下依赖安装

```sh
sudo apt install dirmngr
```

原始内存99M太小，增加内存防卡死（路径可自己定义）

```sh
sudo fallocate -l 1G /opt/swap 
sudo mkswap /opt/swap
sudo swapon /opt/swap
free -m
```

由于树莓派计算能力有限，推荐选择**ROS-Comm**而不是完整安装



在安装到`rosdep install -y --from-paths src --ignore-src --rosdistro melodic -r --os=debian:buster`这一步时，出现了找不到`python-pycryptodome`的情况

故改为

```sh
rosdep install -y --from-paths src --ignore-src --rosdistro melodic -r --os=debian:buster --skip-keys=" python-pycryptodome"
```

顺利完成安装，roscore测试也没问题！

#### 后续

第二天测试action节点的时候catkin_make出了问题——Comm版好像默认只有message和service

需要在这两个地方下载actionlib相关的包

https://github.com/ros/common_msgs/tree/noetic-devel/actionlib_msgs

https://github.com/ros/actionlib

在catkin_make之前，可输入这个加快速度（因为默认只用2或者4线运行）

```sh
export ROS_PARALLEL_JOBS='-j8 -l8'
```

