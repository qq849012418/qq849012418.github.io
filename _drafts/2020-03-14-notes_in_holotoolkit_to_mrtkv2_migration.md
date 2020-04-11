## 前言

了解到，hololens1代使用的sdk为holotookit，但hololens2变成了MRTK v2。尝试在unity2019中运行由unity2017开发的hololens1工程，报错'EditorUserBuildSettings' does not contain a definition for 'wsaGenerateReferenceProjects'，查阅[这个链接](https://github.com/microsoft/MixedRealityToolkit/issues/219) 了解到，HoloToolkit最高只支持到unity2018，因此，尝试着把这一工程升级为MRTK v2

微软官方的升级介绍

https://microsoft.github.io/MixedRealityToolkit-Unity/Documentation/HTKToMRTKPortingGuide.html#setup-and-configuration

https://docs.microsoft.com/en-us/windows/mixed-reality/mrtk-porting-guide

某博客的一个升级例子

https://medium.com/@dongyoonpark/bringing-the-periodic-table-of-the-elements-app-to-hololens-2-with-mrtk-v2-a6e3d8362158

（这个坑后面再填）







cad to unity

传统方法：cad (.catpart)→ 3dmax/c4d/maya等进行读取转换→unity（.fbx）

优点：可详细调参，经过第三方软件的优化，unity读取很快

缺点：步骤比较多，第三方软件模型读取耗时

可直接转换的产品：

PiXYZhttps://www.pixyz-software.com/download/

![](https://forum.unity.com/proxy.php?image=https%3A%2F%2Fwww.pixyz-software.com%2Fwp-content%2Fuploads%2F2017%2F06%2FPR_plugin2.jpg&hash=c3731c225f25cd756c23cf3ab35761ed)

Unity插件的老版本破解版：https://www.liangchan.net/soft/14046.html

PiXYZ Studio&Batchhttp://www.smzy.com/smzy/down402532.html

支持Python脚本，已有现成脚本可以实现文件夹监控和批处理https://gitlab.com/pixyz_co/support/batch/folder-watcher

西门子的例子：

![](https://www.pixyz-software.com/documentations/html/2019.2/studio/lib/NewItem238.png)

模型实测：某数控打印机模型：157子物体+95032三角面片：7S

Tesla-concept模型:

CATPART 27子物体+173310三角面片：39S

STP 50S

由于



CADLink 

Tridify Convert [https://www.tridify.com/convert](https://link.zhihu.com/?target=https%3A//www.tridify.com/convert)

CAD Exchanger 只能进行模型转换，unity插件有介绍页，官方称正在开发



