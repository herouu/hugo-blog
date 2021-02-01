---
title: "Arduino控制led"
subtitle: ""
date: 2020-10-02T01:19:30+08:00
lastmod: 2020-10-02T01:19:30+08:00
draft: false
author: ""
authorLink: ""
description: ""

tags: ['arduino']
categories: ['arduino']

hiddenFromHomePage: false
hiddenFromSearch: false

featuredImage: ""
featuredImagePreview: ""

toc:
  enable: false
math:
  enable: false
lightgallery: false
license: ""
---
闲来无事，在国庆这天折腾下硬件。
<!--more-->
闲着无所事事，就为了验证一下以前的想法，假如我有一个最小电路，如何通过程序控制这个最小电路，比如：控制LED的亮灭。 查询资料知道可以通过单片机进行控制，单片机，其实是一个微型计算机系统，跟我们正常使用的电脑本质上没有什么区别，都是基于冯.诺依曼体系设计的计算机系统，单片机又叫微控机。听说最多的就是51单片机，其实单片机种类有很多，51单片机只是其中的一种，51单片机是兼容Intel8051指令系统的单片机的统称。如果是其他种类的单片机其支持的指令集也不同。这里的指令集其实是汇编指令，也叫汇编语言。刚开始认为汇编语言是一种语言，其实并非如此，他是一类语言的统称。比如51单片机采用MCS-51指令集，avr系列的avr指令集等等，因为属于不同的指令集，所以是不同的汇编语言。单片机里面也存在cpu, 数据存储器，寄存器，计数器，定时器什么的，且使用的是精简指令集，RISC(reduced instruction set computer)。像家用电脑使用的是复杂指令集，支持的指令更多，更复杂。
回到最初的问题，如果想程序控制一个最小电路，至少得让单片机执行汇编语言，用C语言操作汇编语言，因为汇编语言的不同，操作不同的汇编语言就需要包含不同的包，C中称为头文件，所以操作51单片机，跟avr单片机引用的头文件不同。单片机执行指令，输出信号，这个信号本质上是高低电平，也就是计算机为什么只能识别0，1的原因。
8051系列是经典，上手有点难，得基于keil环境下开发，keil是商用的，想找开源的解决方案，sdcc，编译虽然成功，但是烧录失败，也不知道是不是芯片坏了，也就放弃了，感觉应了下面的段子。

{{< bilibili id=BV1Eb411p7CK >}}

手里有arduino,其算是一个硬件开源平台，使用的是Atmel AVR单片机，跟自己使用的51单片机开发板差不多。构成单片机最小系统，电源+晶振+复位电路+单片机，arduino自然也满足,只不过功能更强大，具体强大的地方，有时间再接着研究。

![](https://bj.bcebos.com/v1/alertcode-blog/单片机最小系统/单片机最小系统.png)
自己使用的是vscode+platformIO插件进行开发，写了一个arduino控制led灯闪烁的入门级案例。本质上也是通过程序控制引脚的高低电平。

```c
#include <Arduino.h>

void setup() {
  // put your setup code here, to run once:
   pinMode(8, OUTPUT);
}

void loop() {
  // put your main code here, to run repeatedly:
  digitalWrite(8, LOW);   // turn the LED on (HIGH is the voltage level)
  delay(1000);                       // wait for a second
  digitalWrite(8, LOW);    // turn the LED off by making the voltage LOW
  delay(1000);       
}
```
