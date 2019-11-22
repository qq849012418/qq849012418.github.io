---
layout: post
title: opencv3代码移植到opencv4环境下需要修改什么？
categories:  OpenCV
description: none
keywords: opencv3, opencv4, RM
---

RM中的一个小任务，把19年的代码移植到opencv4环境下，其内涵也就是opencv3to4，考虑到随着opencv4的广泛应用，在其他情景下也可能会遇到这样的移植工作，故简单记录遇到的问题及解决方案，供后来者参考。

------



#### cmake部分不说了，比较容易，把opencv3 改成 opencv4即可



#### ‘IplImage’ does not name a type

添加

```c
#include <opencv2/imgproc/imgproc_c.h>
```



‘CV_BGR2GRAY’ was not declared in this scope

CV开头的一大堆宏定义都报错

添加

```c++
#include <opencv2/imgproc/types_c.h>
```

到这里就能编译成功了

![snapshot](https://qq849012418.github.io/images/posts/opencv/run.png)