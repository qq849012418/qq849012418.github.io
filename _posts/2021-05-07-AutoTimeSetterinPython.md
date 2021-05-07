---
layout: post
title: Python+计划任务的应用——跳过特定系统时间
categories:  Windows
description: none
keywords: python, time
---

最近实验室电脑一到10点自动蓝屏，未发现原因，遂通过计划任务运行python脚本，跳过该时间，暂时避开了这个问题。

------

### 参考文献

1. [Windows系统中设置Python程序定时运行](https://blog.csdn.net/gislaozhang/article/details/90724189)
2. [python脚本设置系统时间的两种方法](https://www.cnblogs.com/ding5688/p/10978265.html)
3. [Python 设置系统时间 for windows](https://www.cnblogs.com/ding5688/p/10978576.html)

### 目标

> 每天9点59将系统时间修改为10点01（跳过引发蓝屏的系统时间10点整），在10点05通过网络将系统时间校准回去。

### 方法

使用python编写两个文件

time1001am.py

```python
import os
import datetime

#设定日期
#_date = datetime.datetime.strptime("2019/05/25","%Y/%m/%d")
#设定时间为 0点30分
_time = '10.01'
#设定时间
os.system('time {}'.format(_time))
#os.system('date {}'.format(_date))
```

timefrominternet.py

```python
import os
import time
import ntplib #pip install ntplib
c = ntplib.NTPClient()
response = c.request('cn.ntp.org.cn')
ts = response.tx_time
_date = time.strftime('%Y-%m-%d',time.localtime(ts)) 
_time = time.strftime('%X',time.localtime(ts))
os.system('date {} && time {}'.format(_date,_time))
```

cn.ntp.org.cn是国家授时中心，您也可以选择其他的ntp授时网站。



打开控制面板——管理工具——任务计划程序_**创建基本任务**

【程序或脚本】文本框中填的是Python编译器的名称，例如D:\Anaconda3\python.exe，

【起始于】文本框中填的是Python编译器的目录，如果上文填了完整目录本项不填

【添加参数】文本框中填的是你的Python程序的完整路径，例如：E:/coding2/Python/TimeChangeTask/time1001am.py

添加完后，双击打开属性，由于修改时间需要管理员权限，此时需要将账户选为Administrators并勾选“使用最高权限运行”。

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/20210507123610.png)

最后，可以通过右侧栏的“运行”来测试计划任务的脚本是否正常执行。

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/20210507123736.png)

