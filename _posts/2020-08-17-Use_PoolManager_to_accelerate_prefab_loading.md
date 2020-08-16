---
layout: post
title: 【Unity开发】使用PoolManager解决Instantiate大量prefab效率低的问题
categories:  Unity
description: none
keywords: pool, unity
---

在项目开发中要实现一个结构树的功能，其中涉及一个初始化节点的过程。如果从resource中直接加载，数十万节点会让设备卡死，即使从别的gameobject中复制实例化，也较慢。PoolManager作为一个对象池插件，其功能全面，使用简单，值得尝试。

------

一个**对象池**包含一组已经初始化过且可以使用的对象，而可以在有需求时创建和销毁对象。池的用户可以从池子中取得对象，对其进行操作处理，并在不需要时归还给池子而非直接销毁它。这是一种特殊的工厂对象。

## 使用场景

1. **资源受限**的, 不需要可伸缩性的环境(cpu\内存等物理资源有限): cpu性能不够强劲, 内存比较紧张, 垃圾收集, 内存抖动会造成比较大的影响, 需要提高内存管理效率, 响应性比吞吐量更为重要;
2. **数量受限**的, 比如数据库连接;
3. **创建成本高的对象**, 可以考虑是否池化, 比较常见的有线程池（ThreadPoolExecutor）, 字节数组池等。

## 参考文献

- [Unity3D研究院之初探PoolManager插件（七十四）](http://www.xuanyusong.com/archives/2974)
- 《Unity插件宝典》
- [对象池模式](https://www.jianshu.com/p/cb81031814f0)

![image-20200817025348736](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20200817025348736.png)

## 插件使用方法

在[这里](https://download.csdn.net/download/qq_39555106/12717997)下载最新的7.0插件，导入工程

在工程中新建一个empty对象，在Asset搜索框中搜索SpawnPool.cs并加入到该对象下

![image-20200817023958369](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20200817023958369.png)

如上图所示，先点+号，并从resource中拖进来一个prefab到pool里，设置好pool的名称以及参数，参数释义如下：

PoolName：缓存池的唯一名称。

MatchPoolScale：勾选后实例化的游戏对象的缩放比例将全是1，不勾选择用Prefab默认的。

MachPool Layer：勾选后实例化的游戏对象的Layer将用Prefab默认的。

Don’t Reparent：勾选后实例化的对象将没有父节点，通通在最上层，建议不要勾选。

Don’t Destroy On Load：这个就不用我解释了吧？切换场景不施放。

Pre-Prefab Pool Options ：缓存池列表，意思就是缓存列表里面可以放各种类型的Prefab。右边有个 “+”按钮点击就添加每个类型的Prefab了，后面会介绍脚本怎么动态添加。

prefab：可以直接把工程里的Prefab直接拖进来。

preloadAmount:缓存池这个Prefab的最大保存数量。

preloadTime：如果都选表示缓存池所有的gameobject可以“异步”加载。

preloadFrames：每几帧加载一组。

preloadDelay：延迟多就开始加载。

limitInstance：是否开始实例的限制功能。

limit Amount：限制缓存池里最大的Prefab的数量，它和上面的preloadAmount是有冲突的，如果同时开启则以limitAmout为准。

limitFIFO：如果我们限制了缓存池里面只能有10个Prefab，如果不勾选它，那么你拿第11个的时候就会返回null。如果勾选它在取第11个的时候他会返回给你前10个里最不常用的那个。

cullDespawend：是否开启缓存池智能自动清理模式。

cull Above：缓存池自动清理，但是始终保留几个对象不清理。

cull Delay：每过多久执行一遍自动清理，单位是秒。

cullMaxPerPass：每次自动清理几个游戏对象。

虽然里面很多可以不勾，但是请时刻注意您的Profiler，有没有内存溢出。建议还是限制一下加载数量，不用的时候清理一下。

![image-20200817020131302](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20200817020131302.png)

在代码中修改设置请参考文献1，现在来说一说在代码中如何用这个代替Instantiate

#### 初始化阶段

```c#
SpawnPool Spool=null;
```

。。。

```c#
if (Spool == null)
{
	Spool = PoolManager.Pools["treenodepool"];
}
```

#### 实例化模型

```c#
//GameObject tt = (GameObject) Instantiate(Resources.Load("TreeNodeTemplate"));
GameObject tt = Spool.Spawn("TreeNodeTemplate").gameObject;
```

实测效果非常快！