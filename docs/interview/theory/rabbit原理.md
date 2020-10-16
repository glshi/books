# RabbitMQ 

## 为什么使用消息队列

消息队列用于系统之间解耦，通过高性能消息中间件，提升系统吞吐量，降低导致系统耦合。 当前有各种消息队列，RabbitMQ、Kafka、ActiveMQ等，为什么使用RabbitMQ？ 消息队列从使用场景来分为两类：

- 一类是大数据的数据流处理，数据采集者作为生产者数据通过消息队列从到后端处理。这种场景要求高吞吐高并发，Kafka专门为这种场景设计。
- 另一类是消息高可靠低时延在系统之间传递，这种场景要求可靠性高，消息不能丢；要求时延低。AMQP协议专门为这种场景设计，RabbitMQ是AMQP协议实现者之一，也是当前使用的最广的AMQP消息队列。

## 消息队列处理流程

以一个提供网页转PDF的Web应用为例，描述消息处理流程。

![RabbitMQ基础插图](https://www.cloudamqp.com/img/blog/rabbitmq-beginners-updated.png)

1. 用户发送网页转PDF请求到Web应用
2. Web应用发送一个消息到RabbitMQ，包括网页内容、用户名、邮箱等
3. RabbitMQ的Exchange接收到消息后，将消息路由到某一个队列
4. PEF生成器消费队列的消息，生成PDF

# RabbitMQ 中的概念模型

## 消息模型

所有 MQ 产品从模型抽象上来说都是一样的过程：
消费者（consumer）订阅某个队列。生产者（producer）创建消息，然后发布到队列（queue）中，最后将消息发送到监听的消费者。

![img](https:////upload-images.jianshu.io/upload_images/5015984-066ff248d5ff8eed.png?imageMogr2/auto-orient/strip|imageView2/2/w/401/format/webp)

​																						消息流

## RabbitMQ 基本概念

上面只是最简单抽象的描述，具体到 RabbitMQ 则有更详细的概念需要解释。上面介绍过 RabbitMQ 是 AMQP 协议的一个开源实现，所以其内部实际上也是 AMQP 中的基本概念：

![img](https:////upload-images.jianshu.io/upload_images/5015984-367dd717d89ae5db.png?imageMogr2/auto-orient/strip|imageView2/2/w/554/format/webp)

​																				RabbitMQ 内部结构

1. Message
    消息，消息是不具名的，它由消息头和消息体组成。消息体是不透明的，而消息头则由一系列的可选属性组成，这些属性包括routing-key（路由键）、priority（相对于其他消息的优先权）、delivery-mode（指出该消息可能需要持久性存储）等。
2. Publisher
    消息的生产者，也是一个向交换器发布消息的客户端应用程序。
3. Exchange
    交换器，用来接收生产者发送的消息并将这些消息路由给服务器中的队列。
4. Binding
    绑定，用于消息队列和交换器之间的关联。一个绑定就是基于路由键将交换器和消息队列连接起来的路由规则，所以可以将交换器理解成一个由绑定构成的路由表。
5. Queue
    消息队列，用来保存消息直到发送给消费者。它是消息的容器，也是消息的终点。一个消息可投入一个或多个队列。消息一直在队列里面，等待消费者连接到这个队列将其取走。
6. Connection
    网络连接，比如一个TCP连接。
7. Channel
    信道，多路复用连接中的一条独立的双向数据流通道。信道是建立在真实的TCP连接内地虚拟连接，AMQP 命令都是通过信道发出去的，不管是发布消息、订阅队列还是接收消息，这些动作都是通过信道完成。因为对于操作系统来说建立和销毁 TCP 都是非常昂贵的开销，所以引入了信道的概念，以复用一条 TCP 连接。
8. Consumer
    消息的消费者，表示一个从消息队列中取得消息的客户端应用程序。
9. Virtual Host
    虚拟主机，表示一批交换器、消息队列和相关对象。虚拟主机是共享相同的身份认证和加密环境的独立服务器域。每个 vhost 本质上就是一个 mini 版的 RabbitMQ 服务器，拥有自己的队列、交换器、绑定和权限机制。vhost 是 AMQP 概念的基础，必须在连接时指定，RabbitMQ 默认的 vhost 是 / 。
10. Broker
     表示消息队列服务器实体。

## AMQP 中的消息路由

AMQP 中消息的路由过程和 Java 开发者熟悉的 JMS 存在一些差别，AMQP 中增加了 Exchange 和 Binding 的角色。生产者把消息发布到 Exchange 上，消息最终到达队列并被消费者接收，而 Binding 决定交换器的消息应该发送到那个队列。

![img](https:////upload-images.jianshu.io/upload_images/5015984-7fd73af768f28704.png?imageMogr2/auto-orient/strip|imageView2/2/w/484/format/webp)



# RabbitMQ

RabbitMQ是一个开源的消息代理和队列服务器，用来通过普通协议在不同的应用之间共享数据(跨平台跨语言)。RabbitMQ是使用Erlang语言编写，并且基于AMQP协议实现。

## RabbitMQ的优势：

- **`可靠性(Reliablity)：`**使用了一些机制来保证可靠性，比如持久化、传输确认、发布确认。
- **`灵活的路由(Flexible Routing)：`**在消息进入队列之前，通过Exchange来路由消息。对于典型的路由功能，Rabbit已经提供了一些内置的Exchange来实现。针对更复杂的路由功能，可以将多个Exchange绑定在一起，也通过插件机制实现自己的Exchange。
- **`消息集群(Clustering)：`**多个RabbitMQ服务器可以组成一个集群，形成一个逻辑Broker。
- **`高可用(Highly Avaliable Queues)：`**队列可以在集群中的机器上进行镜像，使得在部分节点出问题的情况下队列仍然可用。
- **`多种协议(Multi-protocol)：`**支持多种消息队列协议，如STOMP、MQTT等。
- **`多种语言客户端(Many Clients)：`**几乎支持所有常用语言，比如Java、.NET、Ruby等。
- **`管理界面(Management UI)：`**提供了易用的用户界面，使得用户可以监控和管理消息Broker的许多方面。
- **`跟踪机制(Tracing)：`**如果消息异常，RabbitMQ提供了消息的跟踪机制，使用者可以找出发生了什么。
- **`插件机制(Plugin System)：`**提供了许多插件，来从多方面进行扩展，也可以编辑自己的插件。

## RabbitMQ的整体架构

![img](https:////upload-images.jianshu.io/upload_images/17039633-b0adf1dfade2f122.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)



## RabbitMQ的消息流转

![img](https:////upload-images.jianshu.io/upload_images/17039633-6fe89a2074201de7.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)



## RabbitMQ各组件功能



![img](https:////upload-images.jianshu.io/upload_images/5015984-367dd717d89ae5db.png?imageMogr2/auto-orient/strip|imageView2/2/w/554/format/webp)



- **`Broker：`**标识消息队列服务器实体.
- **`Virtual Host：`**虚拟主机。标识一批交换机、消息队列和相关对象。虚拟主机是共享相同的身份认证和加密环境的独立服务器域。每个vhost本质上就是一个mini版的RabbitMQ服务器，拥有自己的队列、交换器、绑定和权限机制。vhost是AMQP概念的基础，必须在链接时指定，RabbitMQ默认的vhost是 /。
- **`Exchange：`**交换器，用来接收生产者发送的消息并将这些消息路由给服务器中的队列。
- **`Queue：`**消息队列，用来保存消息直到发送给消费者。它是消息的容器，也是消息的终点。一个消息可投入一个或多个队列。消息一直在队列里面，等待消费者连接到这个队列将其取走。
- **`Banding：`**绑定，用于消息队列和交换机之间的关联。一个绑定就是基于路由键将交换机和消息队列连接起来的路由规则，所以可以将交换器理解成一个由绑定构成的路由表。
- **`Channel：`**信道，多路复用连接中的一条独立的双向数据流通道。信道是建立在真实的TCP连接内地虚拟链接，AMQP命令都是通过信道发出去的，不管是发布消息、订阅队列还是接收消息，这些动作都是通过信道完成。因为对于操作系统来说，建立和销毁TCP都是非常昂贵的开销，所以引入了信道的概念，以复用一条TCP连接。
- **`Connection：`**网络连接，比如一个TCP连接。
- **`Publisher：`**消息的生产者，也是一个向交换器发布消息的客户端应用程序。
- **`Consumer：`**消息的消费者，表示一个从一个消息队列中取得消息的客户端应用程序。
- **`Message：`**消息，消息是不具名的，它是由消息头和消息体组成。消息体是不透明的，而消息头则是由一系列的可选属性组成，这些属性包括routing-key(路由键)、priority(优先级)、delivery-mode(消息可能需要持久性存储[消息的路由模式])等。

## RabbitMQ的多种Exchange类型

Exchange分发消息时，根据类型的不同分发策略有区别。目前共四种类型：direct、fanout、topic、headers(headers匹配AMQP消息的header而不是路由键(Routing-key)，此外headers交换器和direct交换器完全一致，但是性能差了很多，目前几乎用不到了。所以直接看另外三种类型。)。

> direct

![img](https:////upload-images.jianshu.io/upload_images/5015984-13db639d2c22f2aa.png?imageMogr2/auto-orient/strip|imageView2/2/w/385/format/webp)

MacDown Screenshot



消息中的路由键(routing key)如果和Binding中的binding key一致，交换器就将消息发到对应的队列中。路由键与队列名完全匹配。

> fanout

![img](https:////upload-images.jianshu.io/upload_images/5015984-2f509b7f34c47170.png?imageMogr2/auto-orient/strip|imageView2/2/w/463/format/webp)

MacDown Screenshot



每个发到fanout类型交换器的消息都会分到所有绑定的队列上去。fanout交换器不处理该路由键，只是简单的将队列绑定到交换器上，每个发送到交换器的消息都会被转发到与该交换器绑定的所有队列上。很像子网广播，每台子网内的主机都获得了一份复制的消息。fanout类型转发消息是最快的。

> topic

![img](https:////upload-images.jianshu.io/upload_images/5015984-275ea009bdf806a0.png?imageMogr2/auto-orient/strip|imageView2/2/w/558/format/webp)

MacDown Screenshot



topic交换器通过模式匹配分配消息的路由键属性，将路由键和某个模式进行匹配，此时队列需要绑定到一个模式上。它将路由键(routing-key)和绑定键(bingding-key)的字符串切分成单词，这些单词之间用点隔开。它同样也会识别两个通配符："#"和"*"。#匹配0个或多个单词，匹配不多不少一个单词。

## TTL

TTL(Time To Live)：生存时间。RabbitMQ支持消息的过期时间，一共两种。

- 在消息发送时可以进行指定。通过配置消息体的properties，可以指定当前消息的过期时间。
- 在创建Exchange时可进行指定。从进入消息队列开始计算，只要超过了队列的超时时间配置，那么消息会自动清除。

## 死信队列DLX

**`死信队列(DLX Dead-Letter-Exchange)：`**利用DLX，当消息在一个队列中变成死信(dead message)之后，它能被重新publish到另一个Exchange，这个Exchange就是DLX。

DLX也是一个正常的Exchange，和一般的Exchange没有区别，它能在任何的队列上被指定，实际上就是设置某个队列的属性。

当这个队列中有死信时，RabbitMQ就会自动的将这个消息重新发布到设置的Exchange上去，进而被路由到另一个队列。

可以监听这个队列中消息做相应的处理，这个特性可以弥补RabbitMQ3.0之前支持的immediate参数的功能。

**消息变成死信的几种情况：**

- 消息被拒绝(basic.reject/basic.nack)并且requeue=false
- 消息TTL过期
- 队列达到最大长度

**死信队列设置：**需要设置死信队列的exchange和queue，然后通过routing key进行绑定。**只不过我们需要在队列加上一个参数即可**。



```jsx
Map<String, Object> arguments = Maps.newHashMapWithExpectedSize(3);
arguments.put("x-message-ttl", dlx-ttl);
arguments.put("x-dead-letter-exchange","exchange-name");
arguments.put("x-dead-letter-routing-key", "routing-key");
Queue ret = QueueBuilder.durable("queue-name".withArguments(arguments).build();
```

只需要通过监听该死信队列即可处理死信消息。还可以通过死信队列完成延时队列。

## 消费端ACK与NACK

消费端进行消费的时候，如果由于业务异常可以进行日志的记录，然后进行补偿。由于服务器宕机等严重问题，我们需要手动进行ACK保障消费端消费成功。

**消费端重回队列**是为了对没有成功处理消息，把消息重新返回到Broker。一般来说，实际应用中都会关闭重回队列，也就是设置为false。



```java
// deliveryTag：消息在mq中的唯一标识
// multiple：是否批量(和qos设置类似的参数)
// requeue：是否需要重回队列。或者丢弃或者重回队首再次消费。
public void basicNack(long deliveryTag, boolean multiple, boolean requeue) 
```

## 生产者Confirm机制

- 消息的确认，是指生产者投递消息后，如果Broker收到消息，则会给我们生产者一个应答。
- 生产者进行接受应答，用来确认这条消息是否正常的发送到了Broker，这种方式也是消息的可靠性投递的核心保障！

> 如何实现Confirm确认消息？

![img](https:////upload-images.jianshu.io/upload_images/17039633-95cb101479ed6092.png?imageMogr2/auto-orient/strip|imageView2/2/w/958/format/webp)

确认机制流程



1、在channel上开启确认模式：channel.confirmSelect()
 2、在channel上开启监听：addConfirmListener，监听成功和失败的处理结果，根据具体的结果对消息进行重新发送或记录日志处理等后续操作。

## Return消息机制

Return Listener**用于处理一些不可路由的消息**。

我们的消息生产者，通过指定一个Exchange和Routing，把消息送达到某一个队列中去，然后我们的消费者监听队列进行消息的消费处理操作。

但是在某些情况下，如果我们在发送消息的时候，当前的exchange不存在或者指定的路由key路由不到，这个时候我们需要监听这种不可达消息，就需要使用到Returrn Listener。

基础API中有个关键的配置项`Mandatory`：如果为true，监听器会收到路由不可达的消息，然后进行处理。如果为false，broker端会自动删除该消息。

通过chennel.addReturnListener(ReturnListener rl)传入已经重写过handleReturn方法的ReturnListener。

## 消费端自定义监听(推模式和拉模式pull/push)

- 一般通过while循环进行consumer.nextDelivery()方法进行获取下一条消息进行那个消费。(通过while将拉模式模拟成推模式，但是死循环会耗费CPU资源。)
- 通过自定义Consumer，实现更加方便、可读性更强、解耦性更强的方式。(现默认使用的模式，直接订阅到queue上，如果有数据，就等待mq推送过来)



```css
Basic.Consume将信道(Channel)置为接收模式，直到取消队列的订阅为止。
在接受模式期间，RabbitMQ会不断的推送消息给消费者。
当然推送消息的个数还是受Basic.Qos的限制。
如果只想从队列获得单条消息而不是持续订阅，建议还是使用Basic.Get进行消费。
但是不能将Basic.Get放在一个循环里来代替Basic.Consume，这样会严重影响RabbitMQ的性能。
如果要实现高吞吐量，消费者理应使用Basic.Consume方法。
```

![img](https:////upload-images.jianshu.io/upload_images/17039633-4f6e055501e90ae5.png?imageMogr2/auto-orient/strip|imageView2/2/w/1005/format/webp)

两种模型对比

## 如何保证幂等性

消费端实现幂等性，就意味着我们的消息永远不会消费多次，即使我们收到了多条一样的信息。

- 唯一ID+指纹码机制，利用数据库主键去重

```csharp
select count(1) from table where id = id+指纹码
优点：实现简单
缺点：高并发下有数据库写入的性能瓶颈
解决：跟进ID进行分库分表进行算法路由
```

- 利用redis的原子性去实现

```undefined
问题1：是否需要落库。如果落库，如何保证数据的一致性和原子性？
问题2：如果不进行落库，缓存种的数据如果设置定时同步的策略？
```

## 如何保证可靠性？

> 什么是生产端的可靠性投递？

- 保证消息的成功发出
- 保障MQ节点的成功接受
- 发送端收到MQ节点(Broker)确认应答
- 完善消息的补偿机制

> 解决方案

- 消息落库，对消息状态进行变更。

  ![img](https:////upload-images.jianshu.io/upload_images/17039633-7359ebe49f3da8be.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

  数据落库



```undefined
缺点：对数据库有多次操作。不适用于高并发业务。
```

- 消息的延迟投递，做二次确认，回调检查。

  ![img](https:////upload-images.jianshu.io/upload_images/17039633-cca034c274ab484c.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

  延迟投递



```css
拆出一个回调服务。将落库、检查等操作安排至回调服务上。
1：发送者发送信息至MQ，消费者为下游业务方。
      1.1：成功后，作为发送者发送信息至MQ，消费者为回调服务。
              1.1.1 回调服务接受数据后，落库。
       1.2：失败，等待发送者的延时投递信息。
2、发送者发送延迟投递信息至MQ，消费者为回调服务。
      1.1：查库，确认下游服务方消费已成功。
      1.2：查库，确认下游服务方消费失败，通过rpc调用发送者的接口重新发送。

消息发送者发送的两条信息是同时发送的。

减少了对库的操作，同时解耦，保证了性能，不能百分百保证可靠性
```

## 消费端如何限流

当海量消息瞬间推送过来，单个客户端无法同时处理那么多数据，严重会导致系统宕机。这时，需要削峰。

RabbitMQ提供了一种qos(服务质量保证)功能。即**在非自动确认消息的前提下**(非ACK)，如果一定数目的消息(通过基于consume或者channel设置qos的值)未被确认前，不进行消费新的消息。



```csharp
// prefetchSize：消息体大小限制；0为不限制
// prefetchCount：RabbitMQ同时给一个消费者推送的消息个数。即一旦有N个消息还没有ack，则该consumer将block掉，直到有消息ack。默认是1.
// global：限流策略的应用级别。consumer[false]、channel[true]。
void BasicQos(unit prefetchSize, unshort prefetchCount, bool global);
channel.basicQos(...);
```

## Channel模式和Connection模式

参考：https://www.jianshu.com/p/2c2a7cfdd38a

Connection和Channel是spring-amqp中的概念，并非rabbitmq中的概念，官方文档对Connection和Channel有这样的描述：



```dart
Sharing of the connection 
is possible since the "unit of work" for messaging with AMQP 
is actually a "channel"
 (in some ways, 
this is similar to the relationship 
between a Connection and a Session in JMS).
```

**`CHANNEL模式：`**程序运行期间ConnectionFactory会维护着一个Connection，所有的操作都会使用这个Connection，但一个Connection中可以有多个Channel，操作rabbitmq之前都必须先获取到一个Channel，否则就会阻塞（可以通过setChannelCheckoutTimeout()设置等待时间），这些Channel会被缓存（缓存的数量可以通过setChannelCacheSize()设置）；

**`CONNECTION模式：`**这个模式下允许创建多个Connection，会缓存一定数量的Connection，每个Connection中同样会缓存一些Channel，除了可以有多个Connection，其它都跟CHANNEL模式一样。

关于CONNECTION模式中，可以存在多个Connection的使用场景，官方文档的描述：



```jsx
The use of separate connections 
might be useful in some environments, 
such as consuming from an HA cluster,
in conjunction with a load balancer, 
to connect to different cluster members.
```

setChannelCacheSize：设置每个Connection中（注意是每个Connection）可以缓存的Channel数量，注意只是缓存的Channel数量，不是Channel的数量上限，操作rabbitmq之前（send/receive message等）要先获取到一个Channel，获取Channel时会先从缓存中找闲置的Channel，如果没有则创建新的Channel，当Channel数量大于缓存数量时，多出来没法放进缓存的会被关闭。

> 注意，改变这个值不会影响已经存在的Connection，只影响之后创建的Connection。
>  有时会出现connection closed错误。rabbitTemplate作者对于这种问题的解决方案，他给的方案很简单，单纯的增加connection数：



```css
connectionFactory.setChannelCacheSize(100);
```

setChannelCheckoutTimeout：当这个值大于0时，channelCacheSize不仅是缓存数量，同时也会变成数量上限，从缓存获取不到可用的Channel时，不会创建新的Channel，会等待这个值设置的毫秒数，到时间仍然获取不到可用的Channel会抛出AmqpTimeoutException异常。

同时，在CONNECTION模式，这个值也会影响获取Connection的等待时间，超时获取不到Connection也会抛出AmqpTimeoutException异常。

## 消费端的Concurrency和Prefetch模式

暂未整理，参考https://www.jianshu.com/p/04a1d36f52ba

只有Prefetch模式才可以设置qoshttps://www.jianshu.com/p/4d043d3045ca

## RabbitMQ集群

参考：https://www.jianshu.com/p/b7cc32b94d2a

RabbitMQ最优秀的功能之一就是内建集群，这个功能涉及的目的是允许消费者和生产者在节点崩溃的情况下继续运行，以及通过添加更多的节点来线性扩展消息通信吞吐量。RabbitMQ内部利用Erlang提供的分布式通信框架OTP来满足上述需求，使客户端在失去一个RabbitMQ节点连接的情况下，还是能够重新连接到集群中的其他节点继续胜场、消费信息。

RabbitMQ会始终记录以下四中类型的内部元数据：

- 队列元数据：包括队列名称和他们的属性，比如是否可持久化，是否可持久化，是否自动删除。
- 交换器元数据：交换器名称、类型、属性。
- 绑定元数据：内部是一张表格，记录如何将消息路由到队列。
- vhost元数据：为vhost内部的队列、交换器、绑定提供命名空间和安全属性。

在单一节点中，RabbitMQ会将所有这些信息存储在内存中，同时将标记为可持久化的队列、交换器、 绑定存储在硬盘上。存到硬盘上可以确保队列和交换器在节点重启后能够重建。**而在集群模式下，同样也提供了两种选择：存到硬盘上(独立节点的默认配置)，存在内存中。**

如果在集群中创建队列，集群只会在单个节点而不是所有节点上创建完整的队列信息（元数据、状态、内容）。结果是只有队列的所有者节点知道有关队列的所有信息，因此当集群节点崩溃时，该节点的队列和绑定就消失了，并且任何匹配该队列的绑定的新消息也丢失了。还好RabbitMQ 2.6.0之后提供了镜像队列以避免集群节点故障导致的队列内容不可用。

RabbitMQ 集群中可以共享 user、vhost、exchange等，所有的数据和状态都是必须在所有节点上复制的，例外就是上面所说的消息队列。RabbitMQ 节点可以动态的加入到集群中。

当在集群中声明队列、交换器、绑定的时候，这些操作会直到所有集群节点都成功提交元数据变更后才返回。集群中有内存节点和磁盘节点两种类型，内存节点虽然不写入磁盘，但是它的执行比磁盘节点要好。内存节点可以提供出色的性能，磁盘节点能保障配置信息在节点重启后仍然可用，那集群中如何平衡这两者呢？

RabbitMQ 只要求集群中至少有一个磁盘节点，所有其他节点可以是内存节点，当节点加入或离开集群时，它们必须要将该变更通知到至少一个磁盘节点。如果只有一个磁盘节点，刚好又是该节点崩溃了，那么集群可以继续路由消息，但不能创建队列、创建交换器、创建绑定、添加用户、更改权限、添加或删除集群节点。换句话说集群中的唯一磁盘节点崩溃的话，集群仍然可以运行，但直到该节点恢复，否则无法更改任何东西。



参考:
https://www.jianshu.com/p/78847c203b76
