tshark是`wireshark`网络分析工具下的一个分支，主要用于命令行环境进行抓包、分析，尤其对协议深层解析时，`tcpdump`难以胜任的场景中。

## 主要参数

### 抓包接口类

```bash
-i 设置抓包的网络接口，不设置则默认为第一个非自环接口
-D 列出当前存在的网络接口
-f 设定抓包过滤表达式
-s 设置每个抓包的大小，默认为65535，多于这个大小的数据将不会被程序记入内存、写入文件。（这个参数相当于tcpdump的-s，tcpdump默认抓包的大小仅为68）
-p 设置网络接口以非混合模式工作，即只关心和本机有关的流量。
-B 设置内核缓冲区大小，仅对windows有效。
-y 设置抓包的数据链路层协议，不设置则默认为-L找到的第一个协议，局域网一般是EN10MB等。
-L 列出本机支持的数据链路层协议，供-y参数使用。  
```

### 抓包进度控制

```bash
-c 抓取的packet数，在处理一定数量的packet后，停止抓取，程序退出。
-a 设置tshark抓包停止向文件书写的条件，事实上是tshark在正常启动之后停止工作并返回的条件。条件写为test:value的形式，如“-a duration:5”表示tshark启动后在5秒内抓包然后停止；“-a filesize:10”表示tshark在输出文件达到10kB后停止；“-a files:n”表示tshark在写满n个文件后停止。
```



### 文件控制

```bash
-b 设置ring buffer文件参数。
    ring buffer的文件名由-w参数决定。-b参数采用test:value的形式书写。“-b duration:5”表示每5秒写下一个ring buffer文件；“-b filesize:5”表示每达到5kB写下一个ring buffer文件；“-b files:7”表示ring buffer文件最多7个，周而复始地使用，如果这个参数不设定，tshark会将磁盘写满为止。
-r 设置tshark分析的输入文件。tshark既可以抓取分析即时的网络流量，又可以分析dump在文件中的数据。-r不能是命名管道和标准输入。
```

### 过滤处理

```bash
-R 设置读取（显示）过滤表达式（read filter expression）。不符合此表达式的流量同样不会被写入文件。注意，读取（显示）过滤表达式的语法和底层相关的抓包过滤表达式语法不相同。类似于抓包过滤表达式，在命令行使用时最好将它们quote起来。
-n 禁止所有地址名字解析（默认为允许所有）。
-N 启用某一层的地址名字解析。“m”代表MAC层，“n”代表网络层，“t”代表传输层，“C”代表当前异步DNS查找。如果-n和-N参数同时存在，-n将被忽略。如果-n和-N参数都不写，则默认打开所有地址名字解析。
-d 将指定的数据按有关协议解包输出。如要将tcp 8888端口的流量按http解包，应该写为“-d tcp.port==8888,http”。注意选择子和解包协议之间不能留空格。
```

### 输出格式

```bash
-w 设置raw数据的输出文件。这个参数不设置，tshark将会把解码结果输出到stdout。“-w-”表示把raw输出到stdout。如果要把解码结果输出到文件，使用重定向“>”而不要-w参数。
-F 设置输出raw数据的格式，默认为libpcap。“tshark -F”会列出所有支持的raw格式。
-V 设置将解码结果的细节输出，否则解码结果仅显示一个packet一行的summary。
-x 设置在解码输出结果中，每个packet后面以HEX dump的方式显示具体数据。
-T 设置解码结果输出的格式，包括text,ps,psml和pdml，默认为text。
-t 设置解码结果的时间格式。“ad”表示带日期的绝对时间，“a”表示不带日期的绝对时间，“r”表示从第一个包到现在的相对时间，“d”表示两个相邻包之间的增量时间（delta）。
-S 在向raw文件输出的同时，将解码结果打印到控制台。
-l 在处理每个包时即时刷新输出。
-X 扩展项。
-q 设置安静的stdout输出（例如做统计时）
-z 设置统计参数。
```

### 其它

```bash
-h 显示命令行帮助。
-v 显示tshark的版本信息。
-o 重载选项。
```

## 基本用法

### 在linux下安装(debian或ubuntu)

> sudo apt-get install tshark
>  安装完成后在抓包之前，可以先检查版本、查看帮助等了解tshark初步了解

### 查看tshark版本

> tshark -v

### 列出当前存在的网络接口

> tshark -D
>  网卡描述依据OS有不同的编号方式，在不了解网络设备及编号情况下，一般先用“tshark -D”查看网络接口的编号以供-i参数使用。
>  **注:** linux可以结合ifconfig命令查看

### tshark对指定网卡监听，抓包

> sudo tshark -i <interface>

### 抓取示例

**抓取网卡eth0的流量并写入capture123.pcap**

> tshark -i eth0 -w capture123.pcap

**读取之前的文件capture123.pcap**

> tshark -i eth0 -r capture123.pcap

**抓取网卡eth0的流量10分钟**

> tshark -i eth0 -a duration:600

**注:** 默认时间单位为秒

**抓取网卡eth0的10000个数据包**

> tshark -c 10000 -i eth0

**抓取网卡eth0涉及192.168.1.1的流量报文**

> tshark -i eth0 -f “host 192.168.1.1”
>  **注:** 与wireshark、tcpdump一致，均使用BPF过滤表达式

**抓取网卡eth0指定协议的流量报文**

> tshark -i eth0 -f “<协议名>”
>  协议名可以为： tcp, udp, dns, icmp, http等

## 案例

### 实时打印当前mysql查询语句

> tshark -s 512 -i eth1 -n -f 'tcp dst port 3306' -R 'mysql.query' -T fields -e mysql.query

说明：

- -s 512 :只抓取前512个字节数据
- -i eth0 :监听eth0网卡
- -n :禁止域名解析
- -f ‘tcp dst port 3306’ :只捕捉协议为tcp,目的端口为3306的数据包
- -R ‘mysql.query’ :过滤出mysql.query查询语句的报文
- -T fields -e mysql.query :打印mysql查询语句

### 实时打印当前http请求的url(包括域名)

> tshark -s 512 -i eth1 -n -f 'tcp dst port 8000' -R 'http.host and http.request.uri' -T fields -e http.host -e http.request.uri -l | tr -d '\t'

说明：

- -s 512 :只抓取前512个字节数据
- -i eth1 :监听eth1网卡
- -n :禁止网络对象名称解析
- -f ‘tcp dst port 8000’ :只捕捉协议为tcp,目的端口为8000的数据包
- -R ‘http.host and http.request.uri’ :过滤出http.host和http.request.uri
- -T fields -e http.host -e http.request.uri :打印http.host和http.request.uri
- -l ：输出到标准输出

### 读取之前抓包文件进行报文数据分析

需要从抓包的文件evidence04.pcap中提取出报文相关数据信息，如时间、源IP、目的IP、协议名、源Port、標Port、包大小等信息，最后输出到csv文件。

> tshark -r evidence04.pcap -T fields -e frame.time_relative -e ip.src -e ip.dst -e ip.proto -e tcp.srcport -e tcp.dstport -e frame.len -E header=n -E separator=, -E quote=n -E occurrence=f > output.csv

说明：

```bash
-r evidence04.pcap 需要分析的报文记录文件（pcap格式）
-T fields 输出格式，选fields按字段，也可以选json等其他格式，需结合-e 及 -E使用
-e frame.time_relative 取出相對封包時間的欄位資料
-e ip.src 提取源IP
-e ip.dst 提取目的IP
-e ip.proto 提取协议名
-e tcp.srcport 提取源Port
-e tcp.dstport 提取目的Port
-e frame.len 提取包大小
-E header=n 是否输出字段名称（cvs的第1行）
-E separator=, 指定分割符，/t是tab，/s是一格空格
-E quote=n 指定是否对字段用引号，d是双引号，s是单引号，n是不用
-E occurrence=f 多值时是否保留，f是第一个值，l是最后一个值，a是所有值都列出，默认全部
> output.csv 输出文件路径及名称
```

