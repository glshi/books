## 一、Kafka 简介

Kafka是一种高吞吐量、分布式、基于发布/订阅的消息系统，最初由LinkedIn公司开发，使用Scala语言编写，目前是Apache的开源项目。

跟RabbitMQ、RocketMQ等目前流行的开源消息中间件相比，Kakfa具有高吞吐、低延迟等特点，在大数据、日志收集等应用场景下被广泛使用。

本文主要简单介绍Kafka的设计原理。

## 二、Kafka 架构

![img](https://blog-10039692.file.myqcloud.com/1503038310607_9038_1503038310779.png)

基本概念：

- broker：Kafka服务器，负责消息存储和转发
- topic：消息类别，Kafka按照topic来分类消息
- partition：topic的分区，一个topic可以包含多个partition，topic消息保存在各个partition上
- offset：消息在日志中的位置，可以理解是消息在partition上的偏移量，也是代表该消息的唯一序号
- Producer：消息生产者
- Consumer：消息消费者
- Consumer Group：消费者分组，每个Consumer必须属于一个group
- Zookeeper：保存着集群broker、topic、partition等meta数据；另外，还负责broker故障发现，partition leader选举，负载均衡等功能

## 三、Kafka 设计原理

### 3.1 数据存储设计

partition以文件形式存储在文件系统，目录命名规则：`<topic_name>-<partition_id>`，例如，名为test的topic，其有3个partition，则Kafka数据目录中有3个目录：test-0, test-1, test-2，分别存储相应partition的数据。

**partition的数据文件**

partition中的每条Message包含了以下三个属性：

- offset
- MessageSize
- data

其中offset表示Message在这个partition中的偏移量，offset不是该Message在partition数据文件中的实际存储位置，而是逻辑上一个值，它唯一确定了partition中的一条Message，可以认为offset是partition中Message的id；MessageSize表示消息内容data的大小；data为Message的具体内容。

partition的数据文件由以上格式的Message组成，按offset由小到大排列在一起。 如果一个partition只有一个数据文件：

1. 新数据是添加在文件末尾，不论文件数据文件有多大，这个操作永远都是O(1)的。
2. 查找某个offset的Message是顺序查找的。因此，如果数据文件很大的话，查找的效率就低。

Kafka通过**分段**和**索引**来提高查找效率。

**数据文件分段segment**

partition物理上由多个segment文件组成，每个segment大小相等，顺序读写。每个segment数据文件以该段中最小的offset命名，文件扩展名为.log。这样在查找指定offset的Message的时候，用二分查找就可以定位到该Message在哪个segment数据文件中。

**数据文件索引**

数据文件分段使得可以在一个较小的数据文件中查找对应offset的Message了，但是这依然需要顺序扫描才能找到对应offset的Message。为了进一步提高查找的效率，Kafka为每个分段后的数据文件建立了索引文件，文件名与数据文件的名字是一样的，只是文件扩展名为.index。

![img](https://blog-10039692.file.myqcloud.com/1503038418950_9422_1503038419509.png)

索引文件中包含若干个索引条目，每个条目表示数据文件中一条Message的索引。索引包含两个部分，分别为相对offset和position。

- 相对offset：因为数据文件分段以后，每个数据文件的起始offset不为0，相对offset表示这条Message相对于其所属数据文件中最小的offset的大小。举例，分段后的一个数据文件的offset是从20开始，那么offset为25的Message在index文件中的相对offset就是25-20 = 5。存储相对offset可以减小索引文件占用的空间。
- position，表示该条Message在数据文件中的绝对位置。只要打开文件并移动文件指针到这个position就可以读取对应的Message了。 index文件中并没有为数据文件中的每条Message建立索引，而是采用了稀疏存储的方式，每隔一定字节的数据建立一条索引。这样避免了索引文件占用过多的空间，从而可以将索引文件保留在内存中。但缺点是没有建立索引的Message也不能一次定位到其在数据文件的位置，从而需要做一次顺序扫描，但是这次顺序扫描的范围就很小了。

![img](https://blog-10039692.file.myqcloud.com/1503038445835_8365_1503038446150.png)

**总结**

查找某个offset的消息，先二分法找出消息所在的segment文件（因为每个segment的命名都是以该文件中消息offset最小的值命名）；然后，加载对应的.index索引文件到内存，同样二分法找出小于等于给定offset的最大的那个offset记录（相对offset，position）；最后，根据position到.log文件中，顺序查找出offset等于给定offset值的消息。

由于消息在partition的segment数据文件中是顺序读写的，且消息消费后不会删除（删除策略是针对过期的segment文件），这种顺序磁盘IO存储设计是Kafka高性能很重要的原因。

### 3.2 生产者设计

![img](https://blog-10039692.file.myqcloud.com/1503038464608_3910_1503038464802.png)

- 负载均衡：由于消息topic由多个partition组成，且partition会均衡分布到不同broker上，因此，为了有效利用broker集群的性能，提高消息的吞吐量，producer可以通过随机或者hash等方式，将消息平均发送到多个partition上，以实现负载均衡。
- 批量发送：是提高消息吞吐量重要的方式，Producer端可以在内存中合并多条消息后，以一次请求的方式发送了批量的消息给broker，从而大大减少broker存储消息的IO操作次数。但也一定程度上影响了消息的实时性，相当于以时延代价，换取更好的吞吐量。

### 3.3 消费者设计

![img](https://blog-10039692.file.myqcloud.com/1503038490224_3768_1503038490469.png)

- 任何Consumer必须属于一个Consumer Group
- 同一Consumer Group中的多个Consumer实例，不同时消费同一个partition，等效于队列模式。如图，Consumer Group 1的三个Consumer实例分别消费不同的partition的消息，即，TopicA-part0、TopicA-part1、TopicA-part2。
- 不同Consumer Group的Consumer实例可以同时消费同一个partition，等效于发布订阅模式。如图，Consumer Group 1的Consumer1和Consumer Group 2的Consumer4，同时消费TopicA-part0的消息。
- partition内消息是有序的，Consumer通过pull方式消费消息。
- Kafka不删除已消费的消息

**队列模式**

队列模式，指每条消息只会有一个Consumer消费到。Kafka保证同一Consumer Group中只有一个Consumer会消费某条消息。

- 在Consumer Group稳定状态下，每一个Consumer实例只会消费某一个或多个特定partition的数据，而某个partition的数据只会被某一个特定的Consumer实例所消费，也就是说Kafka对消息的分配是以partition为单位分配的，而非以每一条消息作为分配单元；
- 同一Consumer Group中，如果Consumer实例数量少于partition数量，则至少有一个Consumer会消费多个partition的数据；如果Consumer的数量与partition数量相同，则正好一个Consumer消费一个partition的数据；而如果Consumer的数量多于partition的数量时，会有部分Consumer无法消费该Topic下任何一条消息；
- 设计的优势是：每个Consumer不用都跟大量的broker通信，减少通信开销，同时也降低了分配难度，实现也更简单；可以保证每个partition里的数据可以被Consumer有序消费。
- 设计的劣势是：无法保证同一个Consumer Group里的Consumer均匀消费数据，且在Consumer实例多于partition个数时导致有些Consumer会饿死。

如果有partition或者Consumer的增减，为了保证均衡消费，需要实现Consumer Rebalance，分配算法如下：

![img](https://blog-10039692.file.myqcloud.com/1503038533895_5472_1503038534094.png)

broker对Consumer设计原理：

- 对于每个Consumer Group，选举出一个Broker作为Coordinator（0.9版本以上），由它Watch Zookeeper，从而监控判断是否有partition或者Consumer的增减，然后生成Rebalance命令，按照以上算法重新分配。
- 当Consumer Group第一次被初始化时，Consumer通常会读取每个partition的最早或最近的offset（Zookeeper记录），然后顺序地读取每个partition log的消息，在Consumer读取过程中，它会提交已经成功处理的消息的offsets（由Zookeeper记录）。
- 当一个partition被重新分配给Consumer Group中的其他Consumer，新的Consumer消费的初始位置会设置为(原来Consumer)最近提交的offset。

![img](https://blog-10039692.file.myqcloud.com/1503038555171_8712_1503038555302.png)

如图，Last Commited Offset指Consumer最近一次提交的消费记录offset，Current Position是当前消费的位置，High Watermark是成功拷贝到log的所有副本节点（partition的所有ISR节点，下文介绍）的最近消息的offset，Log End Offset是写入log中最后一条消息的offset+1。

从Consumer的角度来看，最多只能读取到High watermark的位置，后面的消息对消费者不可见，因为未完全复制的数据还没可靠存储，有丢失可能。

**发布订阅模式**

发布订阅模式，又指广播模式，Kafka保证topic的每条消息会被所有Consumer Group消费到，而对于同一个Consumer Group，还是保证只有一个Consumer实例消费到这条消息。

### 3.4 Replication设计

作为消息中间件，数据的可靠性以及系统的可用性，必然依赖数据副本的设计。

Kafka的replica副本单元是topic的partition，一个partition的replica数量不能超过broker的数量，因为一个broker最多只会存储这个partition的一个副本。所有消息生产、消费请求都是由partition的leader replica来处理，其他follower replica负责从leader复制数据进行备份。

Replica均匀分布到整个集群，Replica的算法如下：

- 将所有Broker（假设共n个Broker）和待分配的Partition排序
- 将第i个Partition分配到第（i mod n）个Broker上
- 将第i个Partition的第j个Replica分配到第（(i + j) mode n）个Broker上

![img](https://blog-10039692.file.myqcloud.com/1503038589840_7834_1503038589933.png)

如图，TopicA有三个partition：part0、part1、part2，每个partition的replica数等于2（一个是leader，另一个是follower），按照以上算法会均匀落到三个broker上。

broker对replica管理： 选举出一个broker作为controller，由它Watch Zookeeper，负责partition的replica的集群分配，以及leader切换选举等流程。

**In-Sync-Replica(ISR)**

分布式系统在处理节点故障时，需要预先明确节点的”failure”和”alive”的定义。对于Kafka节点，判断是”alive”有以下两个条件：

- 节点必须和Zookeeper保持心跳连接
- 如果节点是follower，必须从leader节点上复制数据来备份，而且备份的数据相比leader而言，不能落后太多。

Kafka将满足以上条件的replica节点认为是”in sync”（同步中），称为In-Sync-Replica(ISR)。

Kafka的Zookeeper维护了每个partition的ISR信息，理想情况下，ISR包含了partition的所有replica所在的broker节点信息，而当某些节点不满足以上条件时，ISR可能只包含部分replica。例如，上图中的TopicA-part0的ISR列表可能是[broker1,broker2,broker3]，也可能是[broker1,broker3]和[broker1]。

**数据可靠性**

Kafka如何保证数据可靠性？首先看下，Producer生产一条消息，该消息被认为是”committed”（即broker认为消息已经可靠存储）的过程：

- 消息所在partition的ISR replicas会定时异步从leader上批量复制数据log
- 当所有ISR replica都返回ack，告诉leader该消息已经写log成功后，leader认为该消息committed，并告诉Producer生产成功。这里和以上”alive”条件的第二点是不矛盾的，因为leader有超时机制，leader等ISR的follower复制数据，如果一定时间不返回ack（可能数据复制进度落后太多），则leader将该follower replica从ISR中剔除。
- 消息committed之后，Consumer才能消费到。

![img](https://blog-10039692.file.myqcloud.com/1503038622308_9391_1503038622409.png)

![img](https://blog-10039692.file.myqcloud.com/1503038638199_239_1503038638292.png)

![img](https://blog-10039692.file.myqcloud.com/1503038650081_5973_1503038650164.png)

ISR机制下的数据复制，既不是完全的同步复制，也不是单纯的异步复制，这是Kafka高吞吐很重要的机制。同步复制要求所有能工作的follower都复制完，这条消息才会被认为committed，这种复制方式极大的影响了吞吐量。而异步复制方式下，follower异步的从leader复制数据，数据只要被leader写入log就被认为已经committed，这种情况下如果follower都复制完都落后于leader，而如果leader突然宕机，则会丢失数据。而Kafka的这种使用ISR的方式则很好的均衡了确保数据不丢失以及吞吐量，follower可以批量的从leader复制数据，数据复制到内存即返回ack，这样极大的提高复制性能，当然数据仍然是有丢失风险的。

Kafka本身定位于高性能的MQ，更多注重消息吞吐量，在此基础上结合ISR的机制去尽量保证消息的可靠性，但不是绝对可靠的。

**服务可用性**

Kafka所有收发消息请求都由leader节点处理，由以上数据可靠性设计可知，当ISR的follower replica故障后，leader会及时地从ISR列表中把它剔除掉，并不影响服务可用性，那么当leader故障后会怎样呢？如何选举新的leader？

leader选举

- Kafka在Zookeeper存储partition的ISR信息，并且能动态调整ISR列表的成员，只有ISR里的成员replica才会被选为leader，并且ISR所有的replica都有可能成为leader；
- leader节点宕机后，Zookeeper能监控发现，并由broker的controller节点从ISR中选举出新的leader，并通知ISR内的所有broker节点。

![img](https://blog-10039692.file.myqcloud.com/1503038673204_6373_1503038673293.png)

因此，可以看出，只要ISR中至少有一个replica，Kafka就能保证服务的可用性（但不保证网络分区下的可用性）。

**容灾和数据一致性**

分布式系统的容灾能力，跟其本身针对数据一致性考虑所选择的算法有关，例如，Zookeeper的Zab算法，raft算法等。Kafka的ISR机制和这些Majority Vote算法对比如下：

- ISR机制能容忍更多的节点失败。假如replica节点有2f+1个，每个partition最多能容忍2f个失败，且不丢失消息数据；但相对Majority Vote选举算法，只能最多容忍f个失败。
- 在消息committed持久化上，ISR需要等2f个节点返回ack，但Majority Vote只需等f+1个节点返回ack，且不依赖处理最慢的follower节点，因此Majority Vote有优势
- ISR机制能节省更多replica节点数。例如，要保证f个节点可用，ISR方式至少要f个节点，而Majority Vote至少需要2f+1个节点。

如果所有replica都宕机了，有两种方式恢复服务：

1. 等ISR任一节点恢复，并选举为leader；
2. 选择第一个恢复的节点（不一定是ISR中的节点）为leader

第一种方式消息不会丢失（只能说这种方式最有可能不丢而已），第二种方式可能会丢消息，但能尽快恢复服务可用。这是可用性和一致性场景的两种考虑，Kafka默认选择第二种，用户也可以自主配置。

大部分考虑CP的分布式系统（假设2f+1个节点），为了保证数据一致性，最多只能容忍f个节点的失败，而Kafka为了兼顾可用性，允许最多2f个节点失败，因此是无法保证数据强一致的。

![img](https://blog-10039692.file.myqcloud.com/1503038705240_5694_1503038705371.png)

如图所示，一开始ISR数量等于3，正常同步数据，红色部分开始，leader发现其他两个follower复制进度太慢或者其他原因（网络分区、节点故障等），将其从ISR剔除后，leader单节点存储数据；然后，leader宕机，触发重新选举第二节点为leader，重新开始同步数据，但红色部分的数据在新leader上是没有的；最后原leader节点恢复服务后，重新从新leader上复制数据，而红色部分的数据已经消费不到了。

因此，为了减少数据丢失的概率，可以设置Kafka的ISR最小replica数，低于该值后直接返回不可用，当然是以牺牲一定可用性和吞吐量为前提了。

**重复消息**

消息传输有三种方式：

1. At most once：消息可能会丢失，但不会重复传输
2. At least once：消息不会丢失，但可能重复传输
3. Exactly once：消息保证会被传输一次且仅传输一次

Kafka实现了第二种方式，即，可能存在重复消息，需要业务自己保证消息幂等性处理。

### 3.5 高吞吐设计

1. 对于partition，顺序读写磁盘数据，以时间复杂度O(1)方式提供消息持久化能力。
2. Producer批量向broker写数据
3. Consumer批量从broker拉数据
4. 日志压缩
5. Topic分多个partition，提高并发
6. broker零拷贝（Zero Copy），使用sendfile系统调用，将数据直接从page cache发送到socket上
7. Producer可配置是否等待消息committed。如果Producer生产消息，每次都必须等ISR存储后才返回，时延会很高，进而影响整体消息的吞吐量。为了解决这个问题，一方面Producer可以配置减少partition的副本数，例如，ISR大小为1；另一方面，在不太关注消息可靠存储的场景下，Producer可以通过配置选择是否等待消息committed，如下：

| Acks | 持久性等级 | 返回响应时机         | 写延时 |
| :--- | :--------- | :------------------- | :----- |
| all  | 高         | 全部ISR成员返回ACK   | 高     |
| 1    | 中         | ISR中任一成员返回ACK | 中     |
| 0    | 低         | 不等待ISR成员返回ACK | 低     |

这是用户在消息吞吐量和持久化之间做的权衡选择，持久化等级越高，生产消息吞吐量越小，反之，持久化等级越低，吞吐量越高。

### 3.6  HA 基本原理

**broker HA**

broker集群信息由Zookeeper维护，并选举出一个controller。所有partition的leader选举都由controller决定，将leader的变更直接通过rpc方式通知需要为此做出响应的brokers；controller也负责增删topic以及partition replica的重新分配。

controller在Zookeeper上注册watch，一旦有broker宕机，其对应在Zookeeper的临时节点自动被删除，controller对宕机broker上的所有partition重新分配新leader；如果controller宕机，其他broker通过Zookeeper选举出新的controller，然后同样对宕机broker上的所有partition重新分配新leader。

**partition HA**

partition leader所在的broker宕机，如上所述，broker controller根据动态维护的ISR，会重新在剩下的broker机器中选出ISR里面的一个成员成为新的leader。如果ISR中至少有一个follower，则可以确保已经committed的数据不丢失；否则选择任意一个replica作为leader，该场景可能会有潜在的数据丢失；如果partition所有的replica都宕机了，就无法保证数据不丢失了，有两种恢复方案，上文已介绍过。