---

layout: post
title: Unity中自动计算含mesh的子对象包围盒
categories:  [Unity, Hololens2]
description: none
keywords: unity, bounding box, mesh
---

当你在unity中加载一个prefab预置模型，并想对它添加碰撞体时，发现这个prefab的center与面片中心并不重合，这时可以通过代码，把没有MeshRenderer的空白子物体去除。

------

代码如下：

```c#
//本函数用于矫正collider到父物体中心,made by Keenster
    public void renderTrueCollider()
    {
        Transform parent = 	gameObject.transform;
        Vector3 postion = parent.position;
        Quaternion rotation = parent.rotation;
        Vector3 scale = parent.localScale;
        parent.position = Vector3.zero;
        parent.rotation = Quaternion.Euler(Vector3.zero);
        parent.localScale = Vector3.one;
 
        Collider[] colliders = parent.GetComponentsInChildren<Collider>();
        foreach (Collider child in colliders){
            DestroyImmediate(child);
        }
        Vector3 center = Vector3.zero;
        Renderer[] renders = parent.GetComponentsInChildren<Renderer>();
        foreach (Renderer child in renders){
            center += child.bounds.center;   
        }
        center /= parent.GetComponentsInChildren<Renderer>().Length;
        Bounds bounds = new Bounds(center,Vector3.zero);
        foreach (Renderer child in renders){
            bounds.Encapsulate(child.bounds);   
        }
        BoxCollider boxCollider = parent.gameObject.AddComponent<BoxCollider>();
        boxCollider.center = Vector3.zero;
        boxCollider.size = bounds.size;

        Transform child0 = parent.GetChild(0);
        child0.Translate(-bounds.center);
        parent.position = postion;
        parent.rotation = rotation;
        parent.localScale = scale;
    }
```

初始：

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/2020-04-16_234955.png)

脚本施用后：

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/2020-04-16_235041.png)

## 参考链接

[官方Guide-BoundingBox](https://microsoft.github.io/MixedRealityToolkit-Unity/Documentation/README_BoundingBox.html)

[雨松MOMO-自动计算所有子对象包围盒](http://www.xuanyusong.com/archives/3461)



本文章最后编辑于20200416