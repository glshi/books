# dubbo是什么

**dubbo**是一个分布式服务中间件，是高性能和透明化的RPC远程服务调用解决方案，主要通过资源调度和服务治理来解决分布式[架构](http://www.solves.com.cn/it/cxkf/jiagou/)下服务资源浪费以提高集群的使用率。核心部分包含：

- 远程通讯：提供多种基于长连接的NIO的抽象封装，包括多种线程模型，序列化方式，以及请求-响应模式的信息交互
- 集群容错：提供基于接口方法的透明化远程调用，包括多协议支持，软负载均衡，失败容错，地址路由，动态配置的集群策略
- 服务发现：基于注册中心目录服务，使服务消费者能动态查找服务提供者，使地址透明，服务提供者可以根据流量平滑增加节点

 

# 1.dubbo架构-组件角色

在深入架构前，先了解一下dubbo的组件角色，如下图：

 

![dubbo服务治理架构与原理](https://img.solves.com.cn/p/2019/08-05/ee5d776a316d1415d926829f1a899fcb.jpg)

 

 

- Provider：暴露服务的服务提供方
- Container：服务运行容器
- Registry：服务注册与发现的注册中心
- Consumer：调用远程服务的服务消费方
- Monitor：统计服务调用次数和耗时的监控中心

调用关系分析：

- 服务容器Container负责启动，加载，运行服务提供者。
- 服务提供者Provider在启动时，向注册中心注册自己提供的服务。
- 服务消费者Consumer在启动时，向注册中心订阅自己所需的服务。
- 注册中心Registry返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。
- 服务消费者Consumer，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
- 服务消费者Consumer和提供者Provider，在[内存](http://www.solves.com.cn/it/yj/nc/)中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心Monitor。

 

![dubbo服务治理架构与原理](https://img.solves.com.cn/p/2019/08-05/3c17c127764d82eec163c553d0f4f6d9.jpg)

 

 

> 问：各组件角色之间是采用长连接还是短连接交互的？为什么？ 答：服务提供者->消费者，服务提供者->注册中心，服务消费者->注册中心。以上三个交互环节采用的是长连接。至于为什么？笔者分析是因为基于注册中心的动态服务发现与注册机制，所以需要通过心跳的方式实时检测节点的状态

 

# 2.dubbo架构-模型分层

先来看一下dubbo这个中间件的领域模型分层结构，如下图：

 

![dubbo服务治理架构与原理](https://img.solves.com.cn/p/2019/08-05/311b229d8f27520008ee25c8e8fab12b.jpg)

 

 

中间件架构分层[设计](http://www.solves.com.cn/it/rj/ps/)：

- 服务接口层（Service）：该层是与实际业务逻辑相关的，根据服务提供方和服务消费方的业务设计对应的接口和实现。
- 配置层（Config）：对外配置接口，以ServiceConfig和ReferenceConfig为中心，可以直接new配置类，也可以通过spring解析配置生成配置类。
- 服务代理层（Proxy）：服务接口透明代理，生成服务的客户端Stub和服务器端Skeleton，以ServiceProxy为中心，扩展接口为ProxyFactory。
- 服务注册层（Registry）：封装服务地址的注册与发现，以服务URL为中心，扩展接口为RegistryFactory、Registry和RegistryService。可能没有服务注册中心，此时服务提供方直接暴露服务。
- 集群层（Cluster）：封装多个提供者的路由及负载均衡，并桥接注册中心，以Invoker为中心，扩展接口为Cluster、 Directory、Router和LoadBalance。将多个服务提供方组合为一个服务提供方，实现对服务消费方来透明，只需要与一个服务提供方进 行交互。
- 监控层（Monitor）：RPC调用次数和调用时间监控，以Statistics为中心，扩展接口为MonitorFactory、Monitor和MonitorService。
- 远程调用层（Protocol）：封将RPC调用，以Invocation和Result为中心，扩展接口为Protocol、Invoker和 Exporter。Protocol是服务域，它是Invoker暴露和引用的主功能入口，它负责Invoker的生命周期管理。Invoker是实体 域，它是Dubbo的核心模型，其它模型都向它靠扰，或转换成它，它代表一个可执行体，可向它发起invoke调用，它有可能是一个本地的实现，也可能是 一个远程的实现，也可能一个集群实现。
- 信息交换层（Exchange）：封装请求响应模式，同步转异步，以Request和Response为中心，扩展接口为Exchanger、ExchangeChannel、ExchangeClient和ExchangeServer。
- 网络传输层（Transport）：抽象mina和netty为统一接口，以Message为中心，扩展接口为Channel、Transporter、Client、Server和Codec。
- 数据序列化层（Serialize）：可复用的一些工具，扩展接口为Serialization、 ObjectInput、ObjectOutput和ThreadPool。

各层之间关系总结：

- Protocol是核心层，简单组合只需要Protocol,Invoker,Exporter就能完成远程方法调用。Filter是在Invoker过程中增加的拦截层
- Registry和Monitor属于独立角色，Registry属于注册中心层
- Consumer和Provider属于抽象概念，因为一个节点在提供服务的同时调用了别的服务，那么他既属于Consumer又属于Provider
- Proxy层封装过了所有接口了透明代理，但是其他层都以Invoker为中心，只有暴露接口时才用Proxy将Invoker转换为接口（接口转换为Invoker）
- Remoting是dubbo协议的实现，内部分为Transport传输层和Exchange信息交换层，Transport只负责单向数据传输（Netty，Mina的抽象，也可以扩展使用UDP协议传输），Exchange是在Transport上定义了 Request-Response的模型

 

# 3.调用链路

在看源码之前，先看一下dubbo服务调用链路过程。如下图：

 

![dubbo服务治理架构与原理](https://img.solves.com.cn/p/2019/08-05/12d0f7018286a8d5b4b63d08f9d049d0.jpg)

 

 

首先服务消费者通过代理对象 Proxy 发起远程调用，接着通过网络客户端 Client 将编码后的请求发送给服务提供方的网络层上，也就是 Server。Server 在收到请求后，首先要做的事情是对数据包进行解码。然后将解码后的请求发送至分发器 Dispatcher，再由分发器将请求派发到指定的线程池上，最后由线程池调用具体的服务。这就是一个远程调用请求的发送与接收过程。

# 3.1.调用方式

dubbo支持同步和异步两种调用方式，其中异步调用分为“有返回值”的异步调用和“无返回值”的异步调用。所谓“无返回值”异步调用是指服务消费方只管调用，但不关心调用结果，此时 dubbo 会直接返回一个空的 RpcResult。若要使用异步特性，需要服务消费方手动进行配置。默认情况下，Dubbo 使用同步调用方式。

异步调用方式如下：

-  

```
/* 异步方法配置 */
<dubbo:reference id="asyncService" check="false" interface="com.alibaba.dubbo.demo.AsyncService" url="localhost:20880">
 <dubbo:method name="sayHello" async="true" />
</dubbo:reference>
```

Consumer端调用方式如下：

- 调用直接返回null
-  

```
String result = asyncService.sayHello("world");
```

- 通过RpcContext获取Future对象，调用get()阻塞直到返回结果
-  

```
asyncService.sayHello("world");
Future<String> future = RpcContext.getContext().getFuture();
String result = future.get();
```

- 通过ResponseFuture设置回调，执行完成调用done()，异常调用caught()
-  

```
asyncService.sayHello("world");
ResponseFuture responseFuture = ((FutureAdapter)RpcContext.getContext().getFuture()).getFuture();
responseFuture.setCallback(new ResponseCallback() {
 @Override
 public void done(Object response) {
 System.out.println("done");
 }
 @Override
 public void caught(Throwable exception) {
 System.out.println("caught");
 }
});
try {
 System.out.println("result = " + responseFuture.get());
} catch (RemotingException e) {
 e.printStackTrace();
}
```

 

# 4.Dubbo VS SpringCloud VS Other

 

![dubbo服务治理架构与原理](https://img.solves.com.cn/p/2019/08-05/9ff83994f6e923251b2a675b1c75373f.jpg)

 

 

Dubbo实现了服务治理的基础，但是要完成一个完备的微服务架构，还需要在各环节去扩展和完善以保证集群的[健康](http://www.solves.com.cn/sh/jk/)，以减轻开发、测试以及运维各个环节上增加出来的压力，这样才能让各环节人员真正的专注于业务逻辑。而Spring Cloud依然发扬了Spring Source整合一切的作风，以标准化的姿态将一些微服务架构的成熟产品与框架揉为一体，并继承了Spring Boot简单配置、快速开发、轻松部署的特点，让原本复杂的架构工作变得相对容易上手一些。所以，如果选择Dubbo请务必在各个环节做好整套解决方案的准备，不然很可能随着服务数量的增长，整个团队都将疲于应付各种架构上不足引起的困难。而如果选择Spring Cloud，相对来说每个环节都已经有了对应的组件支持，可能有些也不一定能满足你所有的需求。

# 5.踩坑实例

5.1.Reference注解

使用@Reference注解在消费者端注入提供者依赖的时候，MVC容器获取对象时可能出现Null的情况。Reference注解生成的实例是不会交给Spring容器托管的，只是作为Spring管理bean的一个属性赋值去操作，所以无法通过BeanFactory.getBean()获得。如果容器初始化时先加载了MVC容器，随后加载Dubbo实例，这时在Controller层引用的dubbo对象会造成空指针的异常，所以加载时需要进行编排，先加载dubbo实例，再加载MVC容器。另外一种方案是设计一个组件层，将dubbo和业务层在组件层进行组装，提供Controller层使用。

5.2.Dubbo超时和重连机制

Dubbo默认有超时和重连机制，在异常场景下如果不注意会引起异常或者幂等性的异常情况。超时机制是在指定时间内Provider没有返回，则认定本次调用失败。默认重连2次，超时1000ms。另外Dubbo可以以服务，接口，方法3个纬度配置参数，值得注意的是配置以消费者端为准，如果消费者端未配置则以提供者端为准。

5.3.Dubbo序列化协议

Dubbo适合高并发量小数据传输的RPC场景，默认dubbo协议交互，但是出于性能考虑很多公司会采用kryo序列化方式来提升性能，kryo确实性能要高出很多，但是有一个弊端，就是序列化对象修改后不能向上向下兼容，换句话来说就是DTO对象增加了一个字段，服务提供者和消费者两端需要同时升级版本，否则接口调用就会报错。这里可以自己扩展Dubbo预留的Serialization接口在kryo的基础上完善序列化协议。