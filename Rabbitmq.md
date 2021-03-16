## MQ概述

> + MQ，消息队列，存储消息的中间件
> + 分布式系统通信两种方式：直接远程调用 和 借助第三方 完成间接通信
> + 发送方称为生产者Producer，存储临时消息的称之为中间件(MQ)，接收方称为消费者(Consumer)

<img src="../java-notes/image/image-20210120155313911.png" alt="image-20210120155313911" style="zoom:67%;" />

### 优势

+ 应用解耦。提升容错性和可维护性

  <img src="../java-notes/image/image-20210120161123500.png" alt="image-20210120161123500" style="zoom:67%;" />

+ 异步提速，提升用户体验和系统吞吐量（单位时间内处理请求的数目）

  > 生产者不需要从消费者获得反馈

  <img src="../java-notes/image/image-20210120161210563.png" alt="image-20210120161210563" style="zoom:67%;" />

+  削峰填谷，提高系统稳定性

  <img src="../java-notes/image/image-20210120161254416.png" alt="image-20210120161254416" style="zoom:67%;" />

### 应用场景举例

+ **双十一商品秒杀抢票功能实现**
+ **用户注册**

## RabbitMQ

### AMQP

> Advanced Message Queuing Protocol（高级消息队列协议），是一个网络协议，是应用层协议的一个开放标准，为面向消息的中间件设计。（类似HTTP协议）

![image-20210120164221240](../java-notes/image/image-20210120164221240.png)

### 相关概念

![image-20210120164258022](../java-notes/image/image-20210120164258022.png)

+ **VHost**：虚拟主机，一个broker里可以开设多个vhost，用作不同用户的权限分离。
+ **Connection：**publisher／consumer 和 broker 之间的 TCP 连接
+ **Channel：** 消息通道，在客户端的每个连接里，可建立多个channel，每个channel代表一个会话任务
+ **Exchange**：消息交换机，它指定消息按什么规则，路由到哪个队列
+ **Queue**：消息队列载体，每个消息都会被投入到一个或多个队列

### 安装及使用

#### Docker中安装

下载镜像  docker pull rabbitmq:management

创建并运行容器 

docker run -di --name=rabbitmq -p 5671:5617 -p 5672:5672 -p4369:4369 -p 15671:15671 -p 15672:15672 -p 25672:25672 rabbitmq:management

```
解释如下：
15672 (if management plugin is enabled.管理界面 )

15671 management监听端口

5672, 5671 (AMQP 0-9-1 without and with TLS 消息队列协议是一个消息协议)

4369 (epmd) epmd 代表 Erlang 端口映射守护进程

25672 (Erlang distribution)
```

浏览器中输入地址  ip:15672

设置容器开机自动启动  docker update --restart=always 容器ID







### 工程搭建

![image-20210122201544811](../java-notes/image/image-20210122201544811.png)

#### 相关组件创建

方式一：直接在管理平台创建User,Virtual host(添加用户)，**Queue,Exchange**(创建时指定Virtual host，Type并绑定队列)

方式二：通过代码创建 **Queue,Exchange**，绑定关系

```java
//Fanout广播模式
@Configuration
public class RabbitFanoutConfig {
    //exchange名称
    public static final String FANOUTNAME = "javaboy-fanout";
    @Bean
    Queue queueOne() {
        // durable:是否持久化,默认是false,持久化队列：会被存储在磁盘上，当消息代理重启时仍然存在，暂存队列：当前连接有效
        // exclusive:默认也是false，只能被当前创建的连接使用，而且当连接关闭后队列即被删除。此参考优先级高于durable
        // autoDelete:是否自动删除，当没有生产者或者消费者使用此队列，该队列会自动删除。
        //return new Queue("TestDirectQueue",true,true,false);
        return new Queue("queue-one");
    }

    @Bean
    Queue queueTwo() {
        return new Queue("queue-two");
    }

    @Bean
    FanoutExchange fanoutExchange() {
        return new FanoutExchange(FANOUTNAME, true, false);
    }

    @Bean 
    Binding bindingOne() {
        return BindingBuilder.bind(queueOne()).to(fanoutExchange());
    }
    @Bean
    Binding bindingTwo() {
        return BindingBuilder.bind(queueTwo()).to(fanoutExchange());
    }
}
```

```java
//derect直连模式
@Configuration
public class RabbitDirectConfig {
    public final static String DIRECTNAME = "javaboy-direct";

    @Bean
    Queue queue() {
        return new Queue("hello.javaboy");
    }
    @Bean
    DirectExchange directExchange() {
        return new DirectExchange(DIRECTNAME, true, false);
    }

    @Bean
    Binding binding() {
        return BindingBuilder.bind(queue()).to(directExchange()).with("direct");
    }
}
```

```java
//topic通配符模式
@Configuration
public class RabbitTopicConfig {
    public static final String TOPICNAME = "javaboy-topic";
    @Bean
    Queue xiaomi() {
        return new Queue("xiaomi");
    }

    @Bean
    Queue huawei() {
        return new Queue("huawei");
    }

    @Bean
    Queue phone() {
        return new Queue("phone");
    }
    
    @Bean
    TopicExchange topicExchange() {
        return new TopicExchange(TOPICNAME, true, false);
    }

    @Bean
    Binding xiaomiBinding() {
        return BindingBuilder.bind(xiaomi()).to(topicExchange()).with("xiaomi.#");
    }

    @Bean
    Binding huaweiBinding() {
        return BindingBuilder.bind(huawei()).to(topicExchange()).with("huawei.#");
    }
    @Bean
    Binding phoneBinding() {
        return BindingBuilder.bind(phone()).to(topicExchange()).with("#.phone.#");
    }
}
```

#### 生产者工程

 pom文件

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.9.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
<dependencies>
    <!--amqp协议的起步依赖坐标-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-amqp</artifactId>
    </dependency>
    <!--rabbit测试依赖坐标-->
    <dependency>
        <groupId>org.springframework.amqp</groupId>
        <artifactId>spring-rabbit-test</artifactId>
        <scope>test</scope>
    </dependency>
    <!--SpringBoot测试依赖坐标-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

配置文件application.properties

```properties
# RabbitMQ 服务host地址(虚拟机ip地址)
spring.rabbitmq.host=192.168.200.128
# 端口
spring.rabbitmq.port=5672
# 虚拟主机地址
spring.rabbitmq.virtual-host=/itheima（可不设置，默认/)
# rabbit服务的用户名
spring.rabbitmq.username=heima
# rabbit服务的密码
spring.rabbitmq.password=heima
```

启动类

```java
@SpringBootApplication
public class SpringbootRabbitmqProducerApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringbootRabbitmqProducerApplication.class, args);
    }
}
```

##### **发送消息**

```java
@Autowired
private RabbitTemplate rabbitTemplate;

/**
 * 参数1：消息队列名称
 * 参数2：消息内容
 */
rabbitTemplate.convertAndSend("simple_queue","hello 小兔子！");
 /**
  * 参数1：设置交换机
  * 参数2：设置路由键，
  		广播模式，不设置路由键，默认是空字符串
  		路由模式，设置路由键(error，info)
  		通配符模式，设置路由键(item.#，item.*)
  * 参数3：设置发送的消息内容
  */
rabbitTemplate.convertAndSend("fanout_exchange","","你太坏了，小兔子【"+i+"】");
rabbitTemplate.convertAndSend("routing_exchange","info","[info]你太坏了，小兔子【"+i+"】");
rabbitTemplate.convertAndSend("topic_exchange","item.insert.abc","路由键[item.insert.abc]你太坏了，小兔子");

```



##### **接受消息**（可以有多个监听器）

```java
/**
 * 消费者，接收消息队列消息监听器
 * 必须将当前监听器对象注入Spring的容器中
 *会一直等待消息接受
 *跟工作模式无关，用法都相同，都用来接受消息（解耦）
 */
@Component
@RabbitListener(queues = "simple_queue")     //监听的消息队列名
public class SimpleListener {
	
    //接受的消息类型可以是String,List,Map等
    @RabbitHandler
    public void simpleHandler(String msg){
        System.out.println("=====接收消息====>"+msg);
    }
}
```



#### 消费者工程

同生产者工程

### 5中工作模式

> P：生产者，也就是要发送消息的程序，但是不再发送到队列中，而是发给X（交换机）
>
> C：消费者，消息的接受者，会一直等待消息到来。
>
> Queue：消息队列，接收消息、缓存消息。
>
> Exchange：交换机，图中的X。一方面，接收生产者发送的消息。另一方面，知道如何处理消息，例如递交给某个特别队列、递交给所有队列、或是将消息丢弃。到底如何操作，取决于Exchange的类型。

> Exchange有常见以下3种类型：
>
> - **Fanout：广播**  将消息交给所有绑定到交换机的队列, 不处理路由键。只需要简单的将队列绑定到交换机上。**fanout 类型交换机转发消息是最快的。**
>
> - **Direct：直连**  把消息交给符合指定routing key 的队列. 处理路由键。需要将一个队列绑定到交换机上，要求该消息与一个特定的路由键完全匹配。如果一个队列绑定到该交换机上要求路由键 “dog”，则只有被标记为 “dog” 的消息才被转发，不会转发 dog.puppy，也不会转发 dog.guard，只会转发dog。
>
>   **其中，路由模式使用的是 direct 类型的交换机。**
>
> - **Topic：主题(通配符)**  把消息交给符合routing pattern（路由模式）的队列. 将路由键和某模式进行匹配。此时队列需要绑定要一个模式上。符号 “#” 匹配一个或多个词，符号"\*"匹配不多不少一个词。因此“audit.#” 能够匹配到“audit.irs.corporate”，但是“audit.*” 只会匹配到 “audit.irs”。
>
> **Exchange（交换机）只负责转发消息，不具备存储消息的能力，因此如果没有任何队列与Exchange绑定，或者没有符合路由规则的队列，那么消息会丢失**

#### 1.**简单模式 HelloWorld**

![1575274339325](file://D:/java/java-notes/image/1575274339325.png?lastModify=1611318221)

> 在上图的模型中，有以下概念：
>
> ​	**P：生产者: **  也就是要发送消息的程序
>
> ​	**C：消费者：**消息的接受者，会一直等待消息到来。
>
> ​	**queue：**消息队列，图中红色部分。可以缓存消息；生产者向其中投递消息，消费者从其中取出消息。

#### 2.**工作队列模式 Work Queue**

![image-20191205102457994](file://D:/java/java-notes/image/image-20191205102457994.png?lastModify=1611318824)

> 与简单模式相比，多个消费端共同消费同一个队列中的消息（竞争关系，消息只能被一个消费者获得）
>
> **应用场景：对于任务过重或任务较多情况使用工作队列可以提高任务处理的速度。**多个节点分片任务处理，提升任务处理的效率

#### 3.**发布订阅模式 Publish/subscribe**

![image-20191205102917088](file://D:/java/java-notes/image/image-20191205102917088.png?lastModify=1611319272)



> 一个生产者，多个消费者接收任务，每个消费者都接到相同的消息

#### 4.**路由模式 Routing**

![image-20191205103846484](file://D:/java/java-notes/image/image-20191205103846484.png?lastModify=1611320813)

#### 5.**主题(通配符)模式 Topics**

![image-20191205104428234](file://D:/java/java-notes/image/image-20191205104428234.png?lastModify=1611320871)

> `Routingkey`: 一般都是有一个或多个单词组成，多个单词之间以”.”分割，例如：`item.insert`
>
> 通配符规则：
>
> #：匹配一个或多个词，多个词用点号分隔
>
> *：匹配不多不少恰好1个词
>
> 举例：
>
> **item.#：** 能够匹配`item.insert.abc.bbc`或者`item.insert`
>
> **item.*：**只能匹配`item.insert`

### 高级特性

### 使用

发送接口封装参数验证，日志输出信息

