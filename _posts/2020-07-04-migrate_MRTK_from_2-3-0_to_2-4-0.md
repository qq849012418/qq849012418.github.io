---
layout: post
title: [Hololoen2 MRTK开发]升级MRTK2.3.0到2.4.0
categories:  AR, Hololens2
description: none
keywords: Unity,  mixed reality toolkit
---

MRTK2.3.0到2.4.0属于爆炸性更新，微软不得不弄了一个升级工具用来修改一些控件的脚本引用，但升级过程还是比较容易出问题的，所以记录一下。

------

## 前言

MRTK2.3.0到2.4.0属于爆炸性更新，微软不得不弄了一个升级工具用来修改一些控件的脚本引用，但升级过程还是比较容易出问题的，所以记录一下。

## 参考网址

- [【Hololens2-Unity开发】如何升级MRTK2.3.0到2.4.0](https://blog.csdn.net/v2morrow/article/details/106490129)
- [官方的例子](https://microsoft.github.io/MixedRealityToolkit-Unity/version/releases/2.4.0/Documentation/Updating.html)

## 步骤

MRTK2.4下载地址 https://github.com/microsoft/MixedRealityToolkit-Unity/releases/tag/v2.4.0

1.关闭unity，删除MRTK安装目录下的以下文件，连.meta文件一同删除
MixedRealityToolkit
MixedRealityToolkit.Examples
MixedRealityToolkit.Extensions
MixedRealityToolkit.Providers
MixedRealityToolkit.SDK
MixedRealityToolkit.Services
MixedRealityToolkit.Staging
MixedRealityToolkit.Tools

2.删除项目目录下的Library文件夹

3.打开unity，导入2.4版本MRTK，先导入Foundation，再导入其他的，TOOLS等

4.关闭unity删除Library文件夹

5.打开unity，删除所有场景内的 MixedRealityToolkit和MixedRealityPlayspace。然后在上边栏的MixedRealityToolkit里点击Add to Scene and Configure

6.执行一次 MixedRealityToolkit -> Utilities -> Update -> Controller Mapping Profiles

7.运行migration tool检查升级（ MixedRealityToolkit -> Utilities ->Migration Window）

## 注意事项

1、为了防止一些奇怪的错误，最好把.vs ，manifest.json,  QCAR等文件夹都删一删

2、进入unity 的player settings，把变掉的.net的版本改回来

![image-20200704102157404](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20200704102157404.png)

3、最后如果还有少量的代码错误提示，可能是之前对MRTK脚本做过修改，新版本没有修改所致，灵活改动吧

4、最后，需要重新把自己的配置文件加入到mixedrealitytoolkit控件里面

## 最终

![image-20200704102939792](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20200704102939792.png)

编译成功~不过这波把我之前很多ui给改成原始的了，要重新弄了，😣