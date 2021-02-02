---
title: java8 Collector Demo
date: 2018-09-12 21:31:10
tags: ["java8"]

---

&emsp;&emsp;java8 Collector Demo,主要参考了这篇文章[Guide to Java 8’s Collectors](https://www.baeldung.com/java-8-collectors)

<!--more-->

```java
package top.alertcode.trainhigh.other;

import com.alibaba.fastjson.JSON;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.Map;
import java.util.Set;
import java.util.stream.Collectors;
import lombok.Data;

/**
 * 请参考https://www.baeldung.com/java-8-collectors
 */
public class J8Collectors {

  /**
   * The entry point of application.
   *
   * @param args input arguments
   */
  public static void main(String[] args) {
    toListCollectors();
    toSetCollectors();
    toMapCollectors();
    joinCollectors();
    countCollectors();
    partitioningByCollectors();
    collectCollectors();
    collectingAndThen();
  }

  /**
   * toList 操作
   */
  public static void toListCollectors() {
    List<Integer> collect = Arrays.asList(1, 2, 3, 3, 3, 4, 4).stream()
        .collect(Collectors.toList());
    System.out.println(JSON.toJSONString(collect));
  }

  /**
   * toSet 操作
   */
  public static void toSetCollectors() {
    Set<Integer> collect = Arrays.asList(1, 2, 3, 3, 3, 4, 4).stream()
        .collect(Collectors.toSet());
    System.out.println(JSON.toJSONString(collect));
  }

  /**
   * partitioningBy 操作
   */
  public static void partitioningByCollectors() {
    Map<Boolean, List<Integer>> collect = Arrays.asList(1, 2, 3, 3, 3, 4, 4).stream().distinct()
        .collect(Collectors.partitioningBy(e -> e >= 3));
    System.out.println(JSON.toJSONString(collect));
  }

  /**
   * toMap 操作
   */
  public static void toMapCollectors() {
    List<Demo> list = new ArrayList<>();
    list.add(new Demo("1", "1"));
    list.add(new Demo("2", "2"));
    list.add(new Demo("3", "3"));
    Map<String, String> collect = list.stream()
        .collect(Collectors.toMap(Demo::getId, Demo::getName));
    System.out.println(JSON.toJSONString(collect));
  }

  /**
   * joining 操作
   */
  public static void joinCollectors() {
    //只对字符串生效
    String result = Arrays.asList("1", "2", "3", "4", "5").stream()
        .collect(Collectors.joining(" ", "prefix ", " suffix"));
    System.out.println(result);
  }

  /**
   * counting 操作
   */
  public static void countCollectors() {
    Long collect = Arrays.asList("1", "2", "3", "4", "5").stream().collect(Collectors.counting());
    System.out.println(collect);
  }

  /**
   * collect 操作
   */
  public static void collectCollectors() {
    ArrayList<Object> collect = Arrays.asList(1, 2, 3, 4, 5, 6, 7).stream()
        .collect(ArrayList::new, ArrayList::add, ArrayList::addAll);
    System.out.println(collect);
  }

  /**
   * collectingAndThen 操作.
   */
  public static void collectingAndThen() {
    List<Integer> collect = Arrays.asList(1, 2, 3, 4, 5, 6, 7).stream()
        .collect(Collectors.collectingAndThen(Collectors.toList(), result -> {
          result.add(8);
          return result;
        }));
    System.out.println(collect);
  }


  /**
   * Demo类.
   *
   * @author Z佬
   * @version v1.0
   * @date 20180912 18:45:40
   */
  @Data
  static class Demo {

    /**
     * The Id.
     */
    private String id;
    /**
     * The Name.
     */
    private String name;

    /**
     * Instantiates a new Demo.
     */
    public Demo() {
    }

    /**
     * Instantiates a new Demo.
     *
     * @param id the id
     * @param name the name
     */
    public Demo(String id, String name) {
      this.id = id;
      this.name = name;
    }
  }

}

```

结果：

```
[1,2,3,3,3,4,4]
[1,2,3,4]
{"1":"1","2":"2","3":"3"}
prefix 1 2 3 4 5 suffix
5
{false:[1,2],true:[3,4]}
[1, 2, 3, 4, 5, 6, 7]
[1, 2, 3, 4, 5, 6, 7, 8]
```
