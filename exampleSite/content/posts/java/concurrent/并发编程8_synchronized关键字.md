---
title: "并发编程(八)：synchronized关键字"
subtitle: ""
date: 2020-07-03T22:41:56+08:00
lastmod: 2020-07-03T22:41:56+08:00
draft: false
author: ""
authorLink: ""
description: ""

tags: ['多线程']
categories: ['多线程']

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

synchronized关键字在1.6之前是重量级锁，由于是重量级锁，所以性能方面远不如java.util.concurrent并发包下面的Lock接口实现的锁的性能高。 从1.6开始对synchronize进行了优化，引入了偏向锁，轻量级锁。使synchronized的锁具有锁升级的过程。

<!--more-->

升级的过程大概是这样的。

![image-20200704214850588](images/并发编程8_synchronized关键字/image-20200704214850588.png "并发编程")





### synchronized用的锁存在哪里

synchronized实现锁的基础，是java中的任何一个对象都可以作为锁。具体可以表现为以下三种形式：

* 对于普通同步方法，锁是当前实例对象

  ```java
    public class SynchronizedTest {
      public static void main(String[] args){
          // 对实例对象上锁
          new SynchronizedTest().lock1();
      }
      synchronized void lock1(){
  
      }
  }
  ```

  

* 对于静态方法，所示当前类的Class对象

  ```java
     static synchronized void lock() {}
     等价于
     synchronized (SynchronizedTest.class){}
  ```

  

* 对于同步方法块，锁是synchronized括号里配置的对象

  ```java
     // a 可以是任何对象 a=this,a=SynchronizedTest.class,或a = new A();
     synchronized (a){}
  ```

  synchronized用的锁存在java对象头。

  Class A 里面具有的属性

  ```java
  public class A {
      int size;//4字节
      boolean flag;// 1字节
  }
  
  ```

  

  在jvm64位虚拟机下打印A对象的信息为

  ```
   OFFSET  SIZE      TYPE DESCRIPTION                               VALUE
        0     4           (object header)                           01 00 00 00 (00000001 00000000 00000000 00000000) (1)
        4     4           (object header)                           00 00 00 00 (00000000 00000000 00000000 00000000) (0)
        8     4           (object header)                           43 c1 00 f8 (01000011 11000001 00000000 11111000) (-134168253)
       12     4       int A.size                                    0
       16     1   boolean A.flag                                    false
       17     7           (loss due to the next object alignment)
  ```

  前12byte为对象的头信息，后面5byte的为对象的属性信息和7byte对齐位，位数信息为8的倍数。这里12+5+7=24byte 1byte=8bit  12byte=96bit

  12byte头信息由两部分组成**mark word**+ **klass pointer** ，引用自 http://openjdk.java.net/groups/hotspot/docs/HotSpotGlossary.html

  mark word 存储对象的hashCode或索信息，klass pointer用来存储对象类型数据的指针

  >
  >
  >**klass pointer**
  >
  >The second word of every object header. Points to another object (a metaobject) which describes the layout and behavior of the original object. For Java objects, the "klass" contains a C++ style "vtable".
  >
  >**mark word**
  >
  >The first word of every object header. Usually a set of bitfields including synchronization state and identity hash code. May also be a pointer (with characteristic low bit encoding) to synchronization related information. During GC, may contain GC state bits.
  >
  >**object header**
  >
  >Common structure at the beginning of every GC-managed heap object. (Every oop points to an object header.) Includes fundamental information about the heap object's layout, type, GC state, synchronization state, and identity hash code. Consists of two words. In arrays it is immediately followed by a length field. Note that both Java objects and VM-internal objects have a common object header format.

 查看openjdk源码markOop.hpp得知，大端存储的方式下对象头的组成如下。

```c++
// Bit-format of an object header (most significant first, big endian layout below):  //大端存储布局如下
//
//  32 bits:
//  --------
//             hash:25 ------------>| age:4    biased_lock:1 lock:2 (normal object)
//             JavaThread*:23 epoch:2 age:4    biased_lock:1 lock:2 (biased object)
//             size:32 ------------------------------------------>| (CMS free block)
//             PromotedObject*:29 ---------->| promo_bits:3 ----->| (CMS promoted object)
//
//  64 bits:
//  --------
//  unused:25 hash:31 -->| unused:1   age:4    biased_lock:1 lock:2 (normal object) // 无锁
//  JavaThread*:54 epoch:2 unused:1   age:4    biased_lock:1 lock:2 (biased object) // 偏向锁
//  PromotedObject*:61 --------------------->| promo_bits:3 ----->| (CMS promoted object) 
//  size:64 ----------------------------------------------------->| (CMS free block)
//
//  unused:25 hash:31 -->| cms_free:1 age:4    biased_lock:1 lock:2 (COOPs && normal object)
//  JavaThread*:54 epoch:2 cms_free:1 age:4    biased_lock:1 lock:2 (COOPs && biased object)
//  narrowOop:32 unused:24 cms_free:1 unused:4 promo_bits:3 ----->| (COOPs && CMS promoted object)
//  unused:21 size:35 -->| cms_free:1 unused:7 ------------------>| (COOPs && CMS free block)

```

window系统为小端存储，查看对象头关注箭头所示位置，即锁的位置。

![image-20200704232908817](images/并发编程8_synchronized关键字/image-20200704232908817.png "并发编程")

### 各种状态的锁

![image-20200704235730517](images/并发编程8_synchronized关键字/image-20200704235730517.png "并发编程")

