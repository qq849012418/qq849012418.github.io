---
layout: post
title: 【Hololoen2 MRTK开发】使用LeapMotion(by ultraleap)在unity中模拟手势追踪
categories:  [AR, Hololens2, LeapMotion]
description: none
keywords: Unity,  mixed reality toolkit, leapmotion, Ultraleap
---

MRTK2.4.0优化了手势追踪功能，并增加了Ultraleap控制器的支持，今天我们来看一下如何用这个功能来让交互调试不需要模拟器或烧程序。

------

## 前言
![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/LeapHandsGif3.gif)
来自官方的酷炫效果，虽然我感觉这个图是卖家秀不是买家秀。

## 参考网址

- [How to Configure Leap Motion (by Ultraleap) Hand Tracking in MRTK](https://github.com/microsoft/MixedRealityToolkit-Unity/blob/mrtk_development/Documentation/CrossPlatform/LeapMotionMRTK.md)

## 步骤

MRTK2.4 下载地址 https://github.com/microsoft/MixedRealityToolkit-Unity/releases/tag/v2.4.0

leapmotion sdk（4.0.0） 下载地址 https://developer.leapmotion.com/releases/?category=orion

leapmotion unity asset（4.5.0） 下载地址 https://developer.leapmotion.com/unity

个人unity版本 2018.4.3

#### 个人的步骤与官方稍有不同，且比官方更加精简

##### 1、下载上述包（下过的忽略），安装sdk，并将 [Core Assets](https://developer.leapmotion.com/unity)导入unity工程

##### 2、在Assets里新建一个空的csc.rsp，依次点击图中的两个configure

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/20200705092519.png)

##### 3、点击您场景的MixedRealityToolkit game object中，在配置里的input栏目中，增加一个input data provider

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/LeapAddDataProvider.png)

##### 4、克隆一下，以便自己改详细设置

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/20200705094153.png)

##### 5、测试

要测试MRTK是否识别出Leap Motion Unity核心资产的存在，请打开位于MRTK / Examples / Demos / HandTracking /中的LeapMotionHandTrackingExample场景，然后按play。如果识别出Leap Motion Unity Core资产，则场景信息面板上将出现绿色消息。如果未识别出Leap Motion Unity核心资产，则会出现红色消息。

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/20200705104534.png)



ps：感觉leapmotion的定位还不是很稳定……

## 注意事项

1、研究了一下LeapMotionHandTrackingExample。由于leapmotion默认只支持editor和standalone，在测试前需要在build settings里面把platform切换为Standalone，这样在playersettings里才会出现LEAPMOTIONCORE_PRESENT，才会显示绿色。如果平台是UWP，就会变红。

解决方法：直接在UWP下面的playersettings里加上就行

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/20200705125553.png)

