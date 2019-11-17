---
layout: post
title: (持续更新)-Linux实用小工具安装方法合集
categories: [Linux, Tools]
description: none
keywords: Tools, Linux
---

不管你是用win还是Linux，都有很多工具是需要安装的，比如办公/娱乐/效率/系统管理等。我不打算开太多章节，只从安装的角度，说一下这些东西怎么装。对于装机时间较为充裕的朋友，可以做个参考。

------



## 1.多媒体处理类需求

### 录屏

#### SimpleScreenRecorder

屏幕上右键打开终端

添加源：

```Shell
sudo add-apt-repository ppa:maarten-baert/simplescreenrecorder
```

更新源：
```Shell
sudo apt-get update
```
安装：
```Shell
sudo apt-get install simplescreenrecorder
```

效果：![simplescreenrecorder](https://qq849012418.github.io/images/posts/tool/simplescreenrecorder.png)



## 2.多人/多系统协作需求

### 与windows互传文件

### winscp

可在win端操作，有点像ftp的感觉，简洁大方的图形界面

https://winscp.net/eng/docs/lang:chs

linux端需要做的设置（参考https://blog.csdn.net/yangwangnndd/article/details/90676948）

    首先更新源
    
    sudo apt-get update
    
    安装ssh服务
    
    sudo apt-get install openssh-server
    
    检测是否已启动
    
    ps -e | grep ssh
    看到有ssh字样，说明已启动，如果没有就手动启动
    /etc/init.d/ssh start
    
    配置ssh-server,配置文件位于/etc/ssh/sshd_config，默认端口为22，为了安全，一般自定义为其他端口，然后重启
    
    sudo /etc/init.d/ssh resart
使用注意：两台电脑需要在同一局域网下。