---
title: "Hugo博客优化"
subtitle: ""
date: 2020-07-01T23:14:43+08:00
lastmod: 2020-07-01T23:14:43+08:00
draft: false
author: "冷眼"
authorLink: ""
description: ""

tags: ['hugo']
categories: ['hugo']

hiddenFromHomePage: false
hiddenFromSearch: false

featuredImage: ""
featuredImagePreview: ""

toc:
  enable: true
math:
  enable: false
lightgallery: false
license: ""
---

怎么说，使用hugo作为静态博客生成工具也有一段时间了，对hugo也有一定的了解，总体来说hugo的构建速度确实比hexo快，这是使用hugo的优点，当然缺点也是有的，比如：hugo的主题相对来说比hexo要少的多，其次，hugo的主题有特定的模板语法，相对来说上手有点难度。尤其对于想将hexo的主题迁移到hugo上来说工作量比较庞大且不太友好；其次，现有主题对站内搜索支持比较弱。虽然有algolia的商用站内搜索的支持，但较于hexo社区来说还是比较鸡肋。当然，个人还是比较喜欢hugo的，相对hexo来说比较小众。对了，个人使用的是hugo的LoveIt主题，向大家安利一下。

<!--more-->

### 境内访问优化

之前hugo博客使用的是netlify自动部署，这个东西确实比较方便，属于CI/CD一类的工具，如果国内访问速度可以的话，其实自己还是比较推荐这个。但是最近访问netlify上的东西明显比较卡，估计是因为好多国外免费的服务都被国人给玩坏了。可能是被‘墙’给盯上了，曾经见过用netlify发布v2ray的订阅地址的。所以，网络的流量审查，必然导致速度不行。

所以，自然得寻找替换的方案。研究了gitee page、github page、coding.net page

gitee page的话，限制太多，访问速度面向国内，自然在国内的话，访问速度是比较快的，但是gitee page pro才支持自定义域名，收费，想想算了。

本来想把hugo生成的静态资源使用github page的方式进行发布，但是github page使用的是 jekyll 生成的，对于loveIt主题的short code指令无法识别，构建静态页的时候警告且构建不成功。总不能一个一个页面去改，所以放弃github page。如果是使用hexo构建页面，hexo没有特殊的指令，github是完全支持的。个人猜测出现这种情况的原因是因为运行`hugo    --source=exampleSite`生成的文件多了index.md文件，这个在hexo中是不存在的。

![image-20200701234108104](https://bj.bcebos.com/v1/alertcode-blog/hugo博客优化/image-20200701234108104.png)

coding.net  page本来自己也是不想用的。但是相对于netlify的话访问是快了不少。估计是被腾讯收购了以后，线路优化了。恰好静态页部署在coding上，没有出现github page类似的情况。速度上不是最优的，但是可以接受，总比200ms要強上许多。

![image-20200702000044142](https://bj.bcebos.com/v1/alertcode-blog/hugo博客优化/image-20200702000044142.png)

### PicGo+Typora让写飞起来

写markdown文档的话，其实最烦的就是往文档里面添加图片。之前的模式是，批量截图->批量上传图片->获取链接指定位置插入，这个过程是比较耗时的。当Typora支持PicGo的时候这个过程就不一样了，基本上截图+粘贴+上传一气呵成。

演示如下：

![](https://bj.bcebos.com/v1/alertcode-blog/hugo博客优化/20200702003833.gif)

Typora图片配置

![image-20200702002647407](https://bj.bcebos.com/v1/alertcode-blog/hugo博客优化/20200702002649.png)





















