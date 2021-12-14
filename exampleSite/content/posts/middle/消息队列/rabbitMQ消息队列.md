---
title: "RabbitMQ消息队列"
date: 2020-03-10T20:51:18+08:00
draft: false

categories: ["队列"]
tags: ["rabbitMQ"]

---

rabbitMQ的介绍及简单使用
<!--more-->

### 必要的概念

- **Message**（消息）：由消息头和消息体组成。消息头由一系列可选属性（元消息）组成，属性包括 routing-key(路由键)，priority(优先级)，delivery-mode(传输类型：是否持久化)等。消息是不透明的，一些属性可能由 Broker 使用，一些可能由消费消息的客户端使用。

- **Publisher**（发布者/(Producer/生产者)）：向交换器发送消息的应用。

- **Consumer**（消费者）：从消息队列获取消息的应用。

- **Exchange**（交换器）：用于接收生产者发送的消息，并将这些消息路由给服务器中对应的队列。

- **Binding**（绑定）：用于消息队列与交称器之间的关联。一个绑定就是基于路由键将交称器和消息队列连接起来的路由规则，所以可以将交换器理解成一个由绑定构造的路由表。

- **Queue**（队列）：消息容器，保存消息直到发送给消费者。

- **Connection**（网络连接）：比如一个 TCP 连接，客户端连接。

- **Channel**（信道）：多路复用连接中的一条独立的双向数据通道。信道是建立在真实的 TCP 连接内的虚拟连接，AMQP 命令都是通过信息发送出去的，不管是发送消息，还是据接收消息。TCP 连接的创建和销毁是非常昂贵的，引入了信道的概念，可以复用一个 TCP 连接。

- **Virtual Host**（虚拟机）：表示一批交换器、消息队列和相关对象。虚拟机是共享相同身份认证和加密环境的独立服务器域。本质上每个 vhost 相当于一台缩小版的 RabbitMQ 服务器，它拥有自己的队列，交换器，绑定和权限机制。

  vhost 是 AMQP 概念的基础，必须在连接时指定，RabbitMQ 默认的 vhost 是 `/`。

- **Broker**（代理）：消息队列服务器实体

### 模型

![](images/rabbitMQ消息队列/mq-rabbitmq-model.png "rabbitmq")

### 环境搭建

使用docker镜像，安装并运行rabbitmq:3.8.2-management版本，带管理界面的版本

### 交换器类型

RabbitMQ 常用的**交换器类型**有 4 种：**Direct Exchange，Fanout Exchange，Topic Exchange，Headers Exchange**。

#### Direct 交换器

Direct 交换器是把消息发送到**消息中的路由键**（routeing key）与 **绑定键**（binding key）完全匹配的队列中。该类型交换器路由必须完全匹配，属于单播的模式。

#### Fanout 交换器

Fanout 交换器只是简单地将队列绑定到交换器，发送到该交换器的所有信息都会被转发到绑定该交换器的所有队列中，会忽略路由键的处理。转发消息是最快的。该类型交换器类似于广播，所有绑定的队列都将收到消息的副本。

#### Topic 交换器

将消息路中的路由键（routeing key）与 绑定键（binding key）进行**模式匹配**，消息将被路由到匹配的队列中。

##### **模式匹配**

- 路由键 和 绑定键都为一个 `.` 分隔的字符串（被 `.` 分隔的每一个字符串被称为一个单词），如 *NEWS.SPORT.TOP*，*NEWS.SPORT.NBA*。
- 绑定键可以存在两种特殊字符串 `*` 和 `#` 用于做模糊匹配。该类型交换器将识别这两个通配符，其中 `*` 用于匹配一个单词，`#` 用于匹配多个单词（可以是零个）。

#### Headers 交换器

Headers 交换器 与 Direct 交换器类似，但不依赖于路由键的匹配规则来路由信息，是根据消息头（headers）属性进行匹配，消息将被路由到匹配的队列。该类型的交换器性能较差，也不实用，几乎不用。

### springboot集成rabbitMQ
#### maven依赖

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
    <version>2.2.5.RELEASE</version>
</dependency>

```

#### 队列字符串常量

```java
public interface SpikeRabbitMq {

   String SPIKE_QUEUE_CUT_NUM = "spike.cut";
   String SPIKE_EXCHANGE_CUT = "spike.exchange.cut";


   String SPIKE_QUEUE_TTL = "spike.ttl";
   String SPIKE_EXCHANGE_TTL = "spike.exchange.ttl";
   /**
    * 死信交换机
    */
   String SPIKE_QUEUE_DEAD = "spike.dead";
   String SPIKE_EXCHANGE_DEAD = "spike.exchange.dead";
   String SPIKE_ROUTING_KEY_DEAD= "spike.routing.key.dead";

}
```

#### rabbitmq相关对象初始化

```java
import com.google.common.collect.Maps;
import lombok.AllArgsConstructor;
import org.springblade.modules.spike.constant.SpikeRabbitMq;
import org.springframework.amqp.core.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.Map;


/**
 * @author bob
 */
@Configuration
@AllArgsConstructor
public class RabbitmqConfig {

	private AmqpAdmin amqpAdmin;

	/**
	 * 普通队列
	 * @return
	 */

	@Bean
	public void spikeCutDeclare() {
		DirectExchange directExchange = new DirectExchange(SpikeRabbitMq.SPIKE_EXCHANGE_CUT);
		Queue queue = new Queue(SpikeRabbitMq.SPIKE_QUEUE_CUT_NUM);
		Binding binding = BindingBuilder.bind(queue).to(directExchange).with(SpikeRabbitMq.SPIKE_QUEUE_CUT_NUM);
		// 建交换器
		amqpAdmin.declareExchange(directExchange);
		// 建队列
		amqpAdmin.declareQueue(queue);
		// 邦定交换器跟队列的关系
		amqpAdmin.declareBinding(binding);
	}
}
```

#### 生产者

```java
	public Boolean deadTtl(Long spikeSpecId) {
		SpikeRecord spikeRecord = new SpikeRecord();
		spikeRecord.setCreateUser(AuthUtil.getUserId());
		spikeRecord.setSpikeSpecId(spikeSpecId);
		rabbitTemplate.setConfirmCallback(new RabbitTemplate.ConfirmCallback() {
			@Override
			public void confirm(CorrelationData correlationData, boolean ack, String cause) {
				if (ack) {
					log.info("消息发送成功,下面开始处理业务。。。。。。。。。。。。。。。");
					log.info("correlationData=" + correlationData);
				} else {
					log.info("消息发送失败。。。。。。。。。。。。。。。。");
					log.info("cause=" + cause);
				}
			}
		});
		rabbitTemplate.setReturnCallback(new RabbitTemplate.ReturnCallback() {
			@Override
			public void returnedMessage(Message message, int replyCode, String replyText, String exchange, String routingKey) {
				log.info("返回的失败代码=" + replyCode + "  返回的失败信息=" + replyText);
				log.info("交换机=" + exchange + "   绑定的路由键=" + routingKey);
			}
		});
		rabbitTemplate.setBeforePublishPostProcessors(new MessagePostProcessor() {
			@Override
			public Message postProcessMessage(Message message) throws AmqpException {
				return message;
			}
		});
		rabbitTemplate.convertAndSend(SpikeRabbitMq.SPIKE_QUEUE_TTL, spikeRecord);
		return Boolean.TRUE;
	}

```

#### 消费者

```java
@Component
@RabbitListener(queues = SpikeRabbitMq.SPIKE_QUEUE_DEAD)
@Slf4j
@AllArgsConstructor
public class SpikeDeadListener {

   @RabbitHandler
   void deadHandler(SpikeRecord spikeRecord, Channel channel, Message message) throws IOException {
      try {
         Object o = JSON.toJSON(spikeRecord);
         log.info("死信队列已消费，{}", JSON.toJSONString(o));
      } finally {
         channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
      }
   }
}
```

### springboot配置死信队列

ttlSpikeDeclare 设置的是过期队列，当队列中的数据数据未被消费，则进入发送给死信交换机，死信交换机把消息发送给死信队列，消费者监听死信队列，进行消费。

其实，死信队列跟普通队列无区别，死信队列的难点，就是延时队列的参数的配置与邦定的参数routingKey（SPIKE_ROUTING_KEY_DEAD)，生产者跟消费者参照上面的例子，注意SPIKE_QUEUE_TTL 是延时队列，队列消息过期，发送到死信队列 SPIKE_QUEUE_DEAD，消费者监听SPIKE_QUEUE_DEAD,进行消费，并手动确认消费。

```java
@Bean
public void ttlSpikeDeclare() {
   Map<String, Object> args = Maps.newHashMap();
   args.put("x-message-ttl", 10000);
   // 死信交换机
   args.put("x-dead-letter-exchange", SpikeRabbitMq.SPIKE_EXCHANGE_DEAD);
   // 死信队列名
   args.put("x-dead-letter-routing-key", SpikeRabbitMq.SPIKE_ROUTING_KEY_DEAD);
   Queue queue = QueueBuilder.durable(SpikeRabbitMq.SPIKE_QUEUE_TTL).withArguments(args).build();
   DirectExchange directExchange = new DirectExchange(SpikeRabbitMq.SPIKE_EXCHANGE_TTL);
   Binding binding = BindingBuilder.bind(queue).to(directExchange).with(SpikeRabbitMq.SPIKE_EXCHANGE_TTL);
   amqpAdmin.declareQueue(queue);
   amqpAdmin.declareExchange(directExchange);
   amqpAdmin.declareBinding(binding);
}


@Bean
public void deadSpikeDeclare() {
   Queue queue = new Queue(SpikeRabbitMq.SPIKE_QUEUE_DEAD);
   DirectExchange directExchange = new DirectExchange(SpikeRabbitMq.SPIKE_EXCHANGE_DEAD);
   Binding binding = BindingBuilder.bind(queue).to(directExchange).with(SpikeRabbitMq.SPIKE_ROUTING_KEY_DEAD);
   amqpAdmin.declareExchange(directExchange);
   amqpAdmin.declareQueue(queue);
   amqpAdmin.declareBinding(binding);
}
```

### 参考

[消息中间件之RabbitMq](https://chenjiabing666.github.io/2018/11/15/消息中间件之Rabbitmq/)

《RabbitMQ实战指南》
