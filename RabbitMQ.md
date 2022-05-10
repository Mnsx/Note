# MQ的相关概念

## MQ介绍

MQ(message queue)，从字面意思上看，本质是个队列，FIFO 先入先出，只不过队列中存放的内容是
message 而已，还是一种跨进程的通信机制，用于上下游传递消息。在互联网架构中，MQ 是一种非常常
见的上下游“逻辑解耦+物理解耦”的消息通信服务。使用了 MQ 之后，消息发送上游只需要依赖 MQ，不
用依赖其他服务。

## MQ的优势

1. 流量消峰

   举个例子，如果订单系统最多能处理一万次订单，这个处理能力应付正常时段的下单时绰绰有余，正常时段我们下单一秒后就能返回结果。但是在高峰期，如果有两万次下单操作系统是处理不了的，只能限制订单超过一万后不允许用户下单。使用消息队列做缓冲，我们可以取消这个限制，把一秒内下的订单分散成一段时间来处理，这时有些用户可能在下单十几秒后才能收到下单成功的操作，但是比不能下单的体验要好。

2. 应用解耦

   以电商应用为例，应用中有订单系统、库存系统、物流系统、支付系统。用户创建订单后，如果耦合调用库存系统、物流系统、支付系统，任何一个子系统出了故障，都会造成下单操作异常。当转变成基于消息队列的方式后，系统间调用的问题会减少很多，比如物流系统因为发生故障，需要几分钟来修复。在这几分钟的时间里，物流系统要处理的内存被缓存在消息队列中，用户的下单操作可以正常完成。当物流系统恢复后，继续处理订单信息即可，中单用户感受不到物流系统的故障，提升系统的可用性。

3. 异步处理

   有些服务间调用是异步的，例如 A 调用 B，B 需要花费很长时间执行，但是 A 需要知道 B 什么时候可以执行完，以前一般有两种方式，A 过一段时间去调用 B 的查询 api 查询。或者 A 提供一个 callback api， B 执行完之后调用 api 通知 A 服务。这两种方式都不是很优雅，使用消息总线，可以很方便解决这个问题，A 调用 B 服务后，只需要监听 B 处理完成的消息，当 B 处理完成后，会发送一条消息给 MQ，MQ 会将此消息转发给 A 服务。这样 A 服务既不用循环调用 B 的查询 api，也不用提供 callback api。同样 B 服务也不用做这些操作。A 服务还能及时的得到异步处理成功的消息。

## MQ的分类

* ActiveMQ

  优点：单机吞吐量万级，时效性 ms 级，可用性高，基于主从架构实现高可用性，消息可靠性较
  低的概率丢失数据
  缺点:官方社区现在对 ActiveMQ 5.x 维护越来越少，高吞吐量场景较少使用。

* kafka

  大数据的杀手锏，谈到大数据领域内的消息传输，则绕不开 Kafka，这款为大数据而生的消息中间件，以其百万级 TPS 的吞吐量名声大噪，迅速成为大数据领域的宠儿，在数据采集、传输、存储的过程中发挥着举足轻重的作用。目前已经被 LinkedIn，Uber, Twitter, Netflix 等大公司所采纳。
  优点: 性能卓越，单机写入 TPS 约在百万条/秒，最大的优点，就是吞吐量高。时效性 ms 级可用性非常高，kafka 是分布式的，一个数据多个副本，少数机器宕机，不会丢失数据，不会导致不可用,消费者采用 Pull 方式获取消息, 消息有序, 通过控制能够保证所有消息被消费且仅被消费一次;有优秀的第三方Kafka Web 管理界面 Kafka-Manager；在日志领域比较成熟，被多家公司和多个开源项目使用；功能支持：功能较为简单，主要支持简单的 MQ 功能，在大数据领域的实时计算以及日志采集被大规模使用
  缺点：Kafka 单机超过 64 个队列/分区，Load 会发生明显的飙高现象，队列越多，load 越高，发送消息响应时间变长，使用短轮询方式，实时性取决于轮询间隔时间，消费失败不支持重试；支持消息顺序，但是一台代理宕机后，就会产生消息乱序，社区更新较慢；

* RocketMQ

  RocketMQ 出自阿里巴巴的开源产品，用 Java 语言实现，在设计时参考了 Kafka，并做出了自己的一些改进。被阿里巴巴广泛应用在订单，交易，充值，流计算，消息推送，日志流式处理，binglog 分发等场景。

  优点:**单机吞吐量十万级**,可用性非常高，分布式架构,**消息可以做到** **0** **丢失****,**MQ 功能较为完善，还是分布式的，扩展性好,**支持** **10** **亿级别的消息堆积**，不会因为堆积导致性能下降,源码是 java 我们可以自己阅读源码，定制自己公司的 MQ

  缺点：**支持的客户端语言不多**，目前是 java 及 c++，其中 c++不成熟；社区活跃度一般,没有在 MQ核心中去实现 JMS 等接口,有些系统要迁移需要修改大量代码

* RabbitMQ

  2007 年发布，是一个在 AMQP(高级消息队列协议)基础上完成的，可复用的企业消息系统，是**当前最****主流的消息中间件之一**。

  优点:由于 erlang 语言的**高并发特性**，性能较好；**吞吐量到万级**，MQ 功能比较完备,健壮、稳定、易用、跨平台、**支持多种语言** 如：Python、Ruby、.NET、Java、JMS、C、PHP、ActionScript、XMPP、STOMP等，支持 AJAX 文档齐全；开源提供的管理界面非常棒，用起来很好用,**社区活跃度高**；更新频率相当高

  缺点：商业版需要收费,学习成本较高

# RabbitMQ

## RabbitMQ的概念

RabbitMQ 是一个消息中间件：它接受并转发消息。你可以把它当做一个快递站点，当你要发送一个包裹时，你把你的包裹放到快递站，快递员最终会把你的快递送到收件人那里，按照这种逻辑 RabbitMQ 是一个快递站，一个快递员帮你传递快件。RabbitMQ 与快递站的主要区别在于，它不处理快件而是接收，存储和转发消息数据。

## 四大核心概念

* 生产者

  产生数据发送消息的程序是生产者

* 交换机

  交换机是 RabbitMQ 非常重要的一个部件，一方面它接收来自生产者的消息，另一方面它将消息推送到队列中。交换机必须确切知道如何处理它接收到的消息，是将这些消息推送到特定队列还是推送到多个队列，亦或者是把消息丢弃，这个得有交换机类型决定

* 队列

  队列是 RabbitMQ 内部使用的一种数据结构，尽管消息流经 RabbitMQ 和应用程序，但它们只能存储在队列中。队列仅受主机的内存和磁盘限制的约束，本质上是一个大的消息缓冲区。许多生产者可以将消息发送到一个队列，许多消费者可以尝试从一个队列接收数据。这就是我们使用队列的方式

* 消费者

  消费与接收具有相似的含义。消费者大多时候是一个等待接收消息的程序。请注意生产者，消费者和消息中间件很多时候并不在同一机器上。同一个应用程序既可以是生产者又是可以是消费者。

## Rabbit核心部分

![image-20220424144742637](D:\Picture\Note\RabbitMQ\核心模式.png)

## 常用名词解释

![image-20220424145213995](D:\Picture\Note\RabbitMQ\工作原理)

* **Broker**：接收和分发消息的应用，RabbitMQ Server 就是 Message Broker
* **Virtual host**：出于多租户和安全因素设计的，把 AMQP 的基本组件划分到一个虚拟的分组中，类似于网络中的 namespace 概念。当多个不同的用户使用同一个 RabbitMQ server 提供的服务时，可以划分出多个 vhost，每个用户在自己的 vhost 创建 exchange／queue 等
* **Connection**：publisher／consumer 和 broker 之间的 TCP 连接
* **Channel**：如果每一次访问 RabbitMQ 都建立一个 Connection，在消息量大的时候建立 TCP Connection 的开销将是巨大的，效率也较低。Channel 是在 connection 内部建立的逻辑连接，如果应用程序支持多线程，通常每个 thread 创建单独的 channel 进行通讯，AMQP method 包含了 channel id 帮助客户端和 message broker 识别 channel，所以 channel 之间是完全隔离的。**Channel 作为轻量级的Connection 极大减少了操作系统建立 TCP connection 的开销**
* **Exchange**：message 到达 broker 的第一站，根据分发规则，匹配查询表中的 routing key，分发消息到 queue 中去。常用的类型有：direct (point-to-point), topic (publish-subscribe) and fanout (multicast)
* **Queue**：消息最终被送到这里等待 consumer 取走
* **Binding**：exchange 和 queue 之间的虚拟连接，binding 中可以包含 routing key，Binding 信息被保存到 exchange 中的查询表中，用于 message 的分发依据

## 安装

![image-20220509093218104](D:\Picture\Note\RabbitMQ\安装流程)

开启web管理插件

`rabbitmq-plugins enable rabbitmq_management`

添加一个新的用户

* 创建账号

  `rabbitmqctl add_user mnsx 123123`

* 设置用户角色

  `rabbitmqctl set_user_tags admin administrator`

* 设置用户权限

  `set_permissions [-p <vhostpath>] <user> <conf> <write> <read>`

  `rabbitctl set_permissions -p "/" admin ".*" ".*" ".*"`

  用户user_admin具有/vhost1这个virtual host中所有的配置、写、读权限

* 展示当前用户和角色

  `rabbitmqctl list_users`

重置命令

* 关闭应用

  `rabbitmqctl stop_app`

* 清楚命令

  `rabbitmqctl reset`

* 重新启动命令

  `rabbitmqctl start_app`

# 默认模式

在下图中，“ P”是我们的生产者，“ C”是我们的消费者。中间的框是一个队列-RabbitMQ 代 表使用者保留的消息缓冲区

![image-20220509124658457](D:\Picture\Note\RabbitMQ\默认模式.png)

* 添加依赖

  ```xml
  <!--指定 jdk 编译版本-->
  <build>
   <plugins>
   <plugin>
   <groupId>org.apache.maven.plugins</groupId>
   <artifactId>maven-compiler-plugin</artifactId>
   <configuration>
   <source>8</source>
   <target>8</target>
   </configuration>
   </plugin>
   </plugins>
  </build>
  <dependencies>
   <!--rabbitmq 依赖客户端-->
   <dependency>
   <groupId>com.rabbitmq</groupId>
   <artifactId>amqp-client</artifactId>
   <version>5.8.0</version>
   </dependency>
   <!--操作文件流的一个依赖-->
   <dependency>
   <groupId>commons-io</groupId>
   <artifactId>commons-io</artifactId>
   <version>2.6</version>
   </dependency>
  </dependencies>
  ```

* 消息生产者

  ```java
  public class Producer {
   private final static String QUEUE_NAME = "hello";
   public static void main(String[] args) throws Exception {
   //创建一个连接工厂
   ConnectionFactory factory = new ConnectionFactory();
   factory.setHost("182.92.234.71");
   factory.setUsername("admin");
   factory.setPassword("123");
   //channel 实现了自动 close 接口 自动关闭 不需要显示关闭
   try(Connection connection = factory.newConnection();Channel channel = 
  connection.createChannel()) {
   /**
   * 生成一个队列
   * 1.队列名称
   * 2.队列里面的消息是否持久化 默认消息存储在内存中
   * 3.该队列是否只供一个消费者进行消费 是否进行共享 true 可以多个消费者消费
   * 4.是否自动删除 最后一个消费者端开连接以后 该队列是否自动删除 true 自动删除
   * 5.其他参数
   */
   channel.queueDeclare(QUEUE_NAME,false,false,false,null);
   String message="hello world";
   /**
   * 发送一个消息
   * 1.发送到那个交换机
   * 2.路由的 key 是哪个
   * 3.其他的参数信息
   * 4.发送消息的消息体
   */
   channel.basicPublish("",QUEUE_NAME,null,message.getBytes());
   System.out.println("消息发送完毕");
   }
   }
  }
  ```

* 消息消费者

  ```java
  public class Consumer {
   private final static String QUEUE_NAME = "hello";
   public static void main(String[] args) throws Exception {
   ConnectionFactory factory = new ConnectionFactory();
   factory.setHost("182.92.234.71");
   factory.setUsername("admin");
   factory.setPassword("123");
   Connection connection = factory.newConnection();
   Channel channel = connection.createChannel();
   System.out.println("等待接收消息....");
   //推送的消息如何进行消费的接口回调
   DeliverCallback deliverCallback=(consumerTag,delivery)->{
   String message= new String(delivery.getBody());
   System.out.println(message);
   };
   //取消消费的一个回调接口 如在消费的时候队列被删除掉了
   CancelCallback cancelCallback=(consumerTag)->{
   System.out.println("消息消费被中断");
   };
       /**
   * 消费者消费消息
   * 1.消费哪个队列
   * 2.消费成功之后是否要自动应答 true 代表自动应答 false 手动应答
   * 3.消费者未成功消费的回调
   */
   channel.basicConsume(QUEUE_NAME,true,deliverCallback,cancelCallback);
   }
  }
  ```

# 工作队列

工作队列(又称任务队列)的主要思想是避免立即执行资源密集型任务，而不得不等待它完成。 相反我们安排任务在之后执行。我们把任务封装为消息并将其发送到队列。在后台运行的工作进 程将弹出任务并最终执行作业。当有多个工作线程时，这些工作线程将一起处理这些任务。

## 轮询分发消息

* 抽取工具类

  ```java
  public class RabbitMqUtils {
   //得到一个连接的 channel
   public static Channel getChannel() throws Exception{
   //创建一个连接工厂
   ConnectionFactory factory = new ConnectionFactory();
   factory.setHost("182.92.234.71");
   factory.setUsername("admin");
   factory.setPassword("123");
   Connection connection = factory.newConnection();
   Channel channel = connection.createChannel();
   return channel;
   }
  }
  ```

* 启动两个消费者（工作线程）

  ```java
  public class Worker01 {
   private static final String QUEUE_NAME="hello";
   public static void main(String[] args) throws Exception {
   Channel channel = RabbitMqUtils.getChannel();
   DeliverCallback deliverCallback=(consumerTag,delivery)->{
   String receivedMessage = new String(delivery.getBody());
   System.out.println("接收到消息:"+receivedMessage);
   };
   CancelCallback cancelCallback=(consumerTag)->{
   System.out.println(consumerTag+"消费者取消消费接口回调逻辑");
   };
   System.out.println("C2 消费者启动等待消费......");
   channel.basicConsume(QUEUE_NAME,true,deliverCallback,cancelCallback);
   }
  }
  ```

* 启动一个生产者线程

  ```java
  public class Task01 {
   private static final String QUEUE_NAME="hello";
   public static void main(String[] args) throws Exception {
   try(Channel channel=RabbitMqUtils.getChannel();) {
   channel.queueDeclare(QUEUE_NAME,false,false,false,null);
   //从控制台当中接受信息
   Scanner scanner = new Scanner(System.in);
   while (scanner.hasNext()){
   String message = scanner.next();
   channel.basicPublish("",QUEUE_NAME,null,message.getBytes());
   System.out.println("发送消息完成:"+message);
   }
   }
   }
  }
  ```

* 结果展示

  通过程序执行发现生产者总共发送 4 个消息，消费者 1 和消费者 2 分别分得两个消息，并且 是按照有序的一个接收一次消息

  ![image-20220509125122043](D:\Picture\Note\RabbitMQ\工作队列结果展示.png)

# 消息应答

## 概念

消费者完成一个任务可能需要一段时间，如果其中一个消费者处理一个长的任务并仅只完成 了部分突然它挂掉了，会发生什么情况。RabbitMQ 一旦向消费者传递了一条消息，便立即将该消 息标记为删除。在这种情况下，突然有个消费者挂掉了，我们将丢失正在处理的消息。以及后续 发送给该消费这的消息，因为它无法接收到。

为了保证消息在发送过程中不丢失，rabbitmq 引入消息应答机制，消息应答就是:**消费者在接 收到消息并且处理该消息之后，告诉 rabbitmq 它已经处理了，rabbitmq 可以把该消息删除了。**

## 自动应答

 消息发送后立即被认为已经传送成功，这种模式需要在**高吞吐量和数据传输安全性方面做权 衡**,因为这种模式如果消息在接收到之前，消费者那边出现连接或者 channel 关闭，那么消息就丢 失了,当然另一方面这种模式消费者那边可以传递过载的消息，**没有对传递的消息数量进行限制**， 当然这样有可能使得消费者这边由于接收太多还来不及处理的消息，导致这些消息的积压，最终 使得内存耗尽，最终这些消费者线程被操作系统杀死，**所以这种模式仅适用在消费者可以高效并 以某种速率能够处理这些消息的情况下使用**。

## 消息应答的方法

1. `Channel.basicAck`（用于肯定确认）

   RabbitMQ已经知道并且成功的处理消息，可以将其丢弃了

2. `Channel.basicNack`（用于否定确认）

3. `Channel.basicReject`（用于否定确认）

   与Channel.basicNack相比少了一个参数，不处理该消息了直接拒绝可以将其丢弃

## Multiple的解释

**手动应答的好处是可以批量应答并且减少网络拥堵**

![image-20220510101632658](D:\Picture\Note\RabbitMQ\multiple解释) 

multiple 的 true 和 false 代表不同意思

* true 代表批量应答 channel 上未应答的消息

  > 比如说 channel 上有传送 tag 的消息 5,6,7,8 当前 tag 是 8 那么此时 5-8 的这些还未应答的消息都会被确认收到消息应答

* false 同上面相比

  > 只会应答 tag=8 的消息 5,6,7 这三个消息依然不会被确认收到消息应答

## 消息自动重新入队

如果消费者由于某些原因失去连接(其通道已关闭，连接已关闭或 TCP 连接丢失)，导致消息 未发送 ACK 确认，RabbitMQ 将了解到消息未完全处理，并将对其重新排队。如果此时其他消费者 可以处理，它将很快将其重新分发给另一个消费者。这样，即使某个消费者偶尔死亡，也可以确 保不会丢失任何消息。

![image-20220510101953504](D:\Picture\Note\RabbitMQ\消息自动重新入队.png)

## 消息手动应答代码

默认消息采用的是自动应答，所以我们要想实现消息消费过程中不丢失，需要把自动应答改 为手动应答，消费者在上面代码的基础上增加下面画红色部分代码。

1. 消费生产者

   ```java
   public class Producers {
   
       private static final String QUEUE_NAME = "hello";
   
       public static void main(String[] args) throws Exception {
           try (Channel channel = RabbitMQUtils.getChannel()) {
               channel.queueDeclare(QUEUE_NAME, false, false, false, null);
   
               Scanner scanner = new Scanner(System.in);
               System.out.println("请输入信息：");
               while (scanner.hasNext()) {
                   String message = scanner.next();
                   channel.basicPublish("", QUEUE_NAME, null, message.getBytes(StandardCharsets.UTF_8));
                   System.out.println("生产者发出消息");
               }
           }
   
   
       }
   }
   ```

2. 消费者1

   ```java
   public class Consumers1 {
       private static final String QUEUE_NAME = "hello";
   
       public static void main(String[] args) throws Exception {
           Channel channel = RabbitMQUtils.getChannel();
   
           System.out.println("c1等待接受消息处理时间较长");
   
           boolean autoAck = false;
           channel.basicConsume(QUEUE_NAME, autoAck, (consumerTag, delivery) -> {
               String message = new String(delivery.getBody());
               SleepUtils.sleep(1);
               System.out.println("接受消息：" + message);
               channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
           }, (consumerTag) -> {
               System.out.println(consumerTag + "消费者回调逻辑");
           });
       }
   }
   ```

3. 消费者2

   ```java
   public class Consumers2 {
       private static final String QUEUE_NAME = "hello";
   
       public static void main(String[] args) throws Exception {
           Channel channel = RabbitMQUtils.getChannel();
   
           System.out.println("c2等待接受消息处理时间较短");
   
           boolean autoAck = false;
           channel.basicConsume(QUEUE_NAME, autoAck, (consumerTag, delivery) -> {
               String message = new String(delivery.getBody());
               SleepUtils.sleep(10);
               System.out.println("接受消息：" + message);
               channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
           }, (consumerTag) -> {
               System.out.println(consumerTag + "消费者回调逻辑");
           });
       }
   }
   ```

4. 睡眠工具类

   ```java
   public class SleepUtils {
       public static void sleep(int second) {
           try {
               Thread.sleep(second * 1000);
           } catch (InterruptedException e) {
               Thread.currentThread().interrupt();
           }
       }
   }
   ```

# RabbitMQ持久化

# 概念

刚刚我们已经看到了如何处理任务不丢失的情况，但是如何保障当 RabbitMQ 服务停掉以后消 息生产者发送过来的消息不丢失。默认情况下 RabbitMQ 退出或由于某种原因崩溃时，它忽视队列 和消息，除非告知它不要这样做。确保消息不会丢失需要做两件事：我们需要将队列和消息都标 记为持久化。

## 队列实现持久化

之前我们创建的队列都是非持久化的，rabbitmq 如果重启的化，该队列就会被删除掉，如果 要队列实现持久化 需要在声明队列的时候把 durable 参数设置为持久化

![image-20220510111636335](D:\Picture\Note\RabbitMQ\队列持久化.png)

但是需要注意的就是如果之前声明的队列不是持久化的，需要把原先队列先删除，或者重新 创建一个持久化的队列，不然就会出现错误

![image-20220510111708921](D:\Picture\Note\RabbitMQ\队列持久化报错.png)

![image-20220510111749819](D:\Picture\Note\RabbitMQ\队列持久化在控制台的反应.png)

## 消息实现持久化

要想让消息实现持久化需要在消息生产者修改代码，MessageProperties.PERSISTENT_TEXT_PLAIN 添 加这个属性。

![image-20220510111033338](D:\Picture\Note\RabbitMQ\消息实现持久化.png)

将消息标记为持久化并不能完全保证不会丢失消息。尽管它告诉 RabbitMQ 将消息保存到磁盘，但是 这里依然存在当消息刚准备存储在磁盘的时候 但是还没有存储完，消息还在缓存的一个间隔点。此时并没 有真正写入磁盘。持久性保证并不强，但是对于我们的简单任务队列而言，这已经绰绰有余了。如果需要 更强有力的持久化策略：发布确认

```java
public class Producers {

    private static final String QUEUE_NAME = "hello";

    public static void main(String[] args) throws Exception {
        try (Channel channel = RabbitMQUtils.getChannel()) {
            boolean durable = true;
            channel.queueDeclare(QUEUE_NAME, durable, false, false, null);

            Scanner scanner = new Scanner(System.in);
            System.out.println("请输入信息：");
            while (scanner.hasNext()) {
                String message = scanner.next();
                channel.basicPublish("", QUEUE_NAME, MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes(StandardCharsets.UTF_8));
                System.out.println("生产者发出消息");
            }
        }


    }
}
```

# 不公平分发

在最开始的时候我们学习到 RabbitMQ 分发消息采用的轮训分发，但是在某种场景下这种策略并不是 很好，比方说有两个消费者在处理任务，其中有个消费者 1 处理任务的速度非常快，而另外一个消费者 2 处理速度却很慢，这个时候我们还是采用轮训分发的化就会到这处理速度快的这个消费者很大一部分时间 处于空闲状态，而处理慢的那个消费者一直在干活，这种分配方式在这种情况下其实就不太好，但是 RabbitMQ 并不知道这种情况它依然很公平的进行分发。

为了避免这种情况，我们可以设置参数 channel.basicQos(1);

![image-20220510111206381](D:\Picture\Note\RabbitMQ\不公平分发)

![image-20220510111240801](D:\Picture\Note\RabbitMQ\不公平分发流程图.png)

意思就是如果这个任务我还没有处理完或者我还没有应答你，你先别分配给我，我目前只能处理一个 任务，然后 rabbitmq 就会把该任务分配给没有那么忙的那个空闲消费者，当然如果所有的消费者都没有完 成手上任务，队列还在不停的添加新任务，队列有可能就会遇到队列被撑满的情况，这个时候就只能添加 新的 worker 或者改变其他存储任务的策略。

```java
public class Consumers1 {
    private static final String QUEUE_NAME = "hello";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMQUtils.getChannel();

        System.out.println("c1等待接受消息处理时间较长");

        int prefetchCount = 1;
        channel.basicQos(prefetchCount);

        boolean autoAck = false;
        channel.basicConsume(QUEUE_NAME, autoAck, (consumerTag, delivery) -> {
            String message = new String(delivery.getBody());
            SleepUtils.sleep(1);
            System.out.println("接受消息：" + message);
            channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
        }, (consumerTag) -> {
            System.out.println(consumerTag + "消费者回调逻辑");
        });
    }
}
```

```java
public class Consumers2 {
    private static final String QUEUE_NAME = "hello";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMQUtils.getChannel();

        System.out.println("c2等待接受消息处理时间较短");

        int prefetchCount = 1;
        channel.basicQos(prefetchCount);

        boolean autoAck = false;
        channel.basicConsume(QUEUE_NAME, autoAck, (consumerTag, delivery) -> {
            String message = new String(delivery.getBody());
            SleepUtils.sleep(10);
            System.out.println("接受消息：" + message);
            channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
        }, (consumerTag) -> {
            System.out.println(consumerTag + "消费者回调逻辑");
        });
    }
}
```

# 预取值

> 本身消息的发送就是异步发送的，所以在任何时候，channel 上肯定不止只有一个消息另外来自消费 者的手动确认本质上也是异步的。因此这里就存在一个未确认的消息缓冲区，因此希望开发人员能限制此 缓冲区的大小，以避免缓冲区里面无限制的未确认消息问题。这个时候就可以通过使用 basic.qos 方法设 置“预取计数”值来完成的。该值定义通道上允许的未确认消息的最大数量。一旦数量达到配置的数量， RabbitMQ 将停止在通道上传递更多消息，除非至少有一个未处理的消息被确认，例如，假设在通道上有 未确认的消息 5、6、7，8，并且通道的预取计数设置为 4，此时 RabbitMQ 将不会在该通道上再传递任何 消息，除非至少有一个未应答的消息被 ack。比方说 tag=6 这个消息刚刚被确认 ACK，RabbitMQ 将会感知 这个情况到并再发送一条消息。消息应答和 QoS 预取值对用户吞吐量有重大影响。通常，增加预取将提高 向消费者传递消息的速度。虽然自动应答传输消息速率是最佳的，但是，在这种情况下已传递但尚未处理 的消息的数量也会增加，从而增加了消费者的 RAM 消耗(随机存取存储器)应该小心使用具有无限预处理 的自动确认模式或手动确认模式，消费者消费了大量的消息如果没有确认的话，会导致消费者连接节点的 内存消耗变大，所以找到合适的预取值是一个反复试验的过程，不同的负载该值取值也不同 100 到 300 范 围内的值通常可提供最佳的吞吐量，并且不会给消费者带来太大的风险。预取值为 1 是最保守的。当然这 将使吞吐量变得很低，特别是消费者连接延迟很严重的情况下，特别是在消费者连接等待时间较长的环境 中。对于大多数应用来说，稍微高一点的值将是最佳的。

![image-20220510112703519](D:\Picture\Note\RabbitMQ\预取值.png)

个人感觉类似于权值的含义

```java
public class Consumers1 {
    private static final String QUEUE_NAME = "hello";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMQUtils.getChannel();

        System.out.println("c1等待接受消息处理时间较长");

        int prefetchCount = 2;
        channel.basicQos(prefetchCount);

        boolean autoAck = false;
        channel.basicConsume(QUEUE_NAME, autoAck, (consumerTag, delivery) -> {
            String message = new String(delivery.getBody());
            SleepUtils.sleep(1);
            System.out.println("接受消息：" + message);
            channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
        }, (consumerTag) -> {
            System.out.println(consumerTag + "消费者回调逻辑");
        });
    }
}
```

```java
public class Consumers2 {
    private static final String QUEUE_NAME = "hello";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMQUtils.getChannel();

        System.out.println("c2等待接受消息处理时间较短");

        int prefetchCount = 5;
        channel.basicQos(prefetchCount);

        boolean autoAck = false;
        channel.basicConsume(QUEUE_NAME, autoAck, (consumerTag, delivery) -> {
            String message = new String(delivery.getBody());
            SleepUtils.sleep(10);
            System.out.println("接受消息：" + message);
            channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
        }, (consumerTag) -> {
            System.out.println(consumerTag + "消费者回调逻辑");
        });
    }
}
```

