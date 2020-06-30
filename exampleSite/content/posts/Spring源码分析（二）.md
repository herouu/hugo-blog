---
title: Spring源码分析（二）：说说ApplicationContext接口
date: 2019-01-02 20:51:49
tags: ["spring源码"]

---

&emsp;&emsp;使用 spring 的时候，可能并不陌生于下面的这段代码

```java
ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("/org/springframework/context/support/test/context*.xml"，"other.xml");
Service service = (Service) ctx.getBean("service");
```

这段代码在写单元测试的时候经常使用，如果，使用 spring 是基于.xml 配置的话，ApplicationContext 继承的多个接口之一就是 ListableBeanFactory，是 BeanFactory 的子接口，主要获得了 BeanFactory 的 getBean()方法，可以认为 Spring 中的 IoC 容器，即为 BeanFactory 或 ApplicationContext 接口的实现。

<!--more-->

### AppliationContext 继承关系

&emsp;&emsp;ApplicationContext 继承六个接口

- EnvironmentCapable
- ListableBeanFactory
- MessageSource
- HierarchicalBeanFactory
- ApplicationEventPublisher
- ResourcePatternResolver

![ApplicationContext继承关系](https://bj.bcebos.com/v1/alertcode-blog/Spring源码分析（二）：说说ApplicationContext接口/ApplicationContext继承关系.png)

### ApplicationContext 接口的实现

&emsp;&emsp;ApplicationContext 接口的实现主要有三个 ClassPathXmlApplicationContext，AnnotationConfigWebApplicationContext，FileSystemXmlApplicationContext
![ApplicationContext比较重要的几个实现](https://bj.bcebos.com/v1/alertcode-blog/Spring源码分析（二）：说说ApplicationContext接口/ApplicationContext比较重要的几个实现.png)

### 鸡肋的实现类，牛 B 的抽象类

&emsp;&emsp;ApplicationContext 的 3 个比较重要的实现类，中的方法大多比较鸡肋，类中最多的代码可能就是构造方法了（相当多），主要是因为 spring 运用了模板方法模式。让子类继承父类的方法。3 个实现类都是继承自 AbstractApplicationContext 抽象类中的方法，AbstractApplicationContext 中的方法重点关注 refresh()方法，无论是基于 Xml 配置的实现，还是基于注解配置的实现，都会调用 refresh 方法，关于 refresh()中的细节,留到下一篇再分析
