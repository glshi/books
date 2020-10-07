## System Architecture

分布式系统需要解决：分配和管理在集群的计算资源、处理配合、持久和可访问的数据存储、失败恢复。Fink专注分布式流处理。

**Components of a Flink Setup**

- JobManager ：接受application，包含StreamGraph（DAG）、JobGraph（logical dataflow graph，已经进过优化，如task chain）和JAR，将JobGraph转化为ExecutionGraph（physical dataflow graph，并行化），包含可以并发执行的tasks。其他工作类似Spark driver，如向RM申请资源、schedule tasks、保存作业的元数据，如checkpoints。如今JM可分为JobMaster和ResourceManager（和下面的不同），分别负责任务和资源，在Session模式下启动多个job就会有多个JobMaster。
- ResourceManager：一般是Yarn，当TM有空闲的slot就会告诉JM，没有足够的slot也会启动新的TM。kill掉长时间空闲的TM。
- TaskManager类似Spark的executor，会跑多个线程的task、数据缓存与交换。
- Dispatcher（Application Master）提供REST接口来接收client的application提交，它负责启动JM和提交application，同时运行Web UI。

> task是最基本的调度单位，由一个线程执行，里面包含一个或多个operator。多个operators就成为operation chain，需要上下游并发度一致，且传递模式（之前的Data exchange strategies）是forward。
>
> slot是TM的资源子集。结合下面Task Execution的图，一个slot并不代表一个线程，它里面并不一定只放一个task。多个task在一个slot就涉及slot sharing group。一个jobGraph的任务需要多少slot，取决于最大的并发度，这样的话，并发1和并发2就不会放到一个slot中。Co-Location Group是在此基础上，数据的forward形式，即一个slot中，如果它处理的是key1的数据，那么接下来的task也是处理key1的数据，此时就达到Co-Location Group。
>
> 尽管有slot sharing group，但一个group里串联起来的task各自所需资源的大小并不好确定。阿里日常用得最多的还是一个task一个slot的方式。

![img](https://img2018.cnblogs.com/blog/1523511/201902/1523511-20190227123612590-435088.png)

Session模式（上图）：预先启动好AM和TM，每提交一个job就启动一个Job Manager并向Flink的RM申请资源，不够的话，Flink的RM向YARN的RM申请资源。适合规模小，运行时间短的作业。`./bin/flink run ./path/to/job.jar`

Job模式：每一个job都重新启动一个Flink集群，完成后结束Flink，且只有一个Job Manager。资源按需申请，适合大作业。`./bin/flink run -m yarn-cluster ./path/to/job.jar`

下面是简单例子，详细看[官网](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/deployment/yarn_setup.html)。

```bash
# 启动yarn-session，4个TM，每个有4GB堆内存，4个slot
cd flink-1.7.0/
./bin/yarn-session.sh -n 4 -jm 1024m -tm 4096m -s 4
# 启动作业
./bin/flink run -m yarn-cluster -yn 4 -yjm 1024m -ytm 4096m ./examples/batch/WordCount.jar
```

> 细节取决于具体环境，如不同的RM

**Application Deployment**

Framework模式：Flink作业为JAR，并被提交到Dispatcher or JM or YARN。

Library模式：Flink作业为application-specific container image，如Docker image，适合微服务。

**Task Execution**

作业调度：在流计算中预先启动好节点，而在批计算中，每当某个阶段完成计算才启动下一个节点。

资源管理：slot作为基本单位，有大小和位置属性。JM有SlotPool，向Flink RM申请Slot，FlinkRM发现自己的SlotManager中没有足够的Slot，就会向集群RM申请。后者返回可用TM的ip，让FlinkRM去启动，TM启动后向FlinkRM注册。后者向TM请求Slot，TM向JM提供相应Slot。JM用完后释放Slot，TM会把释放的Slot报告给FlinkRM。在Blink版本中，job模式会根据申请slot的大小分配相应的TM，而session模式则预先设置好TM大小，每有slot申请就从TM中划分相应的资源。

任务可以是相同operator (data parallelism)，不同 operator (task parallelism)，甚至不同application (job parallelism)。TM提供一定数量的slots来控制并行的任务数。

![img](https://img2018.cnblogs.com/blog/1523511/201902/1523511-20190223233247133-376816544.jpg)

上图A和C是source function，E是sink function，小数字表示并行度。

![img](https://img2018.cnblogs.com/blog/1523511/201812/1523511-20181215133305533-593631500.png)

一个TM是一个JVM进程，它通过多线程完成任务。线程的隔离不太好，一个线程失败有可能导致整个TM失败。

**Highly-Available Setup**

从失败中恢复需要重启失败进程、作业和恢复它的state。

当一个TM挂掉而RM又无法找到空闲的资源时，就只能暂时降低并行度，直到有空闲的资源重启TM。

当JM挂掉就靠ZK来重新选举，和找到JM存储到远程storage的元数据、JobGraph。重启JM并从最后一个完成的checkpoint开始。

> JM在执行期间会得到每个task checkpoints的state存储路径（task将state写到远程storage）并写到远程storage，同时在ZK的存储路径留下pointer指明到哪里找上面的存储路径。

![img](https://img2018.cnblogs.com/blog/1523511/201812/1523511-20181215133113984-278081201.jpg)

**背压**

数据涌入的速度大于处理速度。在source operator中，可通过Kafka解决。在任务间的operator有如下机制应对：

Local exchange：task1和2在同一个工作节点，那么buffer pool可以直接交给下一个任务，但下一个任务task2消费buffer pool中的信息速度减慢时，当前任务task1填充buffer pool的速度也会减慢。

Remote exchange：TM保证每个task至少有一个incoming和一个outgoing缓冲区。当下游receiver的处理速度低于上有的sender的发送速度，receiver的incoming缓冲区就会开始积累数据（需要空闲的buffer来放从TCP连接中接收的数据），当挤满后就不再接收数据。上游sender利用netty水位机制，当网络中的缓冲数据过多时暂停发送。

## Data Transfer in Flink

TM负责数据在tasks间的转移，转移之前会存储到buffer（这又变回micro-batches）。每个TM有32KB的网络buffer用于接收和发送数据。如果sender和receiver在不同进程，那么会通过操作系统的网络栈来通信。每对TM保持permanent TCP连接来交换数据。每个sender任务能够给所有receiving任务发送数据，反之，所有receiver任务能够接收所有sender任务的数据。TM保证每个任务都至少有一个incoming和outgoing的buffer，并增加额外的缓冲区分配约束来避免死锁。

![img](https://img2018.cnblogs.com/blog/1523511/201812/1523511-20181215133055604-2059310965.jpg)

如果sender和receiver任务在同一个TM进程，sender会序列化结果数据到buffer，如果满了就放到队列。receiver任务通过队列得到数据并进行反序列化。这样的好处是解耦任务并允许在任务中使用可变对象，从而减少了对象实例化和垃圾收集。一旦数据被序列化，就能安全地修改。而缺点是计算消耗大，在一些条件下能够把task穿起来，避免序列化。(C10)

**Flow Control with Back Pressure**

receiver放到缓冲区的数据变为队列，sender将要发送的数据变为队列，最后sender减慢发送速度。

## Event Time Processing

event time处理的数据必须有时间戳（Long unix timestamp）并定义了watermarks。watermark是一种特殊的records holding a timestamp long value。它必须是递增的（防止倒退），有一个timestamp t（下图的5），暗示所有接下来的数据都会大于这个值。后来的，小于这个值，就被视为迟来数据，Flink有其他机制处理。

![img](https://img2018.cnblogs.com/blog/1523511/201812/1523511-20181215133041123-658967497.jpg)

**Watermarks and Event Time**

WM在Flink是一种特殊的record，它会被operator tasks接收和释放。

tasks有时间服务来维持timers（timers注册到时间服务上），在time-window task中，timers分别记录了各个window的结束时间。当任务获得一个watermark时，task会根据这个watermark的timestamp更新内部的event-time clock。任务内部的时间服务确定所有timers时间是否小于watermark的timestamp，如果大于则触发call-back算子来释放记录并返回结果。最后task还会将更新的event-time clock的WM进行广播。（结合下图理解）

只有ProcessFunction可以读取和修改timestamp或者watermark（The `ProcessFunction` can read the timestamp of a currently processed record, request the current event-time of the operator, and register timers）。下面是PF的行为。

![img](https://img2018.cnblogs.com/blog/1523511/201812/1523511-20181215133029233-1547094659.jpg)

当收到WM大于所有目前拥有的WM，就会把event-time clock更新为所有WM中最小的那个，并广播这个最小的WM。即便是多个streams输入，机制也一样，只是增加Paritition WM数量。这种机制要求获得的WM必须是累加的，而且task必须有新的WM接收，否则clock就不会更新，task的timers就不会被触发。另外，当多个streams输入时，timers会被WM比较离散的stream主导，从而使更密集的stream的state不断积累。

**Timestamp Assignment and Watermark Generation**

当streaming application消化流时产生。Flink有三种方式产生：

- SourceFunction：产生的record带有timestamp，一些特殊时点产生WM。如果SF暂时不再发送WM，则会被认为是idle。Flink会从接下来的watermark operators中排除由这个SF生产的分区（上图有4个分区），从而解决timer不触发的问题。
- `AssignerWithPeriodicWatermarks` 提取每条记录的timestamp，并周期性的查询当前WM，即上图的Partition WM。
- `AssignerWithPunctuatedWatermarks` 可以从每条数据提取WM。

> 上面两个User-defined timestamp assignment functions通常用在source operator附近，因为stream一经处理就很难把握record的时间顺序了。所以UDF可以修改timestamp和WM，但在数据处理时使用不是一个好主意。

## State Management

由任务维护并用于计算函数结果的所有数据都属于任务的state。其实state可以理解为task业务逻辑的本地或实例变量。

![img](https://img2018.cnblogs.com/blog/1523511/201812/1523511-20181215133015548-176401136.jpg)

在Flink，state总是和特定的operator关联。operator需要注册它的state，而state有两种类型：

- Operator State：由同一并行任务处理的所有记录都可以访问相同的state，而其他的task或operator不能访问，即一个task专属一个state。这种state有三种primitives
  - *List State* represents state as a list of entries.
  - *Union List State*同上，但在任务失败和作业从savepoint重启的行为不一样
  - *Broadcast State*（v1.5） 同样一个task专属一个state，但state都是一样的（需要自己注意保持一致，对state更新时，实际上只对当前task的state进行更新。只有所有task的更新一样时，即输入数据一样（一开始广播所以一样，但数据的顺序可能不一样），对数据的处理一样，才能保证state一样）。这种state只能存储在内存，所以没有RockDB backend。
- Keyed State：相同key的record共享一个state。
  - *Value State*：每个key一个值，这个值可以是复杂的数据结构.
  - *List State*：每个key一个list
  - *Map State*：每个key一个map

上面两种state的存在方式有两种：raw和managed，一般都是用后者，也推荐用后者（更好的内存管理、不需造轮子）。

**State Backends**

state backend决定了state如何被存储、访问和维持。它的主要职责是本地state管理和checkpoint state到远程。在管理方面，可选择将state存储到内存还是磁盘。checkpoint方面在C8详细介绍。

MemoryStateBackend, FsStateBackend, RocksDBStateBackend适合越来越大的state。都支持异步checkpoint，其中RocksDB还支持incremental的checkpoint。

- 注意：As RocksDB’s JNI bridge API is based on byte[], the maximum supported size per key and per value is 2^31 bytes each. IMPORTANT: states that use merge operations in RocksDB (e.g. ListState) can silently accumulate value sizes > 2^31 bytes and will then fail on their next retrieval. This is currently a limitation of RocksDB JNI.

**Scaling Stateful Operators**

Flink会根据input rate调整并发度。对于stateful operators有以下4种方式：

- keyed state：根据key group来调整，即分为同一组的key-value会被分到相同的task
- list state：所有list entries会被收集并重新均匀分布，当增加并发度时，要新建list
- union list state：增加并发时，广播整个list，所以rescaling后，所有task都有所有的list state。

![img](https://img2018.cnblogs.com/blog/1523511/201812/1523511-20181215132935340-291222631.jpg)

- broadcast state

![img](https://img2018.cnblogs.com/blog/1523511/201812/1523511-20181215132923387-300658437.jpg)

## Checkpoints, Savepoints, and State Recovery

**Flink’s Lightweight Checkpointing Algorithm**

在分布式开照算法Chandy-Lamport的基础上实现。有一种特殊的record叫checkpoint barrier（由JM产生），它带有checkpoint ID来把流进行划分。在CB前面的records会被包含到checkpoint，之后的会被包含在之后的checkpoint。

当source task收到这种信息，就会停止发送recordes，触发state backend对本地state的checkpoint，并广播checkpoint ID到所有下游task。当checkpoint完成时，state backend唤醒source task，后者向JM确定相应的checkpoint ID已经完成任务。

![img](https://img2018.cnblogs.com/blog/1523511/201812/1523511-20181215132839621-838552599.jpg)

当下游获得其中一个CB时，就会暂停处理这个CB对应的source的数据（完成checkpoint后发送的数据），并将这些数据存到缓冲区，直到其他相同ID的CB都到齐，就会把state（下图的12、8）进行checkpoint，并广播CB到下游。直到所有CB被广播到下游，才开始处理排队在缓冲区的数据。当然，其他没有发送CB的source的数据会继续处理。

![img](https://img2018.cnblogs.com/blog/1523511/201812/1523511-20181215132853730-1959584711.jpg)

![img](https://img2018.cnblogs.com/blog/1523511/201812/1523511-20181215132903918-303674089.jpg)

最后，当所有sink会向JM发送BC确定checkpoint已完成。

这种机制还有两个优化：

- 当operator的state很大时，复制整个state并发送到远程storage会很费时。而RocksDB state backend支持asynchronous and incremental的checkpoints。当触发checkpoint时，backend会快照所有本地state的修改（直至上一次checkpoint），然后马上让task继续执行。后台线程异步发送快照到远程storage。
- 在等待其余CB时，已经完成checkpoint的source数据需要排队。但如果使用at-least-once就不需要等了。但当所有CB到齐再checkpoint，存储的state就已经包含了下一次checkpoint才记录的数据。（如果是取最值这种state就无所谓）

**Recovery from Consistent Checkpoints**

![img](https://img2018.cnblogs.com/blog/1523511/201812/1523511-20181215132720518-250742419.jpg)

上图队列中的7和6之所以能恢复，取决于数据源是否resettable，如Kafka，不会因为发送信息就把信息删除。这才能实现处理过程的exactly-once state consistency（严格来讲，数据还是被重复处理，但是在读档后重复的）。但是下游系统有可能接收到多个结果。这方面，Flink提供sink算子实现output的exactly-once，例如给checkpoint提交records释放记录。另一个方法是idempotent updates，详细看C7。

**Savepoints**

checkpoints加上一些额外的元数据，功能也是在checkpoint的基础上丰富。不同于checkpoints，savepoint不会被Flink自动创造（由用户或者外部scheduler触发创造）和销毁。savepoint可以重启不同但兼容的作业，从而：

- 修复bugs进而修复错误的结果，也可用于A/B test或者what-if场景。
- 调整并发度
- 迁移作业到其他集群、新版Flink

也可以用于暂停作业，通过savepoint查看作业情况。





### 前言

Apache Flink（下简称Flink）项目是大数据处理领域最近冉冉升起的一颗新星，其不同于其他大数据项目的诸多特性吸引了越来越多人的关注。本文将深入分析Flink的一些关键技术与特性，希望能够帮助读者对Flink有更加深入的了解，对其他大数据系统开发者也能有所裨益。本文假设读者已对MapReduce、Spark及Storm等大数据处理框架有所了解，同时熟悉流处理与批处理的基本概念。

文章转载自：[深入理解Flink核心技术](https://links.jianshu.com/go?to=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F20585530)

### 一.Flink简介

Flink核心是一个流式的数据流执行引擎，其针对数据流的分布式计算提供了数据分布、数据通信以及容错机制等功能。基于流执行引擎，Flink提供了诸多更高抽象层的API以便用户编写分布式任务：

1. DataSet API， 对静态数据进行批处理操作，将静态数据抽象成分布式的数据集，用户可以方便地使用Flink提供的各种操作符对分布式数据集进行处理，支持Java、Scala和Python。
2. DataStream API，对数据流进行流处理操作，将流式的数据抽象成分布式的数据流，用户可以方便地对分布式数据流进行各种操作，支持Java和Scala。
3. Table API，对结构化数据进行查询操作，将结构化数据抽象成关系表，并通过类SQL的DSL对关系表进行各种查询操作，支持Java和Scala。

此外，Flink还针对特定的应用领域提供了领域库，例如：

- Flink ML，Flink的机器学习库，提供了机器学习Pipelines API并实现了多种机器学习算法。
- Gelly，Flink的图计算库，提供了图计算的相关API及多种图计算算法实现。

 

![img](https://upload-images.jianshu.io/upload_images/14534869-ae0077f260cb6f7d.png)

Flink的技术栈

此外，Flink也可以方便地和Hadoop生态圈中其他项目集成，例如Flink可以读取存储在HDFS或HBase中的静态数据，以Kafka作为流式的数据源，直接重用MapReduce或Storm代码，或是通过YARN申请集群资源等。

### 二.统一的批处理与流处理系统

在大数据处理领域，批处理任务与流处理任务一般被认为是两种不同的任务，一个大数据项目一般会被设计为只能处理其中一种任务，例如Apache Storm、Apache Smaza只支持流处理任务，而Aapche MapReduce、Apache Tez、Apache Spark只支持批处理任务。Spark Streaming是Apache Spark之上支持流处理任务的子系统，看似一个特例，实则不然——Spark Streaming采用了一种micro-batch的架构，即把输入的数据流切分成细粒度的batch，并为每一个batch数据提交一个批处理的Spark任务，所以Spark Streaming本质上还是基于Spark批处理系统对流式数据进行处理，和Apache Storm、Apache Smaza等完全流式的数据处理方式完全不同。通过其灵活的执行引擎，Flink能够同时支持批处理任务与流处理任务。

在执行引擎这一层，流处理系统与批处理系统最大不同在于节点间的数据传输方式。

**对于一个流处理系统，其节点间数据传输的标准模型是：**
当一条数据被处理完成后，序列化到缓存中，然后立刻通过网络传输到下一个节点，由下一个节点继续处理。

**而对于一个批处理系统，其节点间数据传输的标准模型是：**
当一条数据被处理完成后，序列化到缓存中，并不会立刻通过网络传输到下一个节点，当缓存写满，就持久化到本地硬盘上，当所有数据都被处理完成后，才开始将处理后的数据通过网络传输到下一个节点。

这两种数据传输模式是两个极端，对应的是流处理系统对低延迟的要求和批处理系统对高吞吐量的要求。

Flink的执行引擎采用了一种十分灵活的方式，同时支持了这两种数据传输模型。Flink以固定的缓存块为单位进行网络数据传输，用户可以通过缓存块超时值指定缓存块的传输时机。如果缓存块的超时值为0，则Flink的数据传输方式类似上文所提到流处理系统的标准模型，此时系统可以获得最低的处理延迟。如果缓存块的超时值为无限大，则Flink的数据传输方式类似上文所提到批处理系统的标准模型，此时系统可以获得最高的吞吐量。同时缓存块的超时值也可以设置为0到无限大之间的任意值。缓存块的超时阈值越小，则Flink流处理执行引擎的数据处理延迟越低，但吞吐量也会降低，反之亦然。通过调整缓存块的超时阈值，用户可根据需求灵活地权衡系统延迟和吞吐量。

 

![img](https://upload-images.jianshu.io/upload_images/14534869-fa7f089b35e1c243.png)

Flink执行引擎数据传输模式

在统一的流式执行引擎基础上，Flink同时支持了流计算和批处理，并对性能（延迟、吞吐量等）有所保障。相对于其他原生的流处理与批处理系统，并没有因为统一执行引擎而受到影响从而大幅度减轻了用户安装、部署、监控、维护等成本。

### 三.Flink流处理的容错机制

对于一个分布式系统来说，单个进程或是节点崩溃导致整个Job失败是经常发生的事情，在异常发生时不会丢失用户数据并能自动恢复才是分布式系统必须支持的特性之一。本节主要介绍Flink流处理系统任务级别的容错机制。

批处理系统比较容易实现容错机制，由于文件可以重复访问，当某个任务失败后，重启该任务即可。但是到了流处理系统，由于数据源是无限的数据流，从而导致一个流处理任务执行几个月的情况，将所有数据缓存或是持久化，留待以后重复访问基本上是不可行的。Flink基于分布式快照与可部分重发的数据源实现了容错。用户可自定义对整个Job进行快照的时间间隔，当任务失败时，Flink会将整个Job恢复到最近一次快照，并从数据源重发快照之后的数据。**Flink的分布式快照实现借鉴了Chandy和Lamport在1985年发表的一篇关于分布式快照的论文，其实现的主要思想如下：**
**按照用户自定义的分布式快照间隔时间，Flink会定时在所有数据源中插入一种特殊的快照标记消息，这些快照标记消息和其他消息一样在DAG中流动，但是不会被用户定义的业务逻辑所处理，每一个快照标记消息都将其所在的数据流分成两部分：本次快照数据和下次快照数据。**

 

![img](https://upload-images.jianshu.io/upload_images/14534869-5a2e0b3fd1aa2230.png)

Flink包含快照标记消息的消息流

快照标记消息沿着DAG流经各个操作符，当操作符处理到快照标记消息时，会对自己的状态进行快照，并存储起来。当一个操作符有多个输入的时候，Flink会将先抵达的快照标记消息及其之后的消息缓存起来，当所有的输入中对应该次快照的快照标记消息全部抵达后，操作符对自己的状态快照并存储，之后处理所有快照标记消息之后的已缓存消息。操作符对自己的状态快照并存储可以是异步与增量的操作，并不需要阻塞消息的处理。分布式快照的流程如图所示：

 

![img](https://upload-images.jianshu.io/upload_images/14534869-21647fa5067e402d.png)

Flink分布式快照流程图

当所有的Data Sink（终点操作符）都收到快照标记信息并对自己的状态快照和存储后，整个分布式快照就完成了，同时通知数据源释放该快照标记消息之前的所有消息。若之后发生节点崩溃等异常情况时，只需要恢复之前存储的分布式快照状态，并从数据源重发该快照以后的消息就可以了。

**Exactly-Once**是流处理系统需要支持的一个非常重要的特性，它保证每一条消息只被流处理系统处理一次，许多流处理任务的业务逻辑都依赖于Exactly-Once特性。相对于At-Least-Once或是At-Most-Once, Exactly-Once特性对流处理系统的要求更为严格，实现也更加困难。Flink基于分布式快照实现了Exactly-Once特性。

相对于其他流处理系统的容错方案，**Flink基于分布式快照的方案在功能和性能方面都具有很多优点，包括：**

- **低延迟**。由于操作符状态的存储可以异步，所以进行快照的过程基本上不会阻塞消息的处理，因此不会对消息延迟产生负面影响。
- **高吞吐量**。当操作符状态较少时，对吞吐量基本没有影响。当操作符状态较多时，相对于其他的容错机制，分布式快照的时间间隔是用户自定义的，所以用户可以权衡错误恢复时间和吞吐量要求来调整分布式快照的时间间隔。
- **与业务逻辑的隔离**。Flink的分布式快照机制与用户的业务逻辑是完全隔离的，用户的业务逻辑不会依赖或是对分布式快照产生任何影响。
- **错误恢复代价**。分布式快照的时间间隔越短，错误恢复的时间越少，与吞吐量负相关。

### 四.Flink流处理的时间窗口

对于流处理系统来说，流入的消息不存在上限，所以对于聚合或是连接等操作，流处理系统需要对流入的消息进行分段，然后基于每一段数据进行聚合或是连接。消息的分段即称为窗口，流处理系统支持的窗口有很多类型，最常见的就是时间窗口，基于时间间隔对消息进行分段处理。本节主要介绍Flink流处理系统支持的各种时间窗口。

对于目前大部分流处理系统来说，时间窗口一般是根据Task所在节点的本地时钟进行切分，这种方式实现起来比较容易，不会产生阻塞。但是可能无法满足某些应用需求，比如：

消息本身带有时间戳，用户希望按照消息本身的时间特性进行分段处理。

由于不同节点的时钟可能不同，以及消息在流经各个节点的延迟不同，在某个节点属于同一个时间窗口处理的消息，流到下一个节点时可能被切分到不同的时间窗口中，从而产生不符合预期的结果。

**Flink支持3种类型的时间窗口，分别适用于用户对于时间窗口不同类型的要求：**

1. **Operator Time**。根据Task所在节点的本地时钟来切分的时间窗口。
2. **Event Time**。消息自带时间戳，根据消息的时间戳进行处理，确保时间戳在同一个时间窗口的所有消息一定会被正确处理。由于消息可能乱序流入Task，所以Task需要缓存当前时间窗口消息处理的状态，直到确认属于该时间窗口的所有消息都被处理，才可以释放，如果乱序的消息延迟很高会影响分布式系统的吞吐量和延迟。
3. **Ingress Time**。有时消息本身并不带有时间戳信息，但用户依然希望按照消息而不是节点时钟划分时间窗口，例如避免上面提到的第二个问题，此时可以在消息源流入Flink流处理系统时自动生成增量的时间戳赋予消息，之后处理的流程与Event Time相同。Ingress Time可以看成是Event Time的一个特例，由于其在消息源处时间戳一定是有序的，所以在流处理系统中，相对于Event Time，其乱序的消息延迟不会很高，因此对Flink分布式系统的吞吐量和延迟的影响也会更小。

### 五.Event Time时间窗口的实现

Flink借鉴了Google的MillWheel项目，通过WaterMark来支持基于Event Time的时间窗口。

当操作符通过基于Event Time的时间窗口来处理数据时，它必须在确定所有属于该时间窗口的消息全部流入此操作符后才能开始数据处理。但是由于消息可能是乱序的，所以操作符无法直接确认何时所有属于该时间窗口的消息全部流入此操作符。WaterMark包含一个时间戳，Flink使用WaterMark标记所有小于该时间戳的消息都已流入，Flink的数据源在确认所有小于某个时间戳的消息都已输出到Flink流处理系统后，会生成一个包含该时间戳的WaterMark，插入到消息流中输出到Flink流处理系统中，Flink操作符按照时间窗口缓存所有流入的消息，当操作符处理到WaterMark时，它对所有小于该WaterMark时间戳的时间窗口数据进行处理并发送到下一个操作符节点，然后也将WaterMark发送到下一个操作符节点。

为了保证能够处理所有属于某个时间窗口的消息，操作符必须等到大于这个时间窗口的WaterMark之后才能开始对该时间窗口的消息进行处理，相对于基于Operator Time的时间窗口，Flink需要占用更多内存，且会直接影响消息处理的延迟时间。对此，一个可能的优化措施是，对于聚合类的操作符，可以提前对部分消息进行聚合操作，当有属于该时间窗口的新消息流入时，基于之前的部分聚合结果继续计算，这样的话，只需缓存中间计算结果即可，无需缓存该时间窗口的所有消息。

对于基于Event Time时间窗口的操作符来说，流入WaterMark的时间戳与当前节点的时钟一致是最简单理想的状况，但是在实际环境中是不可能的，由于消息的乱序以及前面节点处理效率的不同，总是会有某些消息流入时间大于其本身的时间戳，真实WaterMark时间戳与理想情况下WaterMark时间戳的差别称为Time Skew，如下图所示：

 

![img](https://upload-images.jianshu.io/upload_images/14534869-a6c14281658326e0.png)

WaterMark的Time Skew图

Time Skew决定了该WaterMark与上一个WaterMark之间的时间窗口所有数据需要缓存的时间，Time Skew时间越长，该时间窗口数据的延迟越长，占用内存的时间也越长，同时会对流处理系统的吞吐量产生负面影响。

### 六.基于时间戳的排序

在流处理系统中，由于流入的消息是无限的，所以对消息进行排序基本上被认为是不可行的。但是在Flink流处理系统中，基于WaterMark，Flink实现了基于时间戳的全局排序。**排序的实现思路如下：排序操作符缓存所有流入的消息，当其接收到WaterMark时，对时间戳小于该WaterMark的消息进行排序，并发送到下一个节点，在此排序操作符中释放所有时间戳小于该WaterMark的消息，继续缓存流入的消息，等待下一个WaterMark触发下一次排序。**

由于WaterMark保证了在其之后不会出现时间戳比它小的消息，所以可以保证排序的正确性。需要注意的是，如果排序操作符有多个节点，只能保证每个节点的流出消息是有序的，节点之间的消息不能保证有序，要实现全局有序，则只能有一个排序操作符节点。

通过支持基于Event Time的消息处理，Flink扩展了其流处理系统的应用范围，使得更多的流处理任务可以通过Flink来执行。

### 七.定制的内存管理

Flink项目基于Java及Scala等JVM语言，JVM本身作为一个各种类型应用的执行平台，其对Java对象的管理也是基于通用的处理策略，其垃圾回收器通过估算Java对象的生命周期对Java对象进行有效率的管理。

针对不同类型的应用，用户可能需要针对该类型应用的特点，配置针对性的JVM参数更有效率的管理Java对象，从而提高性能。这种JVM调优的黑魔法需要用户对应用本身及JVM的各参数有深入了解，极大地提高了分布式计算平台的调优门槛。Flink框架本身了解计算逻辑每个步骤的数据传输，相比于JVM垃圾回收器，其了解更多的Java对象生命周期，从而为更有效率地管理Java对象提供了可能。

#### JVM存在的问题

##### 1.Java对象开销

相对于c/c++等更加接近底层的语言，Java对象的存储密度相对偏低，例如[1]，“abcd”这样简单的字符串在UTF-8编码中需要4个字节存储，但采用了UTF-16编码存储字符串的Java则需要8个字节，同时Java对象还有header等其他额外信息，一个4字节字符串对象在Java中需要48字节的空间来存储。对于大部分的大数据应用，内存都是稀缺资源，更有效率地内存存储，意味着CPU数据访问吞吐量更高，以及更少磁盘落地的存在。

##### 2.对象存储结构引发的cache miss

为了缓解CPU处理速度与内存访问速度的差距，现代CPU数据访问一般都会有多级缓存。当从内存加载数据到缓存时，一般是以cache line为单位加载数据，所以当CPU访问的数据如果是在内存中连续存储的话，访问的效率会非常高。如果CPU要访问的数据不在当前缓存所有的cache line中，则需要从内存中加载对应的数据，这被称为一次cache miss。当cache miss非常高的时候，CPU大部分的时间都在等待数据加载，而不是真正的处理数据。Java对象并不是连续的存储在内存上，同时很多的Java数据结构的数据聚集性也不好。

##### 3.大数据的垃圾回收

Java的垃圾回收机制一直让Java开发者又爱又恨，一方面它免去了开发者自己回收资源的步骤，提高了开发效率，减少了内存泄漏的可能，另一方面垃圾回收也是Java应用的不定时炸弹，有时秒级甚至是分钟级的垃圾回收极大影响了Java应用的性能和可用性。在时下数据中心，大容量内存得到了广泛的应用，甚至出现了单台机器配置TB内存的情况，同时，大数据分析通常会遍历整个源数据集，对数据进行转换、清洗、处理等步骤。在这个过程中，会产生海量的Java对象，JVM的垃圾回收执行效率对性能有很大影响。通过JVM参数调优提高垃圾回收效率需要用户对应用和分布式计算框架以及JVM的各参数有深入了解，而且有时候这也远远不够。

##### 4.OOM问题

OutOfMemoryError是分布式计算框架经常会遇到的问题，当JVM中所有对象大小超过分配给JVM的内存大小时，就会出现OutOfMemoryError错误，JVM崩溃，分布式框架的健壮性和性能都会受到影响。通过JVM管理内存，同时试图解决OOM问题的应用，通常都需要检查Java对象的大小，并在某些存储Java对象特别多的数据结构中设置阈值进行控制。但是JVM并没有提供官方检查Java对象大小的工具，第三方的工具类库可能无法准确通用地确定Java对象大小[6]。侵入式的阈值检查也会为分布式计算框架的实现增加很多额外与业务逻辑无关的代码。

#### Flink的处理策略

为了解决以上提到的问题，高性能分布式计算框架通常需要以下技术：

- **定制的序列化工具**。显式内存管理的前提步骤就是序列化，将Java对象序列化成二进制数据存储在内存上（on heap或是off-heap）。通用的序列化框架，如Java默认使用java.io.Serializable将Java对象及其成员变量的所有元信息作为其序列化数据的一部分，序列化后的数据包含了所有反序列化所需的信息。这在某些场景中十分必要，但是对于Flink这样的分布式计算框架来说，这些元数据信息可能是冗余数据。定制的序列化框架，如Hadoop的org.apache.hadoop.io.Writable需要用户实现该接口，并自定义类的序列化和反序列化方法。这种方式效率最高，但需要用户额外的工作，不够友好。
- **显式的内存管理**。一般通用的做法是批量申请和释放内存，每个JVM实例有一个统一的内存管理器，所有内存的申请和释放都通过该内存管理器进行。这可以避免常见的内存碎片问题，同时由于数据以二进制的方式存储，可以大大减轻垃圾回收压力。

缓存友好的数据结构和算法。对于计算密集的数据结构和算法，直接操作序列化后的二进制数据，而不是将对象反序列化后再进行操作。同时，只将操作相关的数据连续存储，可以最大化的利用L1/L2/L3缓存，减少Cache miss的概率，提升CPU计算的吞吐量。以排序为例，由于排序的主要操作是对Key进行对比，如果将所有排序数据的Key与Value分开并对Key连续存储，那么访问Key时的Cache命中率会大大提高。

### 八.定制的序列化工具

分布式计算框架可以使用定制序列化工具的前提是要待处理数据流通常是同一类型，由于数据集对象的类型固定，从而可以只保存一份对象Schema信息，节省大量的存储空间。同时，对于固定大小的类型，也可通过固定的偏移位置存取。在需要访问某个对象成员变量时，通过定制的序列化工具，并不需要反序列化整个Java对象，而是直接通过偏移量，从而只需要反序列化特定的对象成员变量。如果对象的成员变量较多时，能够大大减少Java对象的创建开销，以及内存数据的拷贝大小。Flink数据集都支持任意Java或是Scala类型，通过自动生成定制序列化工具，既保证了API接口对用户友好（不用像Hadoop那样数据类型需要继承实现org.apache.hadoop.io.Writable接口），也达到了和Hadoop类似的序列化效率。

Flink对数据集的类型信息进行分析，然后自动生成定制的序列化工具类。Flink支持任意的Java或是Scala类型，通过Java Reflection框架分析基于Java的Flink程序UDF（User Define Function）的返回类型的类型信息，通过Scala Compiler分析基于Scala的Flink程序UDF的返回类型的类型信息。类型信息由TypeInformation类表示，这个类有诸多具体实现类，例如：

1. **BasicTypeInfo：**任意Java基本类型（装包或未装包）和String类型。
2. **BasicArrayTypeInfo：**任意Java基本类型数组（装包或未装包）和String数组。
3. **WritableTypeInfo：**任意Hadoop的Writable接口的实现类。
4. **TupleTypeInfo：**任意的Flink tuple类型(支持Tuple1 to Tuple25)。 Flink tuples是固定长度固定类型的Java Tuple实现。
5. **CaseClassTypeInfo：**任意的 Scala CaseClass(包括 Scala tuples)。
6. **PojoTypeInfo：**任意的POJO (Java or Scala)，例如Java对象的所有成员变量，要么是public修饰符定义，要么有getter/setter方法。
7. **GenericTypeInfo：**任意无法匹配之前几种类型的类。

前6种类型数据集几乎覆盖了绝大部分的Flink程序，**针对前6种类型数据集，Flink皆可以自动生成对应的TypeSerializer定制序列化工具，非常有效率地对数据集进行序列化和反序列化**。**对于第7种类型，Flink使用Kryo进行序列化和反序列化。**此外，**对于可被用作Key的类型，Flink还同时自动生成TypeComparator，用来辅助直接对序列化后的二进制数据直接进行compare、hash等操作**。对于Tuple、CaseClass、Pojo等组合类型，Flink自动生成的TypeSerializer、TypeComparator同样是组合的，并把其成员的序列化/反序列化代理给其成员对应的TypeSerializer、TypeComparator，如图所示：

 

![img](https://upload-images.jianshu.io/upload_images/14534869-5ae9c7f1f348a621.jpg)

Flink组合类型序列化

此外如有需要，用户可通过集成TypeInformation接口定制实现自己的序列化工具。

### 九.显式的内存管理

垃圾回收是JVM内存管理回避不了的问题，JDK8的G1算法改善了JVM垃圾回收的效率和可用范围，但对于大数据处理实际环境还远远不够。这也和现在分布式框架的发展趋势有所冲突，越来越多的分布式计算框架希望尽可能多地将待处理数据集放入内存，而对于JVM垃圾回收来说，内存中Java对象越少、存活时间越短，其效率越高。通过JVM进行内存管理的话，OutOfMemoryError也是一个很难解决的问题。同时，在JVM内存管理中，Java对象有潜在的碎片化存储问题（Java对象所有信息可能在内存中连续存储），也有可能在所有Java对象大小没有超过JVM分配内存时，出现OutOfMemoryError问题。Flink将内存分为3个部分，每个部分都有不同用途：

- **Network buffers**： 一些以32KB Byte数组为单位的buffer，主要被网络模块用于数据的网络传输。
- **Memory Manager pool**：大量以32KB Byte数组为单位的内存池，所有的运行时算法（例如Sort/Shuffle/Join）都从这个内存池申请内存，并将序列化后的数据存储其中，结束后释放回内存池。
- **Remaining (Free) Heap：**主要留给UDF中用户自己创建的Java对象，由JVM管理。

**Network buffers**在Flink中主要基于Netty的网络传输，无需多讲。
**Remaining Heap**用于UDF中用户自己创建的Java对象，在UDF中，用户通常是流式的处理数据，并不需要很多内存，同时Flink也不鼓励用户在UDF中缓存很多数据，因为这会引起前面提到的诸多问题。
**Memory Manager pool**（以后以内存池代指）通常会配置为最大的一块内存，接下来会详细介绍。

在Flink中，**内存池由多个MemorySegment组成，每个MemorySegment代表一块连续的内存，底层存储是byte[]，默认32KB大小**。MemorySegment提供了根据偏移量访问数据的各种方法，如get/put int、long、float、double等，MemorySegment之间数据拷贝等方法和java.nio.ByteBuffer类似。**对于Flink的数据结构，通常包括多个向内存池申请的MemeorySegment，所有要存入的对象通过TypeSerializer序列化之后，将二进制数据存储在MemorySegment中，在取出时通过TypeSerializer反序列化。数据结构通过MemorySegment提供的set/get方法访问具体的二进制数据。**Flink这种看起来比较复杂的内存管理方式带来的好处主要有：

- 二进制的数据存储大大提高了数据存储密度，节省了存储空间。
- 所有的运行时数据结构和算法只能通过内存池申请内存，保证了其使用的内存大小是固定的，不会因为运行时数据结构和算法而发生OOM。对于大部分的分布式计算框架来说，这部分由于要缓存大量数据最有可能导致OOM。
- 内存池虽然占据了大部分内存，但其中的MemorySegment容量较大（默认32KB），所以内存池中的Java对象其实很少，而且一直被内存池引用，所有在垃圾回收时很快进入持久代，大大减轻了JVM垃圾回收的压力。
- Remaining Heap的内存虽然由JVM管理，但是由于其主要用来存储用户处理的流式数据，生命周期非常短，速度很快的Minor GC就会全部回收掉，一般不会触发Full GC。

Flink当前的内存管理在最底层是基于byte[]，所以数据最终还是on-heap，最近Flink增加了off-heap的内存管理支持。**Flink off-heap的内存管理相对于on-heap的优点主要在于：**

- 启动分配了大内存(例如100G)的JVM很耗费时间，垃圾回收也很慢。如果采用off-heap，剩下的Network buffer和Remaining heap都会很小，垃圾回收也不用考虑MemorySegment中的Java对象了。
- 更有效率的IO操作。在off-heap下，将MemorySegment写到磁盘或是网络可以支持zeor-copy技术，而on-heap的话则至少需要一次内存拷贝。
- off-heap可用于错误恢复，比如JVM崩溃，在on-heap时数据也随之丢失，但在off-heap下，off-heap的数据可能还在。此外，off-heap上的数据还可以和其他程序共享。

### 十.缓存友好的计算

磁盘IO和网络IO之前一直被认为是Hadoop系统的瓶颈，但是随着Spark、Flink等新一代分布式计算框架的发展，越来越多的趋势使得CPU/Memory逐渐成为瓶颈，这些趋势包括：

- 更先进的IO硬件逐渐普及。10GB网络和SSD硬盘等已经被越来越多的数据中心使用。
- 更高效的存储格式。Parquet，ORC等列式存储被越来越多的Hadoop项目支持，其非常高效的压缩性能大大减少了落地存储的数据量。
- 更高效的执行计划。例如很多SQL系统执行计划优化器的Fliter-Push-Down优化会将过滤条件尽可能的提前，甚至提前到Parquet的数据访问层，使得在很多实际的工作负载中并不需要很多的磁盘IO。

由于CPU处理速度和内存访问速度的差距，提升CPU的处理效率的关键在于最大化的利用L1/L2/L3/Memory，减少任何不必要的Cache miss。定制的序列化工具给Flink提供了可能，通过定制的序列化工具，Flink访问的二进制数据本身，因为占用内存较小，存储密度比较大，而且还可以在设计数据结构和算法时尽量连续存储，减少内存碎片化对Cache命中率的影响，甚至更进一步，Flink可以只是将需要操作的部分数据（如排序时的Key）连续存储，而将其他部分的数据存储在其他地方，从而最大可能地提升Cache命中的概率。

### 十一.Flink排序算法

**以Flink中的排序为例，**排序通常是分布式计算框架中一个非常重的操作，Flink通过特殊设计的排序算法获得了非常好的性能，其排序算法的实现如下：

- 将待排序的数据经过序列化后存储在两个不同的MemorySegment集中。数据全部的序列化值存放于其中一个MemorySegment集中。数据序列化后的Key和指向第一个MemorySegment集中值的指针存放于第二个MemorySegment集中。
- 对第二个MemorySegment集中的Key进行排序，如需交换Key位置，只需交换对应的Key+Pointer的位置，第一个MemorySegment集中的数据无需改变。 当比较两个Key大小时，TypeComparator提供了直接基于二进制数据的对比方法，无需反序列化任何数据。
- 排序完成后，访问数据时，按照第二个MemorySegment集中Key的顺序访问，并通过Pointer值找到数据在第一个MemorySegment集中的位置，通过TypeSerializer反序列化成Java对象返回。

 

![img](https://upload-images.jianshu.io/upload_images/14534869-af76bcb579b231bd.png)

Flink排序算法

#### 这样实现的好处有：

- 通过Key和Full data分离存储的方式尽量将被操作的数据最小化，提高Cache命中的概率，从而提高CPU的吞吐量。
- 移动数据时，只需移动Key+Pointer，而无须移动数据本身，大大减少了内存拷贝的数据量。
- TypeComparator直接基于二进制数据进行操作，节省了反序列化的时间。

通过定制的内存管理，Flink通过充分利用内存与CPU缓存，大大提高了CPU的执行效率，同时由于大部分内存都由框架自己控制，也很大程度提升了系统的健壮性，减少了OOM出现的可能。

### 总结

本文主要介绍了Flink项目的一些关键特性，Flink是一个拥有诸多特色的项目，包括其统一的批处理和流处理执行引擎，通用大数据计算框架与传统数据库系统的技术结合，以及流处理系统的诸多技术创新等，因为篇幅有限，Flink还有一些其他很有意思的特性没有详细介绍，比如DataSet API级别的执行计划优化器，原生的迭代操作符等，感兴趣的读者可以通过Flink官网了解更多Flink的详细内容。希望通过本文的介绍能够让读者对Flink有更多的了解，也让更多的人使用甚至参与到Flink项目中去。