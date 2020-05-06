---

layout: post
title: C# 局域网多文件传输实现（TCPclient/TCPlistener）
categories:  C#
description: none
keywords: csharp,  TCP
---

一个小DEMO，实现了客户端向服务端请求文件，服务端循环发送文件，客户端接收存储的功能。

------

## 参考链接

- [C#文件传输](https://blog.csdn.net/weixin_44772948/article/details/100658133)
- [简单的C#TCP通讯](https://www.cnblogs.com/jhlong/p/5799248.html)
- [[使用FileInfo获取文件信息]](https://www.cnblogs.com/ChengWenHao/p/CSharpFileInfo.html)
- C#[扫描文件和文件夹的方法](https://blog.csdn.net/dickens88/article/details/5442229)
- [C#中ToString()格式详解](https://www.cnblogs.com/alsf/p/6247658.html)

## tcpxxx介绍

TcpClient是Socket的基础上的封装,为了简化一部分Socket的功能。
1>Socket支持TCP，UDP，IP包，Stream，Dgram等诸多类型
2>而TcpClient只支持TCP和Stream

## 服务端（先开启）

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Net;
using System.Net.Sockets;
using System.Text;
using System.Threading.Tasks;

namespace FileServer
{
    class Program
    {
        static void Main(string[] args)
        {
            TcpListener listenSocket = new TcpListener(IPAddress.Parse("127.0.0.1"), 9999);
            TcpClient clientSocket;
            byte[] sendFile = new Byte[1024];
            string path=@"D:/Resource/3dmodel/catia/0625SJTU";
            DirectoryInfo di = new DirectoryInfo(path);
/*            DirectoryInfo[] dirs = di.GetDirectories();     //扫描文件夹
            foreach (DirectoryInfo di1 in dirs)
            {
                listBox1.Items.Add(di1.Name);
            }*/

            FileInfo[] files = di.GetFiles();                 //扫描文件 GetFiles("*.prefab");可以实现扫描扫描prefab文件 
            
            try
            {
                listenSocket.Start();
                clientSocket = listenSocket.AcceptTcpClient();

                byte[] buffer = new byte[clientSocket.ReceiveBufferSize];  

                NetworkStream stream = clientSocket.GetStream();//获取网络流  

                stream.Read(buffer, 0, buffer.Length);//读取网络流中的数据  

                

                string receiveString = Encoding.ASCII.GetString(buffer).Trim('\0');//转换成字符串  
                if (receiveString.Equals("getFiles"))
                {
                    stream.Write(Encoding.ASCII.GetBytes(files.Length.ToString()));
                }
                stream.Close();//关闭流  

                clientSocket.Close();//关闭Client  
            }
            catch
            {
                Console.WriteLine("文件发送失败");
                
            }
            
            foreach (FileInfo fi in files)
            {

                try
                {
                    listenSocket.Start();
                    clientSocket = listenSocket.AcceptTcpClient();

                    if (clientSocket.Connected)
                    {
                        NetworkStream netStream = clientSocket.GetStream();
                        
                        byte[] fileNameByte = Encoding.Unicode.GetBytes(fi.Name);

                        byte[] fileNameLengthForValueByte = Encoding.Unicode.GetBytes(fileNameByte.Length.ToString("D10"));
                        byte[] fileAttributeByte = new byte[fileNameByte.Length + fileNameLengthForValueByte.Length];
                       
                        fileNameLengthForValueByte.CopyTo(fileAttributeByte, 0);  //文件名字符流的长度的字符流排在前面。
                      
                        fileNameByte.CopyTo(fileAttributeByte, fileNameLengthForValueByte.Length);  //紧接着文件名的字符流
                     
                        netStream.Write(fileAttributeByte, 0, fileAttributeByte.Length);
                        
                        FileStream FS =
                            new FileStream(fi.FullName,
                                FileMode.Open, FileAccess.Read);
                        int lengthBytes;
                        do
                        {
                            lengthBytes = FS.Read(sendFile, 0, sendFile.Length);
                            netStream.Write(sendFile, 0, lengthBytes);


                        } while (lengthBytes == 1024);

                        FS.Close();
                        netStream.Close();
                        Console.WriteLine("{0}文件传输完毕",fi.Name);
                    }
                }
                catch
                {
                    Console.WriteLine("文件发送失败");
                    continue;
                }

            }

        }
    }
}

```

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