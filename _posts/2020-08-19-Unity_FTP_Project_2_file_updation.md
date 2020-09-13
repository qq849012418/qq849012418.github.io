---
layout: post
title: 【Unity开发】热更新实战[2]-使用HashTable管理文件更新列表
categories:  Unity
description: none
keywords: pool, unity
---

在项目开发中要实现一个结构树的功能，其中涉及一个初始化节点的过程。如果从resource中直接加载，数十万节点会让设备卡死，即使从别的gameobject中复制实例化，也较慢。PoolManager作为一个对象池插件，其功能全面，使用简单，值得尝试。

------

**1.  哈希表(HashTable)简述**

 在.NET Framework中，Hashtable是System.Collections命名空间提供的一个容器，用于处理和表现类似keyvalue的键值对，其中key通常可用来快速查找，同时key是区分大小写；value用于存储对应于key的值。Hashtable中keyvalue键值对均为object类型，所以Hashtable可以支持任何类型的keyvalue键值对.

**2. 什么情况下使用\**哈希表\****

（1）某些数据会被高频率查询
（2）数据量大
（3）查询字段包含字符串类型
（4）数据类型不唯一



## 参考文献

- [C#中哈希表(HashTable)的用法详解](https://www.cnblogs.com/xpvincent/archive/2013/01/15/2860841.html)

- [C#:Hashtable的序列化（保存和读取）](https://www.cnblogs.com/lelf2005/archive/2008/10/07/1305411.html)



由此可见，添加数据时Hashtable快。频繁调用数据时Dictionary快。