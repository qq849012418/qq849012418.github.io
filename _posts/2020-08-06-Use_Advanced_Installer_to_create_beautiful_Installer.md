---
layout: post
title: 使用Advanced Installer制作美观强大的安装包程序
categories:  Tools
description: none
keywords: installer, build
---

在百度上找安装包创建程序，最后发现advanced installer是功能强、易上手且无毒的，可以说是制作安装包的首选。

------



## 参考网址

- [Windows安装包制作指南——Advanced Installer的使用](https://www.cnblogs.com/dreamlofter/p/5785303.html)
- [c++获取注册表中程序的安装路径](https://blog.csdn.net/cqltbe131421/article/details/52687789)

## 下载地址

http://xiazai.zol.com.cn/detail/11/107230.shtml

## 使用案例

以本项目的一个由unity导出的windows平台程序为例，源文件如下

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/20200806212207.png)

新建AdvancedInstaller工程，此处选择通用模板

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/20200806212355.png)



进入工程，在左侧列表中，可以看到该工具功能非常强大，涵盖安装包生命周期的各种需求。

![image-20200806213124941](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20200806213124941.png)



产品信息中，右边的部分是必填的

![image-20200806212608450](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20200806212608450.png)



而我们安装包的文件，则通过资源→文件和文件夹进行添加

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/20200806213227.png)



我们需要选择安装完成后出现在开始菜单中的主程序，这个功能可以通过添加磁贴来实现

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/20200806213449.png)



注册表是一个很重要的功能，本项目需要从另一软件中通过C++代码启动此次安装的主程序，因此需要新建注册表项，和键值，编辑键值时选择“文件...”，选择主程序后，不管软件安装在什么目录，我们都可以通过访问这个键值找到程序入口。

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/20200806213636.png)

如果您有类似的需求，我们实现C++读取注册表打开程序的实现如下，供参考：

```c++
// Created by Keenster
// 2020.8.6
// keenster.cn
//
#include <Windows.h>
#include <iostream>
#include <tchar.h>
#include <string>
#define SIZE 256

std::string pvpath;//PlanningViewer安装目录
int InitUnityPath() {
	HKEY hKEY;
	LPCTSTR data_Set = _T("SOFTWARE\\JNPlanViewer");//主键值
	long ret0, ret1;//返回值
	DWORD dataType;//数据类型
	DWORD dataSize;//数据长度
	char data[SIZE] = { 0 };

	char biosVendor[SIZE];
	memset(biosVendor, 0x41, SIZE);
	ret0 = RegOpenKeyEx(HKEY_LOCAL_MACHINE, data_Set, NULL, KEY_READ, &hKEY);//打开主键
	if (ret0 != ERROR_SUCCESS)        //如果无法打开hKEY,则中止程序的执行  
	{
		printf("注册表打开失败 !!\n");
		return 1;
	}
	ret1 = RegQueryValueEx(hKEY, _T("exepath"), NULL, &dataType, (LPBYTE)&data, &dataSize);//获取数据
	if (ret1 != ERROR_SUCCESS)       //如果无法打开hKEY,则中止程序的执行  
	{
		return 1;
	}
	for (unsigned i = 0, j = 0; i < dataSize; i += 2, j++)
	{
		memcpy(biosVendor + j, data + i, 1);
	}
	printf("路径信息: %s\n", biosVendor);  
	RegCloseKey(hKEY);        // 程序结束前要关闭已经打开的 hKEY。  
	pvpath = biosVendor;
	//pvpath+="\\DPMtoHLL2019.exe";
	return 0;
}
bool OpenUnity() {
	if (pvpath.empty())
		return false;
	std::string dosstr = pvpath;
	LPCSTR lpcstr = dosstr.c_str();
	if (WinExec(lpcstr, SW_SHOW) > 31) {
		return true;
	}
	else return false;
}

int main()
{
	InitUnityPath();
	OpenUnity();
    std::cout << "Hello World!\n";
}
```



回到我们的安装包制作，下一步就是选择一个酷炫的主题了！Advanced Installer很用心地提供了各种不同风格的主题，以满足各个年龄段码农的口味（x）当然你也可以对每个主题进行再次编辑，修改页面元素，增加新的页面。

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/20200806214833.png)

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/20200806215049.png)

![image-20200806215113445](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20200806215113445.png)

![image-20200806215147668](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20200806215147668.png)

etc. 太多了



另一个可以提升安装体验的地方是安装参数，这里可以选择快速安装，加快安装速度

![image-20200806215657948](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20200806215657948.png)



最后就是制作安装包，打开部署→构建，生成安装包

![image-20200806215818659](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20200806215818659.png)



安装包效果如下，简洁大方)

![image-20200806215915123](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20200806215915123.png)



