---
layout: post
title: 配置用于无网络免激活的Unity2019.4.0环境
categories:  Unity
description: none
keywords: Unity,  Crack
---

MR开发需要用到Unity环境，本篇讲解了如何破解海外版Unity2019.4.0，仅供学习交流情景。

------

### 参考文献

[Unity Hub破解方法（需要nodejs环境）](https://www.cnblogs.com/unity3ds/p/11112784.html)

### 资源下载

JD云盘：https://jbox.sjtu.edu.cn/l/Ou6ope

若需尝试其他版本的Unity2019.4，可挂vpn到[https://unity3d.com/unity/whats-new/2019.4.0](https://unity3d.com/unity/whats-new/2019.4.0)下载其他版本

### 步骤

1. 安装unity

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/20210205105956.png)

2. 不要启动，进入Unity目录，删除Editor→Data→Resources→Licensing目录

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/20210205110436.png)

3. 运行Patcher，选择editor目录，create license， patch，replace

   ![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/20210205111105.png)

4. 此时双击Unity，应出现以下界面

   ![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/20210205111219.png)

5. 将经过处理的unityhub文件夹解压，运行hub，切换到“安装”页面，点击“定位”，选择editor的目录

   ![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/20210205111728.png)

   之后就可以通过hub运行unity工程，可根据需要自行安装Support模块