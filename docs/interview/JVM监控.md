https://www.cnblogs.com/jhxxb/p/13279201.html

https://blog.csdn.net/u012550080/article/details/81605189

https://www.jianshu.com/p/bb4ba6612392





## 一、jmx 方式

加上如下启动参数，以 tomcat 为例，修改 bin\catalina 文件，在开始位置添加 JAVA_OPTS

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
JAVA_OPTS="-Djava.rmi.server.hostname=192.168.8.229 -Dcom.sun.management.jmxremote.port=1100 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false"



# 开启 JMX 远程服务权限
# -Dcom.sun.management.jmxremote.port：配置远程 connection 的端口号
# -Dcom.sun.management.jmxremote.ssl：指定 JMX 是否启用 ssl
# -Dcom.sun.management.jmxremote.authenticate：指定 JMX 是否启用密码
# -Djava.rmi.server.hostname：配置 Server IP（不要使用 127.0.0.1）
# -Dcom.sun.management.jmxremote.rmi.port=2222
# -Dcom.sun.management.jmxremote.local.only=false
# -Dcom.sun.management.jmxremote=true
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

JDK8 后不集成 jvisualvm

https://visualvm.github.io/download.html

```
# windows 上启动
start /b visualvm_202\bin\visualvm.exe --jdkhome "D:\PcAPP\jdk-11.0.7" --userdir "data"
```

JDK8 可以直接使用，Windows 下打开 JDK 目录下的 bin/jvisualvm.exe 程序

![img](https://img2018.cnblogs.com/blog/1595409/201908/1595409-20190807231723928-1995964425.png)

添加 JMX 连接，填写地址和端口即可

![img](https://img2018.cnblogs.com/blog/1595409/201908/1595409-20190807231856988-685589844.png)

查看堆栈

![img](https://img2018.cnblogs.com/blog/1595409/201908/1595409-20190807232110011-968590364.png)

 

## 二、Jstatd 方式

在 $JAVA_HOME/bin 下创建 jstatd.all.policy 文件

```
cd /opt/jdk-11.0.7/bin/
vim jstatd.all.policy
```

https://stackoverflow.com/questions/51032095/starting-jstatd-in-java-9

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
# 有 tools.jar（JDK8）
grant codebase "file:${java.home}/lib/tools.jar" {
    permission java.security.AllPermission;
};


# 没有 tools.jar（JDK11）
grant codebase "jrt:/jdk.jstatd" {
    permission java.security.AllPermission;
};

grant codebase "jrt:/jdk.internal.jvmstat" {
    permission java.security.AllPermission;
};
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

启动

```
cd /opt/jdk-11.0.7/bin/
nohup jstatd -J-Djava.rmi.server.hostname=192.168.8.136 -J-Djava.security.policy=./jstatd.all.policy -p 1099 &
jps -l
```

![img](https://img2020.cnblogs.com/blog/1595409/202007/1595409-20200710144118257-1881838586.png)







# jdk自带监控程序jvisualvm的使用

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/original.png)

[慕容羽](https://me.csdn.net/u012550080) 2018-08-12 12:02:05 ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/articleReadEyes.png) 37826 ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollect.png) 收藏 69

版权

1. 监控小程序的配置

1. 生产环境tomcat的配置

编辑应用所在的tomcat服务器下的bin目录下的catalina.sh文件，修改如下：

![img](https://img-blog.csdn.net/20180812120048262?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI1NTAwODA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

配置如下内容：

export JAVA_OPTS="-Xms256m -Xmx512m -Xss256m -XX:PermSize=512m -XX:MaxPermSize=1024m -Djava.rmi.server.hostname=136.64.45.24 -Dcom.sun.management.jmxremote.port=9315 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false"

红字内容需要进行添加，黄色背景的需要根据具体的主机情况配置。

1. 本地jdk小工具的配置

进入到本地的jdk安装目录下，找到jvisualvm.exe,双击打开

 

建立远程连接

图一 添加远程

![img](https://img-blog.csdn.net/20180812120048321?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI1NTAwODA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

图二 建立远程主机ip

 

图三 添加jmx连接

![img](https://img-blog.csdn.net/20180812120048327?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI1NTAwODA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

图四 双击建立好的连接可以实时查看当前程序的运行状况和堆栈信息等

![img](https://img-blog.csdn.net/20180812120048378?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI1NTAwODA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

​     监控小程序菜单截图

图一 概述：显示当前tomcat服务器的整体运行状况

![img](https://img-blog.csdn.net/20180812120048636?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI1NTAwODA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

图二：可事实动态显示cpu、堆栈、类、线程的相关信息

![img](https://img-blog.csdn.net/20180812120048681?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI1NTAwODA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

图三：线程：可实时动态的显示进程的使用状况

![img](https://img-blog.csdn.net/20180812120048709?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI1NTAwODA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

点击线程Dump按钮可以显示具体的进程的内容，可从此页面查看到进程的具体信息以及报错信息

![img](https://img-blog.csdn.net/20180812120048716?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI1NTAwODA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

（4）通过此工具的使用，当ITSM系统僵死时，可看到明显的进程变化

a.所有的请求进程都进入了监控状态，所有请求都无法访问

​      ![img](https://img-blog.csdn.net/20180812120048760?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI1NTAwODA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

1. 查看dump信息，发现ITSM系统首页的报表获取sql存在问题，访问量增多时，打开首页查询比较慢，链接释放缓慢，然后新增请求，导致链接池被沾满，出现了上述问题。

![img](https://img-blog.csdn.net/20180812120048802?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI1NTAwODA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

1. 故障解决方案

通过上述分析，查看连接池配置，对连接池的活动连接数和空闲连接数做了调账，如下图：![img](https://img-blog.csdn.net/20180812120049186?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI1NTAwODA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

同时找到对应的sql语句进行优化，之前的sql存在笛卡尔积，同时数据量大，没有建立索引，通过dba的分析修改了sql语句，并对timeouttime字段建立索引，截图如下（包含原有sql【被注释了】）

![img](https://img-blog.csdn.net/20180812120049237?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI1NTAwODA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

1. 知识扩展

Jdk自带的监控小工具可用来对java程序进行调优和问题分析。

应用僵死后是否有dump或javacore文件生成？拿到后用MemoryAnalyzer 等工具分析一下

如果没有，可以用jdk自带的小工具看一下，是否有资源未释放、线程死锁、内存溢出等问题

 

============================================================

jstack ( 查看jvm线程运行状态，是否有死锁现象等等信息)
jinfo:可以输出并修改运行时的java 进程的opts。 
jps:与unix上的ps类似，用来显示本地的java进程，可以查看本地运行着几个java程序，并显示他们的进程号。 
jstat:一个极强的监视VM内存工具。可以用来监视VM内存内的各种堆和非堆的大小及其内存使用量。 
jmap:打印出某个java进程（使用pid）内存内的所有'对象'的情况（如：产生那些对象，及其数量）。 
jconsole:一个java GUI监视工具，可以以图表化的形式显示各种数据。并可通过远程连接监视远程的服务器VM。 

详细：在使用这些工具前，先用JPS命令获取当前的每个JVM进程号，然后选择要查看的JVM。 
jstat工具特别强大，有众多的可选项，详细查看堆内各个部分的使用量，以及加载类的数量。使用时，需加上查看进程的进程id，和所选参数。以下详细介绍各个参数的意义。 
jstat -class pid:显示加载class的数量，及所占空间等信息。 
jstat -compiler pid:显示VM实时编译的数量等信息。 
jstat -gc pid:可以显示gc的信息，查看gc的次数，及时间。其中最后五项，分别是young gc的次数，young gc的时间，full gc的次数，full gc的时间，gc的总时间。 
jstat -gccapacity:可以显示，VM内存中三代（young,old,perm）对象的使用和占用大小，如：PGCMN显示的是最小perm的内存使用量，PGCMX显示的是perm的内存最大使用量，PGC是当前新生成的perm内存占用量，PC是但前perm内存占用量。其他的可以根据这个类推， OC是old内纯的占用量。 
jstat -gcnew pid:new对象的信息。 
jstat -gcnewcapacity pid:new对象的信息及其占用量。 
jstat -gcold pid:old对象的信息。 
jstat -gcoldcapacity pid:old对象的信息及其占用量。 
jstat -gcpermcapacity pid: perm对象的信息及其占用量。 
jstat -util pid:统计gc信息统计。 
jstat -printcompilation pid:当前VM执行的信息。 
除了以上一个参数外，还可以同时加上 两个数字，如：jstat -printcompilation 3024 250 6是每250毫秒打印一次，一共打印6次，还可以加上-h3每三行显示一下标题。 

jmap是一个可以输出所有内存中对象的工具，甚至可以将VM 中的heap，以二进制输出成文本。 
命令：jmap -dump:format=b,file=heap.bin <pid> 
file：保存路径及文件名 
pid：进程编号 
•jmap -histo:live pid| less :堆中活动的对象以及大小 
•jmap -heap pid : 查看堆的使用状况信息 


jinfo:的用处比较简单，就是能输出并修改运行时的java进程的运行参数。用法是jinfo -opt pid 如：查看2788的MaxPerm大小可以用 jinfo -flag MaxPermSize 2788。 

jconsole是一个用java写的GUI程序，用来监控VM，并可监控远程的VM，非常易用，而且功能非常强。使用方法：命令行里打 jconsole，选则进程就可以了。 
JConsole中关于内存分区的说明。 

Eden Space (heap)： 内存最初从这个线程池分配给大部分对象。 
Survivor Space (heap)：用于保存在eden space内存池中经过垃圾回收后没有被回收的对象。 
Tenured Generation (heap)：用于保持已经在 survivor space内存池中存在了一段时间的对象。 
Permanent Generation (non-heap): 保存虚拟机自己的静态(refective)数据，例如类（class）和方法（method）对象。Java虚拟机共享这些类数据。这个区域被分割为只读的和只写的， 
Code Cache (non-heap):HotSpot Java虚拟机包括一个用于编译和保存本地代码（native code）的内存，叫做“代码缓存区”（code cache） 

•jstack ( 查看jvm线程运行状态，是否有死锁现象等等信息) : jstack pid : thread dump 
•jstat -gcutil pid 1000 100 : 1000ms统计一次gc情况统计100次； 



# JVM 线程dump 导出和分析

[![img](https://cdn2.jianshu.io/assets/default_avatar/10-e691107df16746d4a9f3fe9496fd1848.jpg)](https://www.jianshu.com/u/7c87b7e0b12f)

[码农随想录](https://www.jianshu.com/u/7c87b7e0b12f)关注

0.0732019.01.01 17:03:48字数 1,220阅读 5,653

# 前言

- 线程dump是非常有用的诊断java应用问题的工具，每一个java虚拟机都有及时生成显示所有线程在某一点状态的线程dump的能力。虽然各个java虚拟机线程dump打印输出格式上略微有一些不同，但是线程dump出来的信息包含线程基本信息；线程的运行状态、标识和调用的堆栈；调用的堆栈包含完整的类名，所执行的方法，如果可能的话还有源代码的行数。
- JVM中的许多问题都可以使用线程dump文件来进行诊断，其中比较典型的包括线程阻塞，CPU使用率过高，JVM Crash，堆内存不足和类装载等问题。

# 导出

#### 1、查询java进程pid

- 使用jps [-l]命令查看本机所有java进程pid



```css
jps [-l]
```

![img](https://upload-images.jianshu.io/upload_images/4250933-90cc0c5e200fc9bb.png?imageMogr2/auto-orient/strip|imageView2/2/w/1172/format/webp)

image.png

#### 2、使用top查看目前正在运行的进程使用系统资源情况



```undefined
top
```

![img](https://upload-images.jianshu.io/upload_images/4250933-6b288f693d9601f5.png?imageMogr2/auto-orient/strip|imageView2/2/w/850/format/webp)

image.png



![img](https://upload-images.jianshu.io/upload_images/4250933-850e8d0fea7dcde4.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

image.png

当前占用cpu最高26.5%的进程为27796的java程序

#### 3、导出指定进程pid所有线程信息

- 使用jstack将所有线程信息导出到指定文件中([jstack了解传送门](https://docs.oracle.com/javase/6/docs/technotes/tools/share/jstack.html))

1)将所有线程信息输入到指定文件中



```css
jstack [-l] pid > xxx.log
```

![img](https://upload-images.jianshu.io/upload_images/4250933-2106c58f9a385134.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

image.png

2)-F （当'jstack [-l] pid'没有响应，强制导出堆栈dump）



```css
jstack -F [-m] [-l] pid >xxx.log
```

#### 4、分析

###### 线程状态介绍

- 死锁，Deadlock（重点关注） :一般指多个线程调用间，进入相互资源占用，导致一直等待无法释放的情况。

- 执行中，Runnable :一般指该线程正在执行状态中，该线程占用了资源，正在处理某个请求，有可能正在传递SQL到数据库执行，有可能在对某个文件操作，有可能进行数据类型等转换。

- 等待资源，Waiting on condition（重点关注） :等待资源，或等待某个条件的发生。具体原因需结合 stacktrace来分析。
    1、如果堆栈信息明确是应用代码，则证明该线程正在等待资源。一般是大量读取某资源，且该资源采用了资源锁的情况下，线程进入等待状态，等待资源的读取。
  又或者，正在等待其他线程的执行等。
    2、如果发现有大量的线程都在处在 Wait on condition，从线程 stack看，正等待网络读写，这可能是一个网络瓶颈的征兆。因为网络阻塞导致线程无法执行。
      2.1、一种情况是网络非常忙，几乎消耗了所有的带宽，仍然有大量数据等待网络读写；
      2.2、另一种情况也可能是网络空闲，但由于路由等问题，导致包无法正常的到达。
    3、另外一种出现 Wait on condition的常见情况是该线程在 sleep，等待 sleep的时间到了时候，将被唤醒。

- 等待获取监视器，Waiting on monitor entry（重点关注）

- 对象等待中，Object.wait() 或 TIMED_WAITING
    Waiting for monitor entry 和 in Object.wait()：
    Monitor([Monitor的深入理解传送门](http://www.cnblogs.com/tomsheep/archive/2010/06/09/1754419.html))是 Java中用以实现线程之间的互斥与协作的主要手段，它可以看成是对象或者 Class的锁。每一个对象都有，也仅有一个 monitor。
    从下图中可以看出，每个 Monitor在某个时刻，只能被一个线程拥有，该线程就是 “Active Thread”，而其它线程都是 “Waiting Thread”，分别在两个队列 “ Entry Set”和 “Wait Set”里面等候。
    在 “Entry Set”中等待的线程状态是 “Waiting for monitor entry”，而在 “Wait Set”中等待的线程状态是 “in Object.wait()”

  ![img](https://upload-images.jianshu.io/upload_images/4250933-da4015d034d6806f.png?imageMogr2/auto-orient/strip|imageView2/2/w/1162/format/webp)

  Java Monitor

  

- 暂停，Suspended
- 阻塞，Blocked（重点关注） ：是指当前线程执行过程中，所需要的资源长时间等待却一直未能获取到，被容器的线程管理器标识为阻塞状态，可以理解为等待资源超时的线程。
- 停止，Parked

###### jvm_27796.log展示

![img](https://upload-images.jianshu.io/upload_images/4250933-f8387e32b9ebe219.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

image.png

###### stack trace实例分析



```dart
"consumer_redirectUrl_topic_jmq206_1546013217302" daemon prio=10 tid=0x00007f1bf03f6800 nid=0x693e waiting on condition [0x00007f1b38388000]
   java.lang.Thread.State: TIMED_WAITING (parking)
    at sun.misc.Unsafe.park(Native Method)
    - parking to wait for  <0x00000000f76e21a0> (a java.util.concurrent.CountDownLatch$Sync)
    at java.util.concurrent.locks.LockSupport.parkNanos(LockSupport.java:226)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer.doAcquireSharedNanos(AbstractQueuedSynchronizer.java:1033)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer.tryAcquireSharedNanos(AbstractQueuedSynchronizer.java:1326)
    at java.util.concurrent.CountDownLatch.await(CountDownLatch.java:282)
    at com.jd.jmq.common.network.netty.ResponseFuture.await(ResponseFuture.java:133)
    at com.jd.jmq.common.network.netty.NettyTransport.sync(NettyTransport.java:241)
    at com.jd.jmq.common.network.netty.failover.FailoverNettyClient.sync(FailoverNettyClient.java:94)
    at com.jd.jmq.client.consumer.GroupConsumer.pull(GroupConsumer.java:246)
    at com.jd.jmq.client.consumer.GroupConsumer$QueueConsumer.run(GroupConsumer.java:445)
    at java.lang.Thread.run(Thread.java:745)

   Locked ownable synchronizers:
    - None
```

- 线程名：consumer_redirectUrl_topic_jmq206_1546013217302
- 线程优先级：prio=10
- java线程的identifier：tid=0x00007f1bf03f6800
- native线程的identifier：nid=0x693e
- 线程的状态：waiting on condition [0x00007f1b38388000]
  java.lang.Thread.State: TIMED_WAITING (parking)
- 线程栈起始地址：[0x00007f1b38388000]

###### 找出某进程中要分析的线程ID



```xml
top -H -p <pid>
```

![img](https://upload-images.jianshu.io/upload_images/4250933-6f666e43eed1e8a0.png?imageMogr2/auto-orient/strip|imageView2/2/w/1126/format/webp)

image.png



![img](https://upload-images.jianshu.io/upload_images/4250933-0951e857b916c93a.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

image.png

###### 将线程ID转换为16进制后，在线程dump文件中搜索相关信息

例如：27840==》6cc0



![img](https://upload-images.jianshu.io/upload_images/4250933-cf0e092b312dda7a.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

image.png



```bash
"System_Clock" daemon prio=10 tid=0x00007f1c2cbc6800 nid=0x6cc0 runnable [0x00007f1c24872000]
   java.lang.Thread.State: TIMED_WAITING (parking)
    at sun.misc.Unsafe.park(Native Method)
    - parking to wait for  <0x00000000c0c9d918> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
    at java.util.concurrent.locks.LockSupport.parkNanos(LockSupport.java:226)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.awaitNanos(AbstractQueuedSynchronizer.java:2082)
    at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(ScheduledThreadPoolExecutor.java:1090)
    at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(ScheduledThreadPoolExecutor.java:807)
    at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1068)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1130)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
    at java.lang.Thread.run(Thread.java:745)

   Locked ownable synchronizers:
    - None
```