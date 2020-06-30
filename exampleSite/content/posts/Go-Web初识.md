---
title: Go-Web初识
tags:
  - other
categories: []

date: 2019-02-05 22:48:23
---

&emsp;&emsp;折腾了 1 天多，熟悉了一下 go，当然熟悉的动机基本就是三人成虎，然后自己趁着放假验证了一下。

<!--more-->

### 说说感受(自然是跟 java 对比)

- 缺点：
  &emsp;&emsp;如果用 go 做 web 应用的话，估计会分分钟被搞死，坑多，框架不好用
  &emsp;&emsp;国内社区不成熟，国外的不知道，可查询的资料有点少;
  &emsp;&emsp;如果用 go 做 web 应用，大部分是 api 的接口开发。经了解，主要可能用下面的两组技术的居多
  一种是 beego,自带 orm 框架，另一种是 gin+gorm(或 xorm)。贴出两天搜集的资源的网址
  - [beego 文档](https://beego.me/) 包含一系列组件
  - [gin 英文文档](https://github.com/gin-gonic/gin) 简洁，轻巧，性能不错
  - [gin 中文文档](https://learnku.com/docs/gin-gonic/2018/gin-readme/3819)
  - [gorm 中文文档](http://gorm.book.jasperxu.com/) 连接数据库，文档详细
  - [go 语言实战](https://github.com/threerocks/studyFiles/blob/master/go/%E3%80%8AGo%E8%AF%AD%E8%A8%80%E5%AE%9E%E6%88%98%E3%80%8B.pdf)
    &emsp;&emsp;这两种方案自己都试了，beego 操作数据库相应文档看不明白，gorm 相对的就好很多，然后理解不了 beego 的思路，然后就 gg 了。gin+gorm 方案，在 main 方法里面写的话一些 ok，分包分模块就 gg，归其原因可能是 go 语言奇葩的特性（首字母大写，相当于 java 的 public，小写相当于 private）😈😈😈,唉，真是开心，当然还有其他的坑，最后参考了这篇文章才勉强跑的通[gin+gorm+router 快速搭建 crud restful API 接口](https://learnku.com/articles/23548/gingormrouter-quickly-build-crud-restful-api-interface)
    &emsp;&emsp;代码放在 github 上[train-gin-grom](https://github.com/alertcode/train-gin-gorm)
- 优点
  &emsp;&emsp;性能牛 X，简洁，不像 java 般啰嗦！虽然接口比较简单，但是几毫秒还是比较牛叉的,不知道是不是框架调用的代码行数比较少的原因。

![gin框架性能](https://bj.bcebos.com/v1/alertcode-blog/Go-Web初识/gin框架性能.png)

### 总结一下

&emsp;&emsp;如果说 java 是 AK47,go 估计就是狙。学 java 是万金油，学 go 是一招鲜，特定的场景处理起来一针见血。go 是有优势，但是社区还是差的远。
