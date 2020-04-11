---

layout: post
title: Hololens2 初探秘手册
categories:  [AR, Debug]
description: none
keywords: hololens2, augmented reality
---

微软Hololens 2是微软公司研发的混合现实头显。虽然现在还没货，但是可以研究一下。

------

##### 硬件部分扫盲

1. 后方旋钮调节头带长度

2. 左侧亮度

3. 右侧音量

4. 使用时间2-3h，待机2周

5. 重量566g

6. 应尽量不触摸面罩

   

##### 使用-基操

1. 强制重启：长按10s
2. 睡眠：3min
3. 开始手势：双手——请伸出你的手，掌心朝向自己。 你将看到**开始图标**显示在手腕内侧。 使用另一只手点击此图标。单手——伸出一只手，掌心朝向自己。 你将看到**开始图标**显示在手腕内侧，注视图标并捏合食指拇指。 



#### 重要文档归类及调试注意事项

##### 官方对Hololens2和1的新手教程文档页面（非编程，仅为原理和使用）

https://docs.microsoft.com/zh-cn/hololens/hololens2-hardware



##### 官方Hololens2emulator使用方法

https://docs.microsoft.com/en-us/windows/mixed-reality/using-the-hololens-emulator

安装重点1：首先需要在控制面板-程序与功能-启用或关闭Windows功能-勾选Hyper-V，若看不到Hyper-V，可能是因为系统是家庭版本。解决方法是将以下代码保存为.cmd格式运行，之后按提示重启电脑。

```shell
pushd "%~dp0"
dir /b %SystemRoot%\servicing\Packages\*Hyper-V*.mum >hyper-v.txt
for /f %%i in ('findstr /i . hyper-v.txt 2^>nul') do dism /online /norestart /add-package:"%SystemRoot%\servicing\Packages\%%i"
del hyper-v.txt
Dism /online /enable-feature /featurename:Microsoft-Hyper-V-
```

重点2：BIOS中确认如果有以下的项，将其激活。

```
-Hardware-assisted virtualization
-Second Level Address Translation (SLAT)
-Hardware-based Data Execution Prevention (DEP)
```

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/hololenstest1.png)

emulator里的系统菜单可用F2打开。



如何安装程序包：右侧进入Panel，里面有APP的部分，安装时注意要安装合适的Dependencies，比如这个模拟器好像是x64的，装其他比如ARM的框架会失败。



如何将unity中的scene导出到hololens2 emulator 运行：

首先打开build settings，按数字顺序进行设置，我的unity版本是2019.1.0a12. 注意，第一次点击“Universial Windows Platform”时，可能会需要提前安装 UnitySetup-Universal-Windows-Platform-Support-for-Editor-2019.1.0a12.exe 这个东西，否则不会出这个界面。然后要注意第一步设置一定要选择合适的版本，下面的那几个对号要打上，否则在vs中打开时不会提示使用simulator运行。

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/2020-03-04_174725.png)

build出来之后，文件结构是这个样子的

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/2020-03-04_175110.png)

使用vs打开sln，然后就可以看到模拟器啦，直接点击模拟器，编译通过后就可以在上面跑程序啦

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/2020-03-04_174945.png)

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/2020-03-04_174424.png)

##### 官方unity插件-MixedRealityToolkit（MRTK）的github页

https://github.com/microsoft/MixedRealityToolkit-Unity



##### 官方关于MRTK Examples Hub的github介绍页

https://github.com/microsoft/MixedRealityToolkit-Unity/blob/mrtk_development/Documentation/README_ExampleHub.md

下图是官方的appbundle在emulator中的运行情况

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/hololenstest02.png)

不知道怎么调视角（



#### 开发踩坑

##### unity导包后报错 Feature 'xxx' cannot be used because it is not part of the C# 4.0

https://blog.csdn.net/HarryXYC/article/details/89462647

在Unity选择【Edit】——【Project Settings】——【Player】——【Other Settings】——【Configuration】大项下的【Scripting Runtime Version】选项选择【.NET 4.x Equivalent】即可解决问题。

##### vs 错误 将环境变量 " TraceDesignTime" 设置为 true 解决方案

https://ourcodeworld.com/articles/read/414/visual-studio-2017-ide0006-compiler-error-encountered-while-loading-the-project





#### 备用优秀教程

##### Unity3D项目移植UWP平台踩坑经验总结

https://zhuanlan.zhihu.com/p/63424435



本文章最后编辑于20200304