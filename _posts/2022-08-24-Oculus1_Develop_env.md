---
layout: post
title: Oculus quest2 开发实战（一）环境配置与打包测试
categories:  VR
description: none
keywords: Meta, oculus, unity
---

实验室有一台quest2，一直是休闲游戏用机，本文记录了它的Unity开发环境配置，供快速上手。（本教程只适用windows操作系统。

------



### 配置步骤

此处假设您的设备已经经过基本的调试，能够安装第三方游戏等应用。

**STEP1. 从[meta官网](https://store.facebook.com/quest/setup/)下载oculus的推流软件，这样可以通过数据线直接真机debug unity工程。**

![image-20220824104600954](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20220824104600954.png)

若meta被墙无法载入，可以用我下载好的https://disk.pku.edu.cn:443/link/F48471163B295D80452624AB89F94A29 (有效期限：2027-09-30 23:59)，下同。

**STEP2. 从oculus开发者网站下载oculus developer hub，这样就可以管理设备文件，且上传apk安装你开发好的程序。**

![image-20220824105123251](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20220824105123251.png)

**STEP3. 通过unityhub安装Unity并配置好Android编译工具（以unity2021为例）**

![image-20220824105401739](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20220824105401739.png)

**STEP4. （可选项）安装android studio，以及从[github](https://github.com/orgs/oculus-samples/repositories)下载unity oculus sample工程。**

### Unity内的注意事项

在测试时，我使用了[这个工程](https://codeload.github.com/Georgy256/Endoscopy/zip/refs/heads/main)，它展示了内窥镜手术室的操作环境，代码比较整洁。

1. 调试oculus，应导入“XR Plugin Management”包，并在projectsettings中选中oculus

   ![image-20220824111429006](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20220824111429006.png)

2. playersettings中其他需要注意的设置

   ![image-20220824111733441](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20220824111733441.png)

   ![image-20220824111800002](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20220824111800002.png)

3. 打包apk时，若遇到Could not resolve all artifacts for configuration ‘:classpath‘的问题，打开$Unity安装目录$\Editor\Data\PlaybackEngines\AndroidPlayer\Tools\GradleTemplates\baseProjectTemplate.gradle，增加国内源

   ```java
   mavenCentral()
   google()
   maven{url 'http://maven.aliyun.com/nexus/content/groups/public/'}
   jcenter()
   ```

   ![image-20220824112322563](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20220824112322563.png)

### 串流测试效果

![image-20220824112713419](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20220824112713419.png)