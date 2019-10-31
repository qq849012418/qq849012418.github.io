```
---
layout: post
title: 从零开始的机器学习平台配置（一）Windows制作linux启动u盘并在移动固态硬盘上安装系统
categories: Linux
description: 将 Excel 里的数据使用 VBA 导入 Word 表格中。
keywords: linux, windows, ununtu, ssd
---
```

#  从零开始的机器学习平台配置（一）Windows制作linux启动u盘并在移动固态硬盘上安装系统

> ## 有一说一.JPG

背景：在笔者经历了`rm rf` 惨案之后，下定决心重新折腾一遍系统和环境，供自己以后回忆，并供感兴趣的萌新参考学习。力争易复现，少出毛病，让linux系统成为各位的重要武器。

目标：ubuntu18.04+显卡和摄像头驱动+anaconda3+opencv4.1.2+clion2019+pycharm2019+cuda10.1+pytorch+tensorrt+torch2trt+git+办公软件+系统备份

硬件条件：一个64GB的u盘，一个128GB的移动ssd。

注意：本系列只是实现方式的一种，并不适合只想快速体验下linux的用户。若本系列对您有帮助，欢迎备注转载，或与作者联系，共同学习。

## 1. 前期工作

### 系统下载

网易镜像：http://mirrors.163.com/ubuntu-releases/18.04/，选择`ubuntu-18.04.3-desktop-amd64.iso` 

### 启动盘制作工具

参考了文献1，采用UltraISO，搜索下载试用即可。

### 分区工具

采用常见的~~dick~~DiskGenius，目的是方便后续系统装完所有东西后可以备份到u盘里，先给u盘分个区。这个思想参考了文献2，但工具与其不同。

## 2.启动盘制作与分区

如图，双击iso打开，选择启动→写入硬盘镜像→写入，ok。

![image-20191029102616165](2019-10-29-ML%20on%20Linux%201%20-%20install%20OS.assets/image-20191029102616165.png)

![image-20191029104520402](2019-10-29-ML%20on%20Linux%201%20-%20install%20OS.assets/image-20191029104520402.png)

打开DiskGenius，右击新u盘，点击建立新分区，调整分区空间（分大点）

![image-20191029105617040](2019-10-29-ML%20on%20Linux%201%20-%20install%20OS.assets/image-20191029105617040.png)

在新分区上右键选择格式化，格式化成ext4格式（适用于linux系统备份）。

![image-20191029105844236](2019-10-29-ML%20on%20Linux%201%20-%20install%20OS.assets/image-20191029105844236.png)

至此，u盘预处理完毕，484很简单（

对了，硬盘不需要预处理。

## 4.BIOS的简单修改

开机进入boot，我的机械革命是按F2，其他品牌请百度。

把security boot关了

把启动引导的第一个改成u盘

保存重启，进入Linux grub界面，选择install ubuntu。

## 5.系统安装

说一些注意事项，参考文献3.

1. 安装类型选择其他选项，然后不联网，完整安装，不安装第三方软件。

2.  在分区页面设置efi（800M)，交换空间(8g)和ext4文件系统（剩下的全部容量，挂载点“/”）三个区。
3. 引导文件放置的设备直接选到移动硬盘即可，不要选到下面的分区。
4. 示意图：

## 6.再次修改BIOS（该步骤还需优化）

把硬盘中的第一位引导由windows boot manager改为ubuntu（可能会有两个ubuntu，一个是命令行，一个是图形化界面，选到图形化的那个）

## 7.可能遇到的问题

### 有nvidia显卡的笔记本可能会卡在紫屏界面进不去桌面

临时进入桌面：grub界面按e进入编辑界面， 找到“quiet splash _ _ _”,将后面的“_ _ _”删掉,空一格,然后添加`“$vt_handoff acpi_osi=linux nomodeset”`,然后按F10保存 ，即可进入系统。（这个操作是一次性的）

进入系统后，如果后面不需要跑cuda，在“软件和更新-附加驱动”中选择推荐的nvidia驱动下载安装。若找不到驱动，试一试切换下载源，然后sudo apt-get update一波。

![2019-10-31 00-46-19 的屏幕截图](2019-10-29-ML%20on%20Linux%201%20-%20install%20OS.assets/2019-10-31%2000-46-19%20%E7%9A%84%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png)

![2019-10-31 00-46-19 的屏幕截图](2019-10-29-ML%20on%20Linux%201%20-%20install%20OS.assets/2019-10-31%2000-46-19%20%E7%9A%84%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE-1572454271471.png)

装好nvidia，咱也不能放过默认的显卡驱动nouveau。根据文献4，我们做以下操作：

 **`sudo gedit /etc/modprobe.d/blacklist-nouveau.conf`**

`#添加下列两行`

`blacklist nouveau`

`options nouveau modeset=0`

`#重新生成 kernel initramfs`

`update-initramfs –u`

`#重启`

`reboot`

`#重启后验证驱动是否被禁用 如果无结果显示则表明成功禁用`

`lsmod | grep nouveau`

### 我的linux为何一直处于飞行模式，wifi无法打开

这种情况，首先回windows看一看，你在windows下wifi能不能正常工作。

**若不能，网上大多数解决方案可能不适合你!**进入BIOS，尝试重置一下(机械革命为F3重置，F4退出)，然后，很可能wifi就出现了！

否则，考虑驱动问题，参考下面的文章，此处不过多陈述。

https://blog.csdn.net/weixin_41762173/article/details/79480609

## 8.成果

emm庆祝一下

![2019-10-31 01-02-38 的屏幕截图](2019-10-29-ML%20on%20Linux%201%20-%20install%20OS.assets/2019-10-31%2001-02-38%20%E7%9A%84%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png)

万事开头难

然后中间难

最后结尾难

请期待下一篇。

Keenster，写于2019.10.29-10.31

# 参考文献

[1]: https://blog.csdn.net/neptune4751/article/details/79146885	"win10下制作Ubuntu16.04的U盘安装盘"
[2]: https://www.cnblogs.com/lemos/p/8297071.html	"linux 系统全盘备份"
[3]: https://blog.csdn.net/weixin_44001854/article/details/84896333	"给移动硬盘装上LINUX全攻略"

[4]: https://blog.csdn.net/qq805934132/article/details/82909759	"Ubuntu禁用nouveau驱动"

