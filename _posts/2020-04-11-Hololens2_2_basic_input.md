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



## 眼动控制

Target上可以挂的脚本：

自定义动作：EyeTrackingTarget

想让物体跟着动（如进度条）：MoveObjByEyeGaze

被盯住的反馈：OnLookAtShowHoverFeedback



## 调试中遇到的问题

appx导出时Reference rewriter: Error: field `System.Numerics.Vector3 Windows.Perception.People.HandMeshVertex::Position` doesn't exist in target framework....





本文章最后编辑于20200304