---
layout: post
title: 为博客增加一个很普通的字数统计
categories:  Blog
description: none
keywords: blog,  busuanzi
---

很多公众号，知乎大佬都喜欢弄一个字数统计，这东西看起来严谨而高端，但其实原理非常简单

------

## 参考链接

- [正常人1分钟大约能阅读多少个字](https://zhidao.baidu.com/question/569937287.html)


## 配置方法



在config.yml中添加

```css
# 文章字数统计
word_count:
    enabled: true
```

统一的开关是个好习惯



在post.html页面的合适位置添加引用的格式为

```html
{% if site.word_count.enabled %}
          <span class="meta-info">
            <span class="octicon octicon-clock"></span>
            共 {{ page.content | strip_html | strip_newlines | remove: " " | size }} 字，约 {{ page.content | strip_html | strip_newlines | remove: " " | size | divided_by: 500 | plus: 1 }} 分钟
          </span>
          {% endif %}
```

500字一分钟，考虑到代码段很多所以设置的高了一些

## 效果

![](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/20200702131923.png)