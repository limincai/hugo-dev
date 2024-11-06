---
title: "RabbitMQ 可靠性"
description: "提高 RabbitMQ 可靠性的方案。"
tags: ["RabbitMQ"]
categories: ["RabbitMQ"]
date: 2024-11-02T16:54:50+08:00
image: "rabbitmq-reliability-cover.png"
draft: true
---

## 生产者可靠性

### 生产者重连

由于网络波动，可能出现客户端连接 MQ 失败的情况。可以通过配置生产者的 application.yaml 来开启失败后的重连机制：

```yaml
spring:
  rabbitmq:
    # 设置 MQ 的连接超时时间
    connection-timeout: 1s
    template:
      retry:
        # 开启失败重试机制
        enabled: true
        # 失败后的初始等待时间
        initial-interval: 1000ms
        # 失败后的下次等待时间，下次等待时间 = initial-interval * multiplier
        multiplier: 1
        # 最大重试次数
        max-attempts: 3
```

### 生产者确认

RabbitMQ 有 **Publisher Confirm** 和 **Publisher Return** 两种确认机制。开启确认机制后，在 MQ 成果收到消息后会返回确认信息给生产者。返回到结果一以下几种情况：

- 消息投递到了 MQ，但是路由失败（routing key 没有匹配，一般为自己业务问题）。此时会通过 **Publisher Return** 返回路由异常原因，然后返回 **ACK**，告知投递成功。
- **临时消息**（非持久的消息）投递到了 MQ，并且入列成功，返回 **ACK**， 告知投递成功。
- **持久化消息**投递到了MQ，并且入列完成持久化，返回 **ACK**，告知投递成功。
- 其他情况都会返回 **NACK**，告知投递失败。

1. 在 publisher 的 application.yaml 中添加配置。

```yaml
spring:
  rabbitmq:
  	# 开启 publisher confirm 机制，并设置 confirm 类型
    publisher-confirm-type: correlated
    # 开启 publisher return 机制
    publisher-returns: true
```

Publisher-confirm-type 有三种模式：

- none：关闭 confirm 机制。
- simple：同步阻塞等待 MQ 的回执消息。
- correlated：MQ 异步回调方式返回回执消息。

2. 每个 RabbitTemplate 只能配置一个 ReturnCallback，因此需要在项目启动过程中配置：

```java	
@Configuration
@Slf4j
public class CommonConfig implements ApplicationContextAware {
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        // 获取 RabbitTemplate
        RabbitTemplate rabbitTemplate = applicationContext.getBean(RabbitTemplate.class);
        // 设置 ReturnCallback
        rabbitTemplate.setReturnCallback(new RabbitTemplate.ReturnsCallback() {
            @Override
            public void returnedMessage(ReturnedMessage returnedMessage) {
                log.info(
                        "收到消息的 Return Callback，message:{}，" +
                                "replyCode:{}，replyText:{}，" +
                                "exchange:{}，routingKey:{}", 
                        returnedMessage.getMessage(), 
                        returnedMessage.getReplyCode(), 
                        returnedMessage.getReplyText(), 
                        returnedMessage.getExchange(), 
                        returnedMessage.getRoutingKey());
            }
        });
    }
}
```

3. 发送消息，指定消息 ID、消息 ConfirmCallback。

```java
@SpringBootTest
@Slf4j
class PublisherApplicationTests {

    @Resource
    private RabbitTemplate rabbitTemplate;
  
    @Test
    void testPublisherConfirm() {
        // 1.创建 CorrelationData，并指定 id
        CorrelationData correlationData = new CorrelationData(UUID.randomUUID().toString());
        // 2.给 Future 添加 ConfirmCallback
        correlationData.getFuture().addCallback(new ListenableFutureCallback<CorrelationData.Confirm>() {
            @Override
            public void onFailure(Throwable ex) {
                // Future 发生异常时的处理逻辑
                log.error("handle message ack fail", ex);
            }

            @Override
            public void onSuccess(CorrelationData.Confirm result) {
                // Future 接收到回执的处理逻辑，参数中的 result 就是回执类型
                log.info("收到 confirm callback 回执");
                if (result.isAck()) {
                    log.debug("发送消息成功，收到 ACK！");
                } else {
                    log.error("发送消息失败，收到 NACK，原因：{}", result.getReason());
                }
            }
        });
        // 3.发送消息
        rabbitTemplate.convertAndSend("direct.exchange", "red", "hello", correlationData);
    }
}
```

## MQ 的可靠性 

在默认情况下，RabbitMQ 会将接收到的消息保存在内存中以降低消息收发的延迟。这样会导致两个问题：

- 一旦 MQ 宕机，内存中的消息会丢失。
- 内存空间有限，当消费者故障或处理过慢时，会导致消息积压，引发 MQ 阻塞。

### 数据持久化

RabbitMQ 实现数据持久化包括3个方面：

- 交换机持久化
- 队列持久化
- 消息持久化

其中“**交换机持久化**”和“**队列持久化**” Spring 默认处理就是持久化。

“**消息持久化**”可以通过设置消息的属性来进行持久化。

```java
@SpringBootTest
@Slf4j
class PublisherApplicationTests {

    @Resource
    private RabbitTemplate rabbitTemplate;

    @Test
    void testDurability() {
        Message message = MessageBuilder.
                withBody("hello".getBytes(StandardCharsets.UTF_8)).
                setDeliveryMode(MessageDeliveryMode.PERSISTENT).
                build();
        rabbitTemplate.convertAndSend("simple.queue", message);

    }
}
```

### LazyQueue

惰性队列的特征如下：

- 接收到消息后直接存入磁盘而非内存，内存中只保留最近的消息，默认2048条。
- 消费者要消费消息才会从磁盘中读取而非加载到内存。
- 支持数百万的消息存储。

目前 RabbitMQ 的所有队列都是 Lazy Queue 模式，无法更改。

## 消费者可靠性

### 消费者确认

为了确认消费者是否成功处理消息，RabbitMQ 提供了消费者确认机制（**Cosumer Acknowledegement**）。当消费者处理消息结束后，应该向 RabbitMQ 发送一个回执，告知 RabbitMQ 自己消息处理状态。回执有三种状态：

- **ack**：成功处理消息，RabbitMQ 从队列中删除消息。
- **nack**：消息处理失败，RabbitMQ 需要再次投递消息。
- **reject**：消息处理失败并拒绝消息，RabbitMQ 从队列中删除消息。

其中 SpringAMQP 已经实现了消息确认功能，并且可以通过修改配置文件选择 ACK 处理方式，有三种方式：

- none：不处理。即消息投递给消费者后立刻 ack，消息会立刻从 MQ 中删除，非常不安全。

- manual：手动模式。需要自己在业务中调用 api，发送 ack 或 reject，存在业务入侵，但更灵活。

  - auto：自动模式。Spring AMQP 利用 AOP 对消息处理逻辑做了环绕增强：

    - 当业务正常执行时返回 ack。

    - 当业务出现异常，如果是业务异常，返回 nack。

    - 如果是消息处理或校验异常，返回 reject。

      

application.yaml：

```yaml
spring:
  rabbitmq:
    listener:
      simple:
        acknowledge-mode: auto
```

publisher 端代码：

```java
@SpringBootTest
@Slf4j
class PublisherApplicationTests {

    @Resource
    private RabbitTemplate rabbitTemplate;

    @Test
    void testConsumerAcknowledgement() {
        // 交换机名称
        String exchangeName = "fanout.exchange";
        // 消息
        String message = "hello,fanout exchange";
        // 发送消息
        rabbitTemplate.convertAndSend(exchangeName, "consumerack", message);
    }
}
```

consumer 端代码：

```java
@Component
@Slf4j
public class TestListener {

    @RabbitListener(
            bindings = @QueueBinding(
                    value = @Queue(value = "fanout.queue", durable = "true"),
                    exchange = @Exchange(value = "fanout.exchange", type = ExchangeTypes.FANOUT),
                    key = "consumerack"))
    public void testConsumerAcknowledgement(Message message, Channel channel) throws IOException {
        try {
            // 处理消息
            System.out.println("收到消息: " + new String(message.getBody()));

            // 如果消息成功处理，手动确认
            channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
        } catch (Exception e) {
            // 如果处理失败，拒绝消息，RabbitMQ 会重新投递
            channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, true);
        }
    }
}
```

### 失败重试机制

当消费者出现异常后，消息会不断 requeue 到队列，再重新发给消费者，然后再次异常，再次 requeue，导致 MQ 的消息处理飙升，带来不必要的压力。

可以利用 Spring 的 retry 机制，在消费者出现异常时利用本地重试，而不是无限制的 requeue 到 MQ 队列。

applcation.yaml

```yaml
spring:
  rabbitmq:
    listener:
      simple:
        retry:
          enabled: true # 开启消费者失败重试
          initial-interval: 1000ms # 初始的失败重试等待时长
          multiplier: 1 # 下次失败重试的等待时长 = initial-interval * multiplier
          max-attempts: 3 # 最大重试次数
          stateless: true # true 无状态；false 有状态。如果业务中包含事务，这里改为 true
```

在开启重试模式后，重试次数耗尽，如果消息依然失败，则需要有 MessageRecocver 接口来处理，它包含三种不同的实现：

- RejectAndDontRequeueRecoverer：重试耗尽后，直接 reject，丢弃消息。默认就是这种方法。
- ImmediateRequeueMessageRecoverer：重试耗尽后，返回 nack，消息重新入队。
- RepublishMessageRecoverer：重试耗尽后，将失败消息投递到指定的交换机。

这里采用 RepublishMessageRecoverer 的方案。

（1）定义接收失败消息的交换机、队列以及绑定关系。

（2）定义 RepublishMessageRecoverer：

```java
public class RabbitMQConfig {

    @Bean
    public MessageRecoverer republishMessageRecoverer(RabbitTemplate rabbitTemplate) {
        return new RepublishMessageRecoverer(rabbitTemplate, "error.direct", "error");
    }
}
```

### 业务幂等性

业务 幂等性指同一个业务，执行一次和执行多次对业务状态对影响是一致的。

为了防止消息被多次消费，需要采取措施来实现业务幂等性。

#### 方案一：唯一消息 id

给每一个消息设置一个**唯一 id**，利用 id 区分是否重复消费：

1. 每一条消息都生成一个唯一的 id，与消息一起投递给消费者。
2. 消费者接收到消息后处理自己的业务，业务处理成功后将消息 id 保存到数据库。
3. 如果下次又收到相同消息，去数据库查询判断是否存在，存在则为重复消息放弃处理。

```java
@Configuration
public class RabbitMQMessageConfig {

    @Bean
    public MessageConverter messageConverter() {
        // 定义消息转换器
        Jackson2JsonMessageConverter jjmc = new Jackson2JsonMessageConverter();
        // 配置自动创建 id，用于识别不同消息，也可以在业务中基于 id 判断是否是重复消息
        jjmc.setCreateMessageIds(true);
        return jjmc;
    }

}
```

#### 方案二：业务判断

结合业务逻辑，基于业务本身做判断。举例：在支付订单后修改订单状态为以支付，应该在修改订单状态前先查询订单状态，判断状态是否是未支付。只有未支付订单才需要修改，其他状态不做处理：