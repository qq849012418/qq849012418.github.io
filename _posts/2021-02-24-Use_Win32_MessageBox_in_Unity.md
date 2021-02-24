---
layout: post
title: 在Unity(C#)中快速调用Win32风格的MessageBox
categories:  Unity
description: none
keywords: Unity,  UI
---

在程序中，有时需要对用户的操作进行二次确认，或进行警告和提示。在本篇中，记录了一种比较简单的方案。

------

### 参考文献

[Win32API之MessageBox](https://blog.csdn.net/james_1234/article/details/8672985)

### 介绍

MessageBox 是Windows[系统库](http://wenwen.soso.com/z/Search.e?sp=S系统库&ch=w.search.yjjlink&cid=w.search.yjjlink) user32.dll 的一个导出函数，用于显示一个提示消息对话框，其原型定义如下 ：

```C
int MessageBox(

HWND hWnd, // handle to owner window

LPCTSTR lpText, // text in message box

LPCTSTR lpCaption, // message box title

UINT uType // message box style

);
```

该函数有四个参数，第一个是消息框所有者[窗口句柄](http://wenwen.soso.com/z/Search.e?sp=S窗口句柄&ch=w.search.yjjlink&cid=w.search.yjjlink)，可以是NULL，第二个是消息框的文本内容，第三个是消息框标题，第四个参数是消息框样式（按钮和图标）。

CWnd类对MessageBox进行了封装，对其第一个[参数传递](http://wenwen.soso.com/z/Search.e?sp=S参数传递&ch=w.search.yjjlink&cid=w.search.yjjlink)了CWnd类的[成员变量](http://wenwen.soso.com/z/Search.e?sp=S成员变量&ch=w.search.yjjlink&cid=w.search.yjjlink) m_hWnd，因此，调用CWnd[类的成员函数](http://wenwen.soso.com/z/Search.e?sp=S类的成员函数&ch=w.search.yjjlink&cid=w.search.yjjlink)MessageBox时，不能使用第一个参数，并且，最后两个参数也有默认值。

消息框样式根据输入数字的不同，对应关系如下：

> 0、确定按钮； _MB_OK=@0x0
>
> 1、确定、取消按钮； _MB_OKCANCEL=@0x1
>
> 2、终止、重试、忽略按钮；_MB_ABORTRETRYIGNORE=@0x2
>
> 3、是、否、取消按钮；_MB_YESNOCANCEL=@0x3
>
> 4、是、否按钮；_MB_YESNO=@0x4
>
> 5、重试取消钮；_MB_RETRYCANCEL=@0x5
>
> 6、终止、重试、继续      0x00000006（需声明API才能使用）

函数返回值也是int，对应关系如下：

> 1=确定钮；  IDOK
>
> 2=取消钮；  IDCANCEL
>
> 3=终止钮；  IDABORT
>
> 4=重试钮；  IDRETRY
>
> 5=忽略钮；IDIGNORE
>
> 6=是钮；IDYES
>
> 7=否钮；IDNO  

### 实战

新建Messagebox.cs

```c#
using System;
using System.Runtime.InteropServices;

public class Messagebox
{
    [DllImport("User32.dll", SetLastError = true, ThrowOnUnmappableChar = true, CharSet = CharSet.Auto)]
    public static extern int MessageBox(IntPtr handle, String message, String title, int type);
}
```

调用示例

```C#
Messagebox.MessageBox(IntPtr.Zero, "内容", "标题",  0);
```

或类似

```c#
if(Messagebox.MessageBox(IntPtr.Zero, "内容", "标题",  1) == 2)//按了取消
{
	return;
}
// do sth
```

