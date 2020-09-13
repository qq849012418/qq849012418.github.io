---
layout: post
title: 【Unity开发】热更新实战[2]-使用HashTable管理文件更新列表
categories:  Unity
description: none
keywords: hashtable, unity
---

接上篇，尝试建立哈希表来管理服务器与本地的文件信息，每次若检测到远端有新的更新则自动下载。

------

**1.  哈希表(HashTable)简述**

 在.NET Framework中，Hashtable是System.Collections命名空间提供的一个容器，用于处理和表现类似keyvalue的键值对，其中key通常可用来快速查找，同时key是区分大小写；value用于存储对应于key的值。Hashtable中keyvalue键值对均为object类型，所以Hashtable可以支持任何类型的keyvalue键值对.

**2. 什么情况下使用哈希表**

（1）某些数据会被高频率查询
（2）数据量大
（3）查询字段包含字符串类型
（4）数据类型不唯一

**3. 与dictionary格式的对比**

添加数据时Hashtable快。频繁调用数据时Dictionary快。


## 参考文献

- [C#中哈希表(HashTable)的用法详解](https://www.cnblogs.com/xpvincent/archive/2013/01/15/2860841.html)

- [C#:Hashtable的序列化（保存和读取）](https://www.cnblogs.com/lelf2005/archive/2008/10/07/1305411.html)



## unity代码

本代码中的工具类适用于android es文件管理器建立的FTP，若是windows建立的FTP，可参考[上一篇文章](http://keenster.cn/2020/08/18/Unity_FTP_Project/)的参考文献2

#### UpLoadFiles.cs

```c#

using System;
using System.Collections;
using System.Collections.Generic;
using System.Globalization;
using System.IO;
using System.Linq;
using System.Net;
using System.Security.Cryptography;
using System.Text;
using UnityEngine;
using UnityEngine.UI;
//Keenster 20200818
namespace UpLoad
{

    class UpLoadFiles
    {
        private static string FTPCONSTR = "ftp://10.161.66.187:3721/";//FTP的服务器地址，格式为ftp://192.168.1.234/。ip地址和端口换成自己的，这些建议写在配置文件中，方便修改
        private static string FTPUSERNAME = "";//我的FTP服务器的用户名
        private static string FTPPASSWORD = "";//我的FTP服务器的密码
        public static float uploadProgress;//上传进度
        public static float downloadProgress;//下载进度

        public static Hashtable localFileList, ftpFileList, updateList;

        #region 本地文件上传到FTP服务器

        /// <summary>
        /// 文件上传到ftp
        /// </summary>
        /// <param name="ftpPath">ftp的文件路径</param>
        /// <param name="localPath">本地的文件目录</param>
        /// <param name="fileName">可重命名文件</param>
        /// <param name="pb">进度条</param>
        /// <returns></returns>
        public static bool UploadFiles(string ftpPath, string localPath, string fileName)
        {
            //path = "ftp://" + UserUtil.serverip + path;
            string erroinfo = "";
            float percent = 0;
            FileInfo f = new FileInfo(localPath);
            localPath = localPath.Replace("\\", "/");
            bool b = MakeDir(ftpPath);
            if (b == false)
            {
                return false;
            }
            localPath = FTPCONSTR + ftpPath + "/" + fileName;
            FtpWebRequest reqFtp = (FtpWebRequest)FtpWebRequest.Create(new Uri(localPath));
            reqFtp.UseBinary = true;
            reqFtp.Credentials = new NetworkCredential(FTPUSERNAME, FTPPASSWORD);
            reqFtp.KeepAlive = false;
            reqFtp.Method = WebRequestMethods.Ftp.UploadFile;
            reqFtp.ContentLength = f.Length;
            int buffLength = 2048;
            byte[] buff = new byte[buffLength];
            int contentLen;
            FileStream fs = f.OpenRead();
            int allbye = (int)f.Length;


            int startbye = 0;
            try
            {
                Stream strm = reqFtp.GetRequestStream();
                contentLen = fs.Read(buff, 0, buffLength);
                while (contentLen != 0)
                {
                    strm.Write(buff, 0, contentLen);
                    startbye = contentLen + startbye;
                    percent = (float)startbye / (float)allbye * 100;
                    if (percent <= 100)
                    {
                        //Debug.Log(percent);
                        uploadProgress = percent;
                    }

                    contentLen = fs.Read(buff, 0, buffLength);

                }
                strm.Close();
                fs.Close();
                erroinfo = "完成";
                return true;
            }
            catch (Exception ex)
            {
                erroinfo = string.Format("因{0},无法完成上传", ex.Message);
                return false;
            }
        }



        #endregion

        #region 从ftp服务器下载文件

        /// <summary>
        /// 从ftp服务器下载文件的功能----带进度条
        /// </summary>
        /// <param name="ftpfilepath">ftp下载的地址</param>
        /// <param name="filePath">保存本地的地址</param>
        /// <param name="fileName">保存的名字</param>
        /// <param name="pb">进度条引用</param>
        /// <returns></returns>
        public static bool Download(string ftpfilepath, string filePath, string fileName)
        {
            FtpWebRequest reqFtp = null;
            FtpWebResponse response = null;
            Stream ftpStream = null;
            FileStream outputStream = null;
            bool SUCCESSFLAG;
            try
            {
                filePath = filePath.Replace("我的电脑\\", "");
                String onlyFileName = Path.GetFileName(fileName);
                string newFileName = filePath + onlyFileName;
                if (File.Exists(newFileName))
                {
                    try
                    {
                        File.Delete(newFileName);
                    }
                    catch { }

                }
                ftpfilepath = ftpfilepath.Replace("\\", "/");
                string url = FTPCONSTR + ftpfilepath;
                reqFtp = (FtpWebRequest)FtpWebRequest.Create(new Uri(url));
                reqFtp.UseBinary = true;
                reqFtp.Credentials = new NetworkCredential(FTPUSERNAME, FTPPASSWORD);
                response = (FtpWebResponse)reqFtp.GetResponse();
                ftpStream = response.GetResponseStream();
                long cl = GetFileSize(url);
                int bufferSize = 2048;
                int readCount;
                byte[] buffer = new byte[bufferSize];
                readCount = ftpStream.Read(buffer, 0, bufferSize);
                outputStream = new FileStream(newFileName, FileMode.Create);

                float percent = 0;
                while (readCount > 0)
                {
                    outputStream.Write(buffer, 0, readCount);
                    readCount = ftpStream.Read(buffer, 0, bufferSize);
                    percent = (float)outputStream.Length / (float)cl * 100;
                    if (percent <= 100)
                    {
                        Debug.Log(percent);
                        downloadProgress = percent;
                    }

                }
                SUCCESSFLAG= true;
            }
            catch (Exception ex)
            {
                Debug.LogError("因{0},无法下载" + ex.Message);
                SUCCESSFLAG=false;
            }
            finally
            {
                if (reqFtp != null)
                    reqFtp.Abort();
                if (response != null)
                    response.Close();
                if (ftpStream != null)
                    ftpStream.Close();
                if (outputStream != null)
                    outputStream.Close();
                
            }
            return SUCCESSFLAG;
        }
        #endregion

        #region 获得文件的大小
        /// <summary>
        /// 获得文件大小
        /// </summary>
        /// <param name="url">FTP文件的完全路径</param>
        /// <returns></returns>
        public static long GetFileSize(string url)
        {

            long fileSize = 0;
            try
            {
                FtpWebRequest reqFtp = (FtpWebRequest)FtpWebRequest.Create(new Uri(url));
                reqFtp.UseBinary = true;
                reqFtp.Credentials = new NetworkCredential(FTPUSERNAME, FTPPASSWORD);
                reqFtp.Method = WebRequestMethods.Ftp.GetFileSize;
                FtpWebResponse response = (FtpWebResponse)reqFtp.GetResponse();
                fileSize = response.ContentLength;

                response.Close();
            }
            catch (Exception ex)
            {
                Debug.LogError(ex.Message);
            }
            return fileSize;
        }
        #endregion

        #region 在ftp服务器上创建文件目录

        /// <summary>
        ///在ftp服务器上创建文件目录
        /// </summary>
        /// <param name="dirName">文件目录</param>
        /// <returns></returns>
        public static bool MakeDir(string dirName)
        {
            try
            {
                string uri = (FTPCONSTR + dirName + "/");
                if (DirectoryIsExist(uri))
                {
                    Debug.Log("已存在");
                    return true;
                }

                string url = FTPCONSTR + dirName;
                FtpWebRequest reqFtp = (FtpWebRequest)FtpWebRequest.Create(new Uri(url));
                reqFtp.UseBinary = true;
                // reqFtp.KeepAlive = false;
                reqFtp.Method = WebRequestMethods.Ftp.MakeDirectory;
                reqFtp.Credentials = new NetworkCredential(FTPUSERNAME, FTPPASSWORD);
                FtpWebResponse response = (FtpWebResponse)reqFtp.GetResponse();
                response.Close();
                return true;
            }
            catch (Exception ex)
            {
                Debug.LogError("因{0},无法下载" + ex.Message);
                return false;
            }

        }
        /// <summary>
        /// 判断ftp上的文件目录是否存在
        /// </summary>
        /// <param name="path"></param>
        /// <returns></returns>        
        public static bool DirectoryIsExist(string uri)
        {
            string[] value = GetFileList(uri);
            if (value == null)
            {
                return false;
            }
            else
            {
                return true;
            }
        }
        private static string[] GetFileList(string uri)
        {
            StringBuilder result = new StringBuilder();
            try
            {
                FtpWebRequest reqFTP = (FtpWebRequest)FtpWebRequest.Create(uri);
                reqFTP.UseBinary = true;
                reqFTP.Credentials = new NetworkCredential(FTPUSERNAME, FTPPASSWORD);
                reqFTP.Method = WebRequestMethods.Ftp.ListDirectoryDetails;

                WebResponse response = reqFTP.GetResponse();
                StreamReader reader = new StreamReader(response.GetResponseStream());
                string line = reader.ReadLine();
                while (line != null)
                {
                    result.Append(line);
                    result.Append("\n");
                    line = reader.ReadLine();
                }
                reader.Close();
                response.Close();
                return result.ToString().Split('\n');
            }
            catch
            {
                return null;
            }
        }
        #endregion

        #region 从ftp服务器删除文件的功能
        /// <summary>
        /// 从ftp服务器删除文件的功能
        /// </summary>
        /// <param name="fileName"></param>
        /// <returns></returns>
        public static bool DeleteFtpFile(string fileName)
        {
            try
            {
                string url = FTPCONSTR + fileName;
                FtpWebRequest reqFtp = (FtpWebRequest)FtpWebRequest.Create(new Uri(url));
                reqFtp.UseBinary = true;
                reqFtp.KeepAlive = false;
                reqFtp.Method = WebRequestMethods.Ftp.DeleteFile;
                reqFtp.Credentials = new NetworkCredential(FTPUSERNAME, FTPPASSWORD);
                FtpWebResponse response = (FtpWebResponse)reqFtp.GetResponse();
                response.Close();
                return true;
            }
            catch (Exception ex)
            {
                //errorinfo = string.Format("因{0},无法下载", ex.Message);
                return false;
            }
        }
        #endregion

        #region  从ftp服务器上获得文件夹列表
        /// <summary>
        /// 从ftp服务器上获得文件夹列表
        /// </summary>
        /// <param name="RequedstPath">服务器下的相对路径</param>
        /// <returns></returns>
        public static List<string> GetFtpDirctory(string RequedstPath)
        {
            List<string> strs = new List<string>();
            try
            {
                string uri = FTPCONSTR + RequedstPath;   //根路径+路径
                FtpWebRequest reqFTP = (FtpWebRequest)FtpWebRequest.Create(new Uri(uri));
                // ftp用户名和密码
                reqFTP.Credentials = new NetworkCredential(FTPUSERNAME, FTPPASSWORD);
                reqFTP.Method = WebRequestMethods.Ftp.ListDirectoryDetails;
                WebResponse response = reqFTP.GetResponse();
                StreamReader reader = new StreamReader(response.GetResponseStream());//中文文件名

                string line = reader.ReadLine();
                while (line != null)
                {
                    if (line.Contains("<DIR>"))
                    {
                        string msg = line.Substring(line.LastIndexOf("<DIR>") + 5).Trim();
                        strs.Add(msg);
                    }
                    line = reader.ReadLine();
                }
                reader.Close();
                response.Close();
                return strs;
            }
            catch (Exception ex)
            {
                Console.WriteLine("获取目录出错：" + ex.Message);
            }
            return strs;
        }
        #endregion

        #region 从ftp服务器上获得文件列表
        /// <summary>
        /// 从ftp服务器上获得文件列表
        /// </summary>
        /// <param name="RequedstPath">服务器下的相对路径</param>
        /// <returns></returns>
        public static Hashtable GetFtpFiles(string RequedstPath)
        {
            ftpFileList=new Hashtable();
            try
            {
                string uri = FTPCONSTR + RequedstPath;   //根路径+路径
                FtpWebRequest reqFTP = (FtpWebRequest)FtpWebRequest.Create(new Uri(uri));
                // ftp用户名和密码
                reqFTP.Credentials = new NetworkCredential(FTPUSERNAME, FTPPASSWORD);
                reqFTP.Method = WebRequestMethods.Ftp.ListDirectoryDetails;
                WebResponse response = reqFTP.GetResponse();
                StreamReader reader = new StreamReader(response.GetResponseStream());//中文文件名
                //Debug.Log(reader.ReadToEnd());
                string line = reader.ReadLine();
                while (line != null)
                {
                    //Debug.Log(line);
                    string[] fileDetails = line.Split(new char[] { ' ' }, 9, StringSplitOptions.RemoveEmptyEntries);
                    //Debug.Log($"分割后数组长度：{fileDetails.Length}");
                    //Debug.Log($"month：{fileDetails[5]}");
                    //Debug.Log($"day：{fileDetails[6]}");
                    //Debug.Log($"time：{fileDetails[7]}");
                    //Debug.Log($"filename: {fileDetails[fileDetails.Length - 1]}");
                    string filename = fileDetails[fileDetails.Length - 1];
                    DateTime dt = Convert.ToDateTime($"{fileDetails[7]} {fileDetails[5]} {fileDetails[6]}");//19:48 Aug 18
                    string date = dt.ToString("MM-dd-HH:mm");
                    if(!ftpFileList.Contains(filename))
                        ftpFileList.Add(filename,date);

                    line = reader.ReadLine();
                }
                reader.Close();
                response.Close();
                return ftpFileList;
            }
            catch (Exception ex)
            {
                Console.WriteLine("获取文件出错：" + ex.Message);
            }
            return ftpFileList;
        }
        #endregion

        #region 通过对比获得要更新的文件
        public static Hashtable getUpdateFileList()
        {
            updateList = new Hashtable();
            if (localFileList.Count == 0)
            {
                updateList = ftpFileList;
            }
            else
            {
                foreach(DictionaryEntry de in ftpFileList) 
                {
                    //Console.WriteLine(de.Key);  //de.Key对应于keyvalue键值对key
                    //Console.WriteLine(de.Value);  //de.Key对应于keyvalue键值对value
                    if (!localFileList.Contains(de.Key)|| string.Compare((string)localFileList[de.Key], (string)ftpFileList[de.Key]) < 0)
                    {
                        updateList.Add(de.Key, de.Value);
                    }
                    else
                    {
                        Debug.Log(de.Key + "不需要更新");
                    }

                }
            }
            Debug.Log(updateList.Count);

            return updateList;
        }
        #endregion
    }
}

```

#### FTP_Test.cs

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using System.Threading;
using UnityEngine.UI;
using UpLoad;
using System.IO;
using System.Runtime.Serialization.Formatters.Binary;
//Keenster 20200818
public class FTP_Test : MonoBehaviour
{
    string htPath = @"E:\coding2\unitySource\FTP_Test\Assets\Model\file.dat";//存储位置
     void Start()
     {

        //上传n
        //new Thread(UpLoad).Start();

        //下载
        new Thread(DownLoad).Start();

        //创建
        //bool bl=UpLoadFiles.MakeDir("AaaaAb");        

        //删除
        //UpLoadFiles.DeleteFtpFile("/data/uploadFile/a.zip");

        //得到文件夹列表
        //List<string> dirs = UpLoadFiles.GetFtpDirctory("");

        //得到文件列表
        //new Thread(UpdateList).Start();

    }
  
     void UpLoad()
     {
         UpLoadFiles.UploadFiles("1/uploadFile", @"E:\UNITY\HLL2-Optimized\Assets\StreamingAssets\JNModels_ABtest\crf250x.catproduct.bundle", "1.bundle");
        
     }
     void DownLoad()
     {
        //UpLoadFiles.Download("1/uploadFile/1.bundle", @"E:\UNITY\HLL2-Optimized\Assets\StreamingAssets\JNModels_ABtest\", "crf250xftp.catproduct.bundle");
        UpLoadFiles.localFileList = new Hashtable();//哈希表建立
        if (File.Exists(htPath))//本地表读取
        {
            FileStream fs1 = new FileStream(htPath, FileMode.OpenOrCreate);
            BinaryFormatter bf1 = new BinaryFormatter();
            UpLoadFiles.localFileList = (Hashtable)bf1.Deserialize(fs1);
            fs1.Close();
        }
        UpLoadFiles.GetFtpFiles("1/uploadFile/");//获取文件列表
        Hashtable tb = UpLoadFiles.getUpdateFileList();//寻找差异，得到更新表
        foreach (DictionaryEntry de in tb)//下载更新文件，更新本地表
        {
            if(UpLoadFiles.Download("1/uploadFile/"+ (string)de.Key, @"E:\coding2\unitySource\FTP_Test\Assets\Model\", (string)de.Key))
            {
                UpLoadFiles.localFileList.Add(de.Key,de.Value);
            }
        }
        FileStream fs = new FileStream(htPath, FileMode.Create);
        BinaryFormatter bf = new BinaryFormatter();
        bf.Serialize(fs, UpLoadFiles.localFileList);//存储本地表
        fs.Close();
    }
       
     void UpdateList()
    {
        UpLoadFiles.GetFtpFiles("1/uploadFile/");
    }
}

```

