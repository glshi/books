[TOC]



# wireshark



# 一、过滤器

## 概念

1. 捕获过滤器：当进行数据包捕获时，只有那些满足给定的包含/排除表达式的数据包会被捕获。
2. 显示过滤器：该过滤器根据指定的表达式用于在一个已捕获的数据包集合中，隐藏不想显示的数据包，或者只显示那些需要的数据包。

## 1、过滤源MAC、目的MAC

在wireshark的过滤规则框Filter中输入过滤条件，如查找源和目的地址都为80:f6:bb:cc:dd:00的包，eth.addr== 80:f6:bb:cc:dd:00；查找源地址为eth.src== 80:f6:bb:cc:dd:00；查找目的地址为eth.dst==  80:f6:bb:cc:dd:00；

## 2、 过滤源ip、目的ip

在wireshark的过滤规则框Filter中输入过滤条件，如查找目的地址为192.168.101.8的包，ip.dst==192.168.101.8；查找源地址为ip.src==1.1.1.1；

## 3、端口过滤

如过滤80端口，在Filter中输入，tcp.port==80，这条规则是把源端口和目的端口为80的都过滤出来。使用tcp.dstport==80只过滤目的端口为80的，tcp.srcport==80只过滤源端口为80的包；

## 4、协议过滤

协议过滤比较简单，直接在Filter框中直接输入协议名即可，http udp dns

## 5、http模式过滤

如过滤get包，http.request.method==”GET”,过滤post包，http.request.method==”POST”；

## 6、连接符and的使用

过滤两种条件时，使用and连接，如过滤ip为192.168.101.8并且为http协议的，ip.src==192.168.101.8 and http。

# 二、BPF 语法

## 概念

捕获过滤器应用于WinPcap，并使用BerkeleyPacketFilter(BPF)语法。这个语法被广泛用于多种数据包嗅探软件，主要因为大部分数据包嗅探软件都依赖于使用BPF的libpcap/WinPcap库。掌握BPF语法对你在数据包层级更深入地探索网络来说，非常关键。

## 语法

使用BPF语法创建的过滤器被称为表达式，并且表达式包含一个或多个原语。每个原语包含一个或多个限定词，然后跟着一个ID名字或者数字，如：

使用BPF语法创建的过滤器被称为表达式，并且表达式包含一个或多个原语。每个原语包含一个或多个限定词，然后跟着一个ID名字或者数字，如：

![图片](https://img-blog.csdnimg.cn/img_convert/c46c850fd8e4159db8e7e1806a147f9f.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

BPF语法也是支持下列逻辑运算符的，从而创造更高级的表达式。

1. 连接运算符 与 （&&）
2. 选择运算符 或 （||）
3. 否定运算符 非 （!）

举例说明：

```cpp
src 192.168.0.10 && port 5005
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

上述表达式只对源地址是192.168.0.10和源端口或目标端口是5005的流量进行捕获。



常用的过滤示例

![图片](https://img-blog.csdnimg.cn/img_convert/4b220cdd146185e856da1d43cf3d7793.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

注意灵活使用上文提到的逻辑运算符。









# 三、抓取 https

现在市面上的主流浏览器实现的 HTTP2 都是基于`TLS`的，也就是说要分析`HTTP2`报文得先过了`TLS`这一关，不然只能分析一堆加密的乱码。

`wireshark`支持两种方式来解密`SSL/TLS`报文：

1. 通过网站的私钥
2. 通过浏览器的将 TLS 对称加密秘保存在外部文件中，以供 wireshark 加解密

下面我来一一进行演示

## 1. 通过网站的私钥

如果你想抓取的网站是你自己的，那么可以利用这种方式，因为这需要使用网站生成证书使用的私钥进行解密，就是那个 nginx 上配置的`ssl_certificate_key`对应的私钥文件，把它添加到 wireshark 配置中：

![img](https://segmentfault.com/img/remote/1460000023568907)

然后通过`wireshark`就可以看到明文了：

![img](https://segmentfault.com/img/remote/1460000023568908)

通过上图可以看到，我通过`curl`访问的 https 协议的 URL，在配置了该服务器对应的私钥后可以抓取到对应的 HTTP 明文。

不过缺点也非常明显，只能分析自己持有私钥的网站，如果别人的网站就分析不了了，所幸的是还有第二种方案来支持。

## 2. 通过浏览器的 SSL 日志功能

目前该方案只支持`Chrome`和`Firefox`浏览器，通过设置`SSLKEYLOGFILE`环境变量，可以指定浏览器在访问`SSL/TLS`网站时将对应的密钥保存到本地文件中，有了这个日志文件之后`wireshake`就可以将报文进行解密了。

1. 首先设置`SSLKEYLOGFILE`环境变量：
   ![img](https://segmentfault.com/img/remote/1460000023568906)

   > 注：这是在 windows 系统上进行操作的，其它操作系统同理

2. 配置

   ```
   wireshake
   ```

   ，首选项->Protocls->TLS：

   ![img](https://segmentfault.com/img/remote/1460000023568910)

   将第一步中指定的文件路径配置好

3. 重启浏览器，进行抓包：
   ![img](https://segmentfault.com/img/remote/1460000023568909)

   同样的可以抓取到 HTTP 明文。

   > 注：不抓包时记得把环境变量删掉，以避免性能浪费和安全性问题

方案二的优点非常明显，可以抓取任意网站的`SSL/TLS`加密的报文，唯一的缺点就是只能是浏览器支持的情况才行，而方案一可以针对任何 HTTP 客户端进行抓包。

## 3. 通过 wireshake 抓取 HTTP2 报文

上面都是针对`TLS+HTTP1`进行的抓包，市面上主流的浏览器的`HTTP2`都是基于`TLS`实现的，所以也是一样的，把`TLS`这层解密了自然看到的就是最原始的明文。

这里以分析`https://www.qq.com`为例，为什么不是经典`htts://www.baidu.com`，因为百度首页至今还是`HTTP/1.1`协议。

1. 使用上面的第二种方案配置好`wiresharke`

2. 通过`http2`关键字做过滤

3. 浏览器访问`https://www.qq.com`

4. 查看

   ```
   HTTP2
   ```

   报文：

   ![img](https://segmentfault.com/img/remote/1460000023568911)

   这样就抓取到了`HTTP2`报文了，HTTP2 协议非常复杂，我也还在学习阶段，这里就不多说啥了。



# 四、TcpDump 导入分析

```bash
# 抓包到文件中
sudo tcpdump -i eht0  -w file.pcap
# 现场抓包
sudo tcpdump -i any dst host 127.0.0.1 and port 6379
```

获取到的pcap文件可以使用wireshark分析



# 五、请求包和响应包三种对应方式

以Wireshark2.6.3版本为例，如下图所示，红框中的803是一次HTTP的GET请求包，绿框中的809、810两条记录都是响应包，究竟哪个是803的响应包呢？接下来介绍三种方式识别；
 ![在这里插入图片描述](https://img-blog.csdn.net/20181002124910484?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### 通过传输控制协议信息识别

1. 如下图，点击803这条记录后，在下面的详情窗口打开传输层信息，查看Next sequence number字段的值为282：
    ![在这里插入图片描述](https://img-blog.csdn.net/20181002125505329?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
2. 分别打开809、810这两条记录的详情，查看它们的传输层信息，找到Acknowledgment number字段，等于282的记录就是803的响应信息，如下图：
    ![在这里插入图片描述](https://img-blog.csdn.net/2018100213015966?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
    此时已经找到了803对应的响应，可以继续打开HTTP层的数据查看响应信息的详情了；

### 通过Wireshark的识别结果

通过传输控制协议信息识别的方法略有些麻烦，需要打开所有记录逐个检查，Wireshark已经做了更方便的方式：

1. 展开803号记录的HTTP层，如下图所示，红框中的内容是可以点击的，双击后会立即打开响应记录809的内容：
    ![在这里插入图片描述](https://img-blog.csdn.net/20181002131713511?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
2. 查看响应数据时也有对应的请求包链接，双击链接可打开对应的请求数据包，如下图，以809号记录为例，在HTTP层中可以双击下图红框中的内容，直接打开803的内容：
    ![在这里插入图片描述](https://img-blog.csdn.net/20181002131600340?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### Wireshark的标记

最后介绍的是最简单的方式，如下图，红框中的朝右的箭头是请求，蓝框中朝左的箭头代表这就是对应的响应：
 ![在这里插入图片描述](https://img-blog.csdn.net/20181003172601358?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
