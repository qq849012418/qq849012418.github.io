---
layout: post
title: 从零开始的机器学习平台配置（二）算法环境配置
categories: Linux
description: none
keywords: linux, ununtu, opencv, cuda, pytorch, tensorRT
---

这一章内容很干，新手可能要装一天，但是装完后绝对可以对linux更加熟悉。

------



## 0.修改默认python环境

将原来 python 的软链接重命名：

mv /usr/bin/python /usr/bin/python.bak

将 python 链接至 python3：

ln -s /usr/bin/python3.6 /usr/bin/python

这时，再查看 Python 的版本：

python -V



## 1.opencv

参考https://xinyang-go.github.io/jiaolong-cv-novice-tutorial/environment-setup/及队内培训。

首先安装依赖，在控制台执行以下命令

```Shell
sudo apt update
sudo apt upgrade -y
sudo apt install -y htop cmake cmake-gui gcc g++ vim git wget net-tools zip unzip build-essential intltool curl libcanberra-gtk-module python python-dev python-pip python-tk python3 python3-dev python3-pip python3-tk libssl-dev libgtk-3-dev libeigen3-dev libavcodec-dev libavformat-dev libpng-dev libtiff5-dev libjpeg-dev libwebp-dev libavresample-dev libswscale-dev libavutil-dev libopenexr-dev
sudo pip3 install --upgrade pip setuptools
pip3 install --user numpy
```

下载opencv4.1.2及其contrib源码

https://github.com/opencv/opencv/archive/4.1.2.tar.gz

https://github.com/opencv/opencv_contrib/archive/4.1.2.tar.gz

创建opencv-4.1.2目录，将两个文件夹解压到这里，再创建一个build文件夹。

（参考下面的博客进行camke-gui编译)

https://blog.csdn.net/jindunwan7388/article/details/80397700

下面的更全面，非gui

https://www.jianshu.com/p/f54b0fc13811

注意事项：

OPENCV-GENERATE-PKGCONFIG=ON

CMAKE_INSTALL_PERFIX=/usr/local/opencv4

OPENCV_EXTRA_MODULES_PATH=/你刚创建的opencv-4.1.2目录/opencv_contrib-4.1.2/modules

[20200209更新] 如果你需要使用surf等nonfree模块，应多选这项

OPENCV_ENABLE_NONFREE=ON

环境配置见博客



## 2.CLION

见官网

## 3.PYCHARM

见官网



## 4.CUDA

https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&target_distro=Ubuntu&target_version=1804&target_type=runfilelocal

对于ubuntu18.04下的64为版本，也可用以下代码安装

```Shell
wget http://developer.download.nvidia.com/compute/cuda/10.1/Prod/local_installers/cuda_10.1.243_418.87.00_linux.run
sudo sh cuda_10.1.243_418.87.00_linux.run
```

环境变量设置

```Shell
export PATH=/usr/local/cuda-10.1/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda-10.1/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```

### 4.5 CUDNN

https://developer.nvidia.com/rdp/cudnn-download

下载lib文件并复制到cuda的目录(/usr/local/cuda-10.1)

sudo ldconfig即可

## 5.pytorch

https://download.pytorch.org/whl/torch_stable.html在这个链接里找到你需要的版本，用官网的下载方式会很慢。直接用浏览器下。

比如我的是

[cu101/torch-1.3.0-cp36-cp36m-manylinux1_x86_64.whl](https://download.pytorch.org/whl/cu101/torch-1.3.0-cp36-cp36m-manylinux1_x86_64.whl)

[cu101/torchvision-0.4.1-cp36-cp36m-linux_x86_64.whl](https://download.pytorch.org/whl/cu101/torchvision-0.4.1-cp36-cp36m-linux_x86_64.whl)

下好后进入到相应文件夹中，sudo pip3 install 文件名即可

在py3环境下输入import torch和import torchvision来验证是否安装成功。



## 6.tensorRT

https://developer.nvidia.com/nvidia-tensorrt-6x-download

环境变量设置

```Shell
export LD_LIBRARY_PATH=/home/k/RM/TensorRT-6.0.1.5/lib
```



## 7.torch2trt(用于模型转换)

```Shell
git clone https://github.com/NVIDIA-AI-IOT/torch2trt
cd torch2trt
sudo python setup.py install
```



## 8.装一些常用工具吧～

deepin-wine: 参考https://gitee.com/wszqkzqk/deepin-wine-for-ubuntu即可。