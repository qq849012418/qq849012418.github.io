---
layout: post
title: 【SLAM For AR】Realsense+ORB_SLAM3测试
categories:  [SLAM, AR]
description: none
keywords: SLAM, VINS
---

SLAM是AR的重要技术基础，基于实验室的设备进行一些与AR结合的初步的探索，本篇探索2020年较新的明星级SLAM，ORB_SLAM3。

------

这里假装你已经配好了realsensesdk/ros/opencv/eigen，前两者的安装可以看上一篇文章。

## 安装依赖

### Pangolin

```
git clone https://github.com/stevenlovegrove/Pangolin.git
sudo apt-get install libglew-dev
sudo apt-get install cmake
sudo apt-get install libpython2.7-dev
sudo apt-get install ffmpeg libavcodec-dev libavutil-dev libavformat-dev libswscale-dev libavdevice-dev
sudo apt-get install libdc1394-22-dev libraw1394-dev
sudo apt-get install libjpeg-dev libpng12-dev libtiff5-dev libopenexr-dev
cd Pangolin
mkdir build
cd build
cmake ..
cmake --build 
sudo make install
```

### OpenCV

编好了3.2.0,此处略

### Eigen3

```
sudo apt-get install libeigen3-dev
```

## 编译ORB_SLAM3

进入ROS的src目录

```
git clone https://github.com/UZ-SLAMLab/ORB_SLAM3.git
```

这个版本是官方版本, 期间在下还尝试了3D视觉研习社的注释版本,最终使用了官方版本.

根据文献4, 把build.sh文件中的make -j 改成make -j4,防止编译卡死

根据文献5, 打开LoopClosing.h，将

```cpp
typedef map<KeyFrame*,g2o::Sim3,std::less<KeyFrame*>,
        Eigen::aligned_allocator<std::pair<const KeyFrame*, g2o::Sim3> > > KeyFrameAndPose;
```

改为

```cpp
typedef map<KeyFrame*,g2o::Sim3,std::less<KeyFrame*>,
        Eigen::aligned_allocator<std::pair<KeyFrame *const, g2o::Sim3> > > KeyFrameAndPose;
```

解决以下报错

```
/usr/include/c++/10/bits/stl_map.h:123:71: error: static assertion failed: std::map must have the same value_type as its allocator
```

进入ORB_SLAM3目录,运行以下指令:

```
chmod +x build.sh
./build.sh
```

## 编译ROS接口

首先要把ORB_SLAM3/Examples/ROS加入ros的包目录

```
sudo gedit ~/.bashrc
```

在最后加两句(换成你的相应目录): 

```
export ROS_PACKAGE_PATH=${ROS_PACKAGE_PATH}:/home/k/ROS/catkin_ws/src/ORB_SLAM3_detailed_comments/Examples/ROS
source /home/k/ROS/catkin_ws/src/ORB_SLAM3/Examples/ROS/ORB_SLAM3/build/devel/setup.bash
```

source 一下, 之后进入ORB_SLAM3目录,运行以下指令. 注意:不要瞎用sudo去运行,我加了sudo之后莫名其妙说rosbuild找不到......

```
#安装ros版
chmod +x build_ros.sh
./build_ros.sh
```

## 配置435i相机参数

将ROS/ORBSLAM3下面的Asus.yaml复制改名为435i.yaml

```
roslaunch realsense2_camera rs_rgbd.launch
rostopic echo /camera/color/camera_info
```

得到K = [fx 0 cx 0 fy cy 0 0 1 ], 将相应数值填入yaml # Camera calibration and distortion parameters (OpenCV) 对应项下面

根据文献6, D435i的baseline为50mm，bf的值为bf = baseline (in meters) * fx,因此修改yaml # Close/Far threshold. Baseline times.和# IR projector baseline times fx (aprox.)对应项



## AR测试

进入ORB_SLAM3/Examples/ROS/ORB_SLAM3/src/AR，打开ros_mono_ar.cc，由于使用realsense测试，修改图像获取节点为"/camera/color/image_raw"，重新build_ros.sh

运行

```
roscore
rosrun ORB_SLAM3 MonoAR /home/k/ROS/catkin_ws/src/ORB_SLAM3/Vocabulary/ORBvoc.txt /home/k/ROS/catkin_ws/src/ORB_SLAM3/Examples/ROS/ORB_SLAM3/435i.yaml
roslaunch realsense2_camera rs_camera.launch
```

可能报段错误,可尝试的解决方法是：在build.sh同目录下的cmakelist中，去掉-march=native

，因为它会降低可执行文件的移植性。在指定了-march后，gcc不会生成兼容的指令集，导致了该二进制文件在其他架构上运行时，该架构的cpu并不能识别相应指令，造成illegal instruction & segmentation fault。

效果如图

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/2021-01-23 21-15-56 的屏幕截图.png)

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/2021-01-23 21-33-51 的屏幕截图.png)



## References

1. [用realsense d435i传感器在实际环境中跑ORB_SLAM3，顺带解决一部分编译问题](https://blog.csdn.net/weixin_44350238/article/details/107641849)
2. [ubuntu18.04安装Pangolin](https://www.cnblogs.com/qilai/p/13866082.html)
3. [Ubuntu18.04 两种方式安装eigen3](https://blog.csdn.net/wenxue204/article/details/96957098)
4. [Ubuntu16.04 运行ORBSLAM3](http://www.xiaoheidiannao.com/203760.html)
5. [ORB-SLAM2编译错误](https://blog.csdn.net/lixujie666/article/details/90023059)
6. [realsensed435跑通ORB_SLAMv2](https://www.cnblogs.com/long5683/p/12668396.html)
7. [-march=native引发的segmentation fault问题](https://blog.csdn.net/torresergio/article/details/103253538)

