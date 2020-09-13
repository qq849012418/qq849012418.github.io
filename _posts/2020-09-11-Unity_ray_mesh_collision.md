---
layout: post
title: 【Unity开发】Mesh创建 & 射线碰撞检测，返回所有碰撞结果
categories:  Unity
description: none
keywords: 碰撞检测, 三角面片
---

本篇实现了简化面片txt在unity中的读取，网格创建，碰撞器添加，射线检测。

------

## 参考文献

- [Unity的Mesh合并](https://blog.csdn.net/One_Piece_Fu/article/details/81454030)
- [Unity 代码生成mesh要点总结](https://blog.csdn.net/qq_39185530/article/details/89518262)
- [Unity3D 动态创建Mesh（一）](https://www.cnblogs.com/kyokuhuang/p/4191169.html)
- [Unity3D中使用mesh collider和box collider的区别](https://blog.csdn.net/qq_36991505/article/details/91516409)
- [unity-射线拾取物体的三角面片+显示碰撞点的坐标](https://www.pianshen.com/article/3847739271/)

## 前置知识

### Mesh

在Unity3D中创建一个Object，在Inspector可以看到其中含有MeshFilter、MeshRenderer组件。

MeshFilter含有一个Public成员 Mesh。

在Mesh中存储着三维模型的数据：vertices（顶点数据数组Vector3[]）、triangles（三角形顶点索引数组，int[])、normals（法线向量数组，Vector3[])、uv（纹理坐标数组，Vector2[])。

一般情况下，Mesh都是由三角面片构成，但在立体空间中，三角形有两面，**正面可见，背面不可见**。三角形的渲染顺序与三角形的正面法线呈**左手螺旋定则**。

这就决定了，如果我们需要渲染如下一个正方形面，那么就需要保证组成这个正方形的两个小三角形的正面法线都是**指向屏幕外**的。

![img](https://images0.cnblogs.com/blog/707625/201412/291120353411053.png)

在程序中的顶点顺序为，三角形1: 0->3->1，三角形2: 0->2->3 。

### MeshCollider与Boxcollider

MeshCollider的精度非常高，但是渲染需要消耗巨大资源。
BoxCollider的误差随高度变化而变化，但是资源消耗极少。

在我们的项目中，模型非常复杂，但我们追求的是检测的准确，因此考虑**在面片简化后**，使用meshcollider。

### Unity中的射线检测

unity中的射线检测在Physics类中，Raycast()和RaycastAll()区别：Raycast()只检测当前游戏物体，RaycastAll()检测前方所有游戏物体(返回一个数组)

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/%E5%B0%84%E7%BA%BF%E8%BF%90%E8%A1%8C%E5%8E%9F%E7%90%86.png)

## mesh创建代码

```c#
//made by Keenster
public void BuildMesh(string pointPath,string triPath)
    {
        string[] temp = pointPath.Split('_')[0].Split('/');
        string meshname = temp[temp.Length - 1];
        GameObject meshobj = new GameObject(meshname);
        meshobj.transform.SetParent(this.transform);
        MeshFilter mf = meshobj.AddComponent<MeshFilter>();
        MeshRenderer mr = meshobj.AddComponent<MeshRenderer>();
        List<Vector3> points = new List<Vector3>();
        points = ReadData(pointPath);//因读取的文件类型而异，自行编写
        mf.mesh.Clear();
        mf.mesh.SetVertices(points);
        int[] triangles = ReadTriangle2Int(triPath);//因读取的文件类型而异，自行编写，注意：如果索引不是按规范创建的，可以每行顺着读取一次，倒着读取一次，生成的mesh就可以两面接受碰撞检测
        mf.mesh.triangles = triangles;
        mf.mesh.RecalculateBounds();
        mf.mesh.RecalculateNormals();
        mf.mesh.RecalculateTangents();
        mr.material.SetColor(0, Color.white);
        mr.material.name = meshname;
        meshobj.AddComponent<MeshCollider>().sharedMesh = mf.mesh;
    }
```

![image-20200913175341619](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20200913175341619.png)

## 射线检测类

```c#
using System.Collections;
using System.Collections.Generic;
using System.Text;
using UnityEngine;
// Keenster 20200911
// Inspiration: https://www.pianshen.com/article/3847739271/
public class RayMeshCollision : MonoBehaviour
{
    public float maxDistance=10000;//最大检测距离
    System.Diagnostics.Stopwatch sw;
    void Start()
    {
        sw = new System.Diagnostics.Stopwatch();
    }
    // Update is called once per frame
    void Update()
    {
        if (Input.GetMouseButtonUp(0))
        {
            sw.Restart();
            //RaycastHit hit;
            Ray ray = Camera.main.ScreenPointToRay(Input.mousePosition);
            //if (Physics.Raycast(ray, out hit, 10000))//仅检测第一个
            RaycastHit[] hits = Physics.RaycastAll(ray, maxDistance);//返回所有射线碰撞
            sw.Stop();
            if (hits.Length>0)
            {
                StringBuilder sb = new StringBuilder(20);
                foreach(RaycastHit hit in hits)
                {
                    //拾取三角面前提是物体含有一个MeshCollider碰撞器
                    MeshCollider collider = hit.collider as MeshCollider;
                    if (collider == null || collider.sharedMesh == null)
                        continue;
                    //获取碰撞器所在物体的Mesh网格
                    Mesh mesh0 = collider.sharedMesh;
                    sb.Append(mesh0.name);
                    sb.Append(", ");
                }
                
                
                Debug.Log(hits[0].point+" mesh:"+sb.ToString());//输出第一次碰撞的零件以及所有碰撞零件的名称
                Debug.Log(sw.Elapsed);//输出从检测开始到生成结果所需的时间
            }
        }
    }
}
```

![image-20200913175404150](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20200913175404150.png)

速度还是可以的。