# 1.MQ

全程为Message Queue（消息队列），是在消息的传输过程中保存消息的容器，多用于分布式系统之间进行通信。

<img src="img\image-20221115104653549.png" alt="image-20221115104653549" style="zoom:50%;" />

## 1.1优势

1.应用解耦，提升容错性和可维护性。

<img src="img\image-20221115110337259.png" alt="image-20221115110337259" style="zoom:50%;" />

2.异步提速，提升用户体验和系统吞吐量（单位时间内处理请求的数目）

<img src="img\image-20221115110215259.png" alt="image-20221115110215259" style="zoom:50%;" />

3.削峰填谷，提高系统的稳定性

<img src="img\image-20221115110422864.png" alt="image-20221115110422864" style="zoom:50%;" />

<img src="img\image-20221115110127131.png" alt="image-20221115110127131" style="zoom:50%;" />

​		使用了MQ之后，限制消费消息的速度为1000，这样一来，高峰期产生的数据势必会被积压在 MQ中，高峰就被“削”掉了，但是因为		消息积压，在高峰期过后的一段时间内，消费消息的速度还是会维持在1000，直到消费完积压的消息，这就叫做“填谷”



## 1.2劣势

- **系统可用性降低**

  系统引入的外部依赖越多，系统稳定性越差。一旦 MQ 宕机，就会对业务造成影响。如何保证MQ的高可用?

- **系统复杂度提高**

  MQ 的加入大大增加了系统的复杂度，以前系统间是同步的远程调用，现在是通过 MQ 进行异步调用。如何保证消息没有被重复消费?怎么处理消息丢失情况?那么保证消息传递的顺序性?

- **一致性问题**

  A系统外理完业务，通过MQ给B、C、D三个系统发消息数据如果B 系统、C系统外理成功，D 系统外理失败。如何保证消息数据处理的一致性?



## 1.3使用MQ需要满足的条件

生产者不需要从消费者处获得反馈。引入消息队列之前的直接调用，其接口的返回值应该为空，这才让明明下层的动作还没做，上层却当成动作做完了继续往后走，即所谓异步成为了可能。

容许短暂的不一致性

确实是用了有效果。即解耦、提速、削峰这些方面的收益，超过加入MQ，管理MQ这些成本



# 2.RabbitMQ



2007年，Rabbit 技术公司基于AMQP 标准开发的 RabbitMQ 1.0 发布。RabbitMQ 采用 Erlang 语言开发Erlang 语言由 Ericson 设计，专门为开发高并发和分布式系统的一种语言，在电信领域使用广泛

RabbitMQ基础架构如下图:

<img src="img\image-20221115144745140.png" alt="image-20221115144745140" style="zoom:50%;" />



RabbitMQ中的相关概念：

- **Broker:** 接收和分发消息的应用，RabbitMQ Server就是 Message Broker
- **Virtual host:** 出于多租户和安全因素设计的，把AMQP 的基本组件划分到一个虚拟的分组中，类似于网络中的 namespace 概念。当多个不同的用户使用同一个RabbitMQ server 提供的服务时，可以划分出多个vhost，每个用户在自己的 vhost 创建exchange/queue等
- **Connection:** publisher / consumer 和 broker 之间的 TCP 连接
- **Channel:** 如果每一次访问 RabbitMQ 都建立一个Connection，在消息量大的时候建立TCP Connection的开销将是巨大的，效率也较低。Channel 是在connection 内部建立的罗辑连接，如果应用程席支持多线程，通常每个thread创建单独的 channel 进行通讯，AMQP method 包含了channelid 帮助客户端和message broker识别 channel，所以channel 之间是完全隔离的。Channel作为轻量级的 Connection极大减少了操作系统建立TCP connection的开销
- **Exchange:** message 到达 broker 的第一站，根据分发规则，匹配查询表中的 routing key，分发消息到queue 中去。常用的类型有: direct (point-to-point), topic (publish-subscribe) and fanout (multicast)
- **Queue:** 消息最终被送到这里等待 consumer 取走
- **Binding:** exchange 和 queue 之间的虚拟连接，binding 中可以包含 routing key。Binding 信息被保存到exchange中的查询表中，用于 message的分发依据



**RabbitMQ 提供了 6 种工作模式**: 简单模式、work queues、Publish/Subscribe 发布与订阅模式、Routing路由模式、Topics 主题模式、RPC 远程调用模式(远程调用，不太算 MQ;暂不作介绍)。官网对应模式介绍:https://wwwrabbitmq.com/getstarted.html



## 2.1AMQP

AMQP，即Advanced Message Queuing Protocol (高级消息队列协议)，是一个网络协议，是应用层协议的一个开放标准，为面向消息的中间件设计。基于此协议的客户端与消息中间件可传递消息，并不受客户端/中间件不同产品，不同的开发语言等条件的限制。2006年，AMQP 规范发布。类比HTTP

![image-20221122111554962](img\image-20221122111554962.png)



## 2.2 docker启动RabbitMQ

不使用docker安装并使用MQ -->  [RabbitMQ安装说明文档.md](RabbitMQ安装说明文档.md) 

```markdown
1.拉取rabbitmq
docker pull rabbitmq

2.启动rabbitmq
docker run -id -p 15672:15672 -p 5672:5672 \
> -e RABBITMQ_DEFAULT_USER=admin \
> -e RABBITMQ_DEFAULT_PASS=admin \
> --name rabbitmq \
> rabbitmq:3.11.3

RABBITMQ_DEFAULT_USER：默认的用户名；
RABBITMQ_DEFAULT_PASS：默认的用户密码；

3.进入rabbitmq管理控制台
先进入mq容器内部
docker exec -it 容器名称 /bin/bash
然后开启
rabbitmq-plugins enable rabbitmq_management
最后输入网址进入控制台
http://服务器ip:15672
```



# 3.rabbitmq工作模式

## 3.1rabbitmq小演示(简单模式)

1.创建生产者maven工程，并发送消息

```java
public class ProducerHello {

    public static void main(String[] args) throws IOException, TimeoutException {

        // 创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();

        // 设置参数
        factory.setHost("192.168.108.128");
        factory.setPort(5672);
        factory.setVirtualHost("/");
        factory.setUsername("husky");
        factory.setPassword("123456");

        // 创建连接
        Connection connection = factory.newConnection();

        // 创建channel
        Channel channel = connection.createChannel();

        // 创建Queue(队列)
        /*
        * Params:
                queue – 队列名称
                durable – 是否声明为持久队列
                exclusive – 是否独占队列
                *       只能有一个消费者监听这个队列
                *       当connection关闭时，是否删除
                autoDelete – 是否自动删除，当没有consumer时，删除队列
                arguments – 队列的其他属性（构造参数）
                * */
        // 若队列名称不存在，这创建
        channel.queueDeclare("Hello_mq",true,false,false,null);

        // 发送消息
        String body = "hello, rabbitmq";
        /*
        * Params:
                exchange – 交换机名称，简单模式下交换机名称会使用默认的 ""
                routingKey – 路由名称（若在简单模式下，路由名称要与队列名称一样）
                props – 配置信息
                body – 发送的消息数据
                * */
        channel.basicPublish("", "Hello_mq", null, body.getBytes());

        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        // 释放资源
        /*connection.close();
        channel.close();*/
    }
```



2.创建消费者maven工程，并接收消息

```java
public class ConsumerHello {
    public static void main(String[] args) throws IOException, TimeoutException {
        // 创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();

        // 设置参数
        factory.setHost("192.168.108.128");
        factory.setPort(5672);
        factory.setVirtualHost("/");
        factory.setUsername("husky");
        factory.setPassword("123456");

        // 创建连接
        Connection connection = factory.newConnection();

        // 创建channel
        Channel channel = connection.createChannel();

        // 创建Queue(队列)
        /*
        * Params:
                queue – 队列名称
                durable – 是否声明为持久队列
                exclusive – 是否独占队列
                *       只能有一个消费者监听这个队列
                *       当connection关闭时，是否删除
                autoDelete – 是否自动删除，当没有consumer时，删除队列
                arguments – 队列的其他属性（构造参数）
                * */
        // 若队列名称不存在，这创建
        channel.queueDeclare("Hello_mq",true,false,false,null);

        // 接收消息
        Consumer consumer = new DefaultConsumer(channel){
            /*
            * 回调方法，当收到消息后，会自动执行该方法
            * consumerTag：标识
            * envelope：获取信息，比如交换机、路由key
            * properties：配置信息
            * body：数据
            * */
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("consumerTag:"+consumerTag);
                System.out.println("envelope:"+envelope);
                System.out.println("properties:"+properties);
                System.out.println("body:"+new String(body));
            }
        };
        /*
        * Params:
                queue – 队列名称
                autoAck – 是否自动确认
                callback – 回调对象*/
        channel.basicConsume("Hello_mq",false,consumer);
    }
```

![image-20221123105255463](img\image-20221123105255463.png)

在上图的模型中，有以下概念:

- P:生产者，也就是要发送消息的程序
- C:消费者:消息的接收者，会一直等待消息到来
- queue:消息队列，图中红色部分。类似一个邮箱，可以缓存消息，生产者向其中投递消息，消费者从其中取出消息



## 3.2Work Queue工作队列模式

![image-20221123110456594](img\image-20221123110456594.png)

1.在一个队列中如果有多个消费者，那么消费者之间对于同一个消息的关系是竞争的关系(轮询)

2.Work Queues 对于任务过重或任务较多情况使用工作队列可以提高任务处理的速度。例如: 短信服务部署多个只需要有一个节点成功发送即可。



## 3.3Pub/Sub订阅模式

![image-20221123150902195](img\image-20221123150902195.png)

1. 在订阅模型中，多了一个Exchange角色，而且过程略有变化:

2. P:生产者，也就是要发送消息的程序，但是不再发送到队列中，而是发给X(交换机)

3. C:消费者，消息的接收者，会一直等待消息到来

4. Querle: 消息队列，接收消息，缓存消息

5. Exchange:交换机()。一方面，接收主产者发送的消息。另一方面，知道如何处理消息，例如递交给某个特别队列递交给所有队列、或是将消息丢弃。到底如何操作，取决于Exchange的类型。Exchange有常见以下3种类型：

   - Fanout：广播，将消息文给所有绑定到女换机的队列

   - Direct：定向，把消息交给符合指定routing key 的队列

   - Topic：通配符，把消息交给符合routingpattern(路由模式)的队列

Exchange(交换机)只负责转发消息，不具备存储消息的能力，因此如果没有任何队列与 Exchange 绑定，或者没有符合路由规则的队列，那么消息会丢失!



生产者：

```java
public class Producer_PubSub {
    public static void main(String[] args) throws IOException, TimeoutException {
        // 创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();

        // 设置参数
        factory.setHost("192.168.108.128");
        factory.setPort(5672);
        factory.setVirtualHost("/");
        factory.setUsername("husky");
        factory.setPassword("123456");

        // 创建连接
        Connection connection = factory.newConnection();

        // 创建channel
        Channel channel = connection.createChannel();

        // 创建交换机
        /*
        * Params:
                exchange – 交换机名称
                type – 交换机类型
                *   DIRECT("direct")：定向
                *   FANOUT("fanout")：扇形（广播），发送消息到每一个与之绑定的队列
                *   TOPIC("topic")：通配符方式
                *   HEADERS("headers")：参数匹配
                durable – 是否持久化
                autoDelete – 是否自动删除
                internal – 内部使用，一般为false
                arguments – 参数
                * */
        String exChangeName = "test_fanout";
        channel.exchangeDeclare(exChangeName, BuiltinExchangeType.FANOUT, true, false, false, null);

        // 创建队列
        String queueName1 = "test_fanout_queue1";
        String queueName2 = "test_fanout_queue2";
        channel.queueDeclare(queueName1, true, false, false, null);
        channel.queueDeclare(queueName2, true, false, false, null);

        // 绑定队列和交换机
        /*
        * Params:
            queue – 队列名称
            exchange – 交换机名称
            routingKey – 路由绑定规则
            *   如果交换机类型为fanout，routingKey设置为""
            * */
        channel.queueBind(queueName1, exChangeName, "");
        channel.queueBind(queueName2, exChangeName, "");

        // 发送消息
        String body = "日志消息哦";
        channel.basicPublish(exChangeName, "", null, body.getBytes(StandardCharsets.UTF_8));

        // 释放资源
        channel.close();
        connection.close();
    }
}

```



消费者：

```java
// 创建队列可不写
// 消费者只需将接收消息的队列名称改为生产者的队列名称即可 
channel.basicConsume(queueName1,false,consumer);
```



## 3.4 Routing 路由模式

![image-20221123161907183](img\image-20221123161907183.png)

1.模式说明

- 队列与交换机的绑定，不能是任意绑定了，而是要指定一个RoutingKey (路由key)
- 消息的发送方在向Exchange发送消息时，也必须指定消息的RoutingKey
- Exchange 不再把消息交给每一个绑定的队列，而是根据消息的 Routing Key 进行判断，只有队列的Routingkey 与消息的 Routing key 完全一致，才会接收到消息



生产者：

```java
// 绑定队列和交换机
/*
        * Params:
            queue – 队列名称
            exchange – 交换机名称
            routingKey – 路由绑定规则
            *   如果交换机类型为fanout，routingKey设置为""
            * */
channel.queueBind(queueName1, exChangeName, "error");
channel.queueBind(queueName2, exChangeName, "info");
channel.queueBind(queueName2, exChangeName, "error");
channel.queueBind(queueName2, exChangeName, "warning");

// 发送消息
String body = "Routing日志消息";
channel.basicPublish(exChangeName, "error", null, body.getBytes(StandardCharsets.UTF_8));
```

消费者只能接收到队列的Routingkey 与消息的 Routing key 完全一致的消息



## 3.5topics通配符模式

![image-20221123165643263](img\image-20221123165643263.png)



Topic 主题模式可以实现 Pub/Sub 发布与订阅模式和 Routing 路由模式的功能，只是Topic 在配置routing key的时候可以使用通配符，显得更加灵活。

*：代表一个单词

#：代表0个或多个单词

生产者：

```java
 // 绑定队列和交换机
/*
        * Params:
            queue – 队列名称
            exchange – 交换机名称
            routingKey – 路由绑定规则
            *   如果交换机类型为fanout，routingKey设置为""
            * */
channel.queueBind(queueName1, exChangeName, "*.error");
channel.queueBind(queueName2, exChangeName, "order.*");
channel.queueBind(queueName2, exChangeName, "*.error");

// 发送消息
String body = "Routing日志消息";
channel.basicPublish(exChangeName, "order.info", null, body.getBytes(StandardCharsets.UTF_8));
```



# 4.spring整合rabbitmq

## 4.1.添加依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.2.5.RELEASE</version>
    </dependency>

    <dependency>
        <groupId>org.springframework.amqp</groupId>
        <artifactId>spring-rabbit</artifactId>
        <version>2.4.7</version>
    </dependency>

    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>5.2.5.RELEASE</version>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.8.0</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </plugin>
    </plugins>
</build>

```



## 4.2.rabbitmq.properties资源文件

```properties
rabbitmq.host=192.168.108.128
rabbitmq.port=5672
rabbitmq.username=husky
rabbitmq.password=123456
rabbitmq.virtual-host=/
```



## 4.3生产者配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:rabbit="http://www.springframework.org/schema/rabbit"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/rabbit
       http://www.springframework.org/schema/rabbit/spring-rabbit.xsd">
    <!--加载配置文件-->
    <context:property-placeholder location="classpath:/rabbitmq.properties"/>

    <!-- 定义rabbitmq connectionFactory -->
    <rabbit:connection-factory id="connectionFactory" host="${rabbitmq.host}"
                               port="${rabbitmq.port}"
                               username="${rabbitmq.username}"
                               password="${rabbitmq.password}"
                               virtual-host="${rabbitmq.virtual-host}"/>
    <!--定义管理交换机、队列-->
    <rabbit:admin connection-factory="connectionFactory"/>

    <!--定义持久化队列，不存在则自动创建；不绑定到交换机则绑定到默认交换机
    默认交换机类型为direct，名字为：""，路由键为队列的名称
    -->
    <rabbit:queue id="spring_queue" name="spring_queue" auto-declare="true"/>

    <!-- ~~~~~~~~~~~~~~~~~~~~~~~~~~~~广播；所有队列都能收到消息~~~~~~~~~~~~~~~~~~~~~~~~~~~~ -->
    <!--定义广播交换机中的持久化队列，不存在则自动创建-->
    <rabbit:queue id="spring_fanout_queue_1" name="spring_fanout_queue_1" auto-declare="true"/>

    <!--定义广播交换机中的持久化队列，不存在则自动创建-->
    <rabbit:queue id="spring_fanout_queue_2" name="spring_fanout_queue_2" auto-declare="true"/>

    <!--定义广播类型交换机；并绑定上述两个队列-->
    <rabbit:fanout-exchange id="spring_fanout_exchange" name="spring_fanout_exchange" auto-declare="true">
        <rabbit:bindings>
            <rabbit:binding queue="spring_fanout_queue_1"/>
            <rabbit:binding queue="spring_fanout_queue_2"/>
        </rabbit:bindings>
    </rabbit:fanout-exchange>
<!-- ~~~~~~~~~~~~~~~~~~~~~~~~~~~~Routing路由模式~~~~~~~~~~~~~~~~~~~~~~~~~~~~ -->
    <!--<rabbit:direct-exchange name="aa">
        <rabbit:bindings>
            <rabbit:binding key="error" queue="spring_fanout_queue_1"/>
        </rabbit:bindings>
    </rabbit:direct-exchange>-->
    <!-- ~~~~~~~~~~~~~~~~~~~~~~~~~~~~通配符；*匹配一个单词，#匹配多个单词 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~ -->
    <!--定义广播交换机中的持久化队列，不存在则自动创建-->
    <rabbit:queue id="spring_topic_queue_star" name="spring_topic_queue_star" auto-declare="true"/>
    <!--定义广播交换机中的持久化队列，不存在则自动创建-->
    <rabbit:queue id="spring_topic_queue_well" name="spring_topic_queue_well" auto-declare="true"/>
    <!--定义广播交换机中的持久化队列，不存在则自动创建-->
    <rabbit:queue id="spring_topic_queue_well2" name="spring_topic_queue_well2" auto-declare="true"/>

    <rabbit:topic-exchange id="spring_topic_exchange" name="spring_topic_exchange" auto-declare="true">
        <rabbit:bindings>
            <rabbit:binding pattern="error.*" queue="spring_topic_queue_star"/>
            <rabbit:binding pattern="orders.#" queue="spring_topic_queue_well"/>
            <rabbit:binding pattern="husky.#" queue="spring_topic_queue_well2"/>
        </rabbit:bindings>
    </rabbit:topic-exchange>

    <!--定义rabbitTemplate对象操作可以在代码中方便发送消息-->
    <rabbit:template id="rabbitTemplate" connection-factory="connectionFactory"/>
</beans>
```



## 4.4.生产者测试

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:/spring-rabbit-producer.xml")
public class ProducerRabbitMq {

    // 注入 RabbitTemplate
    @Resource
    private RabbitTemplate rabbitTemplate;

    @Test
    public void testHelloWorld() {
        // 简单模式发送消息
        rabbitTemplate.convertAndSend("spring_queue","helloWorld spring~~");
    }

    @Test
    public void testFanout() {
        // 广播模式发送消息
        rabbitTemplate.convertAndSend("spring_fanout_exchange", "", "rabbitmq fanout~~");
    }

    @Test
    public void testTopics() {
        // 通配符模式发送消息
        rabbitTemplate.convertAndSend("spring_topic_exchange", "orders.io.null", "rabbitmq topic~~");
    }
}
```

## 4.5消费者配置文件

消费者注册监听器，并实现MessageListener接口

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:rabbit="http://www.springframework.org/schema/rabbit"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/rabbit
       http://www.springframework.org/schema/rabbit/spring-rabbit.xsd">
    <!--加载配置文件-->
    <context:property-placeholder location="classpath:/rabbitmq.properties"/>

    <!-- 定义rabbitmq connectionFactory -->
    <rabbit:connection-factory id="connectionFactory" host="${rabbitmq.host}"
                               port="${rabbitmq.port}"
                               username="${rabbitmq.username}"
                               password="${rabbitmq.password}"
                               virtual-host="${rabbitmq.virtual-host}"/>

    <bean id="springQueueListener" class="com.husky.rabbitmq.listener.SpringQueueListener"/>
<!--    <bean id="fanoutListener1" class="com.husky.rabbitmq.listener.FanoutListener1"/>
    <bean id="fanoutListener2" class="com.husky.rabbitmq.listener.FanoutListener2"/>
    <bean id="topicListenerStar" class="com.husky.rabbitmq.listener.TopicListenerStar"/>
    <bean id="topicListenerWell" class="com.husky.rabbitmq.listener.TopicListenerWell"/>
    <bean id="topicListenerWell2" class="com.husky.rabbitmq.listener.TopicListenerWell2"/>-->

    <rabbit:listener-container connection-factory="connectionFactory" auto-declare="true">
        <rabbit:listener ref="springQueueListener" queue-names="spring_queue"/>
<!--        <rabbit:listener ref="fanoutListener1" queue-names="spring_fanout_queue_1"/>
        <rabbit:listener ref="fanoutListener2" queue-names="spring_fanout_queue_2"/>
        <rabbit:listener ref="topicListenerStar" queue-names="spring_topic_queue_star"/>
        <rabbit:listener ref="topicListenerWell" queue-names="spring_topic_queue_well"/>
        <rabbit:listener ref="topicListenerWell2" queue-names="spring_topic_queue_well2"/>-->
    </rabbit:listener-container>
</beans>
```

```java
public class SpringQueueListener implements MessageListener {
    @Override
    public void onMessage(Message message) {
        System.out.println(new String(message.getBody()));
    }
}
```

## 4.6消费者测试

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:spring-rabbitmq-consumer.xml")
public class ConsumerTest {
// 直接运行即可，只要能运行监听器的类，启动监听器，就可以接收到消息
    @Test
    public void test() {
        try {
            Thread.sleep(5 * 60 * 1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```



# 5.springBoot整合rabbitmq

## 5.1 生产者

1.引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```



2.配置文件

```properties
spring:
  rabbitmq:
    host: 192.168.108.128
    port: 5672
    username: admin
    password: admin
    virtual-host: /
```



3.配置类

```java
@Configuration
public class RabbitMqConfig {

    public static final String EXCHANGE_NAME = "boot_topic_exchange";
    public static final String QUEUE_NAME = "boot_queue";

    // 交换机
    @Bean("bootExchange")
    public Exchange bootExchange() {
        return ExchangeBuilder.topicExchange(EXCHANGE_NAME).build();
    }

    // 队列
    @Bean("bootQueue")
    public Queue bootQueue() {
        return QueueBuilder.durable(QUEUE_NAME).build();
    }

    // 队列和交换机绑定关系
    @Bean
    public Binding bindExchangeQueue(@Qualifier("bootQueue") Queue queue, @Qualifier("bootExchange") Exchange exchange) {
        return BindingBuilder.bind(queue).to(exchange).with("boot.#").noargs();
    }
}
```



4.编写测试类

```java
@SpringBootTest
public class ProducerTest {
    @Resource
    private RabbitTemplate rabbitTemplate;
    @Test
    public void sendTest() {
        rabbitTemplate.convertAndSend(RabbitMqConfig.EXCHANGE_NAME,"boot.test","spring boot rabbitmq tess ~~");
    }
}
```



## 5.2消费者

1.依赖和配置文件与生产者一样

2.编写监听器

```java
@Component
public class RabbitMqListener {

    @RabbitListener(queues = "boot_queue")
    public void listenerRabbitMq(Message message){
        // System.out.println(message);
        System.out.println(new String(message.getBody()));
    }
}
```

3.启动消费者（启动类），监听队列消息

## 5.3小结

- SpringBoot提供了快速整合RabbitMQ的方式
- 基本信息再yml中配置，队列交互机以及绑定关系在配置类中使用Bean的方式配置
- 生产端直接注入RabbitTemplate完成消息发送
- 消费端直接使用@RabbitListener完成消息接收



# 6.RabbitMq的高级特性

## 6.1消息的可靠传递

在使用 RabbitMQ 的时候，作为消息发送方希望杜绝任何消息丢失或者投递失败场景。RabbitMQ为我们提供了两种方式用来控制消息的投递可靠性模式。

- **confirm 确认模式**

  - ```java
    /*
        * 确认模式开启：
        *   步骤：
        *       1、确认模式开启：spring.rabbitmq.publisher-confirm-type=correlated
        *       2、在rabbitTemplate定义ConfirmCallback回调函数
        */
    @Test
    public void sendConfirm() {
        // 定义回调
        rabbitTemplate.setConfirmCallback(new RabbitTemplate.ConfirmCallback() {
            /*
                * 第一个参数：相关配置信息
                * 第二个参数：交换机是否成功收到消息
                * 第三个参数：失败的原因
                * */
            @Override
            public void confirm(CorrelationData correlationData, boolean b, String s) {
                System.out.println("confirm被执行、、、");
                if (b) {
                    System.out.println("执行成功"+s);
                }else {
                    System.out.println("执行失败"+s);
                }
            }
        });
    
        // 发送消息
        rabbitTemplate.convertAndSend(RabbitMqConfig.EXCHANGE_NAME_CONFIRM,"boot.confirm","messages confirm ~~~");
    }
    ```

- **return 退回模式**

  - 回退模式: 当消息发送给Exchange后，Exchange路由到Queue失败是 才会执行 ReturnCallBack

  - 步骤:
    开启回退模式:publisher-returns="true"
    设置ReturnCalLBack
    设置Exchange处理消息的模式:

    ​		1.如果消息没有路由到Queue，则丢弃消息（默认)

    ​		2.如果消息没有路由到Queue，返回给消息发送方ReturnCalLBack

    代码示例：

    ```java
    @Test
    public void sendReturn() {
        // 设置交换机处理失败的消息模式
        rabbitTemplate.setMandatory(true);
    
        rabbitTemplate.setReturnsCallback(new RabbitTemplate.ReturnsCallback() {
            // 参数：处理失败退回原因
            @Override
            public void returnedMessage(ReturnedMessage returned) {
                System.out.println("returnedMessage执行了");
                System.out.println(returned);
            }
        });
        // 发送消息
        rabbitTemplate.convertAndSend(RabbitMqConfig.EXCHANGE_NAME_CONFIRM,"bootconfirm","messages confirm ~~~");
    }
    ```

    



rabbitmq整个消息投递的路径为:

producer--->rabbitmq broker--->exchange--->queue--->consumer

- 消息从 producer到exchange 则会返回一个 confirmCallback
- 消息从exchange-->queue 投递失败则会返回一个returnCallback。

我们将利用这两个callback 控制消息的可靠性投递



## 6.2Consumer Ack

ack指Acknowledge，确认。表示消费端收到消息后的确认方式有三种确认方式

- 自动确认:acknowledge="none"

- 手动确认:acknowledge="manual”

- 根据异常情况确认:acknowledge="auto”，

- ```properties
  spring:
    rabbitmq:
      listener:
        direct:
          acknowledge-mode: manual
  ```

- 

其中自动确认是指，当消息一旦被Consumer接收到，则自动确认收到，并将相应message从RabbitMQ的消息缓存中移除。但是在实际业务处理中，很可能消息接收到，业务处理出现异常，那么该消息就会丢失。如果设置了手动确认方式，则需要在业务处理成功后，调用channel.basicAck()，手动签收，如果出现异常，则调用channel.basicNack()方法，让其自动重新发送消息

消费端监听器

```java
/*
     * Consumer Ack 机制：
     *   1.设置手动签收。acknowledge="manual"
     *   2.让监听器类实现ChannelAwareMessageListener接口
     *   3.如果消息成功处理，则调用channel的basicAck()签收
     *   3.如果消息处理失败，则调用channel的basicNack()拒绝签收，broker重新发送到consumer
     * */
@Override
@RabbitListener(queues = "boot_queue_confirm")
public void onMessage(Message message, Channel channel) throws Exception {

    // 获取deliveryTag
    long deliveryTag = message.getMessageProperties().getDeliveryTag();
    try {
        // 接受转换消息
        System.out.println(new String(message.getBody()));

        // 处理业务逻辑
        System.out.println("处理业务逻辑");
        // 手动签收
        channel.basicAck(deliveryTag,true);
    } catch (IOException e) {
        // 拒绝签收
        // 第三个参数：requeue：重回队列，如果设置为true，若消息被拒绝，则消息重新回到queue，broker重新发送到consumer
        channel.basicNack(deliveryTag,true,true);
    }
}
```



## 6.3 消费端限流

```java
@Component
public class QosListener implements ChannelAwareMessageListener {
    /*
     * 开启Consumer限流：
     *   1.确保ack为手动确认
     *   2.spring:
              rabbitmq:
                listener:
                  direct:
                    prefetch: 1
     *     表示消费端每次从mq拉取一条来消费，直到手动消费确认完毕后，才会继续拉取下一条消息
     * */
    @Override
    @RabbitListener(queues = "boot_queue_confirm")
    public void onMessage(Message message, Channel channel) throws Exception {
        Thread.sleep(5000);
        // 获取消息
        System.out.println(new String(message.getBody(), StandardCharsets.UTF_8));
        // 处理业务逻辑
        System.out.println("限流 处理业务逻辑~~");
        // 签收
        channel.basicAck(message.getMessageProperties().getDeliveryTag(),true);
    }
}
```



## 6.4 TTL

- TTL全称 Time To Live (存活时间/过期时间)
- 当消息到达存活时间后，还没有被消费，会被自动清除
- RabbitMQ可以对消息设置过期时间，也可以对整个队列(Queue)设置过期时间



1. 设置队列过期时间在设置队列时使用ttl方法，单位: ms(毫秒)，会对整个队列消息统一过期

   ```java
   // 队列
   @Bean("bootQueueTTL")
   public Queue bootQueueConfirm() {
       return QueueBuilder.durable(QUEUE_TTL).ttl(10000).build();
   }
   ```

   

2. 设置消息过期时间使用参数:**expiration**。单位:ms(毫秒)，当该消息在队列头部时 (消费时)，会单独判断这一消息是否过期。

   比如有10条消息，有五条消息设置了过期时间，另五条没有，每条消息被消费时都会判断这一消息是否过期，若设置队列

   过期时间，则时间一到，队列中的消息将会被清除掉。

   ```java
   @Test
   public void sendTTL() {
       MessagePostProcessor messagePostProcessor = new MessagePostProcessor() {
           @Override
           public Message postProcessMessage(Message message) throws AmqpException {
               // 1.设置message的过期时间
               message.getMessageProperties().setExpiration("5000");
   
               // 2.一定要返回该消息
               return message;
           }
       };
       rabbitTemplate.convertAndSend(RabbitMqConfig.EXCHANGE_TTL,"ttl.test","messages ttl ~~~", messagePostProcessor);
   }
   ```

3. 如果两者都进行了设置，以时间短的为准



## 6.5死信队列

死信队列，英文缩写: DLX。Dead Letter Exchange (死信交换机)，当消息成为Dead message后，可以被重新发送到另一个交换机，这个交换机就是DLX

大部分MQ是没有交换机的概念的，所以叫做死信队列，但rabbitmq有交换机，所以在rabbitmq中一般叫做死信交换机

![image-20221206102324324](img\image-20221206102324324.png)

1.消息成为死信的情况：

- 队列消息长度到达限制;
- 消费者拒接消费消息，basiNack/basicReject,并且不把消息重新放入原目标队列,requeue=false

```java
// 拒绝签收，不重回队列 第三个参数 requeue=false
channel.basicNack(message.getMessageProperties().getDeliveryTag(),true,false);
```



- 原队列存在消息过期设置，消息到达超时时间未被消费



2.正常队列绑定死信交换机

```java
/*
 * 1.在正常队列中绑定死信交换机
 * 2.正常队列发送死信给死信交换机*/
@Bean("bootQueueDlx")
public Queue bootQueueDlx() {
    return QueueBuilder.durable(BOOT_QUEUE_DLX).deadLetterExchange(EXCHANGE_DLX)
        .deadLetterRoutingKey("dlx.hello")
        .ttl(10000)
        .maxLength(10)
        .build();
}
```



## 6.6延迟队列

延迟队列，即消息进入队列后不会立即被消费，只有到达指定时间后，才会被消费

但在RabbitMQ中并未提供延迟队列功能，但是可以使用: **TTL+死信队列 组合实现延迟队列的效果**

<img src="img\image-20221206145326543.png" alt="image-20221206145326543"  />



实现延迟队列的效果 必须要监听死信队列才能实现



## 6.7消息追踪

**FireHost：**

firehose的机制是将生产者投递给rabbitmg的消息，rabbitmq投递给消费者的消息按照指定的格式发送到默认的exchange上。这个默认的exchange的名称为amq.rabbitmq.trace，它是一个topic类型的exchange。发送到这个exchange上的消息的routing key为 publish.exchangename和deliver.queuename。其中exchangename和queuename为实际exchange和queue的名称，分别对应生产者投递到exchange的消息，和消费者从queue上获取的消息。

注意:打开 trace会影响消息写入功能，适当打开后请关闭。

- rabbitmgctl trace on:开启Firehose命令
- rabbitmqctl trace off: 关闭Firehose命令



**rabbitmq_tracing:**
rabbitmq tracing和Firehose在实现上如出一辙，只不过rabbitmg tracing的方式比Firehose多了一层GUI的包装，更容易使用和管理

启用插件:rabbitmq-plugins enable rabbitmq_tracing

可以在开发和测试环境用，若要使用在生产环境可能会影响性能。



## 6.8消息的可靠性保障-消息补偿

​	![image-20221206155545418](img\image-20221206155545418.png)

## 6.9消息的幂等性保障

​	
