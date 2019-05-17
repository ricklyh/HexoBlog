---
title: RabbitMQ入门
date: 2018-08-14 23:32:22
tags:
      - MQ
categories:
      - MQ
---

## 应用场景说明
   - 首先我们以实际的用户下单支付场景为例，其业务主流程如下：
     - 用户提交订单 ---> 生成订单 ---> 发消息（短信&微信推送）---> 支付 ---> 回调通知订单支付成功
   - 上面的流程我们可以拆分为两个模块：
     - 订单模块和支付模块。在订单模块中主流程为生成订单之前的过程，在支付模块中回调通知订单是用户支付完后，第三方支付渠道（比如微信）通知支付回调网关，支付回调网关调用订单模块的回调接口
     
   - 从上述分析我们知道：
     - 发消息如果是同步发送，则会影响主流程业务，比如发送消息慢，则会影响订单服务的性能，考虑到发送消息的状态不是强一致的，所以我们采用异步发送机制
     - 订单模块和支付模块存在依赖关系，这样就会出现，订单模块的回调通知服务接口修改了，则支付模块的代码也会跟着修改，反之也是一样；还有如果订单服务网络超时等异常了则支付的回调通知网关也势必受到影响，所以我们想办法让其解耦
   - 而 MQ 正好满足两大特性 异步 + 解耦 ，其中 RabbitMQ 被企业广泛使用
   
## 而 MQ 正好满足两大特性 异步 + 解耦 ，其中 RabbitMQ 被企业广泛使用
   - RabbitMQ（以下简称 RQ ） 是部署最广泛的开源的消息中间件，由 Erlang 语言开发
   - 在 RQ 中有几个比较重要的理论概念：
     - ### AMQP
       - 是一种消息传递协议，它要求客户端之间、及和消息中间件之间保持一致的消息进行通信
       
     - ### Connections
       - 访问连接，它是建立在可靠的 TCP 连接之上，比如当客户端断开连接时不立即关闭 TCP 连接
      
     - ### Channels
       - 信道，客户端之间消息通过信道传输，一个 Connection 共享多个信道，它的一个主要作用是避免客户端直接对 TCP 建立和关闭所消耗的系统资源代价，可以看出 RQ 从底层设计时就考虑了高性能的应用
       
     - ### Exchanges
       - 交换机，它指定消息发送到哪个队列；流程是生产者将消息发给Exchange, 然后 Exchange 通过不同的类型（主要包括 fanout 、direct、topic ）发送到不同的队列
       
     - ### Queues
       - 队列，存储消息的具体位置
       
     - ### Products
       - 生产者，消息的发送者
       
     - ### Consumers
       - 消费者，它订阅注册到特定队列，队列将消息“推”给消费者进行消息处理 
       
## 重要特性
   - 为了保证消息的可靠传输，RQ 提供了几个比较重要的特性，我们生产环境一般都会采用
   
     - ### 持久化机制
       - 运维侧：在配置 Exchange 和 Queue 时设置 Durability=Durable 避免突然宕机引起的消息丢失
     
     - ### Ack 机制
       - 客户端消费者侧：消费者在处理完消息后通知 RQ 从队列内存中删除消息，保证消息已被消费者接收到
        
     - ### Confirm 机制
       - 客户端生产者侧：生产者将消息发送到 RQ 然后写入到磁盘后通知生成者已收到生产者消息，保证生产者发送的消息不会丢失
         - 支持两种通知方式：
           - 同步方式，即每发一条消息生成者等待 RQ 确认后再继续发送消息
           
           - 异步方式，即生产者提供回调函数入口，生产者发送完消息后不等待 RQ 回应继续发送消息，RQ 会回调通知生产者是否收到消息，一般实际生产环境用此方式比较多
        

## 消息模型
   - RQ 支持灵活的消息模型，概要总结主要包括以下几种
   
   - ### 队列模型
     - 生产者直接将消息发送到队列，有一个消费者或多个消费者获取消息进行消费，如果是多个消费者则队列将采用轮询的方式分发消息到各个消费者，保证消息被均衡消费
     ![RabbitMQ](mq0.png "Optional title")
     
   - 发布订阅模型
     - 生产者将消息发送到交换机，队列绑定到交换机，消费者订阅队列消息进行消费，即 Exchange 的 fanout 类型
     ![RabbitMQ](mq1.png "Optional title")
     
   - 路由模型
        - 生产者将消息发送到交换机并指定路由 key，队列绑定到交换机，并设定好路由规则。消费者从匹配上路由 key 的队列里面获取到推送的消息，即 Exchange 的 direct 类型，和 topic 类型的区别是：topic 可以模糊匹配路由 key 值
        ![RabbitMQ](mq2.png "Optional title")
     
## 基于 Spring Boot 访问核心 API 说明
   - ### Queue 对象
     - name：字符串类型，队列名称，用户自定义名称，符合业务描述即可。
     - durable：布尔类型，是否持久化，比如服务重启队列不丢，生成环境建议设置为 true。
     - autoDelete：布尔类型，是否自动删除，比如最后一个消费者下线后，队列将自动删除，生成环境建议设置为 false
     
   - ### Exchange 对象
     - name：交换机名称。
     - durable：是否持久化，比如服务重启交换机不丢。
     - autoDelete：是否自动删除，比如最后一个绑定的队列被删除，交换机将自动删除
     - 子类：
       - FanoutExchange，无路由规则，发布订阅模型。
       - DirectExchange，有路由规则，匹配路由 key. 路由模型。
       - TopicExchange， 有路由规则，匹配路由 key，路由模型，并支持 key 的模糊匹配
     
   - ### AmqpTemplate 对象
     - 基于 AMQP 协议，同步发送和接收队列消息，生产者侧使用
     - 基于 AMQP 协议，同步发送和接收队列消息，生产者侧使用
     ```
       convertAndSend(java.lang.Object message)
       
       发送消息并指定路由 Key:
       
       convertAndSend(java.lang.String routingKey, java.lang.Object message)
       
       接收消息并转换成 Java 对象：
       
       receiveAndConvert() 
       
     ```
   - ### RabbitListener 对象
     - 通过指定队列或绑定关系监听消息，消费者侧使用
     
   - AsyncAmqpTemplate 对象
     - 基于 AMQP 协议，异步发送和接收队列消息，生产者侧使用
     
   - BindingBuilder 对象
     - 操作将队列绑定到 Exchange 上，并可以指定路由 Key 规则
     
## 简单代码实例
   - 这段代码很简单，就是发送和消费消息，这里交换机和队列绑定通过代码控制配置实现（为演示用，直接感受下）。建议生产环境使用管理系统来创建交换机、队列及绑定关系，因为可视化操作比代码里面维护更方便，即代码级别一般开发人员负责实现消费者、生产者；交换机和队列及绑定关系由运维人员负责配置管理维护，开发人员进行协助
   
   - 添加依赖
   ```
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-amqp</artifactId>
      </dependency>
   ```
   - yml配置或者properties配置
   ```
     spring:
        rabbitmq:
           host: 127.0.0.1
           port: 5672
           userename: admin
           password: 123456
   ```
   - 生产者代码
   ```
      @Autowired
              AmqpTemplate template;
      
              @GetMapping("send")
              public void send() {
      
              //  简单队列模式
                  template.convertAndSend("queue", "hi RQ");   
      
              // 路由模式
                  //template.convertAndSend("exchange", "topic.sms", "hi RQ"); 
      
              }
   ```
   - 消费者代码
   ```
      @Component
      public class Consumer {
      
              @RabbitListener(queues = "queue")
              public void handler(String str)
              {
                  System.out.println(str);
              }
      
              @RabbitListener(queues = "topic.sms")
              public void handlerSms(String str)
              {
                  System.out.println(str);
              }
          }
   ```
   - 生产者端配置类代码
   ```
      @Configuration
          public class SenderConfig {
      
              @Bean
              public Queue queue() {
                  return new Queue("queue",false,false,true);
              }
      
              @Bean(name="smsqueue")
              public Queue smsQueue() {
                  return new Queue("topic.sms",false);
              }
      
      
              @Bean
              public TopicExchange exchange{
                  return new TopicExchange("exchange");
              }
      
              @Bean
              Binding bindingExchangeForSms(Queue smsqueue,TopicExchange exchange){
                  return BindingBuilder.bind(smsqueue).to(exchange).with("topic.sms");
              }
          }
   ```
   
## 监控管理 Web UI 简单介绍
   - 接下来让我们直观感受下 RQ 的管理界面，先上图：
   ![RabbitMQ](mq3.png "Optional title")
   
   - 创建交换机，配置交换机参数
   ![RabbitMQ](mq4.png "Optional title")
   
   - 创建队列，配置队列相关参数
   ![RabbitMQ](mq5.png "Optional title")
     - 其中参数，x-max-length 指定队列中最多包含多少条消息，想一下是不是可以在秒杀系统中使用呢？比如，一个单品 SKU，最多可以多少人下单成功，下单成功的订单都存放到队列中，那如果用户下单了没有支付咱们怎么办，那就配合下面这个参数使用
     
     - x-message-ttl 指定消息在没有消费之前在队列中最多生存多长时间，单位时间为毫秒，也就是说用户下单成功但超过 5 分钟没有支付，则该订单消息将自动从队列中删除，腾出空间让其它用户继续抢单即可
      
   - 将队列绑定到交换机上
   ![RabbitMQ](mq6.png "Optional title")
     - 其中如果你采用发布订阅模型，则路由 Key 可以为空，这个路由 Key 的作用前面已经讲过，我在这里再简要描述下，生产者发送消息指定路由 Key 到交换机，交换机根据路由 Key 匹配到不同的队列上，然后队列将消息推送给订阅了该队列消息的消费者进行消费处理