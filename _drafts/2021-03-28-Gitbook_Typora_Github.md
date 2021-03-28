---
layout: post
title: Gitbook - 适合电子说明文档的Markdown框架
categories:  Blog
description: none
keywords: Gitbook, Typora, Github
---

利用周末闲暇时间，简单配置了下gitbook，用来记录leetcode刷题之路。

------

### 参考文献

1. [使用gitbook时解决“if (cb) cb.apply(this, arguments)”错误](https://yimouleng.com/2020/09/28/if-cb-cb-applythis-arguments-error/)
2. [【经验】使用Profiler工具分析内存占用情况](https://blog.csdn.net/yangyy753/article/details/47025205)



## 步骤

### 1、安装Gitbook

首先确保已经安装node.js

```powershell
npm install -g gitbook-cli
gitbook -V
```

如果第二行代码出现了`TypeError: cb.apply is not a function`的报错，请打开相关报错文件`polyfills.js`，注释掉62-64行调用代码

```javascript
//fs.stat = statFix(fs.stat)
//fs.fstat = statFix(fs.fstat)
//fs.lstat = statFix(fs.lstat)
```

