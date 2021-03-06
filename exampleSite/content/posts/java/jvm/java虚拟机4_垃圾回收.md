---
title: java虚拟机4_垃圾回收
date: 2018-04-10 23:04:27
tags: ["java虚拟机"]

categories: ["java虚拟机"]
---
jvm 的垃圾回收，主要考虑下面的 3 个问题：
&emsp;&emsp; 如何判定对象为垃圾对象
&emsp;&emsp; 如何回收
&emsp;&emsp; 何时回收

<!--more-->

### 1. 何时回收

- 引用计数法  
  &emsp;&emsp;在对象中添加一个引用计数器，当有地方引用这个对象的时候，引用计数器+1，当引用计数器失效的时候，计数器-1
- 可达性分析法
  &emsp;&emsp;定义 gc root 节点，向下级节点查找对象，如果找不到 ，则定义为垃圾。
  可以作为 gc root 的对象：
  a.虚拟机栈；
  b.方法区的类属性所引用的对象；
  c.方法区中常量所引用的对象；
  d.本地方法栈中引用的对象；

### 2. 回收策略

- 标记清除算法
  算法如其名。
  ![img](https://bj.bcebos.com/v1/alertcode-blog/java虚拟机4_垃圾回收/Snipaste_2020-07-29_18-17-34.png)
  <div align="center">黄色-要清除的垃圾;白色-释放的空间;红色-占用的内存</div>
  &emsp;&emsp;标记：标记需要回收的对象（可达性分析法）
  &emsp;&emsp;清除：大对象不连续空间寻址，如果找不到则会再次触发垃圾回收。所以，有效率和空间问题。
- 复制算法
  &emsp;&emsp;首先将内存分为大小相等的两部分（假设 A、B 两部分），每次呢只使用其中的一部分（这里我们假设为 A 区），等这部分用完了，这时候就将这里面还能活下来的对象复制到另一部分内存（这里设为 B 区）中，然后把 A 区中的剩下部分全部清理掉。
- 标记整理算法
  &emsp;&emsp;标记：它的第一个阶段与标记/清除算法是一模一样的，均是遍历 GC Roots，然后将存活的对象标记。
  &emsp;&emsp;整理：移动所有存活的对象，且按照内存地址次序依次排列，然后将末端内存地址以后的内存全部回收。因此，第二阶段才称为整理阶段。
- 分代随机算法
  &emsp;&emsp;是不同算法的集合，对于不同的内存区域，使用不同的算法进行垃圾回收。

### 3. 垃圾回收器

- Serial
- Parnew
- Cms
