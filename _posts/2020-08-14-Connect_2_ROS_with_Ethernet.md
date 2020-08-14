---
layout: post
title: 两台 Ubuntu ROS系统通过网线直连共享节点
categories:  [ROS, Linux, Net]
description: none
keywords: ros, net
---

在机器人开发过程中会遇到上下位机的通信问题，如果是两台Linux系统，我们可以使用串口、usb等连接完成信息传递，但如果他们都是ros系统，使用网线连接也十分方便，而且传输速度快。

------

经测试本文方法只适用于物理机，对虚拟机暂时无效。

## 参考网址

- [两个ROS系统进行通信（网线直连）](https://blog.csdn.net/qq_37719268/article/details/79019044)
- [Ubuntu 16.04 实现有线 无线同时用]([https://blog.csdn.net/Pipcie/article/details/79738516?utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~all~first_rank_v2~rank_v25-1-79738516.nonecase&utm_term=ubuntu%20%E5%90%8C%E6%97%B6%E4%BD%BF%E7%94%A8%E6%9C%89%E7%BA%BF%E5%92%8C%E6%97%A0%E7%BA%BF](https://blog.csdn.net/Pipcie/article/details/79738516?utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~all~first_rank_v2~rank_v25-1-79738516.nonecase&utm_term=ubuntu 同时使用有线和无线))
- [ubuntu--同时使用无线网卡和有线网卡]([https://blog.csdn.net/heroacool/article/details/51819251?utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~all~first_rank_v2~rank_v25-5-51819251.nonecase&utm_term=ubuntu%20%E5%90%8C%E6%97%B6%E4%BD%BF%E7%94%A8%E6%9C%89%E7%BA%BF%E5%92%8C%E6%97%A0%E7%BA%BF](https://blog.csdn.net/heroacool/article/details/51819251?utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~all~first_rank_v2~rank_v25-5-51819251.nonecase&utm_term=ubuntu 同时使用有线和无线))



## 实现方法

> 首先你需要买一根网线
>

将两台电脑分为主机和从机，主机可以用来开roscore，从机比较适合接收、反馈。

主机端进行如下文件的设置：

#### host文件

```sh
sudo gedit /etc/hosts
```

A电脑(主机)

```
127.0.0.1   localhost
127.0.1.1   hostname_A

IP_A      hostname_A
IP_B      hostname_B
```

B电脑(从机)

```
127.0.0.1   localhost
127.0.1.1   hostname_B

IP_B       hostname_B
IP_A       hostname_A
```



#### bashrc文件

```sh
sudo gedit ~/.bashrc
```

A电脑(主机)

```sh
export ROS_HOSTNAME=hostname_A
export ROS_MASTER_URI=http://hostname_A:11311
```

B电脑(从机)

```sh
export ROS_HOSTNAME=hostname_B
export ROS_MASTER_URI=http://hostname_A:11311
```

然后source一下



接下来，在主机和从机上分别新建一个有线网络

![主机](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/20200814213653.png)

![从机](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/20200814213715.png)

依次点击 设置->Network->Wired->Options...->IPv4 Settings，Method选择Manual，点击add，然后填入三个部分，注意IP要在同一个网段，特别要注意勾选上Requare .......那个选项。**从机**上和这个修改步骤一样，只需要把Address改为192.168.2.12，其他两个不变，如果自己不会修改的话直接拷贝我的也没问题。

> 此处有博客建议网关填成0.0.0.0，这样不会干扰到无线网的使用

之后，连接这部分的工作完成，可以试试ping一下对方

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/20200814213822.png)

#### ROS测试

连接完成后用小乌龟测试下，主机运行节点管理器 roscore，另打开一个终端运行 

```
rosrun turtlesim turtlesim_node
```

从机运行键盘发送命令的节点

```
rosrun turtlesim turtle_teleop_key
```

就可以键盘控制小海龟了~爷青回(x)

## 后续

本人当时测试时，是通过下面的方法来保证主机可以访问无线网的

首先ifconfig

删除有线默认网关

sudo route del default gw 内网网关 dev xxx

添加无线网关为默认网关
sudo route add default gw 无线网关 dev xxx

xxx为网卡名