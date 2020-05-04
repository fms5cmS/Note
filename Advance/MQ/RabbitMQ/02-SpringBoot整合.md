 添加 spring-boot-starter-amqp 依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

自动配置原理：

- 自动配置类：`RabbitAutoConfiguration`
  - 配置了ConnectionFactory
  - 配置了`RabbitTemplate`来给RabbitMQ发送和接收消息
  - 配置了`AmqpAdmin`，这是RabbitMQ的系统管理功能组件，创建和删除 Queue、Exchange、Binding
- `RabbitProperties`封装了RabbitMQ的配置：host(默认为localhost)、port(默认为5672)、username(默认为guest)、password(默认为guest) 、virtual-host(如果为空，默认`/`)等。



# RabbitTemplate

- `RabbitTemplate`

  - `send(String exchange, String routingKey, Message message)`：Message需要自己构造，定义消息头和消息体

  - `convertAndSend(String exchange, String routingKey, Object message)`：常用！message默认当成消息体，只需要传入要发送的对象，对象被自动序列化发送给RabbitMQ，默认使用JDK的序列化方式

    ```java
    Map<String, Object> map = new HashMap<>();
    map.put("msg", "这是第一个消息");
    map.put("data", Arrays.asList("hello", 123, true));
    rabbitTemplate.convertAndSend("exchange.direct","zzk.news",map);  //这里测试的是direct，类似的可以使用其他，如：exchange.fanout
    ```

  - `Message receiveAndConvert(String queueName)`：接收数据并自动反序列化

  - `RabbitTemplate`默认的`MessageConverter`是`SimpleMessageConverter`，所以默认使用JDK的序列化方式，如果要使用Json序列化方式，可以在配置类中：

    ```java
        @Bean
        public MessageConverter messageConverter(){
            return new Jackson2JsonMessageConverter();
        }
    ```



# 监听

使用`@RabbitListener`监听消息队列的内容，queues属性指定要监听的消息队列。

要使用`@RabbitListener`注解，需要开启基于注解的RabbitMQ模式：在配置上添加`@EnableRabbit`。

```java
@Service
public class BookService {
	//只要消息队列中有Book相关的消息就会输出
    @RabbitListener(queues = "zzk.news")
    public void receive(Book book) {
        System.out.println("收到消息：" + book);
    }
}
```

如果有一些定制的消息在接收时还想获得消息头，可以将方法参数设置为`Message`

```java
    @RabbitListener(queues = "zzk.news")
    public void receive(Message message) {
        System.out.println(message.getMessageProperties());
        System.out.println(message.getBody());
    }
```



# AmqpAdmin创建

`AmqpAdmin`是RabbitMQ的系统管理功能组件，可以创建和删除 Queue、Exchange、Binding，下面演示了创建过程，还有对应的删除操作。

```java
@Autowired
AmqpAdmin amqpAdmin;

@Test
public void testAmqpAdmin(){
    //创建一个指定名字的Exchange
    amqpAdmin.declareExchange(new DirectExchange("amqpAdmin.exchange"));
    //创建一个指定名字且可持久化的Queue
    amqpAdmin.declareQueue(new Queue("amqpAdmin.queue",true));
    //创建绑定规则
    amqpAdmin.declareBinding(new Binding("amqpAdmin.queue",Binding.DestinationType.QUEUE,
                                         "amqpAdmin.exchange","amqp.h",null));
}
```

