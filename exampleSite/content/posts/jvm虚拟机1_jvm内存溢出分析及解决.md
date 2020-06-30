---
title: jvm虚拟机1_jvm内存溢出问题的分析与解决
date: 2018-03-15 18:16:40
tags: ["jvm虚拟机"]

categories: ["jvm虚拟机"]
---

学习一下 java 虚拟机系列，之一

<!--more-->

添加运行参数
-XX:+HeapDumpOnOutOfMemoryError -Xms30m -Xmx30m

-XX:+HeapDumpOnOutOfMemoryError 这个参数会生成堆栈快照，用于定位异常

模拟内存溢出的场景，简单代码：

```java
package top.alertcode.demo.jvm;

import java.util.ArrayList;

/**
 * @author alertcode
 * @date 2018-04-03
 * @copyright alertcode.top
 */
public class OutOfMemoryDemo {

  /**
   * 运行这段代码最终会出现内存溢出的异常
   * @param args
   */
  public static void main(String[] args) {
    ArrayList<OutOfMemoryDemo> list = new ArrayList<OutOfMemoryDemo>();
    while (true) {
      list.add(new OutOfMemoryDemo());
    }
  }
//出现下面的错误
/*  Connected to the target VM, address: '127.0.0.1:53483', transport: 'socket'
  Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
  at java.util.Arrays.copyOf(Arrays.java:3210)
  at java.util.Arrays.copyOf(Arrays.java:3181)
  at java.util.ArrayList.grow(ArrayList.java:265)
  at java.util.ArrayList.ensureExplicitCapacity(ArrayList.java:239)
  at java.util.ArrayList.ensureCapacityInternal(ArrayList.java:231)
  at java.util.ArrayList.add(ArrayList.java:462)
  at top.alertcode.demo.jvm.OutOfMemoryDemo.main(OutOfMemoryDemo.java:15)
  Disconnected from the target VM, address: '127.0.0.1:53483', transport: 'socket'*/
}
```

&emsp;&emsp;使用分析工具 MAT(Eclipse Memory Analyzer)，进行分析，很容易定位到内存溢出的原因，即频繁的创建对象。参照下图：
![img](内存分析.jpg)
