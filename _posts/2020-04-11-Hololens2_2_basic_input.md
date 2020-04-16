---

layout: post
title: Hololens2开发 (二) 基本输入探索（语音、手势、眼动）
categories:  [AR, Hololens2]
description: none
keywords: hololens2, augmented reality,mrtk v2
---

本系列建立在课题组项目中，可能并不是很全面，最全的教程还是要啃官网。

------

## Package导入设置

需要导入的Package有：

Microsoft.MixedReality.Toolkit.Unity.Foundation.2.3.0（基本）

Microsoft.MixedReality.Toolkit.Unity.Examples.2.3.0（方便改项目参考）

Microsoft.MixedReality.Toolkit.Unity.Tools.2.3.0（这个先前忘记了，想要导出Appx包的话必须要加这个包）

## 语音输入

Hololens2中加入了普通话识别，这时不开发语音怎么行？😄

1. 在MixedRealityInputSystem中的Speech->Speech Commands里输入指令集合
2. 在需要控制的物体上挂载Speech Input Handler，以及包含控制函数的脚本，在handler面板中将指令词与控制函数一一对应，并调节其他参数
3. Unity中可直接控制，模拟器上把语言改成中文即可控制

## 眼动控制

眼动控制修改的主要有三块地方——

1. 修改MixedRealityToolkit中的InputSimulation/InputActions/Pointer为Eyetrackingdemo中使用的profile，或者直接用Assets/MixedRealityToolkit.Examples/Demos/EyeTracking/General/Profiles/EyeTrackingDemoConfigurationProfile.asset里的EyeTrackingDemoConfigurationProfile
2. 将EyeTrackingDemos ManagerComponents.prefab添加到场景中，该预制体中包含了部分语音控制和一个鼠标Cursor
3. 选择接收眼动控制的物体Targets，挂载脚本

Target上可以挂的脚本：

自定义动作：EyeTrackingTarget

想让物体被视线选中拖动（如进度条）：MoveObjByEyeGaze

被盯住的反馈（比如变个色）：OnLookAtShowHoverFeedback





## 调试中遇到的问题

appx导出时Reference rewriter: Error: field `System.Numerics.Vector3 Windows.Perception.People.HandMeshVertex::Position` doesn't exist in target framework....





本文章最后编辑于20200304