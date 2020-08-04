---
layout: post
title: 【Unity开发】在unity中创建一个可多选的打开or保存文件对话窗口
categories:  [Unity]
description: none
keywords: unity, windows, open&save, file
---

项目中的一个需要导出windows平台的部分。需要增加一个ui，可以由用户从对话框中选择某种文件序列，返回这些文件的路径名称。

------



## 参考网址

- [Unity | 打开文件对话框批量选择文件](https://blog.csdn.net/weixin_39766005/article/details/103705178)

## 实现源码及注释

### myOpenFile.cs

这个源文件用来与系统的OpenFileName建立联系，通过一个结构体来放置一些基础的变量

```c#
//created by Keenster 20200802
using UnityEngine;
using System.Collections;
using System;
using System.Runtime.InteropServices;

[StructLayout(LayoutKind.Sequential, CharSet = CharSet.Auto)]
//如果这里写成class而不是struct，最后只能获得单文件，不能获得所有文件
//参考链接：https://blog.csdn.net/weixin_39766005/article/details/103705178
public struct OpenFileName
{
    public int structSize;
    public IntPtr dlgOwner;
    public IntPtr instance;
    public String filter;
    public String customFilter;
    public int maxCustFilter;
    public int filterIndex;
    public String file;
    public int maxFile;
    public String fileTitle;
    public int maxFileTitle;
    public String initialDir;
    public String title;
    public int flags;
    public short fileOffset;
    public short fileExtension;
    public String defExt;
    public IntPtr custData;
    public IntPtr hook;
    public String templateName;
    public IntPtr reservedPtr;
    public int reservedInt;
    public int flagsEx;

}

public class LocalDialog
{
    //链接指定系统函数       打开文件对话框
    [DllImport("Comdlg32.dll", SetLastError = true, ThrowOnUnmappableChar = true, CharSet = CharSet.Auto)]
    public static extern bool GetOpenFileName([In, Out] OpenFileName ofn);
    public static bool GetOFN([In, Out] OpenFileName ofn)
    {
        return GetOpenFileName(ofn);
    }

    //链接指定系统函数        另存为对话框
    [DllImport("Comdlg32.dll", SetLastError = true, ThrowOnUnmappableChar = true, CharSet = CharSet.Auto)]
    public static extern bool GetSaveFileName([In, Out] OpenFileName ofn);
    public static bool GetSFN([In, Out] OpenFileName ofn)
    {
        return GetSaveFileName(ofn);
    }
}
```

### myOpenFileTest.cs

使用OnGUI的绘制方式，在界面上增加一个按钮，初始化对话框功能

```c#
//created by Keenster 20200802
using UnityEngine;
using System.Collections;
using System.Runtime.InteropServices;
using System;
using System.Text.RegularExpressions;

public class myOpenFileTest : MonoBehaviour
{
    void OnGUI()
    {
        if (GUI.Button(new Rect(10, 10, 100, 50), "选择路径文件"))
        {
            OpenFileName openFileName = new OpenFileName();
            openFileName.structSize = Marshal.SizeOf(openFileName);
            openFileName.filter = "路径点文件(*.txt)\0*.txt";
            openFileName.file = new string(new char[1024]);
            openFileName.maxFile = openFileName.file.Length;
            openFileName.fileTitle = new string(new char[64]);
            openFileName.maxFileTitle = openFileName.fileTitle.Length;
            openFileName.initialDir = Application.streamingAssetsPath.Replace('/', '\\');//默认路径
            openFileName.title = "选择多个路径文件";
            openFileName.defExt = "TXT";
            //0x00080000 | ==  OFN_EXPLORER |对于旧风格对话框，目录 和文件字符串是被空格分隔的，函数为带有空格的文件名使用短文件名
            openFileName.flags = 0x00080000 | 0x00001000 | 0x00000800 | 0x00000200 | 0x00000008 ;

            if (LocalDialog.GetOpenFileName(openFileName))
            {
                string[] SplitStr = { "\0" };
                string[] strs = openFileName.file.Split(SplitStr, StringSplitOptions.RemoveEmptyEntries);
                for (int i = 0; i < strs.Length; i++)
                {
                    Debug.Log(strs[i]);
                }
                if (strs.Length > 0)
                {
                    //DO SOMETHING
                    
                }
            }
        }
    }
}
```

这里面需要注意的是，如果openFileName.flags取消选择0x00080000，默认为旧格式（但大家应该不会去用这个，太丑了），多选时分割方式与新格式的对话框是不一样的

本文方法分割出来的字符串，第一个元素是文件夹名，后面各个元素是纯文件名，中间全部用NULL字符串分割，将他们分离时使用"\0"作为分隔字符。