---
layout: post
title: 【SLAM For AR】Realsense+Vins测试
categories:  [SLAM, AR]
description: none
keywords: SLAM, VINS
---

SLAM是AR的重要技术基础，基于实验室的设备进行一些与AR结合的初步的探索。

------

## 前言

Vins-mono是香港科技大学开源的一个VIO算法，[https://github.com/HKUST-Aerial-Robotics/VINS-Mono](https://github.com/HKUST-Aerial-Robotics/VINS-Mono)，是用紧耦合方法实现的，通过单目+IMU恢复出尺度。

环境: ubuntu18.04 ,ros melodic, opencv3(若下载较慢, 建议使用数据流量或科学上网)

## 1. 安装librealsense SDK

参考教程: [intel官方](https://github.com/IntelRealSense/librealsense/blob/master/doc/distribution_linux.md)

1. 注册公钥

```shell
sudo apt-key adv --keyserver keys.gnupg.net --recv-key F6E65AC044F831AC80A06380C8B3A55A6F3EFCDE || sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-key F6E65AC044F831AC80A06380C8B3A55A6F3EFCDE
```

2. 将服务器添加到存储库列表中

- Ubuntu 16 LTS：
  `sudo add-apt-repository "deb http://realsense-hw-public.s3.amazonaws.com/Debian/apt-repo xenial main" -u`
  Ubuntu 18 LTS：
  `sudo add-apt-repository "deb http://realsense-hw-public.s3.amazonaws.com/Debian/apt-repo bionic main" -u`
  Ubuntu 20 LTS：
  `sudo add-apt-repository "deb http://realsense-hw-public.s3.amazonaws.com/Debian/apt-repo focal main" -u`

3. 安装库

```shell
sudo apt-get install librealsense2-dkms
sudo apt-get install librealsense2-utils
```

4. 安装developer和debug packages

```shell
sudo apt-get install librealsense2-dev
sudo apt-get install librealsense2-dbg (这个我没装上但好像不影响跑)
```

5. 测试SDK(终端运行realsense-viewer)（需要usb3.0，否则没有imu输入）

## 2. 安装ROS端realsense工程

提前运行下面的命令可减少安装中出现的错误, 注意命令中ros版本的选择

```shell
sudo apt-get install ros-kinetic-ddynamic-reconfigure
```

(Resource not found: rgbd_launch)

```shell
sudo apt-get install ros-melodic-rgbd-launch
```

启动测试

```BASH
roslaunch realsense2_camera rs_rgbd.launch 
rosrun rviz rviz
```

**rviz中添加话题：**

```cpp
左上角 Displays 中 Fixed Frame 选项中，下拉菜单选择 camera_link，Global Status由红色变绿。
Add -> 上方点击 By topic -> /depth_registered 下的 /points 下的/PointCloud2
Add -> 上方点击 By topic -> /color 下的 /image_raw 下的image
......
```



## 3. 安装VINS并在数据集上测试

下载数据集(可选),本人初次选择EuRoC MAV数据集和AR数据集

[下载链接EuRoC](https://projects.asl.ethz.ch/datasets/doku.php?id=kmavvisualinertialdatasets)

[下载链接AR_BOX](https://pan.baidu.com/s/1geEyHNl?errmsg=Auth+Login+Sucess&errno=0&ssnerror=0&)

安装ceres solver, 根据其他博客的提示,装了旧版本1.14.0

1.安装依赖

```
//第一步，打开sources.list
sudo gedit /etc/apt/sources.list
//第二步，将下面的源粘贴到最上方sources.list
deb http://cz.archive.ubuntu.com/ubuntu trusty main universe 
//第三步，更新源
sudo apt-get update
//第四步，重新输入依赖项安装命令安装依赖项
sudo apt-get install liblapack-dev libsuitesparse-dev libcxsparse3.1.2 libgflags-dev libgoogle-glog-dev libgtest-dev
```

2.下载ceres-solver-1.14.0

```bash
wget ceres-solver.org/ceres-solver-1.14.0.tar.gz
```

3.解压

```bash
tar xvf ceres-solver-1.14.0.tar.gz
```

4.编译

```bash
cd ceres-solver-1.14.0
mkdir build
cd build
cmake ..
make -j4
```

5.安装

```bash
sudo make install
```

此处选择原始vins-mono

```
    cd ~/catkin_ws/src
    下面选一个
# git clone https://github.com/Jichao-Peng/VINS-Mono-Optimization.git
# git clone https://github.com/HKUST-Aerial-Robotics/VINS-Mono.git
    cd ../
    catkin_make
    source ~/catkin_ws/devel/setup.bash
```

测试数据集,以MH01为例

```
   roslaunch vins_estimator euroc.launch 
    roslaunch vins_estimator vins_rviz.launch
    rosbag play YOUR_PATH_TO_DATASET/MH_01_easy.bag 
```

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/2021-01-15 12-08-16 的屏幕截图.png)

## 4. 使用RealSenseD435i测试VINS及AR DEMO

1.**修改realsense包里的rs_camera.launch文件**(也可另存为rs_camera_vins.launch)

第一处，修改unite_imu_method如下，这里是让IMU的角速度和加速度作为一个topic输出

```
 <arg name="unite_imu_method"      default="copy"/>
```

第二处，修改enable_sync参数为true，这里是开机相机和IMU的同步

```
  <arg name="enable_sync"           default="true"/>
```

2.**修改VINS-Mono包里的realsense_color_config.yaml文件**

(1)修改订阅的topic

```
imu_topic: "/camera/imu"
image_topic: "/camera/color/image_raw"
```

(2)修改相机内参，这里先再次打开运行realsesne包，然后可以通过如下命令获取相机内参

```
roslaunch realsense2_camera rs_camera_vins.launch 
rostopic echo /camera/color/camera_info
```

(3)IMU到相机的变换矩阵

```
# Extrinsic parameter between IMU and Camera.
estimate_extrinsic: 2   # 0  Have an accurate extrinsic parameters. We will trust the following imu^R_cam, imu^T_cam, don't change it.
                        # 1  Have an initial guess about extrinsic parameters. We will optimize around your initial guess.
                        # 2  Don't know anything about extrinsic parameters. You don't need to give R,T. We will try to calibrate it. Do some rotation movement at beginning.                        
#If you choose 0 or 1, you should write down the following matrix.
```

(4)IMU参数

```
#imu parameters       The more accurate parameters you provide, the better performance
acc_n: 0.2          # accelerometer measurement noise standard deviation. #0.2
gyr_n: 0.05         # gyroscope measurement noise standard deviation.     #0.05
acc_w: 0.02         # accelerometer bias random work noise standard deviation.  #0.02
gyr_w: 4.0e-5       # gyroscope bias random work noise standard deviation.     #4.0e-5
g_norm: 9.80       # gravity magnitude
```

(5)是否需要在线估计同步时差

```
#unsynchronization parameters
estimate_td: 0                      # online estimate time offset between camera and imu
td: 0.000                           # initial value of time offset. unit: s. readed image clock + td = real image clock (IMU clock)
```

(6)相机曝光改成全局曝光

```
#rolling shutter parameters
rolling_shutter: 0                      # 0: global shutter camera, 1: rolling shutter camera
rolling_shutter_tr: 0               # unit: s. rolling shutter read out time per frame (from data sheet). 
```

**3. 打开摄像头，运行VINS-Mono**

```
roslaunch realsense2_camera rs_camera_vins.launch 
roslaunch vins_estimator realsense_color.launch 
roslaunch vins_estimator vins_rviz.launch
```

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/2021-01-15 13-52-48 的屏幕截图.png)

### ARDEMO

由于不使用鱼眼镜头，故修改相关launch文件

在ar_demo中新建realsense_ar_435.launch

```
<launch>
	<include file="$(find vins_estimator)/launch/realsense_color.launch"/>

    <node pkg="ar_demo" type="ar_demo_node" name="ar_demo_node" output="screen">
        <remap from="~image_raw" to="/camera/color/image_raw"/>
        <remap from="~camera_pose" to="/vins_estimator/camera_pose"/>
        <remap from="~pointcloud" to="/vins_estimator/point_cloud"/>
        <param name="calib_file" type="string" value="$(find feature_tracker)/../config/realsense/realsense_color_config.yaml"/>
        <param name="use_undistored_img" type="bool" value="false"/>
    </node>
</launch>
```



```
roslaunch realsense2_camera rs_camera_vins.launch 
roslaunch ar_demo realsense_ar_435.launch
roslaunch ar_demo ar_rviz.launch
```

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/2021-01-23 23-45-25屏幕截图.png)

## References

https://blog.csdn.net/qq_42800654/article/details/109393646

https://blog.csdn.net/Coderii/article/details/87601836