# RabbitMQ 介绍

RabbitMQ是实现AMQP（高级消息队列协议）的消息中间件的一种，最初起源于金融系统，用于在分布式系统中存储转发消息，在易用性、扩展性、高可用性等方面表现不俗。RabbitMQ主要是为了实现系统之间的双向解耦而实现的。当生产者大量产生数据时，消费者无法快速消费，那么需要一个中间层保存这个数据。

AMQP，即Advanced Message Queuing Protocol，高级消息队列协议，是应用层协议的一个开放标准，为面向消息的中间件设计。消息中间件主要用于组件之间的解耦，消息的发送者无需知道消息使用者的存在，反之亦然。AMQP的主要特征是面向消息、队列、路由（包括点对点和发布/订阅）、可靠性、安全。  

RabbitMQ是一个开源的AMQP实现，服务器端用Erlang语言编写，支持多种客户端，如：Python、Ruby、.NET、Java、JMS、C、PHP、ActionScript、XMPP、STOMP等，支持AJAX。用于在分布式系统中存储转发消息，在易用性、扩展性、高可用性等方面表现不俗。

# RabbitMQ 特点

RabbitMQ 是一个由 Erlang 语言开发的 AMQP 的开源实现。

AMQP ：Advanced Message Queue，高级消息队列协议。它是应用层协议的一个开放标准，为面向消息的中间件设计，基于此协议的客户端与消息中间件可传递消息，并不受产品、开发语言等条件的限制。

RabbitMQ 最初起源于金融系统，用于在分布式系统中存储转发消息，在易用性、扩展性、高可用性等方面表现不俗。具体特点包括：

1. 可靠性（Reliability） RabbitMQ 使用一些机制来保证可靠性，如**持久化、传输确认、发布确认**。
2. 灵活的路由（Flexible Routing） 在消息进入队列之前，通过 Exchange 来路由消息的。对于典型的路由功能，RabbitMQ 已经提供了一些内置的 Exchange 来实现。针对更复杂的路由功能，可以将多个 Exchange 绑定在一起，也通过插件机制实现自己的 Exchange 。
3. 消息集群（Clustering） 多个 RabbitMQ 服务器可以组成一个集群，形成一个逻辑 Broker 。
4. 高可用（Highly Available Queues） 队列可以在集群中的机器上进行镜像，使得在部分节点出问题的情况下队列仍然可用。
5. 多种协议（Multi-protocol） RabbitMQ 支持多种消息队列协议，比如 STOMP、MQTT 等等。
6. 多语言客户端（Many Clients） RabbitMQ 几乎支持所有常用语言，比如 Java、.NET、Ruby 等等。
7. 管理界面（Management UI） RabbitMQ 提供了一个易用的用户界面，使得用户可以监控和管理消息 Broker 的许多方面。
8. 跟踪机制（Tracing） 如果消息异常，RabbitMQ 提供了消息跟踪机制，使用者可以找出发生了什么。
9. 插件机制（Plugin System） RabbitMQ 提供了许多插件，来从多方面进行扩展，也可以编写自己的插件。

作者：预流链接：https://juejin.im/post/5a67f7836fb9a01cb74e8931

# RabbitMQ 基本概念

上面只是最简单抽象的描述，具体到 RabbitMQ 则有更详细的概念需要解释。上面介绍过 RabbitMQ 是 AMQP 协议的一个开源实现，所以其内部实际上也是 AMQP 中的基本概念：

![RabbitMQ 内部结构](https://user-gold-cdn.xitu.io/2018/1/24/161260568dd66584?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

1. Message 消息，消息是不具名的，它由**消息头和消息体**组成。消息体是不透明的，而消息头则由一系列的可选属性组成，这些属性包括**routing-key（路由键）、priority（相对于其他消息的优先权）、delivery-mode（指出该消息可能需要持久性存储）等。**
2. Publisher 消息的生产者，也是一个向交换器发布消息的客户端应用程序。
3. Exchange 交换器，**用来接收生产者发送的消息并将这些消息路由给服务器中的队列**。
4. Binding 绑定，用于**消息队列和交换器之间的关联**。一个绑定就是基于路由键将交换器和消息队列连接起来的路由规则，所以可以将交换器理解成一个由绑定构成的路由表。
5. Queue 消息队列，用来**保存消息直到发送给消费者**。它是消息的容器，也是消息的终点。一个消息可投入一个或多个队列。消息一直在队列里面，等待消费者连接到这个队列将其取走。
6. Connection 网络连接，比如一个TCP连接。
7. Channel 信道，多路复用连接中的一条独立的双向数据流通道。信道是建立在真实的TCP连接内地虚拟连接，AMQP 命令都是**通过信道发出去的**，不管是**发布消息、订阅队列还是接收消息**，这些动作都是通过信道完成。因为对于操作系统来说建立和销毁 TCP 都是非常昂贵的开销，所以**引入了信道的概念，以复用一条 TCP 连接**。
8. Consumer 消息的消费者，表示一个从消息队列中取得消息的客户端应用程序。
9. Virtual Host 虚拟主机，**表示一批交换器、消息队列和相关对象**。虚拟主机是共享相同的身份认证和加密环境的独立服务器域。每个 vhost 本质上就是一个 mini 版的 RabbitMQ 服务器，拥有自己的队列、交换器、绑定和权限机制。vhost 是 AMQP 概念的基础，必须在连接时指定，RabbitMQ 默认的 vhost 是 / 。
10. Broker 表示消息队列服务器实体。


作者：预流链接：https://juejin.im/post/5a67f7836fb9a01cb74e8931

那么，*其中比较重要的概念有 4 个，分别为：虚拟主机，交换机，队列，和绑定。*

- 虚拟主机：一个虚拟主机持有**一组交换机、队列和绑定**。为什么需要多个虚拟主机呢？很简单，RabbitMQ当中，*用户只能在虚拟主机的粒度进行权限控制。* 因此，如果需要禁止A组访问B组的交换机/队列/绑定，必须为A和B分别创建一个虚拟主机。每一个RabbitMQ服务器都有一个默认的虚拟主机“/”。
- 交换机：*Exchange **用于转发消息，但是它不会做存储*** ，如果没有 Queue bind 到 Exchange 的话，它会直接丢弃掉 Producer 发送过来的消息。
  这里有一个比较重要的概念：**路由键** 。消息到交换机的时候，交互机会转发到对应的队列中，那么究竟转发到哪个队列，就要根据该路由键。
- 绑定：也就是**交换机需要和队列相绑定**，这其中如上图所示，是多对多的关系。

## 交换机(Exchange)

交换机的功能主要是**接收消息并且转发到绑定的队列**，交换机不存储消息，在启用ack模式后，交换机找不到队列会返回错误。交换机**有四种类型：Direct, topic, Headers and Fanout** 

- Direct：direct 类型的行为是**"先匹配, 再投送"**. 即在绑定时设定一个 **routing_key**, 消息的**routing_key** 匹配时, 才会被交换器投送到绑定的队列中去.
- Topic：按**规则转发**消息（最灵活）
- Headers：设置header attribute参数类型的交换机
- Fanout：转发消息到**所有绑定队列**

**Direct Exchange**
Direct  Exchange是RabbitMQ默认的交换机模式，也是最简单的模式，根据key全文匹配去寻找队列。

![img](https://user-gold-cdn.xitu.io/2017/7/14/988fe8f789df3fb4540f88a6fc094eb3?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



第一个 X - Q1 就有一个 binding key，名字为 orange； X - Q2 就有 2 个 binding key，名字为 black 和 green。*当消息中的 **路由键 和 这个 binding key 对应上的时候**，那么就知道了该消息去到哪一个队列中。*

Ps：为什么 X 到 Q2 要有 black，green，2个 binding key呢，一个不就行了吗？ - 这个主要是因为可能又有 Q3，而Q3只接受 black 的信息，而Q2不仅接受black 的信息，还接受 green 的信息。

路由键与队列名完全匹配，如果一个队列绑定到交换机要求路由键为“dog”，则只转发 routing key 标记为“dog”的消息，不会转发“dog.puppy”，也不会转发“dog.guard”等等。**它是完全匹配、单播的模式**。

**Topic Exchange**  

*Topic Exchange 转发消息主要是**根据通配符**。* 在这种交换机下，队列和交换机的绑定会**定义一种路由模式**，那么，通配符就要在这种路由模式和路由键之间匹配后交换机才能转发消息。

在这种交换机模式下：

- 路由键必须是一串字符，用句号（`.`） 隔开，比如说 agreements.us，或者 agreements.eu.stockholm 等。
- 路由模式**必须包含一个 星号（`*`）**，主要用于匹配路由键指定位置的一个单词，比如说，一个路由模式是这样子：agreements..b.*，那么就只能匹配路由键是这样子的：第一个单词是 agreements，第四个单词是 b。 井号（#）就表示相当于一个或者多个单词，例如一个匹配模式是agreements.eu.berlin.#，那么，以agreements.eu.berlin开头的路由键都是可以的。

具体代码发送的时候还是一样，**第一个参数表示交换机**，**第二个参数表示routing key**，**第三个参数即消息**。如下：

```java
rabbitTemplate.convertAndSend("testTopicExchange","key1.a.c.key2", " this is  RabbitMQ!");
```

topic 和 direct 类似, 只是匹配上支持了"模式", 在"点分"的 routing_key 形式中, 可以使用两个通配符:

- `*`表示一个词.
- `#`表示零个或多个词.

![topic 交换器](https://user-gold-cdn.xitu.io/2018/1/24/161260569051565f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

 topic 交换器通过模式匹配分配消息的路由键属性，将路由键和某个模式进行匹配，此时队列需要绑定到一个模式上。它将路由键和绑定键的字符串切分成单词，这些单词之间用点隔开。它同样也会识别两个通配符：符号“#”和符号“*”。#匹配0个或多个单词，*匹配不多不少一个单词。
作者：预流链接：https://juejin.im/post/5a67f7836fb9a01cb74e8931

**Headers Exchange** 

headers 也是根据规则匹配, 相较于 direct 和 topic 固定地使用 routing_key , headers 则是一个自定义匹配规则的类型.
在队列与交换器绑定时, 会**设定一组键值对规则**, 消息中也包括一组键值对( headers 属性), 当这些键值对有一对, 或全部匹配时, 消息被投送到对应队列。

**Fanout Exchange** 

Fanout Exchange 消息广播的模式，不管路由键或者是路由模式，*会把消息发给绑定给它的全部队列*，如果配置了routing_key会被忽略。

每个发到 fanout 类型交换器的消息都会分到所有绑定的队列上去。fanout 交换器不处理路由键，只是简单的将队列绑定到交换器上，每个发送到交换器的消息都会被转发到与该交换器绑定的所有队列上。很像子网广播，每台子网内的主机都获得了一份复制的消息。fanout 类型转发消息是最快的。

# RabbitMQ 安装

一般来说安装 RabbitMQ 之前要安装 Erlang ，可以去[Erlang官网](https://link.juejin.im?target=http%3A%2F%2Fwww.erlang.org%2Fdownloads)下载。接着去[RabbitMQ官网](https://link.juejin.im?target=https%3A%2F%2Fwww.rabbitmq.com%2Fdownload.html)下载安装包，之后解压缩即可。根据操作系统不同官网提供了相应的安装说明：[Windows](https://link.juejin.im?target=http%3A%2F%2Fwww.rabbitmq.com%2Finstall-windows.html)、[Debian / Ubuntu](https://link.juejin.im?target=http%3A%2F%2Fwww.rabbitmq.com%2Finstall-debian.html)、[RPM-based Linux](https://link.juejin.im?target=http%3A%2F%2Fwww.rabbitmq.com%2Finstall-rpm.html)、[Mac](https://link.juejin.im?target=http%3A%2F%2Fwww.rabbitmq.com%2Finstall-standalone-mac.html)

如果是Mac 用户，个人推荐使用 HomeBrew 来安装，安装前要先更新 brew：

```
brew update
```

接着安装 rabbitmq 服务器：

```
brew install rabbitmq
```

这样 RabbitMQ 就安装好了，安装过程中会自动其所依赖的 Erlang 。

# RabbitMQ 运行和管理

1. 启动 启动很简单，找到安装后的 RabbitMQ 所在目录下的 sbin 目录，可以看到该目录下有6个以 rabbitmq 开头的可执行文件，直接执行 rabbitmq-server 即可，下面将 RabbitMQ 的安装位置以 . 代替，启动命令就是：

```
./sbin/rabbitmq-server
```

启动正常的话会看到一些启动过程信息和最后的 completed with 7 plugins，这也说明启动的时候默认加载了7个插件。 

![正常启动](https://user-gold-cdn.xitu.io/2018/1/24/16126056ba03d9f0?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

1. 后台启动 如果想让 RabbitMQ 以守护程序的方式在后台运行，可以在启动的时候加上 -detached 参数：

```
./sbin/rabbitmq-server -detached
```

1. 查询服务器状态 sbin 目录下有个特别重要的文件叫 rabbitmqctl ，它提供了 RabbitMQ 管理需要的几乎一站式解决方案，绝大部分的运维命令它都可以提供。 查询 RabbitMQ 服务器的状态信息可以用参数 status ：

```
./sbin/rabbitmqctl status
```

该命令将输出服务器的很多信息，比如 RabbitMQ 和 Erlang 的版本、OS 名称、内存等等

1. 关闭 RabbitMQ 节点 我们知道 RabbitMQ 是用 Erlang 语言写的，在Erlang 中有两个概念：节点和应用程序。节点就是 Erlang 虚拟机的每个实例，而多个 Erlang 应用程序可以运行在同一个节点之上。节点之间可以进行本地通信（不管他们是不是运行在同一台服务器之上）。比如一个运行在节点A上的应用程序可以调用节点B上应用程序的方法，就好像调用本地函数一样。如果应用程序由于某些原因奔溃，Erlang 节点会自动尝试重启应用程序。 如果要关闭整个 RabbitMQ 节点可以用参数 stop ：

```
./sbin/rabbitmqctl stop
```

它会和本地节点通信并指示其干净的关闭，也可以指定关闭不同的节点，包括远程节点，只需要传入参数 -n ：

```
./sbin/rabbitmqctl -n rabbit@server.example.com stop 
```

-n node 默认 node 名称是 rabbit@server ，如果你的主机名是 server.example.com ，那么 node 名称就是 rabbit@server.example.com 。

1. 关闭 RabbitMQ 应用程序 如果只想关闭应用程序，同时保持 Erlang 节点运行则可以用 stop_app：

```
./sbin/rabbitmqctl stop_app
```

这个命令在后面要讲的集群模式中将会很有用。

1. 启动 RabbitMQ 应用程序

```
./sbin/rabbitmqctl start_app
```

1. 重置 RabbitMQ 节点

```
./sbin/rabbitmqctl reset
```

该命令将清除所有的队列。

1. 查看已声明的队列

```
./sbin/rabbitmqctl list_queues
```

1. 查看交换器

```
./sbin/rabbitmqctl list_exchanges
```

该命令还可以附加参数，比如列出交换器的名称、类型、是否持久化、是否自动删除：

```
./sbin/rabbitmqctl list_exchanges name type durable auto_delete
```

1. 查看绑定

```
./sbin/rabbitmqctl list_bindings
```

作者：预流链接：https://juejin.im/post/5a67f7836fb9a01cb74e8931

windows下安装： https://blog.csdn.net/qq_31634461/article/details/79377256

# springboot集成RabbitMQ

springboot集成RabbitMQ非常简单，如果只是简单的使用配置非常少，springboot提供了spring-boot-starter-amqp项目对消息各种支持。

## 简单使用

1、配置pom,主要是添加spring-boot-starter-amqp的支持

```java
<dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

2、配置文件

配置rabbitmq的安装地址、端口以及账户信息

```java
spring.application.name=spirng-boot-rabbitmq

spring.rabbitmq.host=192.168.0.86
spring.rabbitmq.port=5672
spring.rabbitmq.username=admin
spring.rabbitmq.password=123456
```

​	3、队列配置

```java
@Configuration
public class RabbitConfig {

    @Bean
    public Queue Queue() {
        return new Queue("hello");
    }

}
```

3、发送者

rabbitTemplate是springboot 提供的默认实现

```java
public class HelloSender {

    @Autowired
    private AmqpTemplate rabbitTemplate;

    public void send() {
        String context = "hello " + new Date();
        System.out.println("Sender : " + context);
        this.rabbitTemplate.convertAndSend("hello", context);
    }

}
```

4、接收者

```java
@Component
@RabbitListener(queues = "hello")
public class HelloReceiver {

    @RabbitHandler
    public void process(String hello) {
        System.out.println("Receiver  : " + hello);
    }

}
```

5、测试

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class RabbitMqHelloTest {

    @Autowired
    private HelloSender helloSender;

    @Test
    public void hello() throws Exception {
        helloSender.send();
    }

}
```

> 注意，发送者和接收者的queue name必须一致，不然不能接收

## 多对多使用

一个发送者，N个接收者或者N个发送者和N个接收者会出现什么情况呢？

**一对多发送**
对上面的代码进行了小改造,接收端注册了两个Receiver,Receiver1和Receiver2，发送端加入参数计数，接收端打印接收到的参数，下面是测试代码，发送一百条消息，来观察两个接收端的执行效果

```java
@Test
public void oneToMany() throws Exception {
    for (int i=0;i<100;i++){
        neoSender.send(i);
    }
}
```

结果如下：

```java
Receiver 1: spirng boot neo queue ****** 11
Receiver 2: spirng boot neo queue ****** 12
Receiver 2: spirng boot neo queue ****** 14
Receiver 1: spirng boot neo queue ****** 13
Receiver 2: spirng boot neo queue ****** 15
Receiver 1: spirng boot neo queue ****** 16
Receiver 1: spirng boot neo queue ****** 18
Receiver 2: spirng boot neo queue ****** 17
Receiver 2: spirng boot neo queue ****** 19
Receiver 1: spirng boot neo queue ****** 20
```

根据返回结果得到以下结论

> 一个发送者，N个接受者,经过测试会均匀的将消息发送到N个接收者中

**多对多发送**  

复制了一份发送者，加入标记，在一百个循环中相互交替发送

```java
@Test
    public void manyToMany() throws Exception {
        for (int i=0;i<100;i++){
            neoSender.send(i);
            neoSender2.send(i);
        }
}
```

结果如下：

```java
Receiver 1: spirng boot neo queue ****** 20
Receiver 2: spirng boot neo queue ****** 20
Receiver 1: spirng boot neo queue ****** 21
Receiver 2: spirng boot neo queue ****** 21
Receiver 1: spirng boot neo queue ****** 22
Receiver 2: spirng boot neo queue ****** 22
Receiver 1: spirng boot neo queue ****** 23
Receiver 2: spirng boot neo queue ****** 23
Receiver 1: spirng boot neo queue ****** 24
Receiver 2: spirng boot neo queue ****** 24
Receiver 1: spirng boot neo queue ****** 25
Receiver 2: spirng boot neo queue ****** 25
```

> 结论：和一对多一样，接收端仍然会均匀接收到消息

## 高级使用

**对象的支持**

springboot以及完美的支持对象的发送和接收，不需要格外的配置。

```java
//发送者
public void send(User user) {
    System.out.println("Sender object: " + user.toString());
    this.rabbitTemplate.convertAndSend("object", user);
}


//接收者
@RabbitHandler
public void process(User user) {
    System.out.println("Receiver object : " + user);
}
```

结果如下：

```java
Sender object: User{name='neo', pass='123456'}
Receiver object : User{name='neo', pass='123456'}
```

**Topic Exchange**

topic 是RabbitMQ中最灵活的一种方式，可以**根据routing_key**自由的绑定不同的队列

首先对topic规则配置，这里使用两个队列来测试

```java
@Configuration
public class TopicRabbitConfig {

    final static String message = "topic.message";
    final static String messages = "topic.messages";

    @Bean
    public Queue queueMessage() {
        return new Queue(TopicRabbitConfig.message);
    }

    @Bean
    public Queue queueMessages() {
        return new Queue(TopicRabbitConfig.messages);
    }

    @Bean
    TopicExchange exchange() {
        return new TopicExchange("exchange");
    }

    @Bean
    Binding bindingExchangeMessage(Queue queueMessage, TopicExchange exchange) {
        return BindingBuilder.bind(queueMessage).to(exchange).with("topic.message");
    }

    @Bean
    Binding bindingExchangeMessages(Queue queueMessages, TopicExchange exchange) {
        return BindingBuilder.bind(queueMessages).to(exchange).with("topic.#");
    }
}
```

使用queueMessages同时匹配两个队列，queueMessage只匹配"topic.message"队列

```java
public void send1() {
    String context = "hi, i am message 1";
    System.out.println("Sender : " + context);
    this.rabbitTemplate.convertAndSend("exchange", "topic.message", context);
}

public void send2() {
    String context = "hi, i am messages 2";
    System.out.println("Sender : " + context);
    this.rabbitTemplate.convertAndSend("exchange", "topic.messages", context);
}
```

发送send1会匹配到topic.#和topic.message 两个Receiver都可以收到消息，发送send2只有topic.#可以匹配所有只有Receiver2监听到消息

**Fanout Exchange**

Fanout 就是我们熟悉的广播模式或者订阅模式，给Fanout交换机发送消息，**绑定了这个交换机的所有队列**都收到这个消息。

Fanout 相关配置

```java
@Configuration
public class FanoutRabbitConfig {

    @Bean
    public Queue AMessage() {
        return new Queue("fanout.A");
    }

    @Bean
    public Queue BMessage() {
        return new Queue("fanout.B");
    }

    @Bean
    public Queue CMessage() {
        return new Queue("fanout.C");
    }

    @Bean
    FanoutExchange fanoutExchange() {
        return new FanoutExchange("fanoutExchange");
    }

    @Bean
    Binding bindingExchangeA(Queue AMessage,FanoutExchange fanoutExchange) {
        return BindingBuilder.bind(AMessage).to(fanoutExchange);
    }

    @Bean
    Binding bindingExchangeB(Queue BMessage, FanoutExchange fanoutExchange) {
        return BindingBuilder.bind(BMessage).to(fanoutExchange);
    }

    @Bean
    Binding bindingExchangeC(Queue CMessage, FanoutExchange fanoutExchange) {
        return BindingBuilder.bind(CMessage).to(fanoutExchange);
    }

}
```

这里使用了A、B、C三个队列绑定到Fanout交换机上面，发送端的routing_key写任何字符都会被忽略：

```java
public void send() {
        String context = "hi, fanout msg ";
        System.out.println("Sender : " + context);
        this.rabbitTemplate.convertAndSend("fanoutExchange","", context);
}
```

结果如下：

```
Sender : hi, fanout msg 
...
fanout Receiver B: hi, fanout msg 
fanout Receiver A  : hi, fanout msg 
fanout Receiver C: hi, fanout msg复制代码
```

结果说明，绑定到fanout交换机上面的队列都收到了消息
作者：ityouknow链接：https://juejin.im/post/59687c57f265da6c360a23c8

# RabbitMQ 在生产环境下运用和出现的问题

在生产环境中，由于 Spring 对 RabbitMQ 提供了一些方便的注解，所以首先可以使用这些注解。例如：

- @EnableRabbit：@EnableRabbit 和 [@configuration](https://github.com/configuration) 注解在一个类中结合使用，如果该类能够返回一个 RabbitListenerContainerFactory 类型的 bean，那么就相当于能够把该终端（消费端）和 RabbitMQ 进行连接。Ps：（生成端不是通过 RabbitListenerContainerFactory 来和 RabbitMQ 连接，而是通过 RabbitTemplate ）
- @RabbitListener：当对应的队列中有消息的时候，该注解修饰下的方法会被执行。
- @RabbitHandler：接收者可以监听多个队列，不同的队列消息的类型可能不同，该注解可以使得不同的消息让不同方法来响应。

具体这些注解的使用，可以参考这里的代码：[点这里](https://github.com/401Studio/WeekLearn/tree/master/RabbitMQTest_WithSpringAnnotation)

首先，生产环境下的 RabbitMQ 可能**不会在生产者或者消费者本机上**，所以需要**重新定义 ConnectionFactory**，即：

```java
@Bean
ConnectionFactory connectionFactory() {
    CachingConnectionFactory connectionFactory = new CachingConnectionFactory(host, port);
    connectionFactory.setUsername(userName);
    connectionFactory.setPassword(password);
    connectionFactory.setVirtualHost(vhost);
    return connectionFactory;
}
```

这里，可以重新设置需要连接的 RabbitMQ 的 **ip，端口，虚拟主机，用户名，密码。**

然后，可以先从生产端考虑，生产端需要连接 RabbitMQ，那么可以通过 RabbitTemplate 进行连接。 Ps：（RabbitTemplate 用于生产端发送消息到交换机中），如下代码：

```java
@Bean(name="myTemplate")
RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
    RabbitTemplate template = new RabbitTemplate(connectionFactory);
    template.setMessageConverter(integrationEventMessageConverter());
    template.setExchange(exchangeName);
    return template;
}
```

在该代码中，`new RabbitTemplate(connectionFactory);` 设置了生产端连接到RabbitMQ，`template.setMessageConverter(integrationEventMessageConverter());` 设置了生产端发送给交换机的消息是**以什么格式**的，在 `integrationEventMessageConverter()` 代码中：

```java
public MessageConverter integrationEventMessageConverter() {
    Jackson2JsonMessageConverter messageConverter = new Jackson2JsonMessageConverter();
    return messageConverter;
}
```

如上 `Jackson2JsonMessageConverter` **指明了 JSON**。上述代码的最后 `template.setExchange(exchangeName);`指明了 要把生产者要把消息发送到哪个交换机上。

有了上述，那么，我们即可使用 `rabbitTemplate.convertAndSend("spring-boot", xxx);` 发送消息，xxx 表示任意类型，因为上述的设置会帮我们把这些类型转化成 JSON 传输。

接着，生产端发送我们说过了，那么现在可以看看消费端：

对于消费端，我们可以只创建 `SimpleRabbitListenerContainerFactory`，它能够帮我们生成 RabbitListenerContainer，然后我们再使用 @RabbitListener **指定接收者收到信息时处理的方法**。

```java
@Bean(name="myListenContainer")
public SimpleRabbitListenerContainerFactory rabbitListenerContainerFactory() {
    SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
    factory.setMessageConverter(integrationEventMessageConverter());
    factory.setConnectionFactory(connectionFactory());
    return factory;
}
```

这其中 `factory.setMessageConverter(integrationEventMessageConverter());` 指定了我们接受消息的时候，以 JSON 传输的消息可以转换成对应的类型传入到方法中。例如：

```java
@Slf4j
@Component
@RabbitListener(containerFactory = "helloRabbitListenerContainer",queues = "spring-boot")
public class Receiver {
    @RabbitHandler
    public void receiveTeacher(Teacher teacher) {
        log.info("##### = {}",teacher);
    }
}
```

可能出现的问题：

## 消息持久化

在生产环境中，我们需要考虑**万一生产者挂了，消费者挂了，或者 rabbitmq 挂了怎么样**。一般来说，如果生产者挂了或者消费者挂了，其实是没有影响，因为消息就在队列里面。那么万一 rabbitmq 挂了，之前在队列里面的消息怎么办，其实可以**做消息持久化，RabbitMQ 会把信息保存在磁盘上。**

做法是可以先从 Connection 对象中拿到一个 Channel 信道对象，然后再可以通过该对象设置消息持久化。

## 生产者或者消费者断线重连

这里 Spring 有自动重连机制。

## ACK 确认机制

每个Consumer可能需要一段时间才能处理完收到的数据。如果在这个过程中，Consumer出错了，异常退出了，而数据还没有处理完成，那么 非常不幸，这段数据就丢失了。因为我们**采用no-ack的方式进行确认**，也就是说，每次Consumer接到数据后，而不管是否处理完成，**RabbitMQ Server会立即把这个Message标记为完成**，然后**从queue中删除了**。

如果一个Consumer异常退出了，它处理的数据能够被另外的Consumer处理，这样数据在这种情况下就不会丢失了（注意是这种情况下）。
为了保证数据不被丢失，RabbitMQ支持消息确认机制，即acknowledgments。为了保证数据能被正确处理而不仅仅是被Consumer收到，那么我们不能采用no-ack。而应该是在处理完数据后发送ack。

在处理数据后发送的ack，就是告诉RabbitMQ数据已经被接收，处理完成，RabbitMQ可以去安全的删除它了。
如果Consumer退出了但是没有发送ack，**那么RabbitMQ就会把这个Message发送到下一个Consumer**。这样就保证了在Consumer异常退出的情况下数据也不会丢失。

https://github.com/401Studio/WeekLearn/issues/2

# **为什么要使用MQ？**

**异步、解耦、削峰填谷**

- 异步：使用了MQ之后，系统只需发送信息到消息队列，就会返回给用户，耗时大大降低，提高了接口的性能。


- 解耦：只需要把消息发给MQ，其他系统按需订阅就可以，不用像原来修改发送的系统。如果一个系统挂了，则不会影响另外个系统的继续运行。

- 削峰填谷：使用了MQ之后，可以削弱高峰期的并发量，并维持在一定并发量，直到消费完积压的消息，叫做填谷。

# **使用了MQ之后有什么优缺点？**

**系统可用性降低**

大家想想一下，上面的说解耦的场景，本来A系统的哥们要把系统关键数据发送给BC系统的，现在突然加入了一个MQ了，现在BC系统接收数据要通过MQ来接收。

但是大家有没有考虑过一个问题，万一MQ挂了怎么办？这就引出一个问题，加入了MQ之后，系统的可用性是不是就降低了？

因为多了一个风险因素：MQ可能会挂掉。只要MQ挂了，数据没了，系统运行就不对了。

**系统复杂度提高**

本来我的系统通过接口调用一下就能完事的，但是加入一个MQ之后，需要考虑消息重复消费、消息丢失、甚至消息顺序性的问题

为了解决这些问题，又需要引入很多复杂的机制，这样一来是不是系统的复杂度提高了。

**数据一致性问题**

本来好好的，A系统调用BC系统接口，如果BC系统出错了，会抛出异常，返回给A系统让A系统知道，这样的话就可以做回滚操作了

但是使用了MQ之后，A系统发送完消息就完事了，认为成功了。而刚好C系统写数据库的时候失败了，但是A认为C已经成功了？这样一来数据就不一致了。

通过分析引入MQ的优缺点之后，就明白了使用MQ有很多优点，但是会发现它带来的缺点又会需要你做各种额外的系统设计来弥补

最后你可能会发现整个系统复杂了好几倍，所以设计系统的时候要基于这些考虑做出取舍，很多时候你会发现该用的还是要用的。。。

## **怎么保证MQ消息不丢失？**

使用了MQ之后，还要关心消息丢失的问题。这里我们挑RabbitMQ来说明一下吧。

**生产者弄丢了数据**

RabbitMQ生产者将数据发送到rabbitmq的时候,可能数据在网络传输中搞丢了，这个时候RabbitMQ收不到消息，消息就丢了。

RabbitMQ提供了两种方式来解决这个问题：

**事务方式：**

在生产者发送消息之前，通过`channel.txSelect`开启一个事务，接着发送消息

如果消息没有成功被RabbitMQ接收到，生产者会收到异常，此时就可以进行事务回滚`channel.txRollback`然后重新发送。假如RabbitMQ收到了这个消息，就可以提交事务`channel.txCommit`。

但是这样一来，生产者的吞吐量和性能都会降低很多，现在一般不这么干。

另外一种方式就是通过**confirm机制**：

这个confirm模式是在生产者哪里设置的，就是每次写消息的时候会分配一个唯一的id，然后RabbitMQ收到之后会回传一个ack，告诉生产者这个消息ok了。

如果rabbitmq没有处理到这个消息，那么就回调一个nack的接口，这个时候生产者就可以重发。

事务机制和cnofirm机制**最大的不同**在于事务机制是同步的，提交一个事务之后会阻塞在那儿

但是confirm机制是异步的，发送一个消息之后就可以发送下一个消息，然后那个消息rabbitmq接收了之后会异步回调你一个接口通知你这个消息接收到了。

所以一般在**生产者这块避免数据丢失，都是用confirm机制的**。

**Rabbitmq弄丢了数据**

RabbitMQ集群也会弄丢消息，这个问题在官方文档的教程中也提到过，就是说在消息发送到RabbitMQ之后，默认是没有落地磁盘的，万一RabbitMQ宕机了，这个时候消息就丢失了。

所以为了解决这个问题，RabbitMQ提供了一个**持久化**的机制，消息写入之后会**持久化到磁盘**

这样哪怕是宕机了，恢复之后也会自动恢复之前存储的数据，这样的机制可以确保消息不会丢失。

设置持久化有两个步骤：

- 第一个是创建**queue的时候将其设置为持久化的**，这样就可以保证rabbitmq持久化queue的元数据，但是不会持久化queue里的数据
- 第二个是发送消息的时候将消息的deliveryMode设置为2，就是将消息设置为持久化的，此时rabbitmq就会将消息持久化到磁盘上去。

但是这样一来可能会有人说：万一消息发送到RabbitMQ之后，还没来得及持久化到磁盘就挂掉了，数据也丢失了，怎么办？

对于这个问题，其实是配合上面的confirm机制一起来保证的，就是在消息持久化到磁盘之后才会给生产者发送ack消息。

万一真的遇到了那种极端的情况，生产者是可以感知到的，此时生产者可以通过重试发送消息给别的RabbitMQ节点

**消费端弄丢了数据**

RabbitMQ消费端弄丢了数据的情况是这样的：在消费消息的时候，刚拿到消息，结果进程挂了，这个时候RabbitMQ就会认为你已经消费成功了，这条数据就丢了。

对于这个问题，要先说明一下RabbitMQ消费消息的机制：在消费者收到消息的时候，会发送一个ack给RabbitMQ，告诉RabbitMQ这条消息被消费到了，这样RabbitMQ就会把消息删除。

但是默认情况下这个发送ack的操作是自动提交的，也就是说**消费者一收到这个消息就会自动返回ack给RabbitMQ**，所以会出现丢消息的问题。

所以针对这个问题的解决方案就是：**关闭RabbitMQ消费者的自动提交ack,在消费者处理完这条消息之后再手动提交ack。**

这样即使遇到了上面的情况，RabbitMQ也不会把这条消息删除，会在你程序重启之后，重新下发这条消息过来。

## **怎么保证MQ的高可用性性？**

使用了MQ之后，我们肯定是希望MQ有高可用特性，因为不可能接受机器宕机了，就无法收发消息的情况。

这一块我们也是基于RabbitMQ这种经典的MQ来说明一下：

RabbitMQ是比较有代表性的，因为是基于主从做高可用性的，我们就以他为例子讲解第一种MQ的高可用性怎么实现。

rabbitmq有三种模式：**单机模式，普通集群模式，镜像集群模式**

**单机模式**

单机模式就是demo级别的，就是说只有一台机器部署了一个RabbitMQ程序。

这个会存在单点问题，宕机就玩完了，没什么高可用性可言。一般就是你本地启动了玩玩儿的，没人生产用单机模式。

**普通集群模式**

这个模式的意思就是在多台机器上启动多个rabbitmq实例。类似的master-slave模式一样。

但是创建的queue，只会放在一个master rabbtimq实例上，其他实例都同步那个接收消息的RabbitMQ元数据。

在消费消息的时候，如果你连接到的RabbitMQ实例不是存放Queue数据的实例，这个时候RabbitMQ就会从存放Queue数据的实例上拉去数据，然后返回给客户端。

总的来说，这种方式有点麻烦，没有做到真正的分布式，每次消费者连接一个实例后拉取数据，如果连接到不是存放queue数据的实例，这个时候会造成额外的性能开销。如果从放Queue的实例拉取，会导致单实例性能瓶颈。

**如果放queue的实例宕机了，会导致其他实例无法拉取数据，这个集群都无法消费消息了，没有做到真正的高可用**。

所以这个事儿就比较尴尬了，这就没有什么所谓的高可用性可言了，这方案主要是提高吞吐量的，就是说让集群中多个节点来服务某个queue的读写操作。

**镜像集群模式**

镜像集群模式才是真正的rabbitmq的高可用模式，跟普通集群模式不一样的是：**创建的queue无论元数据还是queue里的消息都会存在于多个实例上，每次写消息到queue的时候，都会自动把消息到多个实例的queue里进行消息同步。**这样的话任何一个机器宕机了别的实例都可以用提供服务，这样就做到了真正的高可用了。

但是也存在着不好之处：

- 性能开销过高，消息需要同步所有机器，会导致网络带宽压力和消耗很重

- 扩展性低：无法解决某个queue数据量特别大的情况，导致queue无法线性拓展。

  就算加了机器，那个机器也会包含queue的所有数据，queue的数据没有做到分布式存储。

对于RabbitMQ的高可用一般的做法都是**开启镜像集群模式**，这样起码来说做到了高可用，一个节点宕机了，其他节点可以继续提供服务。

作者：石杉的架构笔记链接：https://juejin.im/post/5d1e201ff265da1b6c5f9423

## 如何保证消息的顺序性

https://juejin.im/post/5c9b1c155188251d806727b2

## 重复消费

# Spring Boot + RabbitMQ实现延迟队列

https://juejin.im/post/5a12ffd451882578da0d7b3a