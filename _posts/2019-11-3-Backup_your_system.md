---
layout: post
title: 从零开始的机器学习平台配置（三）备份及还原你的系统
categories: Linux
description: none
keywords: backup, Linux
---

在好不容易装好系统后。为了防止随时可能出现的系统崩溃，我们需要对系统进行备份。

------



## 1.安装盘分区

还记得那个64G的安装u盘🏇？如果它只是拿来装个系统那实在太大材小用了。可以分出来一部分空间放备份的系统。

### 安装gparted

```Shell
sudo apt-get install gparted
```

其他系统参考https://gparted.org/download.php

### 分区

![gparted](https://qq849012418.github.io/images/posts/linux/gparted.png)

如此分个区，空间要大，确保能装下整个系统。

然后把u盘把出来再插进去，新分区会自动挂载。

*注：笔者开始尝试过在windows下用diskgenius进行分区，格式也是ext4，但ubuntu无法挂载，原因未知。*



## 2.使用Timeshift创建备份

### 安装

```Shell
sudo apt-add-repository -y ppa:teejee2008/ppa
sudo apt update
sudo apt install timeshift
```

### 备份

选择刚才创建的u盘分区，并把home和root，boot等都包含进备份的需求中，创建快照即可。

![snapshot](https://qq849012418.github.io/images/posts/linux/snapshot.png)



## 3.还原操作



至此，本系列正式结束，后续视情况，会更新一些新的内容。

希望本系列对您有帮助。