## tcpdump安装

```bash
yum install tcpdump
apt install tcpdump
```

## 抓包选项

```bash
-c：指定要抓取的包数量。

-i interface：指定tcpdump需要监听的接口。默认会抓取第一个网络接口

-n：对地址以数字方式显式，否则显式为主机名，也就是说-n选项不做主机名解析。

-nn：除了-n的作用外，还把端口显示为数值，否则显示端口服务名。

-P：指定要抓取的包是流入还是流出的包。可以给定的值为"in"、"out"和"inout"，默认为"inout"。

-s len：设置tcpdump的数据包抓取长度为len，如果不设置默认将会是65535字节。
```

## 特殊功能

```bash
-D：列出可用于抓包的接口。将会列出接口的数值编号和接口名，它们都可以用于"-i"后。

-F：从文件中读取抓包的表达式。若使用该选项，则命令行中给定的其他表达式都将失效。

-w：将抓包数据输出到文件中而不是标准输出。

-r：从给定的数据包文件中读取数据。使用"-"表示从标准输入中读取。
```

## 实用操作

```bash
# 抓包到文件中
sudo tcpdump -i eht0  -w file.pcap
# 现场抓包
sudo tcpdump -i any dst host 127.0.0.1 and port 6379
```

获取到的pcap文件可以使用wireshark分析



**Looking at Raw Packets with 'tcpdump'**

 Though tools like Snort do an excellent job of screening everything that comes through our network, sometimes its necessary to see the raw data. To do this, our best tool is 'tcpdump'.


 The most basic way of using tcpdump is to simply issue the command:

```bash
tcpdump
```

 You can get more detailed information using the -v option and you can get even more with -vv.


**Useful Options**

 Let's say you've logged in to a remote machine that you manage.  Normally, you would use SSH. If you ran 'tcpdump' without any options,  the output would be flooded with packets coming from your SSH  connection. To avoid this, simply eliminate port 22 from your output:

```bash
tcpdump not port 22
```

 You can do this with a number of different ports:

```bash
tcpdump not port 143 and not port 25 and not port 22
```

 If you wanted to do the reverse, that is, only monitor a certain port -  which is good for debugging network applications - you would do the  following:

```bash
tcpdump port 143
```

 You can also get data from a specific host on the network:

```
tcpdump host hal9000
```

 If your machine has more than one network interface, you can also specify the one you want to listen to:

```
tcpdump -i eth1
```

 You can also specify a protocol:

```
tcpdump udp
```

 You'll find a list of protocols in /etc/protocols.


**Saving Output for Later**

 In some cases, you may want to redirect the output to a file so that you can study it in detail later or use other programs to parse the output. In the following example, you can still watch the output while saving  it to a file:

```bash
tcpdump -l | tee tcpdump_`date +%Y%m%e-%k.%M`
```

In the above example, we can identify each of the dumps with the date  and time. This might come in handy when dealing with problems that come  at certain times of the day.

 tcpdump also has an option to dump its output into a binary format which it can read later. To create a binary file:

```bash
tcpdump -w tcpdump_raw_`date +%Y%m%e-%k.%M`
```

 Later, you can have tcpdump read the file with

```bash
tcpdump -r tcpdump_raw_YYYMMDD-H.M
```

 You can also use the program ethereal to open up the raw dump and  interpret it. We'll talk more about ethereal in the next section.


**Things to Look For**

 tcpdump gives us information about all the packets traveling to and from our network. But what does it all mean?


**Using Ethereal with tcpdump**

 Ethereal is a tool that can also be used to capture network packets.  Once installed, you can open up the raw dump file you made. It will look something like this:

 That makes it quite a bit easier to see what's going on. You can see  what the source and destination IPs are and what type of packet it was.  It's easy, then to troubleshoot network problems that you might have and to analyze suspicious behavior. Just to add an anecdote, while I was  writing this lesson and interpreting my own dumps, I saw some strange  activity on my personal workstation. Almost at regular intervals I was  querying port 32772 on machines disparate IPs out in the world. I ran a  specific dump for port 32772 like so:

```bash
tcpdump port 32772 -w dump_32772
```

 What I got did look strange indeed. Even after a Google search, I  couldn't find any information on this, so I suspected I might have a  trojan. I ran 'rootkit hunter' (more about that in the next section) and it turned up nothing. Finally, by shutting things down one by one, it  turned out to be Skype, which I always have turned on. Even though this  turned out to be innocuous, I was glad that I had tcpdump to point this  out to me.


**Reading the Raw Output**

 As you can see, reading even the so-called 'human readable' output from  tcpdump can be a bit cryptic. Take a look at the following example, a  random packet I just fished out of the dump:

```bash
17:26:22.924493 IP www.linux.org.www > test.linux.org.34365: P 2845:3739(894) ack 1624 win 9648 <nop,nop,timestamp 326501459 24374272>
```

What we have is a webserver request to [www.linux.org](https://www.linux.org). After the timestamp, you'll notice the .www at the end of the hostname  (which means port 80). This is being sent to port 34365 of the  requesting host, test.linux.org. The 'P' stands for the TCP "oush"  function. This means that the data should be sent immediately. Of the  numbers after, 2845:3739(894), 2845 marks the number of the octet of the first packet. The number 3739 is the number of the last byte it the  packet sent, plus 1. The number 894 is the length of the data packet  that was sent. The part that says: 'ack 1624' is the TCP term for  'acknowledge' - that is the packet has been accepted and the packet  number that is expected next is 1624. After, we see 'win 9648' that the  sending hosts awaits a packet with a window size of 9648 octets. This is followed by a timestamp.
 Now, if you think that is a bit difficult to interpret, if you use the  -x option, it will include the packet contents in hexadecimal output.  Here you'll need an Egyptologist to interpret the output:

```
18:12:45.149977 IP www.linux.org.www > test.linux.org.34536: . 1:1449(1448) 
ack 487 win 6432 <nop,nop,timestamp 329284215 27156244>
        0x0000:  4500 05dc 6a81 4000 4006 493b c0a8 0006  E...j.@.@.I;....
        0x0010:  c0a8 0009 0050 86e8 8fa4 1d47 1c33 e3af  .....P.....G.3..
        0x0020:  8010 1920 b4d9 0000 0101 080a 13a0 7a77  ..............zw
        0x0030:  019e 5f14 4854 5450 2f31 2e31 2032 3030  .._.HTTP/1.1.200
        0x0040:  204f 4b0d 0a44 6174 653a 2054 6875 2c20  .OK..Date:.Thu,.
        0x0050:  3135
```

 We can glean from the output, that this is a HTTP request. As for the  rest, it's not human readable, but we rest easy knowing that this is a  legitimate packet. Another advantage to using this format is that even  if we can't exactly interpret what's going on with this packet, we can  send it to somebody who might be able to. This is, in the end, the raw  data that's going over your network without any filtering.

