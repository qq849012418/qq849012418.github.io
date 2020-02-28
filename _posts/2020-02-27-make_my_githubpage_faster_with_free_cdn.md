---

layout: post
title: 使用星空CDN和JSDelivr CDN对GithubPage未备案.cn域名进行全站加速
categories:  [Net, Tools]
description: none
keywords: github page, keenster.cn, http
---

常言道，时间就是金钱。网站的加载时间从30秒提到了3秒。在这里分享一下方法。

------



> 写在前面

本篇博客是对githubpage进行域名解析后在国内渣访问速度的一种贫穷的补救尝试。若您有条件的话，建议购买已经备案的域名，这样可以享受到国内cdn（如又拍、腾讯）的免费服务；或者购买国内的域名+服务器，并老老实实备案。若您的服务器在海外且是公有的（比如github，很多个人共用几个ip），又购买了.cn这种在国内必须备案的顶级域名，那么你可能需要寻找国外的cdn，或者国内的免备案cdn了，总之，让你的域名对应的节点不在国内，但大大提高连接各大运营商的速度。

我花了一天来寻找这样的东西，但酒香不怕巷子深。在尝试了某御云，freexxx之后，我找到了一个前期免费，后期便宜，无需备案，速度还凑合，回源ip可以直接设置为github.io域名的的国内小型cdn供应商——星空CDN。

接着，由于网站第一次打开的时候需要加载一些资源，找到了专门针对github优化的免费cdn：JSDelivr，通过修改每一个静态资源的加载地址，进一步提高了速度。

## 星空CDN使用方法

官网为https://www.xkcdn.cn/，看起来很简洁

![官网](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/2020-02-28_004802.png)

注册后，会赠送1元，可以在“套餐管理”选择1个月的“免费体验包”（很棒的一点是，这个包是可以续费的，也是1块钱一个月，流量10G，像我这种博客没人看的新博主够用了）

![购买](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/2020-02-28_013202.png)

买完后进入产品管理-域名管理，添加域名（keenster.cn）和加上www的域名，方便用户的不同输入习惯。

这里截一下我的其中一个域名设置，注意源站ip是非常重要的，有服务器当然填你的服务器，但如果是像 github page 这样的，甚至可以写成博客的原始域名！

![域名设置](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/2020-02-28_152045.png)

保存后，进入产品管理-解析管理

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/2020-02-28_013813.png)

复制 CNAME 区域，然后到达你的域名供应商（比如我这里是腾讯云），进入域名解析，删掉现有解析，添加下图的两条CNAME解析。

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/2020-02-28_081729.png)

后面等待解析生效就ok，一般都是秒ok

## JSDelivr CDN使用方法

参考博客为https://www.jianshu.com/p/57f56fb4f219

官网：https://www.jsdelivr.com/，但实际上不需要登录这个网站，只需要在你每个html的代码中修改资源的加载路径，规则如下：

[https://cdn.jsdelivr.net/gh/](https://links.jianshu.com/go?to=https%3A%2F%2Fcdn.jsdelivr.net%2Fgh%2F)你的用户名/你的仓库名@发布的版本号/文件路径

比如可以在site下新建一个cdnurl，值为https://cdn.jsdelivr.net/gh/qq849012418/qq849012418.github.io

然后某个资源的加载可以写成 

```
{{ site.cdnurl }}/assets/js/jquery-ui.js
```



### 可能出现的问题

1. githubpage报502错误，这个问题出现了一次。

   解决方案：浏览器中使用快捷键`CTRL+F5`从服务器刷新缓存。

2.  第一次加载速度非常慢。

    经排查是因为我的源站ssl选项选的是http，改成`http+https（协议跟随模式）`即可。

