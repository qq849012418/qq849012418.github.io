---

layout: post
title: ROS学习（一）在ubuntu上安装ros melodic遇到的问题
categories:  [ROS, Linux]
description: none
keywords: ubuntu,  ros
---

项目原因，开始自学ros系统，本文展示了在ubuntu18.04上安装ros melodic的具体方法及可能会遇到的坑。

------

## 可能用到的链接

官方melodic安装[http://wiki.ros.org/cn/melodic/Installation/Ubuntu](http://wiki.ros.org/cn/melodic/Installation/Ubuntu)

中文wiki[http://wiki.ros.org/cn](http://wiki.ros.org/cn)

【视频】中科院软件所-机器人操作系统入门[https://www.bilibili.com/video/BV1mJ411R7Ni?p=2](https://www.bilibili.com/video/BV1mJ411R7Ni?p=2)

正确安装的效果

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/2020-04-26 13-38-38屏幕截图.png)

### 安装中遇到的坑

-------

 https://raw.githubusercontent.com/ros/rosdistro/master/rosdep/sources.list.d/20-default.list

ERROR: cannot download default sources list from:
https://raw.githubusercontent.com/ros/rosdistro/master/rosdep/sources.list.d/20-default.list
Website may be down.

解决方案：参考[郭润](https://www.cnblogs.com/gary-guo/p/12650552.html)的方法，但是有所不同

首先排查原因到网络问题，这个链接无法在浏览器中打开，代理、证书啥的好像都没问题，所以网上大多数解决方案都没啥用。

具体解决步骤：

1. 

```shell
git clone https://github.com/ros/rosdistro.git
```

我这里是克隆到/home/k/ROS/

2. 在本地rosdistro中搜索20-default.list，将其url由raw.githubusercontent.com/...指向本地repo；

格式为：file:///home/k/ROS/rosdistro/...注意具体的文件夹关系

3. 类似的，搜索rosdep2和rosdistro中出现raw.githubusercontent.com的位置，将其指向本地repo。

   这里我修改的文件路径依次为：

   /usr/lib/python2.7/dist-packages/rosdep2/下面的sources_list.py, rep3.py, gbpdistro_support.py

   以及/usr/lib/python2.7/dist-packages/rosdistro/ __init__.py

###### 注意1：每个文件修改前都要进行sudo chmod 777处理，使之可以修改

###### 注意2：下载文件夹路径最好正常点，避开空格、中文之类的

------

	Traceback (most recent call last): File "/usr/bin/rosdep", line 3, in <module> from rosdep2.main import rosdep_main ImportError: No module named 'rosdep2'

解决方案：将Ubuntu默认python版本设置为2.7。

------

