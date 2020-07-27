---
layout: post
title: 含vuforia9的unity2018.4工程正确升级到2019.4
categories:  [AR, Unity]
description: none
keywords: vuforia, ar, unity, migration
---

unity工程升级过程都比较玄学，使用前先保存。

------

## 前言

在unity2018中，vuforia作为一个可选的unity安装组件，是可以轻松安装的。但是到了unity2019阶段，这个工具包被放在了asset store上，您需要先下载asset store上并不完整的vuforia engine，然后被动地在package.json中加了一句

"scopedRegistries":[

...

"url":"https://registry.packages.developer.vuforia.com"

...

]

意思是会从这个网站上下载最新的vuforia完整包，but——不知是否因为网络原因，我始终无法访问这个网站。unity也会直接报

`unable to connect http://registry.packages.developer.vuforia.com/`

的错误，然后其他package的加载也受到了影响。

如何解决这个问题？


## 参考网址

- [unity Vuforia SDK的开发升级避坑经验](https://blog.csdn.net/u013354943/article/details/105527168)

## 步骤

首先备份您的工程！

直接点击这个链接[packages.developer.vuforia.com](packages.developer.vuforia.com)进入到真正的package下载地址，手动下载最新的tgz包

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/20200727140431.png)

删除工程Library、Packages文件夹，重新导入工程。若提示升级asset database到 version2，同意即可

进入unity主界面后，安装asset store中的非完整vuforia engine

删除Assets下的vuforia文件夹（如果您对这个文件夹里的代码做过修改，记得备份）

打开Window->Package Manager，在project中安装任一版本的vuforia engine ar，如果没有，点击左上角加号，选择from tarball，安装刚下载的tgz包

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/20200727141132.png)

此时再重开一下unity，如果没有报导入错误，工程就已经成功更新