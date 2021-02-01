---
title: android搭建kali linux+java+内网穿透
date: 2018-03-28 22:00:45
categories: ["环境搭建"]
---
&emsp;&emsp;首先，折腾了好几天，已经忘了当初为什么要折腾这个东西，可能是当时想用手机的linux环境玩内网穿透，然后就装上了kali linux(一个渗透测试的环境，搞网络安全的黑帽与白帽)，当然，我的初衷还是玩内网穿透的，kali linux 有机会再学习。
<!--more-->
1. 手机如何装上linux环境？
必备工具下载地址：
【[Linux deploy]( https://github.com/meefik/linuxdeploy/releases) Linux系统支撑软件】
【[Busy Box]( https://github.com/meefik/busybox/releases) Linux deploy支撑软件】
【[ConnectBox]( https://github.com/connectbot/connectbot/releases) 手机端SSH连接软件】
2. 安装Linux deploy中的kali linux镜像
网上类似的教程还是有很多的，这里放几张图，参照下面，来进行部署:
  * 先进行配置，然后进行安装
  ![img1](1.png)
  * 配置参照下图,主要选择版本，源，启动ssh
  ![img2](2.jpg)
  * 安装完毕之后，点击启动
  ![img3](3.png)
  * 启动完成后进入镜像，硬盘占用如下
  ![img4](4.png)
3. 安装java openjdk环境
连接镜像的ssh
然后安装java环境，直接openjdk就好，出问题再说
apt-get install openjdk-8-jdk
然后把maven,tomcat也安装上，然后java环境构建完毕，启动tomcat,手机的ip：port为http://192.168.1.101:8080  , 服务成功访问。  
4. 手机上倒腾frp,内网穿透
  参照 [内网映射frp服务器环境搭建](/内网映射frp服务器环境搭建   )
