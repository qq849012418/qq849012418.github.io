---

layout: post
title: 为博客增加访客统计、浏览统计和文章字数分析
categories:  Blog
description: none
keywords: blog,  TCP
---

一个小DEMO，实现了客户端向服务端请求文件，服务端循环发送文件，客户端接收存储的功能。

------

## 参考链接

- [不蒜子](https://busuanzi.ibruce.info/)
- [不蒜子小白网页计数器使用教程](https://www.dazhuanlan.com/2019/10/27/5db56e10202f4/)
- [[使用FileInfo获取文件信息]](https://www.cnblogs.com/ChengWenHao/p/CSharpFileInfo.html)
- C#[扫描文件和文件夹的方法](https://blog.csdn.net/dickens88/article/details/5442229)
- [C#中ToString()格式详解](https://www.cnblogs.com/alsf/p/6247658.html)

# 不蒜子小白网页计数器

不蒜子是一个极简的网页计数器，因为自己做的静态博客需要用到就把他的js给集成进去，总访问量和总访客数基于域名一对一对应通过远程的js集成，将数据通过/busuanzi.ibruce.info/busuanzi的访问持久化，即便在次生成文章或者网站，只要链接没有变浏览量还是可以在原有的基础上在次往上加的。

## 配置方法

在config.yml中添加
```css
 # 不蒜子访问统计
    busuanzi:
        enabled: true
        start_date: 2020-07-02

```



在_include文件夹下新建一个visit-stat.html的文件，这样可以方便您修改好格式后引入到任何页面中。这里的js目录因为本人使用了cdn加速，所以地址如下，普通使用的话改成<script async src="//busuanzi.ibruce.info/busuanzi/2.3/busuanzi.pure.mini.js">即可

```html
<!-- 不蒜子访问统计 -->
{% if site.busuanzi.enabled == true %}
<script async src="{{ site.cdnurl }}/assets/vendor/busuanzi/2.3/busuanzi.pure.mini.js"></script>
<div class="mobile-hidden" style="margin-top:8px">
  <span id="busuanzi_container_site_pv" style="display:none">
    本站访问量<span id="busuanzi_value_site_pv"></span>次
  </span>
  <span id="busuanzi_container_site_uv" style="display:none">
    / 本站访客数<span id="busuanzi_value_site_uv"></span>人
  </span>
  <span id="busuanzi_container_page_pv" style="display:none">
    / 本页访问量<span id="busuanzi_value_page_pv"></span>次
    {% if site.busuanzi.start_date %}
    / 统计始于{{ site.busuanzi.start_date }}
    {% endif %}
  </span>
</div>
{% endif %}
```

把这个html在任何其他html页面引用的格式为

```html
{% include visit-stat.html %}
```

## 效果



## 客户端

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Net;
using System.Net.Sockets;
using System.Text;
using System.Threading.Tasks;

namespace FileClient
{
    class Program
    {
        static void Main(string[] args)
        {
            TcpClient clientSocket = new TcpClient();
            NetworkStream clientStream;
            IPEndPoint serverIP;
            byte[] receiveFileByte = new byte[1024];

            serverIP = new IPEndPoint(IPAddress.Parse("127.0.0.1"), 9999);
            //     clientStream = clientStream.Read(receiveFileByte, 0, receiveFileByte.Length);
            while (!clientSocket.Connected)
            {
                try
                {
                    clientSocket.Connect(serverIP);
                }
                catch (Exception e)
                {
                    Console.WriteLine(e);
                    throw;
                }
            }
                
            int filenum=0;
            if (clientSocket.Connected)
            {
                clientStream = clientSocket.GetStream();
                clientStream.Write(Encoding.ASCII.GetBytes("getFiles"));
                Byte[] data = new Byte[256];
                String responseData = String.Empty;
 
                // Read the first batch of the TcpServer response bytes.
                Int32 bytes = clientStream.Read(data, 0, data.Length);
                responseData = System.Text.Encoding.ASCII.GetString(data, 0, bytes);
                Console.WriteLine("Received: {0}", responseData);
                filenum = int.Parse(responseData);
                clientStream.Close();
                clientSocket.Close();
            }

            for (int i = 0; i < filenum; i++)
            {
                clientSocket = new TcpClient();
                clientSocket.Connect(serverIP);
                if (clientSocket.Connected)
                {
                    clientStream = clientSocket.GetStream();
                
                    byte[] fileNameLengthForValueByte = Encoding.Unicode.GetBytes((256).ToString("D10"));
                    byte[] fileNameLengByte = new byte[1024];
                    int fileNameLengthSize = clientStream.Read(fileNameLengByte, 0, fileNameLengthForValueByte.Length);
                    string fileNameLength = Encoding.Unicode.GetString(fileNameLengByte, 0, fileNameLengthSize);
                    Console.WriteLine("文件名字符流的长度为：" + fileNameLength);

                    int fileNameLengthNum = Convert.ToInt32(fileNameLength);
                    byte[] fileNameByte = new byte[fileNameLengthNum];

                    int fileNameSize = clientStream.Read(fileNameByte, 0, fileNameLengthNum);
                    string fileName = Encoding.Unicode.GetString(fileNameByte, 0, fileNameSize);
                    Console.WriteLine("文件名为：" + fileName);

                    string dirPath = "./WebFile";
                    if(!Directory.Exists(dirPath))
                    {
                        Directory.CreateDirectory(dirPath);
                    }
                
                    FileInfo fi = new FileInfo(fileName);
                    FileStream fs = new FileStream(dirPath+'/'+fi.Name, FileMode.Append, FileAccess.Write);
                    int length;
                    do
                    {
                        length = clientStream.Read(receiveFileByte, 0, receiveFileByte.Length);
                        fs.Write(receiveFileByte, 0, length);
                        //Console.WriteLine("接受完成");



                    } while (length == 1024);
                    Console.WriteLine("接受完成");
                    fs.Close();
                    clientStream.Close();
                    clientSocket.Close();
                }
            }
          
        }
    }
}
```

## 文件传输效果

客户端根目录下创建了一个名为WebFile的文件夹，服务端目标目录中的所有文件都被保存到这里

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/微信截图_20200507004246.png)