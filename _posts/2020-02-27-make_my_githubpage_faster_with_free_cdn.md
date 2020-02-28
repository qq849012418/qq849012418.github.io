---

layout: post
title: 使用经济实惠的星空CDN对GithubPage未备案.cn域名进行访问加速
categories:  [Net, Tools]
description: none
keywords: github page, keenster.cn, http
---

上车半天，感觉良好，网站的加载时间从13秒提到了3秒。虽然我觉得可能一部分归功于缓存。

------



> 写在前面

本篇博客是对githubpage进行域名解析后在国内渣访问速度的一种贫穷的补救尝试。若您有条件的话，建议购买已经备案的域名，这样可以享受到国内cdn（如又拍、腾讯）的免费服务；或者购买国内的域名+服务器，并老老实实备案。若您的服务器在海外且是公有的（比如github，很多个人共用几个ip），又购买了.cn这种在国内必须备案的顶级域名，那么你可能需要寻找国外的cdn，或者国内的免备案cdn了，总之，让你的域名对应的节点不在国内，但大大提高连接各大运营商的速度。

我花了一天来寻找这样的东西，但酒香不怕巷子深。在尝试了某御云，freexxx之后，我找到了一个前期免费，后期便宜，无需备案，速度还凑合，回源ip可以直接设置为博客地址的的国内小型cdn供应商——星空CDN。（实际速度可刷新本博客测试，我不知道其他地区打开速度如何）

## 使用方法

官网为https://www.xkcdn.cn/，看起来很简洁

![官网](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/2020-02-28_004802.png)

注册后，会赠送1元，可以在“套餐管理”选择1个月的“免费体验包”（很棒的一点是，这个包是可以续费的，也是1块钱一个月，流量10G，像我这种博客没人看的新博主够用了）

![购买](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/2020-02-28_013202.png)

买完后进入产品管理-域名管理，添加域名（keenster.cn）和加上www的域名，方便用户的不同输入习惯。

这里截一下我的其中一个域名设置，注意源站ip是非常重要的，有服务器当然填你的服务器，但如果是像 github page 这样的，甚至可以写成博客的原始域名！

![域名设置](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/2020-02-28_013431.png)

保存后，进入产品管理-解析管理

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/2020-02-28_013813.png)

复制 CNAME 区域，然后到达你的域名供应商（比如我这里是腾讯云），进入域名解析，删掉现有解析，添加下图的两条CNAME解析。

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/2020-02-28_081729.png)

后面等待解析生效就ok

### 可能出现的问题

githubpage报502错误，这个问题出现了一次。

解决方案：浏览器中使用快捷键`CTRL+F5`从服务器刷新缓存。

