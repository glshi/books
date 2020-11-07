

# docker-compose使用

# 参数配置解析

先来看一份 docker-compose.yml 文件，不用管这是干嘛的，只是有个格式方便后文解说：



```ruby
version: '2'
services:
  web:
    image: dockercloud/hello-world
    ports:
      - 8080
    networks:
      - front-tier
      - back-tier

  redis:
    image: redis
    links:
      - web
    networks:
      - back-tier

  lb:
    image: dockercloud/haproxy
    ports:
      - 80:80
    links:
      - web
    networks:
      - front-tier
      - back-tier
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock 

networks:
  front-tier:
    driver: bridge
  back-tier:
driver: bridge
```

可以看到一份标准配置文件应该包含 version、services、networks 三大部分，其中最关键的就是 services 和 networks 两个部分，下面先来看 services 的书写规则。

## 1. image



```yaml
services:
  web:
    image: hello-world
```

在 services 标签下的第二级标签是 web，这个名字是用户自己自定义，它就是服务名称。
 image 则是指定服务的镜像名称或镜像 ID。如果镜像在本地不存在，Compose 将会尝试拉取这个镜像。
 例如下面这些格式都是可以的：



```undefined
image: redis
image: ubuntu:14.04
image: tutum/influxdb
image: example-registry.com:4000/postgresql
image: a4bc65fd
```

## 2. build

服务除了可以基于指定的镜像，还可以基于一份 Dockerfile，在使用 up 启动之时执行构建任务，这个构建标签就是 build，它可以指定 Dockerfile 所在文件夹的路径。Compose 将会利用它自动构建这个镜像，然后使用这个镜像启动服务容器。



```jsx
build: /path/to/build/dir
```

也可以是相对路径，只要上下文确定就可以读取到 Dockerfile。



```undefined
build: ./dir
```

设定上下文根目录，然后以该目录为准指定 Dockerfile。



```undefined
build:
  context: ../
  dockerfile: path/of/Dockerfile
```

注意 build 都是一个目录，如果你要指定 Dockerfile 文件需要在 build 标签的子级标签中使用 dockerfile 标签指定，如上面的例子。
 如果你同时指定了 image 和 build 两个标签，那么 Compose 会构建镜像并且把镜像命名为 image 后面的那个名字。



```undefined
build: ./dir
image: webapp:tag
```

既然可以在 docker-compose.yml 中定义构建任务，那么一定少不了 arg 这个标签，就像 Dockerfile 中的 ARG 指令，它可以在构建过程中指定环境变量，但是在构建成功后取消，在 docker-compose.yml 文件中也支持这样的写法：



```undefined
build:
  context: .
  args:
    buildno: 1
    password: secret
```

下面这种写法也是支持的，一般来说下面的写法更适合阅读。



```undefined
build:
  context: .
  args:
    - buildno=1
    - password=secret
```

与 ENV 不同的是，ARG 是允许空值的。例如：



```undefined
args:
  - buildno
  - password
```

这样构建过程可以向它们赋值。

> 注意：YAML 的布尔值（true, false, yes, no, on, off）必须要使用引号引起来（单引号、双引号均可），否则会当成字符串解析。

## 3. command

使用 command 可以覆盖容器启动后默认执行的命令。



```bash
command: bundle exec thin -p 3000
```

也可以写成类似 Dockerfile 中的格式：



```bash
command: [bundle, exec, thin, -p, 3000]
```

## 4. container_name

前面说过 Compose 的容器名称格式是：<项目名称>*<服务名称>*<序号>
 虽然可以自定义项目名称、服务名称，但是如果你想完全控制容器的命名，可以使用这个标签指定：



```undefined
container_name: app
```

这样容器的名字就指定为 app 了。

## 5. depends_on

在使用 Compose 时，最大的好处就是少打启动命令，但是一般项目容器启动的顺序是有要求的，如果直接从上到下启动容器，必然会因为容器依赖问题而启动失败。
 例如在没启动数据库容器的时候启动了应用容器，这时候应用容器会因为找不到数据库而退出，为了避免这种情况我们需要加入一个标签，就是 depends_on，这个标签解决了容器的依赖、启动先后的问题。
 例如下面容器会先启动 redis 和 db 两个服务，最后才启动 web 服务：



```bash
version: '2'
services:
  web:
    build: .
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres
```

注意的是，默认情况下使用 docker-compose up web 这样的方式启动 web 服务时，也会启动 redis 和 db 两个服务，因为在配置文件中定义了依赖关系。

## 6. dns

和 --dns 参数一样用途，格式如下：



```css
dns: 8.8.8.8
```

也可以是一个列表：



```css
dns:
  - 8.8.8.8
  - 9.9.9.9
```

此外 dns_search 的配置也类似：



```css
dns_search: example.com
dns_search:
  - dc1.example.com
  - dc2.example.com
```

## 7. tmpfs

挂载临时目录到容器内部，与 run 的参数一样效果：



```undefined
tmpfs: /run
tmpfs:
  - /run
  - /tmp
```

## 8. entrypoint

在 Dockerfile 中有一个指令叫做 ENTRYPOINT 指令，用于指定接入点，第四章有对比过与 CMD 的区别。
 在 docker-compose.yml 中可以定义接入点，覆盖 Dockerfile 中的定义：



```jsx
entrypoint: /code/entrypoint.sh
```

格式和 Docker 类似，不过还可以写成这样：



```ruby
entrypoint:
    - php
    - -d
    - zend_extension=/usr/local/lib/php/extensions/no-debug-non-zts-20100525/xdebug.so
    - -d
    - memory_limit=-1
    - vendor/bin/phpunit
```

## 9. env_file

还记得前面提到的 .env 文件吧，这个文件可以设置 Compose 的变量。而在 docker-compose.yml 中可以定义一个专门存放变量的文件。
 如果通过 docker-compose -f FILE 指定了配置文件，则 env_file 中路径会使用配置文件路径。

如果有变量名称与 environment 指令冲突，则以后者为准。格式如下：



```css
env_file: .env
```

或者根据 docker-compose.yml 设置多个：



```jsx
env_file:
  - ./common.env
  - ./apps/web.env
  - /opt/secrets.env
```

注意的是这里所说的环境变量是对宿主机的 Compose 而言的，如果在配置文件中有 build 操作，这些变量并不会进入构建过程中，如果要在构建中使用变量还是首选前面刚讲的 arg 标签。

## 10. environment

与上面的 env_file 标签完全不同，反而和 arg 有几分类似，这个标签的作用是设置镜像变量，它可以保存变量到镜像里面，也就是说启动的容器也会包含这些变量设置，这是与 arg 最大的不同。
 一般 arg 标签的变量仅用在构建过程中。而 environment 和 Dockerfile 中的 ENV 指令一样会把变量一直保存在镜像、容器中，类似 docker run -e 的效果。



```yaml
environment:
  RACK_ENV: development
  SHOW: 'true'
  SESSION_SECRET:

environment:
  - RACK_ENV=development
  - SHOW=true
  - SESSION_SECRET
```

## 11. expose

这个标签与Dockerfile中的EXPOSE指令一样，用于指定暴露的端口，但是只是作为一种参考，实际上docker-compose.yml的端口映射还得ports这样的标签。



```bash
expose:
 - "3000"
 - "8000"
```

## 12. external_links

在使用Docker过程中，我们会有许多单独使用docker run启动的容器，为了使Compose能够连接这些不在docker-compose.yml中定义的容器，我们需要一个特殊的标签，就是external_links，它可以让Compose项目里面的容器连接到那些项目配置外部的容器（前提是外部容器中必须至少有一个容器是连接到与项目内的服务的同一个网络里面）。
 格式如下：



```css
external_links:
 - redis_1
 - project_db_1:mysql
 - project_db_1:postgresql
```

## 13. extra_hosts

添加主机名的标签，就是往/etc/hosts文件中添加一些记录，与Docker client的--add-host类似：



```bash
extra_hosts:
 - "somehost:162.242.195.82"
 - "otherhost:50.31.209.229"
```

启动之后查看容器内部hosts：



```css
162.242.195.82  somehost
50.31.209.229   otherhost
```

## 14. labels

向容器添加元数据，和Dockerfile的LABEL指令一个意思，格式如下：



```csharp
labels:
  com.example.description: "Accounting webapp"
  com.example.department: "Finance"
  com.example.label-with-empty-value: ""
labels:
  - "com.example.description=Accounting webapp"
  - "com.example.department=Finance"
  - "com.example.label-with-empty-value"
```

## 15. links

还记得上面的depends_on吧，那个标签解决的是启动顺序问题，这个标签解决的是容器连接问题，与Docker client的--link一样效果，会连接到其它服务中的容器。
格式如下：



```css
links:
 - db
 - db:database
 - redis
```

使用的别名将会自动在服务容器中的/etc/hosts里创建。例如：



```css
172.12.2.186  db
172.12.2.186  database
172.12.2.187  redis
```

相应的环境变量也将被创建。

## 16. logging

这个标签用于配置日志服务。格式如下：



```bash
logging:
  driver: syslog
  options:
    syslog-address: "tcp://192.168.0.42:123"
```

默认的driver是json-file。只有json-file和journald可以通过docker-compose logs显示日志，其他方式有其他日志查看方式，但目前Compose不支持。对于可选值可以使用options指定。
 有关更多这方面的信息可以阅读官方文档：
 [https://docs.docker.com/engine/admin/logging/overview/](https://link.jianshu.com?t=https://docs.docker.com/engine/admin/logging/overview/)

## 17. pid



```bash
pid: "host"
```

将PID模式设置为主机PID模式，跟主机系统共享进程命名空间。容器使用这个标签将能够访问和操纵其他容器和宿主机的名称空间。

## 18. ports

映射端口的标签。
 使用HOST:CONTAINER格式或者只是指定容器的端口，宿主机会随机映射端口。



```css
ports:
 - "3000"
 - "8000:8000"
 - "49100:22"
 - "127.0.0.1:8001:8001"
```

> 注意：当使用HOST:CONTAINER格式来映射端口时，如果你使用的容器端口小于60你可能会得到错误得结果，因为YAML将会解析xx:yy这种数字格式为60进制。所以建议采用字符串格式。

## 19. security_opt

为每个容器覆盖默认的标签。简单说来就是管理全部服务的标签。比如设置全部服务的user标签值为USER。



```css
security_opt:
  - label:user:USER
  - label:role:ROLE
```

## 20. stop_signal

设置另一个信号来停止容器。在默认情况下使用的是SIGTERM停止容器。设置另一个信号可以使用stop_signal标签。



```undefined
stop_signal: SIGUSR1
```

## 21. volumes

挂载一个目录或者一个已存在的数据卷容器，可以直接使用 [HOST:CONTAINER] 这样的格式，或者使用 [HOST:CONTAINER:ro] 这样的格式，后者对于容器来说，数据卷是只读的，这样可以有效保护宿主机的文件系统。
 Compose的数据卷指定路径可以是相对路径，使用 . 或者 .. 来指定相对目录。
 数据卷的格式可以是下面多种形式：



```jsx
volumes:
  // 只是指定一个路径，Docker 会自动在创建一个数据卷（这个路径是容器内部的）。
  - /var/lib/mysql

  // 使用绝对路径挂载数据卷
  - /opt/data:/var/lib/mysql

  // 以 Compose 配置文件为中心的相对路径作为数据卷挂载到容器。
  - ./cache:/tmp/cache

  // 使用用户的相对路径（~/ 表示的目录是 /home/<用户目录>/ 或者 /root/）。
  - ~/configs:/etc/configs/:ro

  // 已经存在的命名的数据卷。
  - datavolume:/var/lib/mysql
```

如果你不使用宿主机的路径，你可以指定一个volume_driver。



```undefined
volume_driver: mydriver
```

## 22. volumes_from

从其它容器或者服务挂载数据卷，可选的参数是 :ro或者 :rw，前者表示容器只读，后者表示容器对数据卷是可读可写的。默认情况下是可读可写的。



```css
volumes_from:
  - service_name
  - service_name:ro
  - container:container_name
  - container:container_name:rw
```

## 23. cap_add, cap_drop

添加或删除容器的内核功能。详细信息在前面容器章节有讲解，此处不再赘述。



```undefined
cap_add:
  - ALL

cap_drop:
  - NET_ADMIN
  - SYS_ADMIN
```

## 24. cgroup_parent

指定一个容器的父级cgroup。



```undefined
cgroup_parent: m-executor-abcd
```

## 25. devices

设备映射列表。与Docker client的--device参数类似。



```bash
devices:
  - "/dev/ttyUSB0:/dev/ttyUSB0"
```

## 26. extends

这个标签可以扩展另一个服务，扩展内容可以是来自在当前文件，也可以是来自其他文件，相同服务的情况下，后来者会有选择地覆盖原有配置。



```css
extends:
  file: common.yml
  service: webapp
```

用户可以在任何地方使用这个标签，只要标签内容包含file和service两个值就可以了。file的值可以是相对或者绝对路径，如果不指定file的值，那么Compose会读取当前YML文件的信息。
 更多的操作细节在后面的12.3.4小节有介绍。

## 27. network_mode

网络模式，与Docker client的--net参数类似，只是相对多了一个service:[service name] 的格式。
 例如：



```bash
network_mode: "bridge"
network_mode: "host"
network_mode: "none"
network_mode: "service:[service name]"
network_mode: "container:[container name/id]"
```

可以指定使用服务或者容器的网络。

## 28. networks

加入指定网络，格式如下：



```undefined
services:
  some-service:
    networks:
     - some-network
     - other-network
```

关于这个标签还有一个特别的子标签aliases，这是一个用来设置服务别名的标签，例如：



```undefined
services:
  some-service:
    networks:
      some-network:
        aliases:
         - alias1
         - alias3
      other-network:
        aliases:
         - alias2
```

相同的服务可以在不同的网络有不同的别名。

## 29. 其它

还有这些标签：cpu_shares, cpu_quota, cpuset, domainname, hostname, ipc, mac_address, mem_limit, memswap_limit, privileged, read_only, restart, shm_size, stdin_open, tty, user, working_dir
 上面这些都是一个单值的标签，类似于使用docker run的效果。



```ruby
cpu_shares: 73
cpu_quota: 50000
cpuset: 0,1

user: postgresql
working_dir: /code

domainname: foo.com
hostname: foo
ipc: host
mac_address: 02:42:ac:11:65:43

mem_limit: 1000000000
memswap_limit: 2000000000
privileged: true

restart: always

read_only: true
shm_size: 64M
stdin_open: true
tty: true
```

关于配置文件的扩展写法会在有空的时候补上，写得太长了



作者：左蓝
链接：https://www.jianshu.com/p/2217cfed29d7
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



# 多个docker-compose的网络通信

## 1：链接至已有的network

情况说明：链接至`docker run`创建的容器，或者已有 `docker network create YOUR_NET_NAME`创建的网络。

因 links 属性已被废弃，官方建议使用 networks 来将几个container划分至一个网络，从而实现container之间互通。

类似问题地址：https://stackoverflow.com/questions/39067295/docker-compose-external-container

To solve this, you can create your own network:

```
docker network create nacos-net
docker network connect nacos-net nacos-registry
docker network connect nacos-net mysql
docker network connect nacos-net redis
docker network connect nacos-net logstash
```

Then configure your docker-compose.yml with:

```
version: '2'

networks:
  dbnet:
    external:
      name: dbnet

services:
    wordpress:
        image: wordpress
        ports:
            - 80:80
        environment:
            WORDPRESS_DB_USER: root
            WORDPRESS_DB_PASSWORD: password
        volumes:
            - /var/www/somesite.com:/var/www/html
        networks:
          - dbnet
```

Note with recent versions of Docker, you shouldn't need to link the containers, the DNS service should do the name resolution for you.



```bash
# 查看容器网络
docker inspect 容器id
```

找到关键字 NetworkSettings 和 Networks 的第一项

```bash
"NetworkSettings": {
            "Bridge": "",
            "Networks": {
                "bridge": {
"NetworkSettings": {
            "Bridge": "",
            "Networks": {
                "docker_default": {
# 查看已创建的 network
docker network ls

NETWORK ID          NAME                DRIVER              SCOPE
f44b806ac378        bridge              bridge              local
fcd290516578        docker_default      bridge              local
48cb63767bf4        example_default     bridge              local
90eb98f75f53        host                host                local
b27959612ac8        none                null                local
```

设置 network

```yaml
version: "3"

services:
  openresty:
    container_name: openresty
    image: openresty/openresty
    restart: always
    networks:
      - YOUR_NET_NAME
networks:
  YOUR_NET_NAME:
    external: true
```





### 创建一个桥接网络

`docker network create YOUR_NET_NAME`
默认网络方式即为桥接
`YOUR_NET_NAME`为你自定义的网络名称

### 修改`docker-compose.yml`文件

1. 将`YOUR_NET_NAME`网络添加到 `container` 配置中
2. 在与`services`同级的 `networks` 配置中，添加 `YOUR_NET_NAME` 为 `external: true`，如：

**`openresty/docker-compose.yml`**

```yaml
version: "3"

services:
  openresty:
    container_name: openresty
    image: openresty/openresty
    restart: always
    networks:
      - YOUR_NET_NAME
    volumes:
      - ./conf:/usr/local/openresty/nginx/conf
      - ./web:/web
    ports:
      - "80:80"
      - "443:443"

networks:
  YOUR_NET_NAME:
    external: true
```

重新加载配置 `docker-compose up -d`

**`mariadb/docker-compose.yml`**

```yaml
version: "3"

services:
  mariadb:
    container_name: mariadb
    image: mariadb
    restart: always
    networks:
      - YOUR_NET_NAME
    environment:
      MYSQL_ROOT_PASSWORD: root
    volumes:
      - ./data:/var/lib/mysql
      - ./conf/my.cnf:/etc/mysql/my.cnf
    ports:
      - "13306:3306"

networks:
  YOUR_NET_NAME:
    external: true
```

重新加载配置 `docker-compose up -d`

注意：networks 中必须设置 external 为true，否则无法正常通信

### 完成

openresty 和 mariadb 现在可以互通了！
在 `openresty` 中，可以直接通过 `mariadb:3306` 来访问mariadb数据库



## 2：多个compose文件网络互联

实例一：

在下面的示例中，提供了三个服务（`web`，`worker`和`db`），以及两个网络（`new`和`legacy`）。 `db`服务在`new`网络上的主机名`db`或`database`，`legacy`网络中的`db`或`mysql`是可达的。

```yaml
version: '2'

services:
  web:
    build: ./web
    networks:
      - new

  worker:
    build: ./worker
    networks:
    - legacy

  db:
    image: mysql
    networks:
      new:
        aliases:
          - database
      legacy:
        aliases:
          - mysql

networks:
  new:
  legacy:
```

实例二：

> 在同一个docoker-compose中定义的`service`是直接可以通信的,docker-compose在启动后会自动创建默认的`default`网络用于内部通信，但是随着项目服务的增多，不可能所有的服务都定义在一个docker-compose文件中，为了便于维护，和解耦，一般根据业务类型和容器用途等，定义多个dcoker-compose文件来管理容器，比如基础环境（redis、mq服务等）定义，公共业务(注册中心，网关等)定义，普通业务服务等。而多个`docker-compose`定义的service相互不可以直接通信，需要额外配置，这里记录下来,以便后续使用

以基础服务`elk`中的`network`为基础，在`base`服务定义的容器中复用，具体的`docker-compose`文件如下

- 1.elk服务`docker-compose`文件

> 在最后声明一个`nwtworks`网络`elk`，这样运行，启动dockr-compose后会自动创建对应的网络，也可以使用`docker network create ***`来创建



```yaml
version: "3"
services:
  elasticsearch:
    image: "elasticsearch:7.1.1"
    container_name: "elasticsearch"
    restart: "always"
    volumes:
      - "elasticsearch:/usr/share/elasticsearch"
    #vim /etc/sysctl.conf
    #vm.max_map_count=262144
    #sysctl -w vm.max_map_count=262144
    #sysctl -p
    environment:
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - discovery.type=single-node
    networks:
      - "elk"
    ports:
      - "9200:9200"
      - "9300:9300"
  kibana:
    image: "kibana:7.1.1"
    container_name: "kibana"
    restart: "always"
    depends_on:
      - elasticsearch
    volumes:
      - "kibana:/usr/share/kibana"
    networks:
      - "elk"
    ports:
      - "5601:5601"
  cerebro:
    image: "lmenezes/cerebro"
    restart: "always"
    container_name: "cerebro"
    ports: 
      - "9000:9000"
networks:
  elk:

volumes:
  elasticsearch:
  kibana:
```

> 在需要连接通信的`docker-compose`中，文件最后声明networks，主要这里的名称是`elk_elk`,使用`docker network ls`

- 2.基础服务`docker-compose`文件



```yaml
version: '3'
services:
  mysql:
    image: mysql:5.7.22
    restart: always
    privileged: true
    container_name: mysql-5.7
    volumes:
      - ./mysql/data:/var/lib/mysql
      - ./mysql/config/my.cnf:/etc/my.cnf
      - ./mysql/init:/docker-entrypoint-initdb.d/
    environment:
      TZ: Asia/Shanghai
      MYSQL_ROOT_PASSWORD: root
    ports:
      - '3306:3306'
  redis:
    image: redis:4.0
    restart: always
    container_name: redis
    ports:
      - 6379:6379
    volumes:
      - ./redis/data:/data
    command: redis-server
  sjy-api:
    image: sjy-api:latest
    restart: always
    container_name: sjy-api
    environment:
      - JVM_OPTS=-XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=128m -Xms512m -Xmx1024m -Xmn256m -Xss256k -XX:SurvivorRatio=8 -XX:+UseConcMarkSweepGC
    volumes:
      - ./sjy/logs:/logs
      - /etc/localtime:/etc/localtime
    ports:
      - 5050:5050
    links:
      - mysql
      - redis
    external_links:
      - elasticsearch
    depends_on:
      - mysql
      - redis
networks:
  default:
    external:
     name: elk_elk
```

- 使用`docker-compose up -d --build`启动，进入容器内部检查网络情况



```csharp
root@zssy-test:/opt/docker/base# docker exec -it sjy-api /bin/sh
/ # ping elasticsearch
PING elasticsearch (172.18.0.2): 56 data bytes
64 bytes from 172.18.0.2: seq=0 ttl=64 time=0.137 ms
64 bytes from 172.18.0.2: seq=1 ttl=64 time=0.117 ms
64 bytes from 172.18.0.2: seq=2 ttl=64 time=0.110 ms
64 bytes from 172.18.0.2: seq=3 ttl=64 time=0.098 ms
64 bytes from 172.18.0.2: seq=4 ttl=64 time=0.106 ms
64 bytes from 172.18.0.2: seq=5 ttl=64 time=0.107 ms
```



作者：Hi哈娃娃
链接：https://www.jianshu.com/p/1099815985dd