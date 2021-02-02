---
title: 并发编程（五）：线程间交换数据的Exchanger
date: 2018-10-22 04:53:51
tags: ["多线程"]
categories: ["多线程"]

---&emsp;&emsp;Exchanger(交换者)是一个用于线程间协作的工具类。Exchanger 用于进行线程间的数据交换。它提供一个同步点，在这个同步点，两个线程可以交换彼此的数据。这两个线程通过 exchange 方法交换数据，如果第一个线程先执行 exchange 方法，他会一直等待第二个线程也执行 exchange 方法，当两个线程都到达同步点是，这两个线程就可以交换数据，将本线程产生出来的数据传递给对方。

<!-- more  -->

### Exchanger 的应用场景

- 可以用于遗传算法（不懂）
- 可以用于校对工作
  - 比如我们需要将纸质银行流水通过人工的方式录入成电子银行流水，为了避免错误，采用 AB 两人进行录入，录入到 Excel 之后，系统需要加载这两个 excel，并对两个 excel 数据进行校对，看看是否录入一致。

### 代码

```java
package top.alertcode.trainhigh.concurrent;

import java.util.concurrent.Exchanger;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * @author alertcode
 * @date 2018-10-22
 * @copyright alertcode.top
 */
public class TestExchanger {

  private static final Exchanger<String> ex = new Exchanger<>();
  private static ExecutorService service = Executors.newFixedThreadPool(2);

  public static void main(String[] args) {
    service.execute(() -> {
      String a = "a";
      try {
        ex.exchange(a);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
    });

    service.execute(() -> {
      String b = "b";
      try {
        //输出‘a’
        String exchange = ex.exchange(b);
        //进行校对工作
        System.out.println(exchange.equals(b));
        System.out.println(exchange);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
    });
    service.shutdown();
  }
}
```

### 注意

&emsp;&emsp;如果两个线程有一个没有执行 exchange 方法，则会一直等待，如果担心有特殊情况发生，避免一直等待，可以使用`public V exchange(V x, long timeout, TimeUnit unit)`设置最大等待时长。
