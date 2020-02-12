---

layout: post
title: linux的cuda突然报cuda initialization failure with error 100的解决方案
categories:  [Linux, Debug]
description: none
keywords: clion, linux, cuda, nvidia, ubuntu
---

VIDIA-SMI has failed because it couldn't communicate with the NVIDIA driver.
我就关了个电脑。

------



### 个人解决方案如下，可能不适合于所有情况

#### 首先，在终端中输入nvcc -V命令

![终端图1](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/2020-02-12%2013-45-17%20%E7%9A%84%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png)

如图 ，表示cuda正常安装，因此考虑nvidia驱动问题。

#### 

#### 接下来，输入nvidia-smi，没有出现驱动信息，考虑重装驱动。

#### 打开“软件与更新”，进入“附加驱动”，发现驱动被改成nvidia_driver_418了，重新改到430，并点击 应用更改，进行安装。

![软件和更新](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/2020-02-12%2014-07-23%20%E7%9A%84%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png)

#### 最后，输入nvidia-smi，可以看到出现了配置信息

![终端](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/2020-02-12%2014-50-01%20%E7%9A%84%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png)

重新运行程序，问题解决





本文首次使用腾讯云图床，看看能否提高加载速度