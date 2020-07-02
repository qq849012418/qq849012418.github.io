---
layout: post
title: 使用不蒜子为博客增加访客统计+浏览统计
categories:  Blog
description: none
keywords: blog,  busuanzi
---

非常良心的免费计数器。

------

## 参考链接

- [不蒜子](https://busuanzi.ibruce.info/)
- [不蒜子小白网页计数器使用教程](https://www.dazhuanlan.com/2019/10/27/5db56e10202f4/)

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



在_include文件夹下新建一个visit-stat.html的文件，这样可以方便您修改好格式后引入到任何页面中。这里的js目录因为本人使用了cdn加速（防止远程地址跑路），所以地址如下，普通使用的话改成<script async src="//busuanzi.ibruce.info/busuanzi/2.3/busuanzi.pure.mini.js">即可

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

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/20200702120849.png)