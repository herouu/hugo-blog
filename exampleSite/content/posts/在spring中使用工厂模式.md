---
title: "在spring中使用工厂模式"
subtitle: ""
date: 2021-01-27T19:51:48+08:00 
lastmod: 2021-01-27T19:51:48+08:00 
draft: false 
author: ""
authorLink: ""
description: ""

tags: ["设计模式","spring"]
categories: ["设计模式","spring"]

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
在spring中使用工厂模式
<!--more-->

### 定义接口

```java
package com.github.herouu.trainhigh.design.factory;

public interface IHandler {

    void handler();
}

```

### 定义抽象类

```java
public abstract class AbstractHandle implements IHandler {
    // 没有任何共用的方法，不进行实现
}
```

### 实现类A,B

```java

package com.github.herouu.trainhigh.design.factory.impl;

import com.github.herouu.trainhigh.design.factory.AbstractHandle;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

@Component
@Slf4j
public class AHandleImpl extends AbstractHandle {
    @Override
    public void handler() {
        log.info("AHandleImpl");
    }
}

```

```java
package com.github.herouu.trainhigh.design.factory.impl;

import com.github.herouu.trainhigh.design.factory.AbstractHandle;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

@Component
@Slf4j
public class BHandleImpl extends AbstractHandle {
    @Override
    public void handler() {
        log.info("BHandleImpl");
    }
}
```

### 定义工厂

```java
package com.github.herouu.trainhigh.design.factory;


import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.util.Map;

@Component
public class HandleFactory {

    @Autowired
    private Map<String, AbstractHandle> map;

    public IHandler getHandler(String beanName) {
        return map.get(beanName);
    }
}

```

### 调用

```java

@RestController
@Slf4j
public class QuestionController {

    @Autowired
    HandleFactory handleFactory;

    @GetMapping(path = "/test")
    public void test() {
        handleFactory.getHandler("AHandleImpl").handler();
    }
}
```
