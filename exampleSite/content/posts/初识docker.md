---
title: 初识docker
date: 2018-03-27 21:58:00
tags: ["环境搭建",'docker']
categories: ["docker"]

---
&emsp;&emsp;早就对docker产生兴趣，最近正好赶上项目不是很急，然后趁着闲暇时间折腾一下docker，没想到docker如此简单。
<!--more-->
1. 什么是docker?
&emsp;&emsp;基本上就是一句话：docker是集装箱，主机是货轮。
细节内容：可以看下阮一峰的这篇博客【[Docker入门教程](http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html)】，和逼乎上的【[如何通俗解释Docker是什么？](https://www.zhihu.com/question/28300645)】，基本上就知道docker是什么了。docker的目的就是保持程序运行环境的一致性，特别适合开发，测试，生产各个环境的统一。
2. 学习docker
&emsp;&emsp;主要参照【[docker教程](http://www.runoob.com/docker/docker-tutorial.html)】和【[docker中文社区](http://www.docker.org.cn/)】，当然还有【[docker中文文档](https://docs.docker-cn.com/)】。重复的内容这里就不再赘述。
3. docker on windows
&emsp;&emsp;下载地址：win7、win8.1可以使用国内镜像【[daocloud](https://get.daocloud.io/)】，win10 版本可以直接使用【[官网地址](https://www.docker.com/products/docker#/windows)】,这个地址速度超级慢。配置镜像加速，【[阿里云docker加速](https://cr.console.aliyun.com/)】或者【[daocloud加速](https://www.daocloud.io/mirror)】,国内访问docker pub基本速度，呵呵!
用这个教程【[Docker 容器使用](http://www.runoob.com/docker/docker-container-usage.html)】进行测试，加速效果明显。
按下图进行配置：
![](配置加速.jpg)
4. 为什么个人要使用docker？
&emsp;&emsp;初衷是为了模拟分布式环境,然后折腾一些环境和小实验，用Redis做分布式锁;模拟mycat的使用场景;模拟微服务的架构等等。
&emsp;&emsp;其次，是为了方便折腾大数据的应用。
&emsp;&emsp;由于VMware 太耗费资源，不得不考虑docker。
5. 未来可能还要掌握的技术栈？
&emsp;&emsp;因为docker属于容器技术，然后又是用go语言写的，最近两年go语言很火，然后很多涉及网络编程方面的都是用的go语言。所以，抱着好奇的态度，可能今后要深入了解一下go语言。
&emsp;&emsp;还需要了解一下devOps这种微服务的架构模式。根据这篇文章【[微服务并非Spring Cloud和Dubbo，下一代微服务是什么？](http://developer.51cto.com/art/201803/568218.htm)】的描述，然后结合一下现状，觉得甚有道理。
  * 未来Ops可能需要掌握的技术栈是这样的：
![](下一代微服务.jpg)
  * Dev需要掌握的技术栈是这样的：

** Spring Cloud与Dubbo进行一番对比 **

微服务需要的功能	| Dubbo	| Spring Cloud
  - | - | -
服务注册中心	| Zookeeper	| Spring Cloud Netflix Eureka
服务调用方式	| RPC	| REST API
服务监控	| Dubbo-monitor	| Spring Boot Admin
断路器	| 有，不完善	| Spring Cloud Netflix Hystrix
服务网关	| 无	| Spring Cloud Netflix Zuul
分布式配置	| 无	| Spring Cloud Config
服务跟踪	| 无	| Spring Cloud Sleuth
消息总线	| 无	| Spring Cloud Bus
数据流	| 无	| Spring Cloud Stream
批量任务	| 无	| Spring Cloud Task
... | ... | ...
