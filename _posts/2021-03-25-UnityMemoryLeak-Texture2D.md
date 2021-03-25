---
layout: post
title: Unity Texture2D引起的内存泄漏排查+Profiler内存分析
categories:  Unity
description: none
keywords: Texture2D, Profiler, Memory
---

记录一次内存泄漏问题，代码规范很重要~

------

### 参考文献

1. [Unity3d Texture2d Memery Leak 贴图导致的内存泄漏](https://www.douban.com/note/748126260/?type=like)
2. [【经验】使用Profiler工具分析内存占用情况](https://blog.csdn.net/yangyy753/article/details/47025205)



### 问题发现

本地两个Unity客户端通过服务器实现图像传输和渲染，运行时内存占用一直上升，导致系统崩溃。

于是乎，将其中一个客户端在Editor模式下运行，打开Profiler，再次复现问题，对Memory跟踪采样，定位到了一直在增加的Scene Memory，而其中Texture2D应该是罪魁祸首。

![image-20210325102137071](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20210325102137071.png)

### 问题分析

在接收端中频繁地使用了texture的直接赋值，而上一帧的没有直接被回收，变成了垃圾……

修改后的源代码，在赋值前增加了一个清理操作：

```csharp
    private void OnReceiveRGBD(SLAMPacket packet)
    {
        int width = packet.Width;
        int height = packet.Height;
        byte[] RGBbyte = packet.RGBtex.ToByteArray();
        byte[] Dbyte = packet.Dtex.ToByteArray();

        Texture2D RGBtex = new Texture2D(width, height);
        Texture2D Dtex = new Texture2D(width, height);
        ImageConversion.LoadImage(RGBtex, RGBbyte, false);
        ImageConversion.LoadImage(Dtex, Dbyte, false);
        //回收上一帧的texture
        Destroy(rgb2show.texture);
        Destroy(d2show.texture);
        //
        rgb2show.texture = RGBtex;
        d2show.texture = Dtex;
    }
```

在发送端中，由于在转换操作中使用了临时的Texture2D，但这种一条龙操作导致后续没有任何代码能够引用到这个Texture2D，造成泄漏。原代码段为：

```csharp
private Texture2D TextureToTexture2D(Texture texture, TextureFormat format);//略，可参考https://blog.csdn.net/weixin_42565127/article/details/106321570
packet.RGBtex = ByteString.CopyFrom(TextureToTexture2D(rgb2show.texture, TextureFormat.RGB24).EncodeToJPG());//造成内存泄漏

```

修改为下方的代码，逻辑更加清晰，也能够及时清理掉不用的Texture2D：

```csharp
Texture2D rgbtex2d = TextureToTexture2D(rgb2show.texture, TextureFormat.RGB24);
packet.RGBtex = ByteString.CopyFrom(rgbtex2d.EncodeToJPG());
//...
Destroy(rgbtex2d);
```

再次运行，发现发送和接收端的内存泄漏问题已解除，Scene Memory降下来了。

![image-20210325103819770](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20210325103819770.png)

### 拓展阅读

在此次分析中使用了Profiler，可以进一步记录一下内存中其他部分的分析方法，下面的内容参考了参考文献2.

首先打开Profiler选择Memory选项，在游戏运行的某一帧查看Detailed选项数据（*Simple模式的数据很直观，可以知道内存大体被哪部分占用了，网上也有很多相关介绍，我就不再啰嗦了*），如下图所示：

![img](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/20150723170500499)

选中后，unity会自动获取这一帧的内存占用数据项，主要分为：Other、Assets、BuiltinResources、Scene Memory、NotSaved这五大部分，下面我们就来一一分析。

#### Other

![img](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/20150723170626976)

记录数据项很多，篇幅时间有限，我们就专挑占用大小排行榜靠前的几项来详细分析吧。

System.ExecutableAndDlls：系统可执行程序和DLL，是只读的内存，用来执行所有的脚本和DLL引用。不同平台和不同硬件得到的值会不一样，可以通过修改Player Setting的Stripping Level来调节大小。
Ricky：我试着修改了一下Stripping Level似乎没什么改变，感觉虽占用内存大但不会影响游戏运行。我们暂时忽略它吧(- -)!

GfxClientDevice：GFX（图形加速\图形加速器\显卡 (GraphicsForce Express)）客户端设备。
Ricky：虽占用较大内存，但这也是必备项，没办法优化。继续忽略吧(- -)!!

ManagedHeap.UsedSize：托管堆使用大小。
Ricky：重点监控对象，不要让它超过20MB，否则可能会有性能问题！

ShaderLab：Unity自带的着色器语言工具相关资源。
Ricky：这个东西大家都比较熟悉了，忽略它吧。

SerializedFile：序列化文件，把显示中的Prefab、Atlas和metadata等资源加载进内存。
Ricky：重点监控对象，这里就是你要监控的哪些预设在序列化中在内存中占用大小，根据需求进行优化。

PersistentManager.Remapper：持久化数据重映射管理相关
Ricky：与持久化数据相关，比如AssetBundle之类的。注意监控相关的文件。

ManagedHeap.ReservedUnusedSize：托管堆预留不使用内存大小，只由Mono使用。
Ricky：无法优化。

#### Assets

#### ![img](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/20150723170958766)

- **Texture2D：** 2D贴图及纹理。

![img](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/20150723171218585)

Ricky：重点优化对象，有以下几点可以优化：

许多贴图采用的Format格式是ARGB 32 bit所以保真度很高但占用的内存也很大。在不失真的前提下，适当压缩贴图，使用ARGB 16 bit就会减少一倍，如果继续Android采用RGBA Compressed ETC2 8 bits（iOS采用RGBA Compressed PVRTC 4 bits），又可以再减少一倍。把不需要透贴但有alpha通道的贴图，全都转换格式Android：RGB Compressed ETC 4 bits，iOS：RGB Compressed PVRTC 4 bits。
当加载一个新的Prefab或贴图，不及时回收，它就会永驻在内存中，就算切换场景也不会销毁。应该确定物体不再使用或长时间不使用就先把物体制空(null)，然后调用Resources.UnloadUnusedAssets()，才能真正释放内存。
有大量空白的图集贴图，可以用TexturePacker等工具进行优化或考虑合并到其他图集中。

- AudioManager：音频管理器
  Ricky：随着音频文件的增多而增大。

- AudioClip：音效及声音文件
  Ricky：重点优化对象，播放时长较长的音乐文件需要进行压缩成.mp3或.ogg格式，时长较短的音效文件可以使用.wav 或.aiff格式。

- Cubemap：立方图纹理
  Ricky：这个一般在天空盒中比较常见，我也不知道如何优化这个。。。

- Mesh：模型网格
  Ricky：主要检查是否有重复的资源，还有尽量减少点面数。

#### Scene Memory

![img](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/20150723171140415)

- **Mesh：**场景中使用的网格模型

  *Ricky：*注意网格模型的点面数，能合并的mesh尽量合并。

#### Builtin Resources

![img](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/20150723171306601)

* Ricky：这些都是Unity的一些内部资源，对于项目内存没有什么分析价值，所以我就暂时不对其进行分析了。

#### Profiler内存重点关注优化项目

1）ManagedHeap.UsedSize: 移动游戏建议不要超过20MB.

2）SerializedFile: 通过异步加载(LoadFromCache、WWW等)的时候留下的序列化文件,可监视是否被卸载.

3）WebStream: 通过异步WWW下载的资源文件在内存中的解压版本，比SerializedFile大几倍或几十倍，不过我们现在项目中展示没有。

4）Texture2D: 重点检查是否有重复资源和超大Memory是否需要压缩等.

5）AnimationClip: 重点检查是否有重复资源.

6）Mesh： 重点检查是否有重复资源.

#### 项目中可能遇到的问题

##### 1.Device.Present:

1）GPU的presentdevice确实非常耗时，一般出现在使用了非常复杂的shader.

2）GPU运行的非常快，而由于Vsync的原因，使得它需要等待较长的时间.

3）同样是Vsync的原因，但其他线程非常耗时，所以导致该等待时间很长，比如：过量AssetBundle加载时容易出现该问题.

4）Shader.CreateGPUProgram:Shader在runtime阶段（非预加载）会出现卡顿(华为K3V2芯片).

5）StackTraceUtility.PostprocessStacktrace()和StackTraceUtility.ExtractStackTrace(): 一般是由Debug.Log或类似API造成，游戏发布后需将Debug API进行屏蔽。

##### 2.Overhead:

1）一般情况为Vsync所致.

2）通常出现在Android设备上.

##### 3.GC.Collect:

原因：

1）代码分配内存过量(恶性的)

2）一定时间间隔由系统调用(良性的).

占用时间：

1）与现有Garbage size相关

2）与剩余内存使用颗粒相关（比如场景物件过多，利用率低的情况下，GC释放后需要做内存重排)

##### 4.GarbageCollectAssetsProfile:

1）引擎在执行UnloadUnusedAssets操作（该操作是比较耗时的,建议在切场景的时候进行）。

2）尽可能地避免使用Unity内建GUI，避免GUI.Repaint过渡GCAllow.

3）if(other.tag == a.tag)改为other.CompareTag(a.tag).因为other.tag为产生180B的GC Allow.

4）少用foreach，因为每次foreach为产生一个enumerator(约16B的内存分配)，尽量改为for.

5）Lambda表达式，使用不当会产生内存泄漏.

##### 5.尽量少用LINQ：

1）部分功能无法在某些平台使用.

2）会分配大量GC Allow.

##### 6.控制StartCoroutine的次数：

1）开启一个Coroutine(协程)，至少分配37B的内存.

2）Coroutine类的实例 -> 21B.

3）Enumerator -> 16B.

##### 7.使用StringBuilder替代字符串直接连接.

##### 8.缓存组件:

1）每次GetComponent均会分配一定的GC Allow.

2）每次Object.name都会分配39B的堆内存.

