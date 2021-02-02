---
title: spring-boot-kafka整合
date: 2019-04-11 10:53:48
tags: ["springboot"]

---

&emsp;&emsp;spring boot kafka 整合工程

<!--more-->

### maven 依赖

```xml
   <dependency>
      <groupId>org.apache.kafka</groupId>
      <artifactId>kafka-streams</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.kafka</groupId>
      <artifactId>spring-kafka</artifactId>
    </dependency>
```

### yml 配置文件

```yml
server:
  port: 8081
spring:
  kafka:
    producer:
      bootstrap-servers: 192.168.0.103:9092
    consumer:
      group-id: test

kafka.consumer.topic: topictest
```

### producer 生产者

```java
@Component
@EnableScheduling
public class KafkaProducer {

  @Autowired
  private KafkaTemplate kafkaTemplate;
  @Value("${kafka.consumer.topic}")
  private String topic;

  /**
   * 定时任务 模拟异步发送 kafkaTemplate下的发送机制都是异步的
   */
  @Scheduled(cron = "* * * * * ?")
  public void send() {
    ExecutorService executorService = Executors.newFixedThreadPool(1000);
    Stream.iterate(1, seed -> seed + 1).limit(10).forEach(
        task -> executorService
            .execute(() -> Stream.iterate(1, seed -> seed + 1).limit(1000).forEach(item -> {
              String message = UUID.randomUUID().toString();
              ListenableFuture future = kafkaTemplate.send(topic, message);
              future.addCallback(o -> System.out.println("send-消息发送成功：" + message),
                  throwable -> System.out.println("消息发送失败：" + message));
            }))
    );

  }
}
```

### consumer 消费者

```java
@Component
public class KafkaComsumer {


  @KafkaListener(topics = {"${kafka.consumer.topic}"})
  public void listen(ConsumerRecord<?, ?> record) throws IOException {
    String value = (String) record.value();
    System.out.println("消息接收成功" + value);
  }

}
```

### demo 地址

https://github.com/alertcode/springboot-kafka-demo.git
