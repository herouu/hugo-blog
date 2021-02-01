---
title: "Mint安装wechat"
date: 2020-12-16T19:47:21+08:00
draft: false
---
mint安装wechat!换上固态硬盘的老电脑又活过来了。
<!--more-->
### 前言
旧笔记本12年买的机器，18年的时候以为坏了，然后就换了现在正在使用的ThinkPad E580，坏了的电脑就放在那里也没有修。前段时间无意中查到开不了机，可能是由于硬盘坏了，网上买了的固态硬盘，准备周末回家装上，看看，还好使不。到家发现自己的旧笔记本找不到了，各种翻找，电脑跟适配器分开放置，找这俩货委实废了不少时间，最后在卫生间的储物箱里找到了，拿到的时候是有些惊呆的。这洗澡的时候这潮，放了快3年了，不会生锈坏了吧。心情忐忑，装上之后，发现bios可以进。确实是硬盘坏了，影响了开机。由于是老机器，装上了linux，这个linux选择发行版本折腾了好久，最后选定了Mint。因为有点接近windows，所以使用起来也比较顺手。使用了一段时间发现，确实桌面级应用，Linux是比不过windows生态的。没有太多的软件可以选择。比如常用的wechat在Linux下没有适配版本，但工作生活中又很需要这个软件进行交流。折腾了好久，最后使用wine+安装包才勉强可以用起来，发送消息了。这里就简单的介绍下自己的安装过程。
### 安装需要工具
* wechat.exe 官方下载
* wine
* winetricks
### 安装步骤
#### wine
这个通过mint系统自带的软件管理器，搜索wine-stable就可以进行安装，正常安装不会有什么问题，这里不在赘述。
#### winetricks
```
wget  https://raw.githubusercontent.com/Winetricks/winetricks/master/src/winetricks
chmod +x winetricks
sudo mv -v winetricks /usr/local/bin
```
