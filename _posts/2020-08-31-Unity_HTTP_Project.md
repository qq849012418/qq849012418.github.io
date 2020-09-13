---
layout: post
title: 【Unity开发】热更新实战[3]-使用HTTP文件服务器在Hololens2上管理下载模型列表
categories:  [Hololens2, Unity]
description: none
keywords: http, unity
---

接上篇，尝试建立http文件服务器来管理服务器与本地的文件信息，实现了Hololens2上的手动下载。

------

本文禁止转载。

## 服务器搭建工具

HfS（https://www.rejetto.com/hfs/）

[此处提供汉化版链接](https://jbox.sjtu.edu.cn/l/L04xuK)

界面如下，请在根目录下建立一个文件夹，虚实均可

![image-20200903204324964](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20200903204324964.png)

如果要存储assetbundle文件，请将该文件夹命名为`ABModel`

ip是可以设置的，如果电脑有公网ip，尽量使用公网ip，否则，请保证服务器ip和hololens2在同一个局域网下。

部署完成，即可在浏览器中查看文件夹，点击upload可上传模型，理论上，每台电脑都有上传下载的权限

![image-20200903204729571](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20200903204729571.png)

## unity端关键代码

仅WindowsPC可用的联网方式：*HttpWebRequest*（class in System.Net）

Hololens2可用的联网方式：*UnityWebRequest*（class in UnityEngine.Networking）

#### HttpDldFile.cs

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.IO;
using System.Net;
using System.Collections;
using UnityEngine.Networking;
using UnityEngine;
using System.Text.RegularExpressions;
using System.Media;
//Created by Keenster 20200902
namespace ConsoleTest
{
    public class HttpDldFile
    {
        /// <summary>
        /// Http方式下载文件
        /// </summary>
        /// <param name="url">http地址</param>
        /// <param name="localfile">本地文件</param>
        /// <returns></returns>
        /// 
        public static string serverip = "http://10.161.166.80";
        UnityWebRequest webRequest;
       
        AudioSource SoundPlayer;

        public static void InitIP()
        {
            if (PlayerPrefs.HasKey("ServerIP"))
            {
                serverip = "http://" + PlayerPrefs.GetString("ServerIP");
            }
        }

        #region 模型下载
        public IEnumerator unityDownload(string url, string localfile)
        {
            //发送请求
            webRequest = UnityWebRequest.Get(serverip+url);
            webRequest.timeout = 30;//设置超时，若webRequest.SendWebRequest()连接超时会返回，且isNetworkError为true
            yield return webRequest.SendWebRequest();

            if (webRequest.isNetworkError)
            {
                Debug.Log("Download Error:" + webRequest.error);
                PlaySound("fail");
            }
            else
            {
                //获取二进制数据

                var File = webRequest.downloadHandler.data;

                //创建文件写入对象

                FileStream nFile = new FileStream(localfile, FileMode.Create);

                //写入数据

                nFile.Write(File, 0, File.Length);

                nFile.Close();
                PlaySound("complete");
            }
        }

        //普通下载方法（已弃用）
        public bool Download(string url, string localfile)
        {
            bool flag = false;
            long startPosition = 0; // 上次下载的文件起始位置
            FileStream writeStream; // 写入本地文件流对象

            // 判断要下载的文件夹是否存在
            if (File.Exists(localfile))
            {

                writeStream = File.OpenWrite(localfile);             // 存在则打开要下载的文件
                startPosition = writeStream.Length;                  // 获取已经下载的长度
                writeStream.Seek(startPosition, SeekOrigin.Current); // 本地文件写入位置定位
            }
            else
            {
                writeStream = new FileStream(localfile, FileMode.Create);// 文件不保存创建一个文件
                startPosition = 0;
            }


            try
            {
                HttpWebRequest myRequest = (HttpWebRequest)HttpWebRequest.Create(url);// 打开网络连接

                if (startPosition > 0)
                {
                    myRequest.AddRange((int)startPosition);// 设置Range值,与上面的writeStream.Seek用意相同,是为了定义远程文件读取位置
                }


                Stream readStream = myRequest.GetResponse().GetResponseStream();// 向服务器请求,获得服务器的回应数据流


                byte[] btArray = new byte[512];// 定义一个字节数据,用来向readStream读取内容和向writeStream写入内容
                int contentSize = readStream.Read(btArray, 0, btArray.Length);// 向远程文件读第一次

                while (contentSize > 0)// 如果读取长度大于零则继续读
                {
                    writeStream.Write(btArray, 0, contentSize);// 写入本地文件
                    contentSize = readStream.Read(btArray, 0, btArray.Length);// 继续向远程文件读取
                }

                //关闭流
                writeStream.Close();
                readStream.Close();

                flag = true;        //返回true下载成功
            }
            catch (Exception)
            {
                writeStream.Close();
                flag = false;       //返回false下载失败
            }

            return flag;
        }
        #endregion

        #region 文件列表信息获取,已转移到myUITreeControl
        public IEnumerator Get(string url)
    {
        UnityWebRequest webRequest = UnityWebRequest.Get(HttpDldFile.serverip+url);

        yield return webRequest.SendWebRequest();
        //异常处理，很多博文用了error!=null这是错误的，请看下文其他属性部分
        if (webRequest.isHttpError || webRequest.isNetworkError)
            Debug.Log(webRequest.error);
        else
        {
            //Debug.Log(webRequest.downloadHandler.text);
            string response = webRequest.downloadHandler.text;
            MatchCollection matchlist = Regex.Matches(response, @"<div class=""item-link"">[\s]+<a href=""(.+)"">", RegexOptions.Compiled);
            foreach (Match m in matchlist)
            {
				//do your own work
            }
        }

    }
        #endregion

        //播放音效
        public void PlaySound(string name)
        {
            AudioClip clip = Resources.Load<AudioClip>(name);
            SoundPlayer = GameObject.Find("MainStateManager").GetComponent<AudioSource>();
            SoundPlayer.clip = clip;
            SoundPlayer.PlayOneShot(clip);
        }
        /// <summary>
        /// 获取下载进度
        /// </summary>
        /// <returns></returns>
        public float GetProcess()
        {
            if (webRequest != null)
            {
                return webRequest.downloadProgress;
            }
            return 0;
        }
        /// <summary>
        /// 获取当前下载内容长度
        /// </summary>
        /// <returns></returns>
        public long GetCurrentLength()
        {
            if (webRequest != null)
            {
                return (long)webRequest.downloadedBytes;
            }
            return 0;
        }
    }
}

```

