---
title: springboot事件驱动编程
date: 2019-03-16 17:33:28
tags: ["springboot"]

---

&emsp;&emsp;观察者模式，定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新
&emsp;&emsp;又称,发布/订阅、消息 通知机制、事件监听、事件驱动编程

<!--more-->

&emsp;&emsp;事件监听机制结构图如下：
![spring事件监听机制结构图.png](images/springboot事件驱动编程/事件监听架构图.png "springboot事件驱动编程")

### 业务处理完成后发送事件（生产者）

```java
package top.alertcode.train.boot.demo.event;


import lombok.extern.slf4j.Slf4j;
import org.springframework.context.ApplicationContext;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;

@Service
@Slf4j
public class EventService {


    @Resource
    ApplicationContext applicationContext;

    public void createEvent() {
        log.info("业务完成，产生NoticeEvent事件");
        applicationContext.publishEvent(new NoticeEvent(""));
    }
}

```

### 事件类（消息）

```java
package top.alertcode.train.boot.demo.event;

import org.springframework.context.ApplicationEvent;

public class NoticeEvent extends ApplicationEvent {

    public NoticeEvent(Object source) {
        super(source);
    }
}

```

### 监听事件（第一个消费者）

```java
package top.alertcode.train.boot.demo.event;

import lombok.extern.slf4j.Slf4j;
import org.springframework.context.ApplicationEvent;
import org.springframework.context.event.SmartApplicationListener;
import org.springframework.stereotype.Component;


@Slf4j
@Component
public class EventListener implements SmartApplicationListener {


    /**
     * 只监听需要监听的事件
     *
     * @param aClass
     * @return
     */
    @Override
    public boolean supportsEventType(Class<? extends ApplicationEvent> aClass) {
        return aClass == NoticeEvent.class;
    }

    /**
     * 当事件发送的时候调用
     *
     * @param applicationEvent
     */
    @Override
    public void onApplicationEvent(ApplicationEvent applicationEvent) {
        //    业务逻辑的实现
        log.info("处理发送的事件, EventListener 处理 NoticeEvent");
    }

    /**
     * 多个监听，执行顺序，值越大，越靠后
     *
     * @return
     */
    @Override
    public int getOrder() {
        return 0;
    }
}

```

### 监听事件（第二个消费者）

```java
package top.alertcode.train.boot.demo.event;

import lombok.extern.slf4j.Slf4j;
import org.springframework.context.ApplicationEvent;
import org.springframework.context.event.SmartApplicationListener;
import org.springframework.stereotype.Component;

@Slf4j
@Component
public class EventListenerTwo implements SmartApplicationListener {
    @Override
    public boolean supportsEventType(Class<? extends ApplicationEvent> aClass) {
        return aClass == NoticeEvent.class;
    }


    @Override
    public int getOrder() {
        return 1;
    }

    @Override
    public void onApplicationEvent(ApplicationEvent applicationEvent) {
        log.info("处理发送的事件, EventListenerTwo 处理 NoticeEvent");
    }
}

```

### 单元测试

```java
package top.alertcode.train.boot.demo.event;

import org.junit.Test;
import top.alertcode.train.boot.demo.TrainBootDemoApplicationTests;

import javax.annotation.Resource;

import static org.junit.Assert.*;

public class EventServiceTest extends TrainBootDemoApplicationTests {

    @Resource
    private EventService eventService;

    @Test
    public void createEvent() {
        eventService.createEvent();
    }
}
```

### 日志打印

```
2019-03-16 17:21:26.127  INFO 13144 --- [           main] t.a.train.boot.demo.event.EventService   : 业务完成，产生NoticeEvent事件
2019-03-16 17:21:26.128  INFO 13144 --- [           main] t.a.train.boot.demo.event.EventListener  : 处理发送的事件, EventListener 处理 NoticeEvent
2019-03-16 17:21:26.128  INFO 13144 --- [           main] t.a.t.boot.demo.event.EventListenerTwo   : 处理发送的事件, EventListenerTwo 处理 NoticeEvent

```
