---
layout: post
title: 让Linux终端和Windows PowerShell走代理
categories:  [Net, Linux, Windows]
description: none
keywords: vpn, v2ray
---

当你在用vcpkg下载包或者在终端运行git clone时，有时网速特别慢。vpn类软件虽可实现全局代理，但似乎对终端或powershell无效，如何给这些系统窗口设置代理呢？

------

## 前言

代理是通过客户端与服务端通信,传输服务端能够访问到的资源文件,再由服务端客户端通信返回给客户端,从而间接访问服务端能访问的资源。

## 参考文献

[WIN10给powershell设置全局代理](https://blog.csdn.net/weixin_44120025/article/details/110950434)

[Linux 让终端走代理的几种方法](https://zhuanlan.zhihu.com/p/46973701)

## 主要步骤

### 0.一个代理软件

我使用的代理服务是相对价廉物美的[心阶云](https://www.xinjiecloud.vip/auth/register?code=Dho7xaD8sx58xop9cPjaU2wiGt4wMl6o)，在其用户界面提供了支持的客户端，在Windows端我选用了[V2RayN](https://mirrors.ohmy.cat/Windows/v2rayN.zip)，在Linux端我探索了[QV2Ray](https://qv2ray.net/getting-started/)。

![image-20210119091720199](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20210119091720199.png)

### 1.Windows端的实现

1. 在代理软件下方查看域名端口

![image-20210119092510075](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20210119092510075.png)

2. 打开powershell（在目标路径下按住shift，鼠标右键，选择“在此处打开powershell窗口”即可）
3. 依次输入如下两行命令`$env:HTTP_PROXY="http://域名:端口"`和`$env:HTTPS_PROXY="http://域名:端口`。比如：`$env:HTTP_PROXY="http://127.0.0.1:10809"`即可。

### 2.Linux端的实现

在linux中可以使用环境变量，许多命令行程序（如 `curl` 和 `wget`）会使用 `<PROTOCOL>_PROXY` 环境变量提供的代理。

首先`sudo gedit ~/.bashrc`，在最后添加

```bash
export ALL_PROXY=socks5://127.0.0.1:xxxx（软件中查看的端口号）
```

再`source ~/.bashrc`，即可实现终端走代理。

