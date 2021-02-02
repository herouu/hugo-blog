---
title: "Java虚拟机5_对象强引用GC验证"
subtitle: ""
date: 2020-07-29T16:27:37+08:00
lastmod: 2020-07-29T16:27:37+08:00
draft: false
author: ""
authorLink: ""
description: ""

tags: ['java虚拟机']
categories: ['java虚拟机']

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
最近看java虚拟机，初看时各种疑惑，再细看，发现纠结的点，都说了。比如根据下面这段代码，提出如下的几个问题：
* 静态变量a存在jvm的方法区，那么a的实例存在哪？
* 局部变量b存在java虚拟机栈，那b的实例存在哪？
* a,b都是强引用，不会进行垃圾回收，但假如强引用在线程内部，线程销毁，b的实例会不会被回收？

带着这几个问题，从《深入理解java虚拟机》中找答案。


<!--more-->
```java
public class GcOOM {

    private byte[] static a = new byte[1024*1024*1];

    public void test() {
        byte[] b= new byte[1024*1024*1];
    }
}
```
### 问题1
{{< admonition type=question title="" open=true >}}
局部变量b存在java虚拟机栈，那b的实例存在哪？
{{< /admonition >}}

#### 验证代码
```java
public class GcOOM {
    // -XX:+PrintGCDetails -Xms6m -Xmx6m
    // private static byte[] a = new byte[1024*1024*1];
    // private static byte[] a;

    @SneakyThrows
    public static void main(String[] args) {
        Thread.sleep(1000L * 60);
        new GcOOM().test();
    }

    private void test() {
        byte[] a= new byte[1024*1024*1];
        // a = null;
        while (true) {

        }
    }
}

```
#### 验证结果
这两个问题，用上面的代码进行验证，a,b的实例存在java堆中，当引用为null,执行gc，堆中的内存被释放
删除a=null,执行gc
![](https://bj.bcebos.com/v1/alertcode-blog/java虚拟机5_对象强引用GC验证/Snipaste_2020-07-29_12-19-54.png)
添加a=null这行代码，执行gc
![](https://bj.bcebos.com/v1/alertcode-blog/java虚拟机5_对象强引用GC验证/Snipaste_2020-07-29_12-16-01.png)
### 问题2
{{< admonition type=question title="" open=true >}}
静态变量a存在jvm的方法区，那么a的实例存在哪？
{{< /admonition >}}
#### 验证代码
```java
public class GcOOM {
    // -XX:+PrintGCDetails -Xms6m -Xmx6m
    private static byte[] a = new byte[1024*1024*1];

    @SneakyThrows
    public static void main(String[] args) {
        Thread.sleep(1000L * 60);
        new GcOOM().test();
    }

    private void test() {
        // a = null;
        while (true) {

        }
    }
}
```
#### 验证结果
![](https://bj.bcebos.com/v1/alertcode-blog/java虚拟机5_对象强引用GC验证/Snipaste_2020-07-29_12-32-16.png)
### 问题3
{{< admonition type=question title="" open=true >}}
a,b都是强引用，是不是真的不会进行垃圾回收，但假如强引用在线程内部，线程销毁，b的实例会不会被回收？
{{< /admonition >}}
#### 验证代码
```java
package top.alertcode.adelina.oom;

import lombok.SneakyThrows;

public class ThreadGc {

    @SneakyThrows
    public static void main(String[] args) {
        Thread.sleep(1000 * 60L);
        ThreadGc threadGc = new ThreadGc();
        Runnable runnable = threadGc.threadTest();
        Thread thread1 = new Thread(runnable);
        Thread thread2 = new Thread(runnable);
        thread1.start();
        thread2.start();
        while (true) {

        }
    }

    Runnable threadTest() {
        return () -> {
            // 线程中的强引用随着线程的销毁,gc会回收对象
            byte[] a = new byte[1024 * 1024 / 2];
            System.out.println("123");
            try {
                Thread.sleep(10_000L);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        };
    }

}
```
#### 验证结果
![](https://bj.bcebos.com/v1/alertcode-blog/java虚拟机5_对象强引用GC验证/Snipaste_2020-07-29_15-16-26.png)
