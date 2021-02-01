---
title: java虚拟机2_内存管理
date: 2018-03-15 18:16:40
tags: ["java虚拟机"]
categories: ["java虚拟机"]

---

java 虚拟机的内存关系:
&emsp;&emsp;a.java 虚拟机并不是只有 Hotspot
&emsp;&emsp;b.java 虚拟机内存管理

<!--more-->

## 1. java 虚拟机并不是只有 Hotspot

- Sun Classic VM 始祖级
- Exact VM 过度产品,英雄气短
- Hotspot VM 目前使用广泛的虚拟机
- Kvm (Kilobyte)
- JRockit
- IBM j9
- Azul vm
- Liquid vm
- Dalvik vm
- Microsoft vm

## 2. java 虚拟机内存管理

### 内存分配区域如下：

![img](https://bj.bcebos.com/v1/alertcode-blog/java虚拟机2_内存管理/Snipaste_2020-07-29_18-09-57.png)

### 由所有线程共享的数据区

**堆内存**

- 存放对象实例
- 垃圾收集器管理的主要区域
- 新生代，老年代
- 新生代包含一块Eden区，两块Survivor(S1,S2)区，Hotspot虚拟机默认大小比例为8：1：1

**方法区**

- 存储虚拟机加载的类信息，常量，静态变量，即时编译器变异后的代码等数据
  &emsp;&emsp; a. 类的版本 b. 方法 c. 字段 d. 接口
- 方法区与永久代
- 垃圾回收在方法区的行为，回收效率比较低

**运行时常量池，属于方法区内存中的一块**
**直接内存**

- 不受 java 虚拟机内存的制约

### 线程隔离的数据区

**程序计数器**

- 一块较小的内存空间，可以看成当前线程所执行的字节码的行号指示器，分支、跳转、循环、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成。
- 程序计数器，处于线程独占区；

**java 虚拟机栈**

- 虚拟机栈描述的是 java 方法执行的动态内存模型，为执行 java 方法服务
- 栈帧：每个方法执行时都会创建一个栈帧，伴随着方法从创建到执行完成。存储局部变量表，操作数栈，动态链接，方法出口等。
- 局部变量表： 存放编译期可知的各种基本数据类型，引用类型，returnAddress 类，内存空间在编译期完成分配，内存空间在运行期间不变，存放对象的引用。
- 大小
  &emsp;&emsp;a.StackOverflowError 栈内存溢出 b.outOfMemory 内存溢出

**本地方法栈**

- 本地方法栈为虚拟机执行 native 方法服务
