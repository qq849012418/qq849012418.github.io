---
layout: post
title: 将python脚本打包为exe——pyinstaller的使用
categories:  [Tools, Python]
description: none
keywords: exe, python, pyinstaller
---

有时我们需要通过其他语言调用我们写的python脚本，或者将我们编写的python程序放在其他没有python环境的电脑上调用，这个时候，我们就可以将python打包成可执行文件。

------
### 前言

在某项目中，需要使用到命令行调用UnityEditor，因此使用了某python工程进行实时日志输出，[unity-realtime-log](https://github.com/qq849012418/unity_realtime_log)。

但并不是所有机器上都有python环境，如何让该部分代码永久可用？

询问了师姐，发现了神奇的pyinstaller，遂学习之。



### 参考文献

[Python程序打包成.exe(史上最全面讲解)](https://www.cnblogs.com/ZFJ1094038955/p/13375740.html?utm_source=tuicool)

[pyinstaller参数介绍以及总结](https://blog.csdn.net/bearstarx/article/details/81054134)



### 主要步骤

#### 1.pyinstaller安装

```shell
pip install pyinstaller
```

![image-20201122215259636](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20201122215259636.png)

#### 2.pyinstaller的主要参数介绍（from参考文献2，少量修改）

| -F, –onefile            | 打包成一个单个文件                                           |
| ----------------------- | :----------------------------------------------------------- |
| -D, –onedir             | 打包多个文件，在dist中生成很多依赖文件，适合以框架形式编写工具代码，程序运行速度快 |
| -K, –tk                 | 在部署时包含 TCL/TK                                          |
| -a, –ascii              | 不包含编码.在支持Unicode的python版本上默认包含所有的编码.    |
| -d, –debug              | 产生debug版本的可执行文件                                    |
| -w,–windowed,–noconsole | 使用Windows子系统执行.当程序启动的时候不会打开命令行(只对Windows有效) |
| -c,–nowindowed,–console | 使用控制台子系统执行(默认)(只对Windows有效)pyinstaller -c xxxx.pypyinstaller xxxx.py --console |
| -s,–strip               | 可执行文件和共享库将run through strip.注意Cygwin的strip往往使普通的win32 Dll无法使用. |
| -X, –upx                | 如果有UPX安装(执行Configure.py时检测),会压缩执行文件(Windows系统中的DLL也会)(参见note) |
| -o DIR, –out=DIR        | 指定spec文件的生成目录,如果没有指定,而且当前目录是PyInstaller的根目录,会自动创建一个用于输出(spec和生成的可执行文件)的目录.如果没有指定,而当前目录不是PyInstaller的根目录,则会输出到当前的目录下. |
| -p DIR, –path=DIR       | 设置导入路径(和使用PYTHONPATH效果相似).可以用路径分割符(Windows使用分号,Linux使用冒号)分割,指定多个目录.也可以使用多个-p参数来设置多个导入路径，让pyinstaller自己去找程序需要的资源 |
| –icon=<FILE.ICO>        | 将file.ico添加为可执行文件的资源(只对Windows系统有效)，改变程序的图标 pyinstaller -i ico路径 xxxxx.py |
| –icon=<FILE.EXE,N>      | 将file.exe的第n个图标添加为可执行文件的资源(只对Windows系统有效) |
| -v FILE, –version=FILE  | 将verfile作为可执行文件的版本资源(只对Windows系统有效)       |
| -n NAME, –name=NAME     | 可选的项目(产生的spec的)名字.如果省略,第一个脚本的主文件名将作为spec的名字 |

#### 3.实践

由于我的工程中python文件unity_realtime_log.py是程序入口，同时有依赖文件tail.py等(-p)，需要进行参数输入(-w不能用)，需要打包成单个文件（-F），因此使用的打包参数为

```shell
pyinstaller -F unity_realtime_log.py -p tail.py -p __init__.py
```

![5eaf5cbbc22a965329a9eadefb5ade35](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/5eaf5cbbc22a965329a9eadefb5ade35.jpg)

程序会生成build和dist两个文件夹，dist中的就是我们需要的exe文件，终于可以不用bat调用了。

![image-20201122220815033](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20201122220815033.png)

后续可尝试将文件修改为自定义的图标，本篇暂不讨论。