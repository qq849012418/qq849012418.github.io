---
layout: wiki
title: 常用正则表达式（regular expression）
categories: Debug
description: 静态分析神器
keywords: debug, regular expression
---

### 前言

本篇主要记录Keenster在遇到一些小需求的时候用到的表达式，原理也会逐步介绍



### 删除代码行号

在从博客园等地粘贴代码时，有时会连着行号一起粘下来，这时就需要在vs里面（或者其他常见IDE）中进行批量删除，使用Ctrl+H进行替换。

^\s*\d+	这个是单纯数字的行号

^\s*\d+\.	这个是数字后有一个点的行号 

![image-20200818164805470](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20200818164805470.png)

