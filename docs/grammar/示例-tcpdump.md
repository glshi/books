## [tcpdump详细讲解](https://baijiahao.baidu.com/s?id=1671144485218215170&wfr=spider&for=pc)

## 1、tcpdump 核心参数图解

大家都知道，网络上的流量、数据包，非常的多，因此要想抓到我们所需要的数据包，就需要我们定义一个精准的过滤器，把这些目标数据包，从巨大的数据包网络中抓取出来。

所以学习抓包工具，其实就是学习如何定义过滤器的过程。

而在 tcpdump 的世界里，过滤器的实现，都是通过一个又一个的参数组合起来，一个参数不够精准，那就再加一个，直到我们能过滤掉无用的数据包，只留下我们感兴趣的数据包。

tcpdump 的参数非常的多，初学者在没有掌握 tcpdump 时，会对这个命令的众多参数产生很多的疑惑。

就比如下面这个命令，我们要通过 host 参数指定 host ip 进行过滤

$ tcpdump host 192.168.10.100

主程序 + 参数名+ 参数值  这样的组合才是我们正常认知里面命令行该有的样子。

可 tcpdump 却不走寻常路，我们居然还可以在 host 前再加一个限定词，来缩小过滤的范围？

$ tcpdump src host 192.168.10.100

从字面上理解，确实很容易理解，但是这不符合编写命令行程序的正常逻辑，导致我们会有所疑虑：

除了 src ，dst，可还有其它可以用的限定词？src，host 应该如何理解它们，叫参数名？不合适，因为 src 明显不合适。如果你在网上看到有关 tcpdump 的博客、教程，无一不是给你一个参数组合，告诉你这是实现了怎样的一个过滤器？这样的教学方式，很容易让你依赖别人的文章来使用 tcpdump，而不能将 tcpdump 这样神器消化，达到灵活应用，灵活搭配过滤器的效果。

上面加了 src 本身就颠覆了我们的认知，你可知道在 src 之前还可以加更多的条件，比如 tcp, udp, icmp 等词，在你之前的基础上再过滤一层。

$ tcpdump tcp src host 192.168.10.100

这种参数的不确定性，让大多数人对 tcpdump 的学习始终无法得其精髓。

因此，在学习 tcpdump 之前，我觉得有必要要先让你知道：tcpdump 的参数是如何组成的？这非常重要。

为此，我画了一张图，方便你直观的理解 tcpdump 的各种参数：

![img](https://pics1.baidu.com/feed/908fa0ec08fa513ddc9fa0046821bcfdb3fbd983.png?token=e7392cf49539920399c00ac832b70ff7)

option 可选参数：将在后边一一解释，对应本文 第四节：可选参数解析proto 类过滤器：根据协议进行过滤，可识别的关键词有：upd, udp, icmp, ip, ip6, arp, rarp,ether,wlan, fddi, tr, decnettype 类过滤器：可识别的关键词有：host, net, port, portrange，这些词后边需要再接参数。direction 类过滤器：根据数据流向进行过滤，可识别的关键字有：src, dst，同时你可以使用逻辑运算符进行组合，比如 src or dstproto、type、direction 这三类过滤器的内容比较简单，也最常用，因此我将其放在最前面，也就是 第三节：常规过滤规则一起介绍。

而 option 可选的参数非常多，有的甚至也不经常用到，因此我将其放到后面一点，也就是 第四节：可选参数解析

当你看完前面六节，你对 tcpdump 的认识会上了一个台阶，至少能够满足你 80% 的使用需求。

你一定会问了，还有 20% 呢？

其实 tcpdump 还有一些过滤关键词，它不符合以上四种过滤规则，可能需要你单独记忆。关于这部分我会在  第六节：特殊过滤规则  里进行介绍。

## 2、理解 tcpdump 的输出

2.1 输出内容结构

tcpdump 输出的内容虽然多，却很规律。

这里以我随便抓取的一个 tcp 包为例来看一下

21:26:49.013621 IP 172.20.20.1.15605 > 172.20.20.2.5920: Flags [P.], seq 49:97, ack 106048, win 4723, length 48

从上面的输出来看，可以总结出：

第一列：时分秒毫秒 21:26:49.013621第二列：网络协议 IP第三列：发送方的ip地址+端口号，其中172.20.20.1是 ip，而15605 是端口号第四列：箭头 >， 表示数据流向第五列：接收方的ip地址+端口号，其中 172.20.20.2 是 ip，而5920 是端口号第六列：冒号第七列：数据包内容，包括Flags 标识符，seq 号，ack 号，win 窗口，数据长度 length，其中 [P.] 表示 PUSH 标志位为 1，更多标识符见下面2.2 Flags 标识符

使用 tcpdump 抓包后，会遇到的 TCP 报文 Flags，有以下几种：

[S] : SYN（开始连接）[P] : PSH（推送数据）[F] : FIN （结束连接）[R] : RST（重置连接）[.] : 没有 Flag，由于除了 SYN 包外所有的数据包都有ACK，所以一般这个标志也可表示 ACK



## 3、常规过滤规则

3.1 基于IP地址过滤：host

使用 host 就可以指定 host ip 进行过滤

$ tcpdump host 192.168.10.100

数据包的 ip 可以再细分为源ip和目标ip两种

\# 根据源ip进行过滤$ tcpdump -i eth2 src 192.168.10.100# 根据目标ip进行过滤$ tcpdump -i eth2 dst 192.168.10.200

3.2 基于网段进行过滤：net

若你的ip范围是一个网段，可以直接这样指定

$ tcpdump net 192.168.10.0/24

网段同样可以再细分为源网段和目标网段

\# 根据源网段进行过滤$ tcpdump src net 192.168# 根据目标网段进行过滤$ tcpdump dst net 192.168

3.3 基于端口进行过滤：port

使用 port 就可以指定特定端口进行过滤

$ tcpdump port 8088

端口同样可以再细分为源端口，目标端口

\# 根据源端口进行过滤$ tcpdump src port 8088# 根据目标端口进行过滤$ tcpdump dst port 8088

如果你想要同时指定两个端口你可以这样写

$ tcpdump port 80 or port 8088

但也可以简写成这样

$ tcpdump port 80 or 8088

如果你的想抓取的不再是一两个端口，而是一个范围，一个一个指定就非常麻烦了，此时你可以这样指定一个端口段。

$ tcpdump portrange 8000-8080$ tcpdump src portrange 8000-8080$ tcpdump dst portrange 8000-8080

对于一些常见协议的默认端口，我们还可以直接使用协议名，而不用具体的端口号

比如 http  == 80，https == 443 等

$ tcpdump tcp port http

3.4 基于协议进行过滤：proto

常见的网络协议有：tcp, udp, icmp, http, ip,ipv6 等

若你只想查看 icmp 的包，可以直接这样写

$ tcpdump icmp

protocol 可选值：ip, ip6, arp, rarp, atalk, aarp, decnet, sca, lat, mopdl,  moprc,  iso,  stp, ipx,  or  netbeui

3.5 基本IP协议的版本进行过滤

当你想查看 tcp 的包，你也许会这样子写

$ tcpdump tcp

这样子写也没问题，就是不够精准，为什么这么说呢？

ip 根据版本的不同，可以再细分为 IPv4 和 IPv6 两种，如果你只指定了 tcp，这两种其实都会包含在内。

那有什么办法，能够将 IPv4 和 IPv6 区分开来呢？

很简单，如果是 IPv4 的 tcp 包 ，就这样写（友情提示：数字 6 表示的是 tcp 在ip报文中的编号。）

$ tcpdump 'ip proto tcp'# or$ tcpdump ip proto 6# or$ tcpdump 'ip protochain tcp'# or $ tcpdump ip protochain 6

而如果是 IPv6 的 tcp 包 ，就这样写

$ tcpdump 'ip6 proto tcp'# or$ tcpdump ip6 proto 6# or$ tcpdump 'ip6 protochain tcp'# or $ tcpdump ip6 protochain 6

关于上面这几个命令示例，有两点需要注意：

跟在 proto 和 protochain 后面的如果是 tcp, udp, icmp ，那么过滤器需要用引号包含，这是因为 tcp,udp, icmp 是 tcpdump 的关键字。跟在ip 和 ip6 关键字后面的 proto 和 protochain 是两个新面孔，看起来用法类似，它们是否等价，又有什么区别呢？关于第二点，网络上没有找到很具体的答案，我只能通过 man tcpdump 的提示， 给出自己的个人猜测，但不保证正确。

proto 后面跟的 <protocol> 的关键词是固定的，只能是 ip, ip6, arp, rarp, atalk, aarp,  decnet, sca, lat, mopdl,  moprc,  iso,  stp, ipx,  or  netbeui 这里面的其中一个。

而 protochain 后面跟的 protocol 要求就没有那么严格，它可以是任意词，只要 tcpdump 的 IP 报文头部里的 protocol 字段为 <protocol> 就能匹配上。

理论上来讲，下面两种写法效果是一样的

$ tcpdump 'ip && tcp'$ tcpdump 'ip proto tcp'

同样的，这两种写法也是一样的

$ tcpdump 'ip6 && tcp'$ tcpdump 'ip6 proto tcp'



## 4、可选参数解析

4.1 设置不解析域名提升速度

-n：不把ip转化成域名，直接显示  ip，避免执行 DNS lookups 的过程，速度会快很多-nn：不把协议和端口号转化成名字，速度也会快很多。-N：不打印出host 的域名部分.。比如,，如果设置了此选现，tcpdump 将会打印'nic' 而不是 'nic.ddn.mil'.4.2 过滤结果输出到文件

使用 tcpdump 工具抓到包后，往往需要再借助其他的工具进行分析，比如常见的 wireshark 。

而要使用wireshark ，我们得将 tcpdump 抓到的包数据生成到文件中，最后再使用 wireshark 打开它即可。

使用 -w 参数后接一个以 .pcap 后缀命令的文件名，就可以将 tcpdump 抓到的数据保存到文件中。

$ tcpdump icmp -w icmp.pcap

4.3 从文件中读取包数据

使用 -w 是写入数据到文件，而使用 -r 是从文件中读取数据。

读取后，我们照样可以使用上述的过滤器语法进行过滤分析。

$ tcpdump icmp -r all.pcap

4.4 控制详细内容的输出

-v：产生详细的输出. 比如包的TTL，id标识，数据包长度，以及IP包的一些选项。同时它还会打开一些附加的包完整性检测，比如对IP或ICMP包头部的校验和。-vv：产生比-v更详细的输出. 比如NFS回应包中的附加域将会被打印, SMB数据包也会被完全解码。（摘自网络，目前我还未使用过）-vvv：产生比-vv更详细的输出。比如 telent 时所使用的SB, SE 选项将会被打印, 如果telnet同时使用的是图形界面，其相应的图形选项将会以16进制的方式打印出来（摘自网络，目前我还未使用过）4.5 控制时间的显示

-t：在每行的输出中不输出时间-tt：在每行的输出中会输出时间戳-ttt：输出每两行打印的时间间隔(以毫秒为单位)-tttt：在每行打印的时间戳之前添加日期的打印（此种选项，输出的时间最直观）4.6 显示数据包的头部

-x：以16进制的形式打印每个包的头部数据（但不包括数据链路层的头部）-xx：以16进制的形式打印每个包的头部数据（包括数据链路层的头部）-X：以16进制和 ASCII码形式打印出每个包的数据(但不包括连接层的头部)，这在分析一些新协议的数据包很方便。-XX：以16进制和 ASCII码形式打印出每个包的数据(包括连接层的头部)，这在分析一些新协议的数据包很方便。4.7 过滤指定网卡的数据包

-i：指定要过滤的网卡接口，如果要查看所有网卡，可以 -i any4.8 过滤特定流向的数据包

-Q：选择是入方向还是出方向的数据包，可选项有：in, out, inout，也可以使用  --direction=[direction] 这种写法4.9 其他常用的一些参数

-A：以ASCII码方式显示每一个数据包(不显示链路层头部信息). 在抓取包含网页数据的数据包时, 可方便查看数据-l : 基于行的输出，便于你保存查看，或者交给其它工具分析-q : 简洁地打印输出。即打印很少的协议相关信息, 从而输出行都比较简短.-c : 捕获 count 个包 tcpdump 就退出-s :  tcpdump 默认只会截取前 96 字节的内容，要想截取所有的报文内容，可以使用 -s number， number 就是你要截取的报文字节数，如果是 0 的话，表示截取报文全部内容。-S : 使用绝对序列号，而不是相对序列号-C：file-size，tcpdump 在把原始数据包直接保存到文件中之前, 检查此文件大小是否超过file-size. 如果超过了,  将关闭此文件,另创一个文件继续用于原始数据包的记录. 新创建的文件名与-w 选项指定的文件名一致,  但文件名后多了一个数字.该数字会从1开始随着新创建文件的增多而增加. file-size的单位是百万字节(nt:  这里指1,000,000个字节,并非1,048,576个字节, 后者是以1024字节为1k, 1024k字节为1M计算所得, 即1M=1024 ＊ 1024 ＝ 1,048,576)-F：使用file 文件作为过滤条件表达式的输入, 此时命令行上的输入将被忽略.4.10 对输出内容进行控制的参数

-D : 显示所有可用网络接口的列表-e : 每行的打印输出中将包括数据包的数据链路层头部信息-E : 揭秘IPSEC数据-L ：列出指定网络接口所支持的数据链路层的类型后退出-Z：后接用户名，在抓包时会受到权限的限制。如果以root用户启动tcpdump，tcpdump将会有超级用户权限。-d：打印出易读的包匹配码-dd：以C语言的形式打印出包匹配码.-ddd：以十进制数的形式打印出包匹配码

## 5、过滤规则组合

有编程基础的同学，对于下面三个逻辑运算符应该不陌生了吧

and：所有的条件都需要满足，也可以表示为 &&or：只要有一个条件满足就可以，也可以表示为 ||not：取反，也可以使用 !举个例子，我想需要抓一个来自10.5.2.3，发往任意主机的3389端口的包

$ tcpdump src 10.5.2.3 and dst port 3389

当你在使用多个过滤器进行组合时，有可能需要用到括号，而括号在 shell 中是特殊符号，因为你需要使用引号将其包含。例子如下：

$ tcpdump 'src 10.0.2.4 and (dst port 3389 or 22)'

而在单个过滤器里，常常会判断一条件是否成立，这时候，就要使用下面两个符号

=：判断二者相等==：判断二者相等!=：判断二者不相等当你使用这两个符号时，tcpdump 还提供了一些关键字的接口来方便我们进行判断，比如

if：表示网卡接口名、proc：表示进程名pid：表示进程 idsvc：表示 service classdir：表示方向，in 和 outeproc：表示 effective process nameepid：表示 effective process ID比如我现在要过滤来自进程名为 nc 发出的流经 en0 网卡的数据包，或者不流经 en0 的入方向数据包，可以这样子写

$ tcpdump "( if=en0 and proc =nc ) || (if != en0 and dir=in)"

**特殊过滤规则**

5.1 根据 tcpflags 进行过滤

通过上一篇文章，我们知道了 tcp 的首部有一个标志位。

![img](https://pics5.baidu.com/feed/21a4462309f790526c2a7b3d5bbf3ecc7bcbd54b.png?token=61e9cbe7ec7eb764b642aa55d66eb36c)

TCP 报文首部

tcpdump 支持我们根据数据包的标志位进行过滤

proto [ expr:size ]

proto：可以是熟知的协议之一（如ip，arp，tcp，udp，icmp，ipv6）expr：可以是数值，也可以是一个表达式，表示与指定的协议头开始处的字节偏移量。size：是可选的，表示从字节偏移量开始取的字节数量。接下来，我将举几个例子，让人明白它的写法，不过在那之前，有几个点需要你明白，这在后面的例子中会用到：

1、tcpflags 可以理解为是一个别名常量，相当于 13，它代表着与指定的协议头开头相关的字节偏移量，也就是标志位，所以 tcp[tcpflags] 等价于 tcp[13] ，对应下图中的报文位置。

![img](https://pics2.baidu.com/feed/4e4a20a4462309f729c9da8c2a42e5f5d6cad659.png?token=91edebd753127aa088f914da16b66162)

2、tcp-fin, tcp-syn, tcp-rst, tcp-push, tcp-ack, tcp-urg 这些同样可以理解为别名常量，分别代表 1，2，4，8，16，32，64。这些数字是如何计算出来的呢？

以 tcp-syn 为例，你可以参照下面这张图，计算出来的值 是就是 2

![img](https://pics7.baidu.com/feed/09fa513d269759eedc1f418fe7b7aa106c22dffe.png?token=7a8b5395c8859a01f54e28e450b40b6a)

由于数字不好记忆，所以一般使用这样的“别名常量”表示。

因此当下面这个表达式成立时，就代表这个包是一个 syn 包。

tcp[tcpflags] == tcp-syn

要抓取特定数据包，方法有很多种。

下面以最常见的 syn包为例，演示一下如何用 tcpdump 抓取到 syn 包，而其他的类型的包也是同样的道理。

据我总结，主要有三种写法：

1、第一种写法：使用数字表示偏移量

$ tcpdump -i eth0 "tcp[13] & 2 != 0"

2、第二种写法：使用别名常量表示偏移量

$ tcpdump -i eth0 "tcp[tcpflags] & tcp-syn != 0"

3、第三种写法：使用混合写法

$ tcpdump -i eth0 "tcp[tcpflags] & 2 != 0"# or$ tcpdump -i eth0 "tcp[13] & tcp-syn != 0"

如果我想同时捕获多种类型的包呢，比如 syn + ack 包

1、第一种写法

$ tcpdump -i eth0 'tcp[13] == 2 or tcp[13] == 16'

2、第二种写法

$ tcpdump -i eth0 'tcp[tcpflags] == tcp-syn or tcp[tcpflags] == tcp-ack'

3、第三种写法

$ tcpdump -i eth0 "tcp[tcpflags] & (tcp-syn|tcp-ack) != 0"

4、第四种写法：注意这里是 单个等号，而不是像上面一样两个等号，18（syn+ack） = 2（syn） + 16（ack）

$ tcpdump -i eth0 'tcp[13] = 18'# or$ tcpdump -i eth0 'tcp[tcpflags] = 18'

tcp 中有 类似 tcp-syn 的别名常量，其他协议也是有的，比如 icmp 协议，可以使用的别名常量有

icmp-echoreply, icmp-unreach, icmp-sourcequench, icmp-redirect, icmp-echo, icmp-routeradvert,icmp-routersolicit, icmp-timx-ceed, icmp-paramprob, icmp-tstamp, icmp-tstampreply,icmp-ireq, icmp-ireqreply, icmp-maskreq, icmp-maskreply

5.2  基于包大小进行过滤

若你想查看指定大小的数据包，也是可以的

$ tcpdump less 32 $ tcpdump greater 64 $ tcpdump <= 128

5.3 根据 mac 地址进行过滤

例子如下，其中 ehost 是记录在 /etc/ethers 里的 name

$ tcpdump ether host [ehost]$ tcpdump ether dst    [ehost]$ tcpdump ether src    [ehost]

5.4 过滤通过指定网关的数据包

$ tcpdump gateway [host]

5.5 过滤广播/多播数据包

$ tcpdump ether broadcast$ tcpdump ether multicast$ tcpdump ip broadcast$ tcpdump ip multicast$ tcpdump ip6 multicast

## 6、如何抓取到更精准的包？

先给你抛出一个问题：如果我只想抓取 HTTP 的 POST 请求该如何写呢？

如果只学习了上面的内容，恐怕你还是无法写法满足这个抓取需求的过滤器。

在学习之前，我先给出答案，然后再剖析一下，这个过滤器是如何生效的，居然能让我们对包内的内容进行判断。

$ tcpdump -s 0 -A -vv 'tcp[((tcp[12:1] & 0xf0) >> 2):4]'

命令里的可选参数，在前面的内容里已经详细讲过了。这里不再细讲。

本节的重点是引号里的内容，看起来很复杂的样子。

将它逐一分解，我们只要先理解了下面几种用法，就能明白

tcp[n]：表示 tcp 报文里 第 n 个字节tcp[n:c]：表示 tcp 报文里从第n个字节开始取 c 个字节，tcp[12:1]  表示从报文的第12个字节（因为有第0个字节，所以这里的12其实表示的是13）开始算起取一个字节，也就是 8 个bit。查看 tcp  的报文首部结构，可以得知这 8 个bit 其实就是下图中的红框圈起来的位置，而在这里我们只要前面  4个bit，也就是实际数据在整个报文首部中的偏移量。

![img](https://pics0.baidu.com/feed/5d6034a85edf8db187405da75c6f3452574e745d.png?token=5ce942613f5110d22e02323fb2684410)

&：是位运算里的 and 操作符，比如 0011 & 0010 = 0010，>>：是位运算里的右移操作，比如 0111 >> 2 = 0011，0xf0：是 10 进制的 240 的 16  进制表示，但对于位操作来说，10进制和16进制都将毫无意义，我们需要的是二进制，将其转换成二进制后是：11110000，这个数有什么特点呢？前面个 4bit 全部是 1，后面4个bit全部是0，往后看你就知道这个特点有什么用了。分解完后，再慢慢合并起来看

1、tcp[12:1] & 0xf0 其实并不直观，但是我们将它换一种写法，就好看多了，假设 tcp 报文中的 第12 个字节是这样组成的  10110000，那么这个表达式就可以变成 10110110 && 11110000 = 10110000，得到了  10110000 后，再进入下一步。

2、tcp[12:1] & 0xf0) >> 2 ：如果你不理解 tcp 报文首部里的数据偏移，请先点击这个前往我的上一篇文章，搞懂数据偏移的意义，否则我保证你这里会绝对会听懵了。

tcp[12:1] & 0xf0) >> 2 这个表达式实际是 (tcp[12:1] & 0xf0) >> 4 )  << 2 的简写形式。所以要搞懂 tcp[12:1] & 0xf0) >> 2 只要理解了(tcp[12:1]  & 0xf0) >> 4 ) << 2  就行了 。

从上一步我们算出了 tcp[12:1] & 0xf0  的值其实是一个字节，也就是 8 个bit，但是你再回去看下上面的 tcp  报文首部结构图，表示数据偏移量的只有 4个bit，也就是说 上面得到的值 10110000，前面 4  位（1011）才是正确的偏移量，那么为了得到 1011，只需要将 10110000 右移4位即可，也就是 tcp[12:1] &  0xf0) >> 4，至此我们是不是已经得出了实际数据的正确位置呢，很遗憾还没有，前一篇文章里我们讲到 Data Offset  的单位是 4个字节，因为要将 1011 乘以 4才可以，除以4在位运算中相当于左移2位，也就是 <<2，与前面的 >>4 结合起来一起算的话，最终的运算可以简化为 >>2

至此，我们终于得出了实际数据开始的位置是 tcp[12:1] & 0xf0) >> 2 （单位是字节）。

找到了数据的起点后，可别忘了我们的目的是从数据中打到 HTTP 请求的方法，是 GET 呢 还是 POST ，或者是其他的？

有了上面的经验，我们自然懂得使用 tcp[((tcp[12:1] & 0xf0) >> 2):4] 从数据开始的位置再取出四个字节，然后将结果与 GET  （注意 GET最后还有个空格）的 16进制写法（也就是 0x47455420）进行比对。

0x47   -->   71    -->  G0x45   -->   69    -->  E0x54   -->   84    -->  T0x20   -->   32    -->  空格

![img](https://pics2.baidu.com/feed/d01373f082025aaf5945f930a3a14262024f1aad.png?token=d45afa034aa2c7f03b2c40116e898ee7)

如果相等，则该表达式为True，tcpdump 认为这就是我们所需要抓的数据包，将其输出到我们的终端屏幕上。

## 7、抓包实战应用例子

以下例子摘自：https://fuckcloudnative.io/posts/tcpdump-examples/

7.1 提取 HTTP 的 User-Agent

从 HTTP 请求头中提取 HTTP 用户代理：

$ tcpdump -nn -A -s1500 -l | grep "User-Agent:"

通过 egrep 可以同时提取用户代理和主机名（或其他头文件）：

$ tcpdump -nn -A -s1500 -l | egrep -i 'User-Agent:|Host:'

7.2 抓取 HTTP GET 和 POST 请求

抓取 HTTP GET 请求包：

$ tcpdump -s 0 -A -vv 'tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x47455420'# or$ tcpdump -vvAls0 | grep 'GET'

可以抓取 HTTP POST 请求包：

$ tcpdump -s 0 -A -vv 'tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x504f5354'# or $ tcpdump -vvAls0 | grep 'POST'

注意：该方法不能保证抓取到 HTTP POST 有效数据流量，因为一个 POST 请求会被分割为多个 TCP 数据包。

7.3 找出发包数最多的 IP

找出一段时间内发包最多的 IP，或者从一堆报文中找出发包最多的 IP，可以使用下面的命令：

$ tcpdump -nnn -t -c 200 | cut -f 1,2,3,4 -d '.' | sort | uniq -c | sort -nr | head -n 20

cut -f 1,2,3,4 -d '.' : 以 . 为分隔符，打印出每行的前四列。即 IP 地址。sort | uniq -c : 排序并计数sort -nr : 按照数值大小逆向排序8.4 抓取 DNS 请求和响应

DNS 的默认端口是 53，因此可以通过端口进行过滤

$ tcpdump -i any -s0 port 53

7.5 切割 pcap 文件

当抓取大量数据并写入文件时，可以自动切割为多个大小相同的文件。例如，下面的命令表示每 3600 秒创建一个新文件 capture-(hour).pcap，每个文件大小不超过 200*1000000 字节：

$ tcpdump  -w /tmp/capture-%H.pcap -G 3600 -C 200

这些文件的命名为 capture-{1-24}.pcap，24 小时之后，之前的文件就会被覆盖。

7.6 提取 HTTP POST 请求中的密码

从 HTTP POST 请求中提取密码和主机名：

$ tcpdump -s 0 -A -n -l | egrep -i "POST /|pwd=|passwd=|password=|Host:"

7.7 提取 HTTP 请求的 URL

提取 HTTP 请求的主机名和路径：

$ tcpdump -s 0 -v -n -l | egrep -i "POST /|GET /|Host:"

7.8 抓取 HTTP 有效数据包

抓取 80 端口的 HTTP 有效数据包，排除 TCP 连接建立过程的数据包（SYN / FIN / ACK）：

$ tcpdump 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'

7.9 结合 Wireshark 进行分析

通常 Wireshark（或 tshark）比 tcpdump 更容易分析应用层协议。一般的做法是在远程服务器上先使用 tcpdump 抓取数据并写入文件，然后再将文件拷贝到本地工作站上用 Wireshark 分析。

还有一种更高效的方法，可以通过 ssh 连接将抓取到的数据实时发送给 Wireshark 进行分析。以 MacOS 系统为例，可以通过 brew cask install wireshark 来安装，然后通过下面的命令来分析：

$ ssh root@remotesystem 'tcpdump -s0 -c 1000 -nn -w - not port 22' |  /Applications/Wireshark.app/Contents/MacOS/Wireshark -k -i -

例如，如果想分析 DNS 协议，可以使用下面的命令：

$ ssh root@remotesystem 'tcpdump -s0 -c 1000 -nn -w - port 53' | /Applications/Wireshark.app/Contents/MacOS/Wireshark -k -i -

抓取到的数据：

![img](https://pics6.baidu.com/feed/7aec54e736d12f2e94e9887e168e3c6484356896.png?token=b0cf725a1db52c348a717c325e0b22c5)

-c 选项用来限制抓取数据的大小。如果不限制大小，就只能通过 ctrl-c 来停止抓取，这样一来不仅关闭了 tcpdump，也关闭了 wireshark。

到这里，我已经将我所知道的 tcpdump 的用法全部说了一遍，如果你有认真地看完本文，相信会有不小的收获，掌握一个上手的抓包工具，对于以后我们学习网络、分析网络协议、以及定位网络问题，会很有帮助，而 tcpdump 是我推荐的一个抓包工具。

## [tcpdump高级过滤](https://www.cnblogs.com/jiujuan/p/9017495.html)

## 一：查看帮助选项

```
tcpdump --help

Usage: tcpdump [-aAbdDefhHIJKlLnNOpqStuUvxX#] [ -B size ] [ -c count ]
[ -C file_size ] [ -E algo:secret ] [ -F file ] [ -G seconds ]
[ -i interface ] [ -j tstamptype ] [ -M secret ] [ --number ]
[ -Q|-P in|out|inout ]
[ -r file ] [ -s snaplen ] [ --time-stamp-precision precision ]
[ --immediate-mode ] [ -T type ] [ --version ] [ -V file ]
[ -w file ] [ -W filecount ] [ -y datalinktype ] [ -z postrotate-command ]
[ -Z user ] [ expression ]
```

 **选项的解释:**

```
-a：尝试将网络和广播地址转换成名称；
-c<数据包数目>：收到指定的数据包数目后，就停止进行倾倒操作；
-d：把编译过的数据包编码转换成可阅读的格式，并倾倒到标准输出；
-dd：把编译过的数据包编码转换成C语言的格式，并倾倒到标准输出；
-ddd：把编译过的数据包编码转换成十进制数字的格式，并倾倒到标准输出；
-e：在每列倾倒资料上显示连接层级的文件头；
-f：用数字显示网际网络地址；
-F<表达文件>：指定内含表达方式的文件；
-i<网络界面>：使用指定的网络截面送出数据包；
-l：使用标准输出列的缓冲区；
-n：不把主机的网络地址转换成名字；
-N：不列出域名；
-O：不将数据包编码最佳化；
-p：不让网络界面进入混杂模式；
-q ：快速输出，仅列出少数的传输协议信息；
-r<数据包文件>：从指定的文件读取数据包数据；
-s<数据包大小>：设置每个数据包的大小；
-S：用绝对而非相对数值列出TCP关联数；
-t：在每列倾倒资料上不显示时间戳记；
-tt： 在每列倾倒资料上显示未经格式化的时间戳记；
-T<数据包类型>：强制将表达方式所指定的数据包转译成设置的数据包类型；
-v：详细显示指令执行过程；
-vv：更详细显示指令执行过程；
-x：用十六进制字码列出数据包资料；
-w<数据包文件>：把数据包数据写入指定的文件。
```

注意：查看更多的信息，可以用命令：man tcpdump
或者网址：https://www.tcpdump.org/tcpdump_man.html

## 二：用法

1：直接启动 tcpdump 将监视第一个网络接口所有流过的数据包

```
tcpdump
```

2：监控某一网络接口的数据包

```
tcpdump -i enp0s3
```

 3：过滤主机

3.1 抓取所有经过enp0s3，目的或源地址是 192.168.1.101 的网络数据

```
tcpdump -i enp0s3 host 192.168.1.101 
```

 3.2 指定源地址

```
tcpdump -i enp0s3 src host 192.168.1.101
```

3.3 指定目的地址

```
tcpdump -i enp0s3 dst host 192.168.1.101
```

3.4 截获主机192.168.1.101 和主机192.168.1.102 或192.168.1.103的通信

```
tcpdump -i enp0s3 host 192.168.1.101 and \(192.168.1.102 or 192.168.1.103 \)
```

3.5 如果想要获取主机192.168.1.101除了和主机192.168.1.102之外所有主机通信的ip包，使用命令：

```
tcpdump ip host 192.168.1.101 and !192.168.1.102
```

 4：过滤端口

4.1 抓取所有经过 enp0s3，目的或源端口是22的网络数据

```
tcpdump -i enp0s3 port 22
```

4.2 指定源端口

```
tcpdump -i enp0s3 src port 22
```

4.3 指定目的端口

```
tcpdump -i enp0s3 dst port 22
```

 5：网络过滤

```
tcpdump -i enp0s3 net 192.168
tcpdump -i enp0s3 src net 192.168
tcpdump -i enp0s3 dst net 192.168
```

 6：协议过滤

```
tcpdump -i enp0s3 arp
tcpdump -i enp0s3 ip
tcpdump -i enp0s3 tcp
tcpdump -i enp0s3 udp
tcpdump -i enp0s3 icmp
```

 7：常用表达式

非 : ! or "not" (without the quotes)
且 : && or "and"
或 : || or "or"

7.1： 抓取目的地址是192.168.1.254或192.168.1.200端口是80的TCP数据

```
tcpdump  '((tcp) and (port 80) and ((dst host 192.168.1.254) or (dst host 192.168.1.200)))'
```

当然上也可以像之前的加上指定网卡 -i enp0s3
tcpdump -i enp0s3 '((tcp) and (port 80) and ((dst host 192.168.1.254) or (dst host 192.168.1.200)))'

7.2： 抓取目标MAC地址是00:01:02:03:04:05的ICMP数据

```
tcpdump  '((icmp) and ((ether dst host 00:01:02:03:04:05)))'
```

可以加上具体网卡

7.3：抓取目的网络是192.168，但目的主机不是192.168.1.200的TCP数据

```
tcpdump  '((tcp) and ((dst net 192.168) and (not dst host 192.168.1.200)))'
```

## 三：高级过滤包头

当我们继续之前，必须了解tcp/ip包头的头部信息

```
proto[x:y]          : 过滤从x字节开始的y字节数。比如ip[2:2]过滤出3、4字节（第一字节从0开始排）
proto[x:y] & z = 0  : proto[x:y]和z的与操作为0
proto[x:y] & z !=0  : proto[x:y]和z的与操作不为0
proto[x:y] & z = z  : proto[x:y]和z的与操作为z
proto[x:y] = z      : proto[x:y]等于z
```

操作符：

```
>  : greater 大于
<  : lower 小于
>= : greater or equal 大于或者等于
<= : lower or equal 小于或者等于
=  : equal  等于
!= : different  不等于
```

第一次在这地方看见这个你可能不是很清楚
当然，在深入理解过滤头部包，首先要了解协议头是很重要的

**1：IP头部**

```
0                   1                   2                   3
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|Version|  IHL  |Type of Service|          Total Length         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|         Identification        |Flags|      Fragment Offset    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Time to Live |    Protocol   |         Header Checksum       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                       Source Address                          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Destination Address                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Options                    |    Padding    | <-- optional
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                            DATA ...                           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

我们只考虑IPv4协议

**2：IP 选项设置**

“一般”的IP头是20字节，但IP头有选项设置，不能直接从偏移21字节处读取数据。IP头有个长度字段可以知道头长度是否大于20字节。

```
+-+-+-+-+-+-+-+-+
 |Version|  IHL  |
 +-+-+-+-+-+-+-+-+
```

通常第一个字节的二进制值是：01000101，分成两个部分：
0100 = 4 表示IP版本 0101 = 5 表示IP头32 bit的块数，5 x 32 bits = 160 bits or 20 bytes
如果第一字节第二部分的值大于5，那么表示头有IP选项。

下面介绍两种过滤方法（第一种方法比较操蛋，可忽略）


2.1. 比较第一字节的值是否大于01000101，这可以判断IPv4带IP选项的数据和IPv6的数据。
01000101十进制等于69，计算方法如下（小提示：用计算器更方便）

```
0 : 0  \
1 : 2^6 = 64 \ 第一部分 (IP版本)
0 : 0   /
0 : 0  /
-
0 : 0  \
1 : 2^2 = 4  \ 第二部分 (头长度)
0 : 0   /
1 : 2^0 = 1 /64 + 4 + 1 = 69
```

如果设置了IP选项，那么第一自己是01000110（十进制70），过滤规则：
tcpdump 'ip[0] > 69'
当然可以加上网卡选项：-i enp0s3

IPv6的数据也可以匹配，第二种方法
2.2 位操作

```
0100 0101 : 第一字节的二进制
0000 1111 : 与操作
<=========
0000 0101 : 结果
正确的过滤方法
tcpdump  'ip[0] & 15 > 5'

或者
tcpdump  'ip[0] & 0x0f > 5'
```

我用了16进制掩码.
That's rather simple, if you want to:
\- keep the last 4 bits intact, use 0xf (binary 00001111)
\- keep the first 4 bits intact, use 0xf0 (binary 11110000)

2.3 分片标记 -Exercise: Is DF bit (don't fragment) set?

当发送端的MTU大于到目的路径链路上的MTU时就会被分片
分片信息在IP头的第七和第八字节：

```
Bit 0: 保留，必须是0
Bit 1: (DF) 0 = 可能分片, 1 = 不分片
Bit 2: (MF) 0 = 最后的分片, 1 = 还有分片
Fragment Offset字段只有在分片的时候才使用。
要抓带DF位标记的不分片的包，第七字节的值应该是：
01000000 = 64

tcpdump  'ip[6] = 64'
```

 

2.4 抓分片包

a：匹配MF，分片包

```
tcpdump  'ip[6] = 32'
```

b：匹配分片和最后分片

```
tcpdump  '((ip[6:2] > 0) and (not ip[6] = 64))'
```

测试分片可以用下面命令：
ping -M want -s 3000 192.168.1.101

 

2.5 匹配小于ttl的数据报

TTL字段在第九字节，并且正好是完整的一个字节，TTL最大值是255，二进制为11111111。

可以来验证下，我们试着制定一个特需的ttl长度为 256
$ ping -M want -s 3000 -t 256 192.168.1.200
ping: ttl 256 out of range

TTL 字段：

```
+-+-+-+-+-+-+-+-+
|  Time to Live |
+-+-+-+-+-+-+-+-+
```

在网关可以用下面的命令看看网络中谁在使用traceroute
tcpdump 'ip[8] < 5'

 

**3： 更多的过滤**

tcp报文的基本结构

TCP 头

```
0                   1                   2                   3
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Source Port          |       Destination Port        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        Sequence Number                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Acknowledgment Number                      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Data |       |C|E|U|A|P|R|S|F|                               |
| Offset|  Res. |W|C|R|C|S|S|Y|I|            Window             |
|       |       |R|E|G|K|H|T|N|N|                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|           Checksum            |         Urgent Pointer        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Options                    |    Padding    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                             data                              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

3.1 抓取源端口大于1024的TCP数据包

```
tcpdump 'tcp[0:2] > 1024'
or
tcpdump 'tcp src portrange 1025-65535'
```

3.2 匹配TCP数据包的特殊标记

TCP标记定义在TCP头的第十四个字节

```
+-+-+-+-+-+-+-+-+
|C|E|U|A|P|R|S|F|
|W|C|R|C|S|S|Y|I|
|R|E|G|K|H|T|N|N|
+-+-+-+-+-+-+-+-+
```

在TCP 3次握手中，两个主机是如何交换数据
1、源端发送 SYN
2、目标端口应答 SYN,ACK
3、源端发送 ACK

\- 只抓取SYN包，第十四字节是二进制的00000010，也就是十进制的2

```
tcpdump 'tcp[13] = 2'
```

\- 抓取 SYN,ACK （00010010 or 18）

```
tcpdump 'tcp[13] = 18'
```

\- 抓取SYN或者SYN-ACK

```
tcpdump 'tcp[13] & 2 = 2'
```

我们使用了掩码，它会返回任何事情，当ACK是二进制设置时候
让我们看看下面的例子（SYN-ACK)

```
00010010 : SYN-ACK packet
00000010 : mask (2 in decimal)
==========
00000010 : result (2 in decimal)
```

\- 抓取PSH-ACK

```
tcpdump 'tcp[13] = 24'
```

\- 抓所有包含FIN标记的包（FIN通常和ACK一起，表示幽会完了，回头见）

```
tcpdump 'tcp[13] & 1 = 1'
```

\- 抓取RST

```
tcpdump 'tcp[13] & 4 = 4'
```

 

TCP标记值：

tcp-fin, tcp-syn, tcp-rst, tcp-push, tcp-push, tcp-ack, tcp-urg

\- 抓取TCP标志位

实际上有一个很简单的方法过滤 flags（man pcap-filter and look for tcpflags）

```
tcpdump 'tcp[tcpflags] == tcp-ack'
```

\- 抓取所有的包，用TCP-SYN 或者 TCP-FIN 设置

```
tcpdump 'tcp[tcpflags] & (tcp-syn|tcp-fin) != 0
```

 

下图表示了TCP各状态转换的标记

[![img](https://images2018.cnblogs.com/blog/650581/201805/650581-20180510003225443-782852645.jpg)](https://images2018.cnblogs.com/blog/650581/201805/650581-20180510003225443-782852645.jpg)

```
tcpdump 提供了常用的字段偏移名字：
icmptype (ICMP类型字段)
icmpcode (ICMP符号字段)
tcpflags (TCP标记字段)
```

ICMP类型值有：
icmp-echoreply, icmp-unreach, icmp-sourcequench, icmp-redirect, icmp-echo, icmp-routeradvert, icmp-routersolicit,
icmp-timxceed, icmp-paramprob, icmp-tstamp, icmp-tstampreply, icmp-ireq, icmp-ireqreply, icmp-maskreq, icmp-maskreply

 

4： SMTP 数据过滤

我们将弄一个匹配任意包的过滤，这个包包括 “MAIL”
你可以用网址 http://www.easycalculation.com/ascii-hex.php 把ASCII转化为 十六进制， 也可以用python来转化
$ python -c 'print "MAIL".encode("hex")'
4d41494c

 

所以 “MAIL” 的十六进制是：0x4d41494c
那么规则就是

```
tcpdump '((port 25) and (tcp[20:4] = 0x4d41494c))'
```

这是一个包的例子



```
# tshark -V -i eth0 '((port 25) and (tcp[20:4] = 0x4d41494c))'
Capturing on eth0
Frame 1 (92 bytes on wire, 92 bytes captured)
    Arrival Time: Sep 25, 2007 00:06:10.875424000
    [Time delta from previous packet: 0.000000000 seconds]
    [Time since reference or first frame: 0.000000000 seconds]
    Frame Number: 1
    Packet Length: 92 bytes
    Capture Length: 92 bytes
    [Frame is marked: False]
    [Protocols in frame: eth:ip:tcp:smtp]
Ethernet II, Src: Cisco_X (00:11:5c:X), Dst: 3Com_X (00:04:75:X)
    Destination: 3Com_X (00:04:75:X)
        Address: 3Com_X (00:04:75:X)
        .... ...0 .... .... .... .... = IG bit: Individual address (unicast)
        .... ..0. .... .... .... .... = LG bit: Globally unique address (factory default)
    Source: Cisco_X (00:11:5c:X)
        Address: Cisco_X (00:11:5c:X)
        .... ...0 .... .... .... .... = IG bit: Individual address (unicast)
        .... ..0. .... .... .... .... = LG bit: Globally unique address (factory default)
    Type: IP (0x0800)
Internet Protocol, Src: 62.163.X (62.163.X), Dst: 192.168.X (192.168.X)
    Version: 4
    Header length: 20 bytes
    Differentiated Services Field: 0x00 (DSCP 0x00: Default; ECN: 0x00)
        0000 00.. = Differentiated Services Codepoint: Default (0x00)
        .... ..0. = ECN-Capable Transport (ECT): 0
        .... ...0 = ECN-CE: 0
    Total Length: 78
    Identification: 0x4078 (16504)
    Flags: 0x04 (Don't Fragment)
        0... = Reserved bit: Not set
        .1.. = Don't fragment: Set
        ..0. = More fragments: Not set
    Fragment offset: 0
    Time to live: 118
    Protocol: TCP (0x06)
    Header checksum: 0x08cb [correct]
        [Good: True]
        [Bad : False]
    Source: 62.163.X (62.163.X)
    Destination: 192.168.X (192.168.XX)
Transmission Control Protocol, Src Port: 4760 (4760), Dst Port: smtp (25), Seq: 0, Ack: 0, Len: 38
    Source port: 4760 (4760)
    Destination port: smtp (25)
    Sequence number: 0    (relative sequence number)
    [Next sequence number: 38    (relative sequence number)]
    Acknowledgement number: 0    (relative ack number)
    Header length: 20 bytes
    Flags: 0x18 (PSH, ACK)
        0... .... = Congestion Window Reduced (CWR): Not set
        .0.. .... = ECN-Echo: Not set
        ..0. .... = Urgent: Not set
        ...1 .... = Acknowledgment: Set
        .... 1... = Push: Set
        .... .0.. = Reset: Not set
        .... ..0. = Syn: Not set
        .... ...0 = Fin: Not set
    Window size: 17375
    Checksum: 0x6320 [correct]
        [Good Checksum: True]
        [Bad Checksum: False]
Simple Mail Transfer Protocol
    Command: MAIL FROM:<wguthrie_at_mysickworld--dot--com>\r\n
        Command: MAIL
        Request parameter: FROM:<wguthrie_at_mysickworld--dot--com>
```

 

5： HTTP数据过滤

http请求开始格式
GET / HTTP/1.1\r\n (16 bytes counting the carriage return but not the backslashes !)

“GET ” 十六进制是 47455420

```
tcpdump 'tcp[32:4] = 0x47455420'
```

HTTP数据（从man tcpdump 看到的例子）

打印所有源或目的端口是80, 网络层协议为IPv4, 并且含有数据,而不是SYN,FIN以及ACK-only等不含数据的数据包.(ipv6的版本的表达式可做练习)

```
tcpdump 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'
```



```
         +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
ip[2:2] = |          Total Length         |
            +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

        +-+-+-+-+-+-+-+-+
ip[0] =    |Version|  IHL  |
        +-+-+-+-+-+-+-+-+

              +-+-+-+-+-+-+-+-+
ip[0]&0xf =  |# # # #|  IHL  | <-- that's right, we masked the version bits
              +-+-+-+-+-+-+-+-+     with 0xf or 00001111 in binary

            +-+-+-+-+
            |  Data |
tcp[12] = | Offset|
           |       |
            +-+-+-+-+
```

 6： SSH过滤

我们看看ssh server
OpenSSH 常常应答一些内容，比如"SSH-2.0-OpenSSH_3.6.1p2"， 这第一个4 bytes (SSH-)的十六进制值是 0x5353482D

```
tcpdump 'tcp[(tcp[12]>>2):4] = 0x5353482D'
```

如果我们想要找到老版本的OpenSSH的任意链接
这时候OpenSSH服务器应答的内容：比如 “SSH-1.99..”

```
tcpdump '(tcp[(tcp[12]>>2):4] = 0x5353482D) and (tcp[((tcp[12]>>2)+4):2] = 0x312E)'
```

 7： UDP头

```
0      7 8     15 16    23 24    31  
 +--------+--------+--------+--------+
 |     Source      |   Destination   |
 |      Port       |      Port       |
 +--------+--------+--------+--------+
 |                 |                 |
 |     Length      |    Checksum     |
 +--------+--------+--------+--------+
 |                                   |
 |              DATA ...             |
 +-----------------------------------+
```

如果我们想要过滤，我们可以用下面的方法

```
tcpdump udp dst port 53
```

 

8：ICMP头

这里本来显示一张图片，可是原文图片没有正确显示，如实我找了一个图，如下

```
0                   1                   2                   3
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     Type      |     Code      |          Checksum             |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                             unused                            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|      Internet Header + 64 bits of Original Data Datagram      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

在ICMP报文中，我们经常过滤 type（1 byte）和code（1 byte）

下图是ICMP报文中 Type ，我们经常用到的几个值：

```
0    Echo Reply                 [RFC792]   回显应答报文
3    Destination Unreachable     [RFC792]   目的不可达
4    Source Quench             [RFC792]     源冷却报文
5    Redirect                 [RFC792]      重定向报文
8    Echo                     [RFC792]     请求回显报文
11    Time Exceeded             [RFC792]    超时报文
```

如果我们要过滤报文 type = 4 

```
tcpdump 'icmp[0] = 4'
```

如我我们仅仅是要找到ICMP 的回显 应答报文，同时 ID是500。

```
tcpdump -i eth0 '(icmp[0] = 0) and (icmp[4:2] = 0x1f4)'
```





## wt项目实战

抓取指定端口数据包并保存

```bash
# 获取十六进制数据
tcpdump -s 0 -nn -X -v -i enp0s8 port 26667
tcpdump -s 0 -nn -X -v -i enp0s3 port 26667
# 获取二进制数据
tcpdump -s 0 -nn -A -v -i enp0s8 port 26667
tcpdump -s 0 -nn -A -v -i enp0s8 port 26667 | grep 'logout'
# 按照bid的值进行过滤 -c 1 表示捕获一个包就退出
tcpdump -s 0 -nn -A -v -c 1 '((port 25) and (tcp[31:4] = 0x00010100))' -i eth0 -w client.pcap
tcpdump -s 0 -nn -A -v -c 1 '((port 25) and (udp[19:4] = 0x00010100))' -i eth0 -w client.pcap
# 所以 “MAIL” 的十六进制是：0x4d41494c
tcpdump '((port 25) and (tcp[20:4] = 0x4d41494c))'
00010100H   qifeilingdian 
00210301H   qianxiang     

# 正式使用的命令如下------------------------
tcpdump -s 0 -nn -X -v -c 1 '((host 192.168.1.101) and (port 25) and (udp[19:4] = 0x00010100))' -i eth0 -w client.pcap
tcpdump -s 0 -nn -X -v -c 1 '((host 192.168.1.101) and (port 25) and (tcp[(((tcp[12:1] & 0xf0) >> 2)+11):4] = 0x00010100))' -i eth


tcpdump -s 0 -nn -X -v '((port 25) and (udp[8+11:4] = 0x303030313031303048))' -i enp0s8

tcp[((tcp[12:1] & 0xf0) >> 2)+11:4]
tcp[((tcp[12:1] & 0xf0) >> 2):4]
tcpdump '(tcp[(tcp[12]>>2):4] = 0x5353482D) and (tcp[((tcp[12]>>2)+4):2] = 0x312E)'

# 待测试命令
# 2.4 抓分片包
# a：匹配MF，分片包
tcpdump  'ip[6] = 32'
# b：匹配分片和最后分片
tcpdump  '((ip[6:2] > 0) and (not ip[6] = 64))'
# 测试分片可以用下面命令：
ping -M want -s 3000 192.168.1.101

ps -ef |grep tcpdump
192.168.43.123
192.168.115.82

# 获取所有tcpdump的进程号
ps aux|grep tcpdump|grep -v "grep"|awk '{print $2}'
# 杀死所有tcpdump进程
killall -9 tcpdump
# 杀死指定的tcpdump进程
kill -9 pid

# 设置tcpdump运行十秒钟后关闭：
timeout 10 tcpdump -i any -w package.pcap
# 注意不要使用 kill -9 pid 或者 killall tcpdump
# 会造成tcpdump异常终止，然后截包文件出现损坏

vncviewer 138.30.20.2:32

# 1.先将文件导入到容器
docker cp **.sql 【容器名】:/root/
# 2.进入容器
docker exec -it 【容器名/ID】/bin/bash
# 3.将文件导入数据库
mysql -uroot -proot 【数据库名】 < ***.sql

# 也可切换数据库使用source命令
mysql -uroot -p123456
# 查看一下数据库
show databases;
# 新建一个要导入的数据库（databaseName是数据库的名称）
create database databaseName;
# 使用该数据库
use databaseName;
# 导入
source /tmp/aoh_cms_dev.sql;
# 查看是否导入成功
show tables;
```

## 远程实时抓包解析

```bash
ssh root@192.168.43.136 'tcpdump tcp -nn -w - port 80' | tshark -T jsonraw -j "tcp" -x -r -

sudo ssh root@192.168.43.136 'tcpdump -s 0 -nn -v -i enp0s8 port 26667 -w -' | tshark -T fields -e data.data -r -
```



```bash
 sudo tcpdump -s 0 -nn -v -i enp0s8 port 26667 -w - | tshark -T fields -e data.data -l
```



```bash
# 保存为txt格式的文件
tail -n 1000 -f client.pcap |tshark -T fields -e data.data -r - >dataTest.txt
# 保存为json格式的文件
tail -n 1000 -f client.pcap |tshark -T json -e data.data -r - >dataTest.json
# 输出为流
tail -n 1000 -f client.pcap |tshark -T fields -e data.data -r - 
```

```bash
# 加上 -l参数才有实时输出的效果
sudo tshark -i enp0s8 -f 'port 26667' -T fields -e data.data -l
```

sudo tshark -i enp0s8 -f 'port 26667' -T fields -e data.data

ssh 192.168.43.136





坑

在抓包的时候，如果过滤了端口，如：tcpdump -i any udp port 5000。从上面的分析得到，UDP分片中，只有第一个分片是带有源端口、目的端口，因此只能抓取到第一个分片。
解决

把分片包全部抓取下来，使用：tcpdump -i any -s 0 udp port 5000 or 'ip[6:2] & 0x3fff != 0'

```bash
glshi@glshi-VirtualBox:~$ sudo tcpdump -s 0 -nn -A -v -i enp0s8 port 26667
[sudo] password for glshi:
tcpdump: listening on enp0s8, link-type EN10MB (Ethernet), capture size 262144 bytes
14:54:43.885505 IP (tos 0x0, ttl 128, id 47516, offset 0, flags [none], proto UDP (17), length 62)
    192.168.77.159.8888 > 192.168.77.82.26667: UDP, length 34
E..>......d...M...MR".h+.*..{"cmd":"logout","ip":"99.2.1.146"}
14:56:27.853418 IP (tos 0x0, ttl 128, id 47517, offset 0, flags [none], proto UDP (17), length 772)
    192.168.77.159.8888 > 192.168.77.82.26667: UDP, length 744
E.........b     ..M...MR".h+....{"cmd":"login","ip":"99.2.1.146","status":1,"config":{"index":2,"securtitylevel":"247~255","qoslevel":"0,1,2,6,4,5,10,13","centers":[{"index":0,"name":"............","bandwidth":28030,"AccessServername":"..........1","AccessServerIP":"99.1.0.10","AccessServerport":5050,"signalport":6666},{"index":1,"name":"............","bandwidth":17796,"AccessServername":"..........1","AccessServerIP":"99.3.0.10","AccessServerport":0,"signalport":6866},{"index":2,"name":"............","bandwidth":28030,"AccessServername":"..........1","AccessServerIP":"99.1.0.10","AccessServerport":5050,"signalport":7066}],"keynoteservice":[{"centerid":0,"taskid":347,"taskname":"test2","tasksecurity":"248","taskstation":[["stationname","stationip","securities"]]}]}}
^C
2 packets captured
2 packets received by filter
0 packets dropped by kernel
glshi@glshi-VirtualBox:~$ sudo tcpdump -s 0 -nn -A -v -i enp0s8 port 26667 -w test.pcap
tcpdump: listening on enp0s8, link-type EN10MB (Ethernet), capture size 262144 bytes
^C2 packets captured
2 packets received by filter
0 packets dropped by kernel
```



## 安装 jdk1.8

```bash
sudo mkdir /usr/local/java
cp jdk-8u66-linux-x64.tar.gz /usr/local/java
cd /usr/local/java
sudo tar zxvf jdk-8u66-linux-x64.tar.gz
sudo rm jdk-8u66-linux-x64.tar.gz
```

```
vi /etc/profile
```

打开之后在末尾添加

```bash
JAVA_HOME=/usr/local/java/jdk1.8.0_66

JRE_HOME=/usr/local/java/jdk1.8.0_66/jre

CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib

PATH=$JAVA_HOME/bin:$PATH

export PATH JAVA_HOME CLASSPATH
```

使环境变量生效

```bash
source /etc/profile
```

看看自己的配置是否都正确

```bash
echo $JAVA_HOME
echo $CLASSPATH
echo $PATH
```

如果系统已经安装了其他版本的Java

```bash
update-alternatives --install /usr/bin/java java /usr/java/jdk1.8.0_66/bin/java 300

update-alternatives --install /usr/bin/javac javac /usr/java/jdk1.8.0_66/bin/javac 300

update-alternatives --config java

update-alternatives --config javac
```

在终端

```bash
java -version
```

看看是否安装成功，成功则显示如下

```bash
java version "1.8.0_66"
Java(TM) SE Runtime Environment (build 1.8.0_66-b17)
Java HotSpot(TM) 64-Bit Server VM (build 25.65-b01, mixed mode)
```

## 安装 mysql5.5.59 docker

```bash
docker save mysql:5.5.59 > mysql5.5.59.tar
docker load < mysql5.5.59.tar

docker run -itd --name mysql -p 3306:3306 -v /home/wtzx/config/mysql:/etc/mysql/conf.d 
-v /home/wtzx/data/mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7


docker run -itd --name mysql55 -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql:5.5.59

docker run -itd --name mysql8 -v mysql8:/var/lib/mysql -p 3307:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql:8
```



## 安装 mysql5.7

1.检测系统是否已经安装过mysql或其依赖，若已装过要先将其删除，否则第4步使用yum安装时会报错：

```
1 # yum list installed | grep mysql
2 mysql-libs.i686         5.1.71-1.el6      @anaconda-CentOS-201311271240.i386/6.5
3 # yum -y remove mysql-libs.i686
```

 

2.从mysql的官网下载mysql57-community-release-el6-5.noarch.rpm（注意这里的el6-5即适配RHEL6.5的版本，如果下载了其它版本后面的安装过程中可能会报错）：

```
wget dev.mysql.com/get/mysql-community-release-el6-5.noarch.rpm
```

 

3.安装第一步下载的rpm文件：

```
 yum install mysql-community-release-el6-5.noarch.rpm
```

安装成功后，我们可以看到/etc/yum.repos.d/目录下增加了以下两个文件

```
1 # ls /etc/yum.repos.d
2 mysql-community-source.repo
3 mysql-community.repo
```

查看mysql57的安装源是否可用，如不可用请自行修改配置文件（/etc/yum.repos.d/mysql-community.repo）使mysql57下面的enable=1

若有mysql其它版本的安装源可用，也请自行修改配置文件使其enable=0

```
1 # yum repolist enabled | grep mysql
2 mysql-connectors-community MySQL Connectors Community                        13
3 mysql-tools-community      MySQL Tools Community                             18
4 mysql57-community-dmr      MySQL 5.7 Community Server Development Milesto    65
```

 

4.使用yum安装mysql：

```
yum install mysql-community-server
```

 

5.启动mysql服务：

```
service mysqld start
```

查看root密码：

```
1 # grep "password" /var/log/mysqld.log
2 2016-08-10T15:03:02.210317Z 1 [Note] A temporary password is generated for root@localhost: AYB(&-3Cz-rW
```

现在必须立刻修改密码，不然会报错：

```
ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.
```

修改密码（如果在此步报错ERROR 1819，请向下翻查看原因及解决方法）：

```
mysql> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('newpass');
```

 

6.查看mysqld是否开机自启动，并设置为开机自启动：

```
1 chkconfig --list | grep mysqld
2 chkconfig mysqld on
```

 

7.修改字符集为UTF-8：

```
vim /etc/my.cnf
```

在[mysqld]部分添加：

```
character-set-server=utf8
```

在文件末尾新增[client]段，并在[client]段添加：

```
default-character-set=utf8
```

修改好之后重启mysqld服务：

```
service mysqld restart
```

查看修改结果：

```
mysql> show variables like "%character%";
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | utf8                       |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)
```

 

注：在修改密码步骤，若设置的密码为简单密码，可能会出现如下错误：

```
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
```

这一错误其实与validate_password_policy值的设置有关：

![img](https://images2015.cnblogs.com/blog/782067/201608/782067-20160811110902746-619153901.png)

validate_password_policy值默认为1，即MEDIUM，所以刚开始设置的密码必须符合长度要求，且必须含有数字，小写或大写字母，特殊字符

如果我们只是做为测试用而不需要如此复杂的密码，可使用如下方式修改validate_password_policy值

```
mysql> set global validate_password_policy=0;
Query OK, 0 rows affected (0.00 sec)
```

这样，对密码要求就只有长度了，而密码的最小长度由validate_password_length值决定

validate_password_length参数默认为8，它有最小值的限制，最小值为：

```
validate_password_number_count+ validate_password_special_char_count+ (2 * validate_password_mixed_case_count)
```

其中，validate_password_number_count指定了密码中数字的长度，validate_password_special_char_count指定了密码中特殊字符的长度，validate_password_mixed_case_count指定了密码中大小字母的长度。这些参数的默认值均为1，所以validate_password_length最小值为4，如果显性指定validate_password_length的值小于4，尽管不会报错，但validate_password_length的值将设为4

设置validate_password_length的值：

```
mysql> set global validate_password_length=4;
Query OK, 0 rows affected (0.00 sec)
```

如果修改了validate_password_number_count，validate_password_special_char_count，validate_password_mixed_case_count中任何一个值，则validate_password_length将进行动态修改。

## 安装 mysql5.7 docker

依旧国际惯例，启动容器，设置root密码为root

```bash
docker run -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=123456 -d ibex/debian-mysql-server-5.7
```

查看容器

```bash
docker container ls
```



## 安装 docker1.7

[Docker系列（一）CentOS 6.5 离线安装、不升级内核 - 小鸟的士林 - 博客园 (cnblogs.com)](https://www.cnblogs.com/hdulzt/p/7834312.html)

[中标麒麟系统-离线内网环境-安装docker ( -io 1.x版本) - 走看看 (zoukankan.com)](http://t.zoukankan.com/lxgbky-p-13654769.html)

```
yum install -y cgroup
```



## 安装 node14.16

**一.准备工作**

下载nodejs：https://nodejs.org/download/release/v12.10.0/node-v12.10.0-linux-x64.tar.gz

（1）查看GLIBCXX版本,node需要 GLIBCXX_3.4.18版本以上，如果版本过低需要升级libstdc++.so.6.0.26 否则直接跳过 这一步

```
strings /usr/lib64/libstdc++.so.6 | grep GLIBC

1.下载libstdc++.so.6.0.26 
2.解压并且把解压的文件复制到 /usr/lib64/目录下
    cp libstdc++.so.6.0.26 /usr/lib64/
    
3. 进入到/usr/lib64/ 目录下删除软连接
    cd /usr/lib64/
    rm libstdc++.so.6
    
4.新建软连接
    ln -s libstdc++.so.6.0.26 libstdc++.so.6
```

（2）查看glibc，node需要GLIBC_2.17 ，如果版本过低需要升级。否则跳过这一步。

```bash
strings /lib64/libc.so.6 |grep GLIBC_
# 1.升级glibc至 2.17版本 下载7个包
wget http://copr-be.cloud.fedoraproject.org/results/mosquito/myrepo-el6/epel-6-x86_64/glibc-2.17-55.fc20/glibc-utils-2.17-55.el6.x86_64.rpm &
wget http://copr-be.cloud.fedoraproject.org/results/mosquito/myrepo-el6/epel-6-x86_64/glibc-2.17-55.fc20/glibc-static-2.17-55.el6.x86_64.rpm &
wget http://copr-be.cloud.fedoraproject.org/results/mosquito/myrepo-el6/epel-6-x86_64/glibc-2.17-55.fc20/glibc-2.17-55.el6.x86_64.rpm &
wget http://copr-be.cloud.fedoraproject.org/results/mosquito/myrepo-el6/epel-6-x86_64/glibc-2.17-55.fc20/glibc-common-2.17-55.el6.x86_64.rpm &
wget http://copr-be.cloud.fedoraproject.org/results/mosquito/myrepo-el6/epel-6-x86_64/glibc-2.17-55.fc20/glibc-devel-2.17-55.el6.x86_64.rpm &
wget http://copr-be.cloud.fedoraproject.org/results/mosquito/myrepo-el6/epel-6-x86_64/glibc-2.17-55.fc20/glibc-headers-2.17-55.el6.x86_64.rpm &
wget http://copr-be.cloud.fedoraproject.org/results/mosquito/myrepo-el6/epel-6-x86_64/glibc-2.17-55.fc20/nscd-2.17-55.el6.x86_64.rpm &
# 3.执行升级命令
  rpm -Uvh *-2.17-55.el6.x86_64.rpm --force --nodeps
```

**二.正式安装**

1.获取Node.js 安装包并安装
Node.js 安装包及源码下载地址为：https://nodejs.org/en/download/

```bash
wget https://nodejs.org/dist/v12.18.1/node-v12.18.1-linux-x64.tar.xz    # 下载
tar xf node-v12.18.1-linux-x64.tar.xz                                   # 解压

tar zxvf node-v12.18.1-linux-x64.tar.gz
```

vim /etc/profile

```bash
export NODE_HOME=/nodejs解压路径
export PATH=$PATH:$NODE_HOME/bin 
export NODE_PATH=$NODE_HOME/lib/node_modules
```

source /etc/profile 刷新配置

node -v 查看node版本



## service chkconfig systemctl

```bash
chkconfig --list                #列出所有的系统服务
chkconfig --add httpd           #增加httpd服务
chkconfig --del httpd           #删除httpd服务
chkconfig --level 2345 httpd on #设置httpd在运行级别为2、3、4、5的情况下都是on（开启）的状态
chkconfig --list                #列出系统所有的服务启动情况
chkconfig --list mysqld         #列出mysqld服务设置情况
chkconfig --level 35 mysqld on  #设定mysqld在等级3和5为开机运行服务，
								#--level 35表示操作只在等级3和5执行，on表示启动，off表示关闭
chkconfig mysqld on    			#设定mysqld在各等级为on，“各等级”包括2、3、4、5等级
```

![img](https://images2015.cnblogs.com/blog/1127982/201703/1127982-20170326000443611-1501812638.png)

