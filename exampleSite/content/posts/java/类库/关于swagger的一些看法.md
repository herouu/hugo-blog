---
title: 关于swagger的一些想法
date: 2018-03-21 20:59:34
tags: ['java工具']
categories: ['java工具']

---

&emsp;&emsp;因为最近的项目总是涉及到前后端的交互，所以为了减少前后端的沟通与测试，简单的了解了一下 swagger(丝袜哥)，这个东西还是比较屌的，尤其是 springboot+swagger2,前后端分离，对于后台同学来说，开发速度快的飞起。

<!--more-->

这样的开发模式，唯一缺点就是，沟通的成本，要维护接口的文档效率还是很低的，一般维护的状况就是自己写一个 word,要么就是自己部署一个文档服务器（这里推荐一个叫做【[RAP](https://github.com/thx/RAP)】的框架，虽然要手写，但是比自己维护 word 估计效率强多了），然后就接触了 swagger，简单的看了一下[swagger](https://swagger.io/)官网，基本上有四个模块

- swagger-editor 文档编辑器
- swagger-codegen 代码生成器
- swagger-ui 一套接口 UI
- swagger-inspector 可以根据任何语言的 api 自动生成 openApi 文档(可以跟各种语言整合)

&emsp;&emsp;这里，一直有个误解，以为 springboot 使用 swagger,其实使用的是 swagger-ui,而真正根据代码，自动生成 api 是一个名叫 springfox 的框架，一直误解，以为 swagger 可以自动生成 api,其实不是。springboot 工程中使用 swagger,其实是使用 springfox+swagger-ui+swagger-inspector 结合的产物，也就是为什么 springboot 工程中引用的是这两个包。

```xml
<dependency>
     <groupId>io.springfox</groupId>
     <artifactId>springfox-swagger2</artifactId>
     <version>2.7.0</version>
</dependency>
<dependency>
     <groupId>io.springfox</groupId>
     <artifactId>springfox-swagger-ui</artifactId>
     <version>2.7.0</version>
</dependency>
```
