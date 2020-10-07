##### 1、Hive的优点和缺点

**优点：**
 1）Hive 使用类SQL 查询语法，最大限度的实现了和SQL标准的兼容，大大降低了传统数据分析人员处理大数据的难度；
 2）使用JDBC 接口/ODBC接口，开发人员更易开发应用；
 3）以MR 作为计算引擎、HDFS 作为存储系统，为超大数据集设计的计算/ 扩展能力；
 4）统一的元数据管理（Derby、MySql等），并可与Pig 、spark等共享； 元数据：hive表所对应的字段、属性还有表所对应存储的HDFS目录。

![img](https:////upload-images.jianshu.io/upload_images/16338962-e3c4f0391cb0741f.png?imageMogr2/auto-orient/strip|imageView2/2/w/817/format/webp)



**缺点：**
 1）Hive 的HQL 表达的能力有限，比如不支持UPDATE、非等值连接、DELETE、INSERT单条等，insert单条代表的是 创建一个文件；
 2）由于Hive自动生成MapReduce 作业， HQL 调优困难；
 3）粒度较粗，可控性差，是因为数据是读的时候进行类型的转换，mysql关系型数据是在写入的时候就检查了数据的类型；
 4）hive生成MapReduce作业，高延迟，不适合实时查询。

##### 2、Hive与关系数据库的区别

1）hive和关系数据库存储文件的系统不同，hive使用的是hadoop的HDFS，关系数据库则是服务器本地的文件系统；
 2）hive使用mapreduce做运算，与传统数据库相比运算数据规模要大得多；
 3）关系数据库都是为实时查询的业务进行设计的，而hive则是为海量数据做数据挖掘设计的，实时性很差；实时性差导致hive的应用场景和关系数据库有很大的区别；



4）Hive很容易扩展自己的存储能力和计算能力，这个是继承hadoop的，而关系数据库在这个方面要比Hive差很多。

![img](https:////upload-images.jianshu.io/upload_images/16338962-d032b3bb600d12a1.png?imageMogr2/auto-orient/strip|imageView2/2/w/566/format/webp)

##### 3、hive的服务端组件和客户端组件

![img](https:////upload-images.jianshu.io/upload_images/16338962-1fdca9ed6c53241f.png?imageMogr2/auto-orient/strip|imageView2/2/w/551/format/webp)

**3.1、服务端组件：**

**Driver组件：**该组件包括Complier、Optimizer和Executor，它的作用是将HiveQL（类SQL）语句进行解析、编译优化，生成执行计划，然后调用底层的MapReduce计算框架。

**Metastore组件：**元数据服务组件，这个组件存取Hive的元数据，Hive的元数据存储在关系数据库里，Hive支持的关系数据库有Derby和Mysql。作用是：客户端连接metastore服务，metastore再去连接MySQL数据库来存取元数据。

![img](https:////upload-images.jianshu.io/upload_images/16338962-3cc3a363d043ad43.png?imageMogr2/auto-orient/strip|imageView2/2/w/615/format/webp)



**Thrift服务：**Thrift是Facebook开发的一个软件框架，它用来进行可扩展且跨语言的服务的开发，Hive集成了该服务，能让不同的编程语言调用Hive的接口。

**3.2、客户端组件：**

**CLI：**命令行接口。

**JDBC/ODBC：**Hive架构的JDBC和ODBC接口是建立在Thrift客户端之上。

**WEBGUI：**Hive客户端提供了一种通过网页的方式访问Hive所提供的服务。这个接口对应Hive的HWI组件（Hive Web Interface），使用前要启动HWI服务。

##### 4、Hive查询的执行过程



![img](https:////upload-images.jianshu.io/upload_images/16338962-777fd73a46de95e1.png?imageMogr2/auto-orient/strip|imageView2/2/w/686/format/webp)


**1）Execute Query：**hive界面如命令行或Web UI将查询发送到Driver(任何数据库驱动程序如JDBC、ODBC,等等)来执行。



**2）Get Plan：**Driver根据查询编译器解析query语句,验证query语句的语法,查询计划或者查询条件。

**3）Get Metadata：**编译器将元数据请求发送给Metastore(数据库)。

**4）Send Metadata：**Metastore将元数据作为响应发送给编译器。

**5）Send Plan：**编译器检查要求和重新发送Driver的计划。到这里,查询的解析和编译完成。

**6）Execute Plan：**Driver将执行计划发送到执行引擎。

**6.1）Execute Job：**hadoop内部执行的是mapreduce工作过程,任务执行引擎发送一个任务到资源管理节点(resourcemanager)，资源管理器分配该任务到任务节点，由任务节点上开始执行mapreduce任务。

**6.2）Metadata Ops：**在执行引擎发送任务的同时,对hive的元数据进行相应操作。

**7）Fetch Result：**执行引擎接收数据节点(data node)的结果。

**8）Send Results：**执行引擎发送这些合成值到Driver。

**9）Send Results：**Driver将结果发送到hive接口。



作者：喵感数据
链接：https://www.jianshu.com/p/afe2248aa86e
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。













一、Hive 架构

下面是Hive的架构图。



![img](https://ask.qcloudimg.com/http-save/developer-news/4nx34w94kz.jpeg?imageView2/2/w/1620)

Hive的体系结构可以分为以下几部分：

1、用户接口主要有三个：CLI，Client 和 WUI。其中最常用的是CLI，Cli启动的时候，会同时启动一个Hive副本。Client是Hive的客户端，用户连接至Hive Server。在启动 Client模式的时候，需要指出Hive Server所在节点，并且在该节点启动Hive Server。 WUI是通过浏览器访问Hive。

2、Hive将元数据存储在数据库中，如mysql、derby。Hive中的元数据包括表的名字，表的列和分区及其属性，表的属性（是否为外部表等），表的数据所在目录等。

3、解释器、编译器、优化器完成HQL查询语句从词法分析、语法分析、编译、优化以及查询计划的生成。生成的查询计划存储在HDFS中，并在随后有MapReduce调用执行。

4、Hive的数据存储在HDFS中，大部分的查询、计算由MapReduce完成（包含*的查询，比如select * from tbl不会生成MapRedcue任务）。

二、元数据存储

Hive将元数据存储在RDBMS中，有三种模式可以连接到数据库：

1、元数据库内嵌模式：此模式连接到一个In-memory 的数据库Derby，一般用于Unit Test。



![img](https://ask.qcloudimg.com/http-save/developer-news/2rnc8awgu8.jpeg?imageView2/2/w/1620)

2、元数据库mysql模式：通过网络连接到一个数据库中，是最经常使用到的模式。



![img](https://ask.qcloudimg.com/http-save/developer-news/yhm8eky5ce.jpeg?imageView2/2/w/1620)

3、MetaStoreServe访问元数据库模式：用于非Java客户端访问元数据库，在服务器端启动MetaStoreServer，客户端利用Thrift协议通过MetaStoreServer访问元数据库。



![img](https://ask.qcloudimg.com/http-save/developer-news/y1p4ferkw2.jpeg?imageView2/2/w/1620)

对于数据存储，Hive没有专门的数据存储格式，也没有为数据建立索引，用户可以非常自由的组织Hive中的表，只需要在创建表的时候告诉Hive数据中的列分隔符和行分隔符，Hive就可以解析数据。Hive中所有的数据都存储在HDFS中，存储结构主要包括数据库、文件、表和视图。Hive中包含以下数据模型：Table内部表，External Table外部表，Partition分区，Bucket桶。Hive默认可以直接加载文本文件，还支持sequence file 、RCFile。

4、三种模式汇总：



![img](https://ask.qcloudimg.com/http-save/developer-news/towioe4sia.jpeg?imageView2/2/w/1620)

三、Hive 工作原理

Hive 工作原理如下图所示。



![img](https://ask.qcloudimg.com/http-save/developer-news/exggeujtaa.jpeg?imageView2/2/w/1620)

Hive构建在Hadoop之上

1、HQL中对查询语句的解释、优化、生成查询计划是由Hive完成的

2、所有的数据都是存储在Hadoop中

3、查询计划被转化为MapReduce任务，在Hadoop中执行（有些查询没有MR任务，如：select * from table）

4、Hadoop和Hive都是用UTF-8编码的

Hive编译器的组成：



![img](https://ask.qcloudimg.com/http-save/developer-news/og4elfsug2.jpeg?imageView2/2/w/1620)

Hive编译流程如下



![img](https://ask.qcloudimg.com/http-save/developer-news/hjnxv6wclo.jpeg?imageView2/2/w/1620)

![img](https://ask.qcloudimg.com/http-save/developer-news/c8l86t14an.jpeg?imageView2/2/w/1620)

四、Hive安装和部署

1、解压缩 hive 安装包

2、安装并启动mysql

3、修改 mysql 密码

4、给root用户访问授权

5、建立链接,该命令在 hive 安装目录的 lib 目录下建立软链接，指向/usr/share/java/mysql-connector-java.jar

6、配置环境变量

生效环境变量

7、修改hadoop的配置文件core-site.xml

增加如下：

8、修改hive的配置文件hive-site.xml

增加如下：

注释：

hive.metastore.uris 中的“hadoop“含义为 metastore 所在的机器（启动metastore 的方法见下一节）

javax.jdo.option.ConnectionURL 中的“hadoop”为 mysql 安装机器的hostname

javax.jdo.option.ConnectionUserName 和javax.jdo.option.ConnectionPassword

分别为 mysql 的访问用户和密码

fs.defaultFS 为 HDFS 的 namenode 启动的机器地址

beeline.hs2.connection.user 和 beeline.hs2.connection.password 是 beeline 方式

访问的用户名和密码，可任意指定，但在 beeline 访问时要写入你指定的这个

（具体参考最后一部分）

9、启动hive metastore

10、启动 hiveserver2

11、hive 常见两种访问方式