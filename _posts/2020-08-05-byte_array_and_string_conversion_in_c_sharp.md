---
layout: post
title: 解决C#中Byte.toString()后文本框中只显示System.Byte[]的问题
categories:  C#
description: none
keywords: unity, string
---

toString用习惯了，觉得啥格式都能toString，直到遇见了Byte[].

------



## 参考网址

- [为什么C# 中 文本框里有 System.Byte[] 字符](https://zhidao.baidu.com/question/178171538.html)

## 解决方法

byte[] a=New Byte[3];
你可能是想把数组的数据显bai示到文本框中，结果你只是在类型后调用了toString(),而数据没有读出来，
下面可能是你的代码
byte[] a=new byte[3];
this.textBox1.Text = a.ToString();
这样就会在文本框里显示System.Byte[],
你可以这样改
byte[] a=new byte[3];
for (int i = 0; i < a.Length; i++)
{
this.textBox1.Text += a[i].ToString("X2") + " ";
}
就可以显示a[]中的各个值了，已经转换成了16进制两位显示！