---
layout: post
title: 【Hololens2 MRSV开发】三方视角[1]-在Unity2019.4中配置MixedReality-SpectatorView
categories:  [AR, Hololens2]
description: none
keywords: 第三人称, 空间锚点
---

一个项目外的小尝试，实现多平台共享AR内容。

------
Spectator View（以下简称MRSV）是一种增强现实产品，可从辅助设备查看HoloLens体验。Spectator View具有多种配置，并支持从拍摄快速原型到制作主题演示的各种场景。

### 参考文献

- [github仓库地址](https://github.com/microsoft/MixedReality-SpectatorView)
- [在unity2019使用ARCore](https://zhuanlan.zhihu.com/p/104301078)
- [HowToSetUpSpectatorView](https://github.com/GooDtoLivE/HowToSetUpSpectatorView)

### 搭建方法

#### 1、新建一个unity空工程，并在工程目录下输入git init

因为后续会用到github

#### 2、添加子模块

工程目录下输入`git submodule add https://github.com/microsoft/MixedReality-SpectatorView.git sv`

#### 3、切换sv到稳定版本

```shell
cd sv

git pull

git checkout release/1.1.0
```

#### 4、添加sv的依赖包

管理员身份进powershell，导航到sv

运行 tools/Scripts/SetupRepository.bat命令，自动下载[MixedRealityToolkit-Unity](https://github.com/microsoft/MixedRealityToolkit-Unity)、[Azure-Spatial-Anchors-Samples](https://github.com/Azure/azure-spatial-anchors-samples) 、 [ARCore-Unity-SDK](https://github.com/google-ar/arcore-unity-sdk) 以上模块是当前项目中所包含的子模块

> Note: 如果下载过程中出现了中断或者未完成的情况，请直接在该处下载[MixedRealityToolkit-Unity](https://github.com/microsoft/MixedRealityToolkit-Unity/tree/b7dbeb6e9b14355ed176a388ddac3e4a4a1946f9)、[Azure-Spatial-Anchors-Samples](https://github.com/Azure/azure-spatial-anchors-samples/tree/61a1e390cb09ab7544da9304460f5b88e331a3ef) 、 [ARCore-Unity-SDK](https://github.com/google-ar/arcore-unity-sdk/tree/05829541bdf24c6dcbbeb5976dc1673c6a482471)将下载的文件解压后复制到工程下的sv\external\模块中的文件

#### 5、设置依赖关系，创建符号链接

管理员身份进powershell，导航到sv

运行tools\Scripts\AddDependencies.bat -AssetPath "Assets" -SVPath "sv"

#### 6、进入unity设置

如果在这里报很多与ARCore相关的错误，比如The type or namespace name 'NetworkBehaviour' could not be found...原因是unity版本太高了，相关库过时了，在unity中打开window=>package manager，安装这两个组件

![image-20200915191129131](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20200915191129131.png)

![image-20200915191201802](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20200915191201802.png)

然后就不报错了，基本配置完成

