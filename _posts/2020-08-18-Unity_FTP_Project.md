---
layout: post
title: 【Unity开发】热更新实战[1]-简易FTP服务器搭建与Unity使用
categories:  Unity
description: none
keywords: unity
---

在unity应用开发中，有时需要通过网络向远程地址上传下载文件，这里有两种文件传输方式——FTP和HTTP，本篇尝试使用了FTP的方法。

------

文件传输协议（英文：File Transfer Protocol，缩写：FTP）是用于在网络上进行文件传输的一套标准协议，使用客户/服务器模式。它属于网络传输协议的应用层。

![image-20200913155805847](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20200913155805847.png)

## 参考文献

- [如何分析Ftp.ListDirectoryDetails给出的一行信息](https://blog.csdn.net/fdc2017/article/details/102608590)
- [Unity Ftp相关操作](https://www.cnblogs.com/jietian331/p/12841262.html)

## 建立过程

Android：使用ES文件浏览器直接创建(网络=>远程管理器)

![image-20200913160009216](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20200913160009216.png)

Windows：控制面板打开FTP服务并配置绑定IP和用户权限

![image-20200913155821625](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20200913155821625.png)

## unity ftp工具类

请参考参考文献2，或者系列的下一篇[完整代码](http://keenster.cn/2020/08/19/Unity_FTP_Project_2_file_updation/)

## 访问效果

![image-20200913155859784](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20200913155859784.png)

## 注意

在hololens等uwp平台上，没有ftp的库，开发前需要调研好。

