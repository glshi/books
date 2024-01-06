## 代理生效条件

配置文件示例

```nginx
server {
    listen       80;
    server_name  glshi.top;

    location /test/ {
        proxy_pass http://127.0.0.1:8080/;
    }
}
```

1：`nginx.conf` 配置文件中的`server_name` （glshi.top）必须和前端项目请求中的域名一致

2：请求的端口必须一致

3：查询下面的路径问题

## Nginx反向代理之路径替换

在使用nginx进行反向代理时，有时需要使用别名，或者说需要进行路径的替换。听不懂？那直接看下面的需求：

### 1.代理静态资源

在目录"E:\test\data\upload\20221104"下有一张图片1.jpg，在目录"E:\test\data\temp\20221022"也下有一张图片2.jpg，现需要通过nginx来代理访问资源。

1）需求：通过在浏览器访问http://127.0.0.1/img/upload/20221104/1.jpg 和 http://127.0.0.1/img/temp/20221022/2.jpg 和 访问两张图片

nginx默认是通过配置root来代理静态资源，而刚好上述是通过真实的目录去访问的，故在root上配置路径即可

```nginx
server {
    listen       80;
    server_name  127.0.0.1;

    location /img/ {
        root  E:/test/data;
        index  index.html index.htm;
    }
}
```

这里相当于nginx代理到目录data下，然后即可访问到img目录下的所有文件。其中img也可以配置为在后面不加 "/"，为 location /img，此处后面加不加 "/" 效果一样，但为了规范，建议都加。

2）需求：通过在浏览器访问http://127.0.0.1/file/upload/20221104/1.jpg 和 http://127.0.0.1/file/temp/20221022/2.jpg 和 访问两张图片

很明显，在目录中并未有file目录，那么此时就需要设置别名，进行访问

```nginx
server {
    listen       80;
    server_name  127.0.0.1;

    location /file/ {
        alias  E:/test/data/img/;
        index  index.html index.htm;
    }
}
```

需要注意的是，在alias后指定的路径，最后一定要加 "/"，否则很有可能会因为location配置不当而出现404.

### 2.代理动态服务

所谓动态服务也就是后端的各种请求，有时直接使用匹配转发，有时需要处理后转发。

现有一个查询用户信息的接口，地址是http://127.0.0.1:8080/api/user/getById?id=123

1）需求：通过nginx转发，使用http://127.0.0.1/api/user/getById?id=123访问用户查询服务

```nginx
server {
    listen       80;
    server_name  127.0.0.1;

    location /api/ {
        proxy_pass http://127.0.0.1:8080;
    }
}
```

通过原有地址直接准发非常简单。

2）需求：通过nginx转发，使用http://127.0.0.1/test/api/user/getById?id=123访问用户查询服务

```nginx
server {
    listen       80;
    server_name  127.0.0.1;

    location /test/ {
        proxy_pass http://127.0.0.1:8080/;
    }
}
```

这里相当于对请求添加了前缀，但在转发的过程中是没有前缀的，故需要去掉。关键点就是地址后面的 "/".

### 3.关于斜杆"/"的案例对比

以服务地址http://127.0.0.1:8080/api/user/getById进行说明，访问地址是http://127.0.0.1/api/user/getById。location后斜杆与proxy_pass后斜杆问题如下：

1）**location、proxy_pass都不加斜杠**

```nginx
location /api {
　　proxy_pass http://127.0.0.1:8080;
}
```

实际代理地址：http://127.0.0.1:8080/api/user/getById。正确的

2）**location加斜杠，proxy_pass不加斜杠**

```nginx
location /api/ {
　　proxy_pass http://127.0.0.1:8080;
}
```

实际代理地址：http://127.0.0.1:8080/api/user/getById。正确的

3）**location不加斜杠，proxy_pass加斜杠**

```nginx
location /api {
　　proxy_pass http://127.0.0.1:8080/;
}
```

实际代理地址：http://127.0.0.1:8080//user/getById。错误的，也出现了双斜杠

4）**location、proxy_pass都加斜杠**

```nginx
location /api/ {
　　proxy_pass http://127.0.0.1:8080/;
}
```

实际代理地址：http://127.0.0.1:8080/user/getById

5）**location不加斜杠，proxy_pass加"api"**

```nginx
location /api {
   proxy_pass http://127.0.0.1:8080/api;
}
```

实际代理地址：http://127.0.0.1:8080/api/user/getById。正确的

6）**location加斜杠，proxy_pass加"api"**

```nginx
location /api/ {
   proxy_pass http://127.0.0.1:8080/api;
}
```

实际代理地址：http://127.0.0.1:8080/apiuser/getById。错误的，少了一个斜杆

7）**location不加斜杠，proxy_pass加"api/"**

```nginx
location /api {
   proxy_pass http://127.0.0.1:8080/api/;
```

实际代理地址：http://127.0.0.1:8080/api//user/getById。这种情况会出现双斜杠问题，后端在认证请求时会校验失败。

8）**location加斜杠，proxy_pass加"api/"**

```nginx
location /api/ {
   proxy_pass http://127.0.0.1:8080/api/;
}
```

实际代理地址：http://127.0.0.1:8080/api/user/getById。正确的

可以看出，两者加不加斜杆的区别还是很大的，不同的场景使用不同的配置即可，但我建议要么两者都加斜杆，要么都不加，这样转发的地址一般不会错。

使用一句标准的话来说，总结如下：（与location是否有斜杆关系不大）

第一，若proxy_pass代理地址端口后无任何字符，则转发后地址为：代理地址+访问的uri。例如第1、2种情况

第二，若proxy_pass代理地址端口后有目录（包括"/"），则转发后地址为：代理地址+访问的uri**去除**location匹配的路径。例如第3-8种情况，但有些情况可能是错的，配置时需要谨慎。

## nginx 常用示例

### 1.nginx基本概念

**Nginx的概念**

- 正向代理和反向代理：
  - 正向代理：正向代理就是在客户端配置代理服务器，通过代理服务器去进行互联网操作。(VPN代理客户端)
  - 反向代理：客户端发送请求到反向代理服务器，由反向代理服务器去选择目标服务器获取它的数据，在返回给客户端。此时反向代理服务器和目标服务器对外就是一台服务器，暴露的是代理服务器地址，隐藏了真实的服务器地址。（代理服务端）
- 负载均衡
  - 在多个服务器的情况下，我们将请求发放到各个服务器上，将原先请求集中到单个服务器的情况改为将请求发送到多个服务器上，将负载分发到不同的服务器，也就是负载均衡
- 动静分离
  - 简单理解就是把静态资源和动态资源分开部署。为了加快网站解析的速度，可以把静态资源和动态资源部署到不同的服务器来解析，加快解析速度。降低单个服务器的压力！

### 2.常用命令以及配置文件

**Win下nginx的常用命令**

- 前提：需要进入到nginx目录下在进行操作
- nginx -v：查看nginx版本号
- nginx -s stop：关闭nginx
- start nginx：启动nginx
- nginx -s reload：重新加载nginx
- nginx -t 检查默认配置conf
- taskkill /f /im nginx.exe win杀掉nginx

**nginx配置文件**

- 全局块
  - 主要设置一些影响nginx服务器运行的配置指令。主要包括配置运行nginx服务器的用户，允许生成的worker process数，进程PID存放路径，日志存放路径和类型以及配置文件的引入等
  - 比如：worker_process 1; 这个就代表nginx服务器并发处理服务的关键配置，它的值越大，表示支持的并发处理量越多，但是会受到硬件，软件等设备的约束。
- events块
  - events块主要影响nginx服务器与用户的网络连接，是否开启同时多个网络连接
  - 比如：worker_connections：1024；表示最大连接数为1024个
- http块（http块包含了http全局块和server块）
  - 这时nginx配置最频繁的部分，代理，缓存，日志等都是在这里配置。
  - http全局块：主要是配置日志等等的配置
  - server块：
    - server全局快：listen：参数对应的是端口号；server_name：地址参数
    - location块：用来配置响应反向代理的。增加参数proxy_pass 服务器（反向代理去到的服务器）地址（比如：127.0.0.1:8080）

### 3.nginx配置实例之反向代理

1. 实现效果：监听9000端口。根据不同的路径跳转到不同的端口服务中

2. 准备工作

准备好两个服务器：8080和8081

在8080tomcat的webapps里面添加一个shisan01文件夹，编写一个shisan.html。内容自己定就好，同理8081也是一样，添加一个shisan02文件夹，编写一个shisan.html。跑完之后访问不同的服务测试

3. 前往nginx配置文件配置

进入配置文件，如下配置

```nginx
server {
  listen        9000;
  server_name        192.168.12.127；
  location ~/shisan01/ {
      proxy_pass http://127.0.0.1:8080;
  }
  location ~/shisan02/ {
      proxy_pass http://127.0.0.1:8081;
  }
}
```

PS:注意要记得开发端口号：8080 8081 9000

PS：location的指令可以百度了解一下

### 4.nginx配置实例之负载均衡

1. 准备工作

同样是两个tomcat（8080和8081）服务器，当然要是你真有两台服务器最好！哈哈哈。条件问题还是模拟。

在两个服务器里面的webapps里面创建一个shisan文件夹，在shisan文件夹里创建一个shisan.html，添加内容自己定就好

2. 在nginx配置文件中修改配置

在http块中配置

```nginx
upstream myserver {
    server        192.168.12.127:8080;
    server        192.168.12.127:8081;
}
server {
    listen        80;
    server_name        192.168.12.127;
    location / {
        proxy_pass        http://myserver;
        root        html;
        index        index.html index.htm;
    }
}
```

3. nginx分配服务器的策略

轮询：默认策略；每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除

权重：指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。

```nginx
upstream myserver {
    server        192.168.12.127:8080 weight=10;
    server        192.168.12.127:8081 weight=5;
}
```

IP绑定 ip_hash：每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题

```nginx
upstream myserver { 
  ip_hash; 
  server        192.168.12.127:8080;
  server        192.168.12.127:8081;
}
```

fair：按后端服务器的响应时间来分配请求，响应时间短的优先分配。

```nginx
upstream backserver {
    server 192.168.12.127：8080;
    server 192.168.12.127：8081;
    fair;
}
```

url_hash：按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。

### 5.nginx配置实例之动静分离

1. 准备工作

弄好一个动态资源一个静态资源：比如

- /shisan/image/01.jpg

- /shisan/html/01.html
2. 配置nginx文件

```nginx
server {
    listen        80;
    server_name        192.168.12.127;
    location /html/ {
        root        /shisan/;
        index        index.html index.htm;
    }
    location /image/ {
        root        /shisan/;
        autoindex        on;            // 列出文件的目录，可以自行观看对比就知道
    }
}
```

### 6.nginx配置高可用集群

1. 因为我这里环境不允许，可以[参考这个文章](https://www.jb51.net/article/256093.htm)

### 7. Location规则

语法规则： `location [=||*|^~] /uri/ {… }`

首先匹配 =，其次匹配^~,其次是按文件中顺序的正则匹配，最后是交给 /通用匹配。当有匹配成功时候，停止匹配，按当前匹配规则处理请求。

| 符号                    | 含义                                                                                                 |
| --------------------- | -------------------------------------------------------------------------------------------------- |
| =                     | = 开头表示精确匹配                                                                                         |
| ^~                    | ^~开头表示uri以某个常规字符串开头，理解为匹配 url路径即可。nginx不对url做编码，因此请求为/static/20%/aa，可以被规则^~ /static/ /aa匹配到（注意是空格） |
| ~                     | ~ 开头表示区分大小写的正则匹配                                                                                   |
| **~***                | ~ 开头表示不区分大小写的正则匹配*                                                                                 |
| !和!*                  | !和!*分别为区分大小写不匹配及不区分大小写不匹配的正则                                                                       |
| /                     | 用户所使用的代理（一般为浏览器）                                                                                   |
| $http_x_forwarded_for | 可以记录客户端IP，通过代理服务器来记录客户端的ip地址                                                                       |
| $http_referer         | 可以记录用户是从哪个链接访问过来的                                                                                  |

比这些 location 规则来选择一个 location,对比的顺序可以总结为:

1. 首先匹配前缀匹配(没有 RE 表达式),针对当前这个请求,每个前缀匹配都匹配一遍.
2. 搜索=匹配,如果当前请求匹配上了,搜索将会停止,直接使用这个这个 location.
3. 如果第二步没有匹配上,nginx 会按照如下步骤继续搜索最长前缀匹配:  
   3.1 如果最长前缀匹配有^~这个modifier,nginx 会停止搜索并直接使用这个 location.  
   3.2 如果没有使用 ^~,暂存这个 location并且继续搜索.
4. 只要最长前缀匹配被暂存和选中,nginx 就会看当前的 location 是否有大小写敏感的 RE(~和~*),第一个匹配上这种会被当做有效的 location来处理这个请求.
5. 如果没有 RE 的 location 匹配上,前面暂存的 location 就会被选中来处理这个请求.

**举例**

如下是一些 location 配置的例子,用来详细描述上面所说的处理顺序,你也可以按照具体实际情况来修改这些例子.

```nginx
location  = / {
    # 只处理请求 /.
}
location /data/ {
    # 所有以 /data/ 匹配,但是还会继续搜索.
    # 如果没有其他 location 匹配上,就用这个处理请求.
}
```

```nginx
location ^~ /img/ {
    # 所有以 /img/ 开头的请求并且会停止搜索.
}
location ~* .(png|gif|ico|jpg|jpeg)$ {
    # 以png, gif, ico, jpg ,jpeg结尾的请求. 
    # 如果请求是到 /img/ 路径的话 还是会被上面👆的 location 处理
}
```

文档链接：

[Nginx如何配置根据路径转发详解_nginx_脚本之家](https://www.jb51.net/article/256088.htm)

[Nginx反向代理之路径替换 - 码农教程](http://www.manongjc.com/detail/40-hhriemappoydwww.html)
