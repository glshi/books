## zabbix 搭建

```bash
docker-compose -f ./docker-compose_v3_ubuntu_mysql_latest.yaml up -d
```

## Jenkins 搭建

```bash
docker run -p 8080:8080 -p 50000:50000 -v /your/home:/var/jenkins_home jenkins
```

## varnish 搭建

1：新建文件`default.vcl` file:

```bash
# we need both a configuration file at /etc/varnish/default.vcl
# and our workdir to be mounted as tmpfs to avoid disk I/O
```

```bash
vcl 4.0;

backend default {
  .host = "www.nytimes.com:80";
}
```

2：启动容器

```bash
docker run --name varnish -v /path/to/default.vcl:/etc/varnish/default.vcl:ro --tmpfs /var/lib/varnish:exec -itd -e VARNISH_SIZE=2G -p 8080:80 varnish

docker run --name varnish -v /mnt/d/shareFiles/docker/varnish/default.vcl:/etc/varnish/default.vcl:ro --tmpfs /var/lib/varnish:exec -itd -e VARNISH_SIZE=2G -p 8080:80 varnish -p http_max_hdr=256 -p http_req_hdr_len=8192
```

## nginx搭建

### 一：部署web项目

```bash
# 启动nginx
docker run --name nginx-test -p 80:80 -d nginx
# 创建映射目录
mkdir -p /home/nginx/www /home/nginx/logs /home/nginx/conf
# 将nginx-test容器配置文件copy到本地
docker cp nginx-test:/etc/nginx/nginx.conf /home/nginx/conf
# 创建新nginx容器nginx-web,并将www,logs,conf目录映射到本地
docker run -d -p 80:80 --name nginx-web -v /home/nginx/www:/usr/share/nginx/html -v /home/nginx/conf/nginx.conf:/etc/nginx/nginx.conf -v /home/nginx/logs:/var/log/nginx nginx
# 在本机/home/nginx/www目录下放入打包好的vue或者web项目
```

### 二：配置反向代理

```bash
# 在conf.d目录下面，编辑default.conf，添加下面的代理内容
location /wiki/ {
    rewrite ^/wiki/(.*)$ /$1 break;
    proxy_pass http://xxx.com.cn;
}
```

rewrite 的语法是（来自[文档](https://link.segmentfault.com/?url=http%3A%2F%2Fnginx.org%2Fen%2Fdocs%2Fhttp%2Fngx_http_rewrite_module.html%23rewrite)）：rewrite regex replacement [flag];

所以上面的效果是匹配 `^/wiki/(.*)$` 然后替换为 `/` 加匹配到的后面括号后的分块。

按照上面的配置，重启 nginx `./nginx -s reload`，直接在浏览器输入 `http://localhost/wiki/rest/api/2/user/picker` 就等于访问 `http://xxx.com.cn/rest/api/2/user/picker` 啦。

### 三：配置负载均衡

```bash
# 1、轮询：轮询方式是Nginx负载默认的方式
upstream  geobase-server {
       server    localhost:10001;
       server    localhost:10002;
}
# 2、权重：指定每个服务的权重比例，weight和访问比率成正比
upstream  geobase-server {
       server    localhost:10001 weight=1;
       server    localhost:10002 weight=2;
}
# 3、iphash：每个请求都根据访问ip的hash结果分配
upstream  geobase-server {
       ip_hash; 
       server    localhost:10001 weight=1;
       server    localhost:10002 weight=2;
}
# 4、最少连接：将请求分配到连接数最少的服务上
upstream  geobase-server {
       least_conn;
       server    localhost:10001 weight=1;
       server    localhost:10002 weight=2;
}
# 5、fair：按后端服务器的响应时间来分配请求，响应时间短的优先分配
upstream  geobase-server {
       server    localhost:10001 weight=1;
       server    localhost:10002 weight=2;
       fair;  
}
```

以轮训为例，如下是nginx.conf完整代码

```
worker_processes  1;

events {
    worker_connections  1024;
}

http {
   upstream  geobase-server {
       server    localhost:10001;
       server    localhost:10002;
   }

   server {
       listen       10000;
       server_name  localhost;

       location / {
        proxy_pass http://geobase-server;
        proxy_redirect default;
      }
    }
}
```



## portainer 搭建

### 一：安装

```bash
docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce
```

### 二：远程2375端口异常

第一步：查看 `docker.service` 的位置

```bash
systemctl status docker
```

第二步：编辑`docker.service` 文件

```bash
# 1. 编辑docker.service
vim /usr/lib/systemd/system/docker.service
# 找到 ExecStart字段修改如下
ExecStart=/usr/bin/dockerd-current -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock 
# 2. 重启docker重新读取配置文件，重新启动docker服务
systemctl daemon-reload
systemctl restart docker
```

第三步：开发防火墙

```bash
# 3. 开放防火墙端口
systemctl status firewalld
firewall-cmd --zone=public --add-port=2375/tcp --permanent
# 4. 刷新防火墙
firewall-cmd --reload
# 5. 再次配置远程docker就可以了
# 6.如果重启不起来 估计是这个 unix://var/run/docker.sock 文件位置不对 
find / -name docker.sock 查找一下正确位置就好了
```

第四步：远程客户端测试：

```bash
docker -H 192.168.1.248 info 
```

### 三：异常

**1：Error response from daemon: OCI runtime create failed: container_linux.go:349: starting container process caused “process_linux.go:449: container init caused “write /proc/self/attr/keycreate: permission denied””: unknown.****


```bash
vi /etc/selinux/config
# 更改SELINUX=disable
reboot
```



## gitlab 搭建

### 一：安装

```bash
export GITLAB_HOME=/data/gitlab

docker run --detach \
  --hostname gitlab.example.com \
  --publish 443:443 --publish 8080:80 --publish 22:22 \
  --name gitlab \
  --restart always \
  --volume $GITLAB_HOME/config:/etc/gitlab \
  --volume $GITLAB_HOME/logs:/var/log/gitlab \
  --volume $GITLAB_HOME/data:/var/opt/gitlab \
  gitlab/gitlab-ce:latest
```

### 二：数据备份与恢复

Depending on your version of GitLab, use the following command if you installed GitLab using the Omnibus package:

- GitLab 12.2 or later:

  ```bash
  sudo gitlab-backup create
  ```

- GitLab 12.1 and earlier:

  ```bash
  gitlab-rake gitlab:backup:create
  ```

If you installed GitLab from source, use the following command:

```bash
sudo -u git -H bundle exec rake gitlab:backup:create RAILS_ENV=production
```

If you’re running GitLab from within a Docker container, run the backup from the host, based on your installed version of GitLab:

- GitLab 12.2 or later:

  ```bash
  docker exec -t <container name> gitlab-backup create
  ```

- GitLab 12.1 and earlier:

  ```bash
  docker exec -t <container name> gitlab-rake gitlab:backup:create
  ```

### **三：gitlab runner**

1：docker 安装

```bash
docker run -d --name gitlab-runner --restart always \
-v /data/gitlab-runner/config:/etc/gitlab-runner \
-v /var/run/docker.sock:/var/run/docker.sock \
gitlab/gitlab-runner:latest
```

2：注册一个runner

```bash
docker run --rm -it -v /data/gitlab-runner/config:/etc/gitlab-runner gitlab/gitlab-runner register
```

1. Enter your GitLab instance URL (also known as the `gitlab-ci coordinator URL`).
2. Enter the token you obtained to register the runner.
3. Enter a description for the runner. You can change this value later in the GitLab user interface.
4. Enter the [tags associated with the runner](https://docs.gitlab.com/ee/ci/runners/#using-tags), separated by commas. You can change this value later in the GitLab user interface.
5. Provide the [runner executor](https://docs.gitlab.com/runner/executors/index.html). For most use cases, enter `docker`.
6. If you entered `docker` as your executor, you’ll be asked for the default image to be used for projects that do not define one in `.gitlab-ci.yml`.

第二种注册方法：https://cloud.tencent.com/developer/article/1803787

```bash
docker exec -it gitlab-runner bash
gitlab-runner register
gitlab-runner restart
gitlab-runner list 
```

### 四：CI、CD集成

1、项目根目录添加 `.gitlab-ci.yml` 文件

1.1、springboot集成：

```yml
image: maven:3.6.3-jdk-8

variables:
  MAVEN_CLI_OPTS: "-s .m2/settings.xml --batch-mode"
  MAVEN_OPTS: "-Dmaven.repo.local=.m2/repository"
  DOCKER_DRIVER: overlay2
  DOCKER_HOST: tcp://docker:2375
  DOCKER_TLS_CERTDIR: ""

# 定义缓存
# 如果gitlab runner是shell或者docker，此缓存功能没有问题
# 如果是k8s环境，要确保已经设置了分布式文件服务作为缓存
cache:
  key: docker-ci-cache
  paths:
  - .m2/repository/
  - target/*.jar

# 本次构建的阶段：build package
stages:
- package
- build

# 生产jar的job
make_jar:
  image: maven:3.6.3-jdk-8
  stage: package
  tags:
  - k8s
  script:
  - echo "=============== 开始编译源码，在target目录生成jar文件 ==============="
  - mvn $MAVEN_CLI_OPTS clean compile package -Dmaven.test.skip=true
  - echo "target文件夹" `ls target/`

# 生产镜像的job
make_image:
  image: docker:latest
  stage: build
  tags:
  - k8s
  script:
  - echo "从缓存中恢复的target文件夹" `ls target/`
  - echo "==打包Docker镜像 ： " gitlabci-java-demo:$CI_COMMIT_SHORT_SHA "=="
  - docker build -t 192.168.140.50:5000/geobase:$CI_COMMIT_SHORT_SHA .
  - docker tag 192.168.140.50:5000/geobase:$CI_COMMIT_SHORT_SHA 192.168.140.50:5000/geobase:latest
  - echo "=============== 推送到镜像仓库  ==============="
  - docker push 192.168.140.50:5000/geobase:$CI_COMMIT_SHORT_SHA
  - docker push 192.168.140.50:5000/geobase:latest
  - echo "清理掉本次构建的jar文件"
  - rm -rf target/*.jar
```

1.2、vue项目集成

```

```

2、项目根目录添加 `docker-compose.yml` 文件

```yaml
version: '2.0'
services:
	geobase:
		image: 192.168.140.50:5000/geobase:latest
		container_name: geobase
		ports:
		- "8090:8090"
```

### 五：git 配置 ssh key

**1、配置 git**

```bash
$ git config --global user.name "John Doe"
$ git config --global user.email johndoe@example.com
$ git config --list
```

**2、生成密钥**

```bash
ssh-keygen -t rsa -C "your_email@youremail.com" 
# 在~/.ssh/下会生成两个文件，id_rsa和id_rsa.pub
# 复制 id_rsa.pub 中的内容到 gitlab 的 SSH Keys 中
```



## nexus 搭建

### 一：安装

```bash
# 方式一
$ docker volume create --name nexus-data
$ docker run -d -p 8081:8081 --name nexus -v nexus-data:/nexus-data sonatype/nexus3

# 方式二
$ mkdir /some/dir/nexus-data && chown -R 200 /some/dir/nexus-data
$ docker run -d -p 8081:8081 --name nexus -v /some/dir/nexus-data:/nexus-data sonatype/nexus3
```

**Nexus3首次登录默认密码**

目前的Nexus3用户名是admin，但是默认密码不再是以前的amdin123了

那么就需要在启动的容器中去寻找密码

```bash
docker exec -it 容器id bash  # 进入容器
cat /opt/sonatype/sonatype-work/nexus/admin.password
```

### **二：nexus3重置admin密码**

停止服务

```bash
# 进入到/opt/sonatype/nexus/bin目录
./nexus stop
```

进入OrientDB控制台

```bash
# 进入到/opt/sonatype/nexus/lib目录
java -jar ./support/nexus-orient-console.jar
```

进入数据库
注意改…/sonatype-work此处为相对目录，所以上一步要求切换到nexus目录下，如果用绝对目录则不需要

```bash
# 进入到/opt/sonatype/目录
connect plocal:../sonatype-work/nexus3/db/security admin admin
```

重置admin密码为admin123

```bash
# 重置密码
update user SET password="$shiro1$SHA-512$1024$NE+wqQq/TmjZMvfI7ENh/g==$V4yPw8T64UQ6GfJfxYq2hLsVrBY8D1v+bktfOxGdt4b/9BthpWPNUy/CBk6V9iA0nHpzYzJFWO8v/tZFtES8CA==" UPSERT WHERE id="admin"
```

重新登录修改密码
登录之后打开设置，重新设置密码即可



### **三：命令导入本地repository到nexus**

1：创建如下安装脚本，`mavenimport.sh`

```bash
#!/bin/bash
# copy and run this script to the root of the repository directory containing files
# this script attempts to exclude uploading itself explicitly so the script name is important
# Get command line params
while getopts ":r:u:p:" opt; do
    case $opt in
        r) REPO_URL="$OPTARG"
        ;;
        u) USERNAME="$OPTARG"
        ;;
        p) PASSWORD="$OPTARG"
        ;;
    esac
done
  
find . -type f -not -path './mavenimport\.sh*' -not -path '*/\.*' -not -path '*/\^archetype\-catalog\.xml*' -not -path '*/\^maven\-metadata\-local*\.xml' -not -path '*/\^maven\-metadata\-deployment*\.xml' | sed "s|^\./||" | xargs -I '{}' curl -u "$USERNAME:$PASSWORD" -X PUT -v -T {} ${REPO_URL}/{} ;
```

2：在maven仓库的`repository`目录下执行脚本

```bash
./mavenimport.sh -u admin -p geovis123 -r http://192.168.140.40:8081/repository/maven-releases/
```



## registry 搭建

### 一：registry安装

```bash
$ docker run -d \
  -p 5000:5000 \
  --restart=always \
  --name registry \
  -v /mnt/registry:/var/lib/registry \
  registry:2
  
  
 $ docker run -d \
  --restart=always \
  --name registry \
  -v "$(pwd)"/certs:/certs \
  -e REGISTRY_HTTP_ADDR=0.0.0.0:443 \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
  -p 443:443 \
  registry:2
```

**放开http访问，在/etc/docker/daemon.json 加入如下配置**

```bash
{
     "insecure-registries":["192.168.140.50:5000"]
 }
```

重启docker

```bash
systemctl restart docker
docker image tag mariadb:10.5 192.168.140.50:5000/mariadb:10.5
docker push localhost:5000/mariadb:10.5
```

查看已上传镜像：http://192.168.140.50:5000/v2/_catalog

### **二：registry-web UI 安装**

```bash
docker pull hyper/docker-registry-web

docker run -itd -p 8080:8080 --name registry-web --link registry -e REGISTRY_URL=http://registry:5000/v2 -e REGISTRY_NAME=localhost:5000 hyper/docker-registry-web 

# 如果使用https，用下面的启动
docker run -it -p 8080:8080 --name registry-web --link registry-srv \
           -e REGISTRY_URL=https://registry-srv:5000/v2 \
           -e REGISTRY_TRUST_ANY_SSL=true \
           -e REGISTRY_BASIC_AUTH="YWRtaW46Y2hhbmdlbWU=" \
           -e REGISTRY_NAME=localhost:5000 hyper/docker-registry-web
 
# 使用 joxit/docker-registry-ui 成功启动
docker run -p 8080:80 --name registry-ui \
-e DELETE_IMAGES="true" \
-e REGISTRY_TITLE="Registry" \
-d joxit/docker-registry-ui:latest


docker run -p 8080:80 --name registry-ui \
--link registry:registry \
-e REGISTRY_URL="http://registry:5000" \
-e DELETE_IMAGES="true" \
-e REGISTRY_TITLE="Registry" \
-d joxit/docker-registry-ui:latest
```

**joxit/docker-registry-ui 配置跨域和删除镜像**

1：进入 `registry` 容器

```bash
docker exec -it registry /bin/sh
```

修改`/etc/docker/registry/config.yml` 文件如下

```yaml
version: 0.1
log:
  fields:
    service: registry
storage:
  delete:
    enabled: true
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry
http:
  addr: :5000
  headers:
    X-Content-Type-Options: [nosniff]
    Access-Control-Allow-Origin: ['*']
    Access-Control-Allow-Methods: ['HEAD', 'GET', 'OPTIONS', 'DELETE']
    Access-Control-Expose-Headers: ['Docker-Content-Digest']
auth:
  htpasswd:
    realm: basic-realm
    path: /etc/docker/registry/htpasswd
```

## 服务器列表

### 一：服务器地址

大家好，请大家自行配置电脑ip的时候配置为192.168.140.60至192.168.140.230以内的ip地址，不要和服务器地址冲突，请140.1至140.60段的电脑IP地址修改为140.60至140.230以内。

一：现在的服务器IP地址

> 192.168.140.61
>
> 192.168.140.62
>
> 192.168.140.63
>
> 192.168.140.66
>
> 192.168.140.68
>
> 192.168.140.83
>
> 192.168.140.100
>
> 192.168.140.140
>
> 192.168.140.198
>
> 192.168.140.199
>
> 192.168.140.200
>
> 192.168.140.250
>
> 192.168.140.251

二：后期ip地址改造

192.168.140.1至192.168.140.60为服务器IP地址段；

> 服务器临时环境IP段：140.1-140.19
>
> 服务器开发环境IP段：140.20-140.29
>
> 服务器测试环境IP段：140.30-140.39
>
> 服务器生产环境IP段：140.40-140.49
>
> 服务器其他环境IP段：140.50-140.59
>

大家自己电脑配置ip的时候请配置140.60至140.230以内的ip地址。

三：服务器IP地址后期迁移

> 192.168.140.61   迁移至 192.168.140.21
>
> 192.168.140.62   迁移至 192.168.140.22
>
> 192.168.140.63   迁移至 192.168.140.23
>
> 192.168.140.66   迁移至 192.168.140.26
>
> 192.168.140.68   迁移至 192.168.140.28
>
> 192.168.140.83   迁移至 192.168.140.40
>
> 192.168.140.100 迁移至 192.168.140.53
>
> 192.168.140.140 迁移至 192.168.140.20
>
> 192.168.140.198 迁移至 192.168.140.46
>
> 192.168.140.199 迁移至 192.168.140.43
>
> 192.168.140.200 迁移至 192.168.140.52
>
> 192.168.140.250 迁移至 192.168.140.50
>
> 192.168.140.251 迁移至 192.168.140.51

2021-07-19 服务器暂时规划

```bash
192.168.140.66 -- 140.26 开发应用 原来是国产麒麟环境 
192.168.140.140---140.20 开发数据库 

192.168.140.198---140.46 生产应用
192.168.140.83--  140.40 生产数据库 

192.168.140.250---140.50 事业部数据存档 registry portainer seafile
```

### 二：服务器密码

物理机

> 140.46：root/geovis12#$
>
> 140.26：root/geovis12#$
>
> 140.20：root/geovis12#$
>
> 140.40：root/geovis12#$
>
> 140.50：root/geovis12#$
>
> 140.61：root/123.com
>
> 140.60：root/123.com

软件

> seafile:admin@example.com/geovis123、public@geovis.com/123456、data@geovis.com/geovis123
>
> gitlab: root/geovis123、shiguangli/geovis123
>
> nexus:admin/geovis123
>
> portainer：admin/geovis123、dev/123456、pro/geovis123
>

数据库

> 140.40 oracle 1521  system/oracle,sys/oracle
>
> 140.40 postgres 5432  postgres/geovis123
>
> 140.40 mysql 3306  root/geovis123
>
> 140.40 sqlserver 1433  SA/geovis123

虚拟机

Geovis123、geovis123、123456

## 数据库常用操作

### 一：mysql

**1、安装**

```bash
docker run --name mysql --restart=always -v mysqldata:/var/lib/mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7.29
```

**2、修改root账户密码**

```bash
# 记得密码
mysql -u root -p
mysql> use mysql;
mysql> UPDATE user SET password = PASSWORD('新密码') WHERE user = 'root'; 
mysql> FLUSH PRIVILEGES;
# 忘记密码
# KILL掉系统里的MySQL进程；
mysqld_safe --skip-grant-tables & mysql -u root mysql
mysql> UPDATE user SET password=PASSWORD('新密码') WHERE user='root';
mysql> flush privileges;
```

**3、连接到server**

```bash
sudo docker exec -it mysql "bash"
mysql -u root -p 
```

**4、创建数据库**

```bash
create database testDB default charset utf8mb4 collate utf8mb4_unicode_ci;
# 本地授权
grant all privileges on testDB.* to 'myuser'@'localhost' identified by 'mypassword';
# 远程授权
grant all privileges on testDB.* to 'myuser'@'%' identified by 'mypassword'; 
# 刷新系统权限表
flush privileges;

# 导入数据
docker cp ./data/szdq/pg/alldata.dump 93f03cde72c6:/home/
mysql -uroot -p < ./alldata.dump
```

**5、创建用户并授权**

```bash
create user base@'%' identified by 'base';
grant all privileges on base.* to base;

CREATE USER 'myuser'@'localhost' IDENTIFIED BY 'mypassword'; #本地登录 
CREATE USER 'myuser'@'%' IDENTIFIED BY 'mypassword'; #远程登录 
quit 
mysql -u myuser -p #测试是否创建成功
```

**6、卸载**

```bash
sudo docker stop mysql
sudo docker rm mysql
```



### 二：postgresql

**1、安装**

```bash
docker run -v pgdata:/root/data --name pg12 --restart=always -e POSTGRES_PASSWORD=postgres -p 5432:5432 -d postgres:12.0.1
```

**2、修改账户密码**

```bash
# 默认用户名postgres
psql -U postgres
alter user postgres with password 'new password';
```

**3、连接到server**

```bash
sudo docker exec -it pg12 "bash"
psql -U postgres
```

**4、创建数据库**

```bash
# 创建数据库 --指定字符集  --设置数据库所有者
create DATABASE ktxmch ENCODING='utf-8' TABLESPACE=pg_default owner ktchsys;

# 导入数据
docker cp ./data/szdq/pg/alldata.dump 93f03cde72c6:/home/
psql -U postgres < ./alldata.dump
```

**5、创建用户并授权**

```bash
# 创建用户
create  user ktchsys  with password 'ktchsys';

# 将 ktxmch 数据库的所有权限赋予 ktchsys，否则 ktchsys 只能登录psql，没有任何数据库操作权限：
grant all privileges on database ktxmch  to ktchsys;
```

**6、卸载**

```bash
sudo docker stop pg12
sudo docker rm pg12
```



### 三：oracle

**1、安装**

```bash
docker run -itd -p 1521:1521 --restart=on-failure:5 -e ORACLE_ALLOW_REMOTE=true --name oracle11g  wnameless/oracle-xe-11g-r2:latest
```

**2、修改账户密码**

```bash
# 默认用户和密码
# Password for SYS & SYSTEM is oracle
```

**3、连接到server**

```bash
cd /u01/app/oracle/product/11.2.0/xe/bin
su oracle
./sqlplus / as sysdba
```

**4、创建数据库**

```mysql
create tablespace GFWZDB datafile '/u01/app/oracle/GFWZDB.dbf' size 200m autoextend on next 100m maxsize unlimited;
```

**5、创建用户并授权**

```mysql
create user GFWZDB identified by GFWZDB default tablespace GFWZDB temporary tablespace temp;
# 一般此授权即可
grant connect,resource to GFWZDB;
grant connect,resource,dba to GFWZDB;
revoke connect,resource,dba from GFWZDB;
# 删除用户及数据
drop user GFWZDB cascade;

# 导入数据
start d:\document\dbdata\GFWZDB.sql
```

**6、卸载**

```bash
sudo docker stop sql1
sudo docker rm sql1
```



### 四：sqlserver

**1、安装**

```bash
sudo docker pull mcr.microsoft.com/mssql/server:2019-latest
sudo docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=Geovis123com" \
   -p 1433:1433 --name sql1 -h sql1 \
   -v sqlvolume:/var/opt/mssql \
   -d mcr.microsoft.com/mssql/server:2019-latest
```

**2、修改sa账户密码**

```bash
sudo docker exec -it sql1 /opt/mssql-tools/bin/sqlcmd \
   -S localhost -U SA -P "<YourStrong@Passw0rd>" \
   -Q 'ALTER LOGIN SA WITH PASSWORD="<YourNewStrong@Passw0rd>"'
```

**3、连接到sqlserver**

```bash
# 方式一
sudo docker exec -it sql1 "bash"
/opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P "<YourNewStrong@Passw0rd>"
# 方式二
sqlcmd -S 192.168.140.20,1433 -U SA -P 'Geovis123#com'
```

**4、创建数据库**

```bash
CREATE DATABASE TestDB
SELECT Name from sys.Databases
GO
QUIT
```

**5、创建用户并授权**

```bash

# 方式一，此登录账户默认拥有数据库JIAOGUAN的权限
create login test with password='test',default_database=JIAOGUAN
# 方式二
use JIAOGUAN
# dbo为DataBaseOwner的简写，每个数据库都有一个dbo用户
create user test for login test with default_schema=dbo 
# 将test加入db_owner角色
exec sp_addrolemember 'db_owner', 'test'
go


# 创建用户
create login username with password='密码' , default_database=数据库;
create user username for login username with default_schema=dbo;
# 增删改查授权
grant select,insert,UPDATE,DELETE on 表 to username
# 存储过程授权
GRANT EXECUTE ON 存储过程名 TO username
# 禁止对表授权
DENY UPDATE ON 表 TO username CASCADE;
# 回收权限
REVOKE DELETE ON 表 FROM username
go
# Granting INSERT permission on schema HumanResources to guest
GRANT INSERT ON SCHEMA :: HumanResources TO guest;
```

**6、卸载**

```bash
sudo docker stop sql1
sudo docker rm sql1
```

**7、18456或者10054异常**

```bash
# 安装sqlcmd连接
# 配置密码必须包含大写字母、小写字母、数字和特殊字符（#%）四项
```



### 五：mariadb

**1、安装**

```bash
# 映射配置文件
docker run --name some-mariadb -v /my/custom:/etc/mysql/conf.d -e MARIADB_ROOT_PASSWORD=my-secret-pw -d mariadb:tag
# 映射存储路径
docker run --name some-mariadb -v /my/own/datadir:/var/lib/mysql -e MARIADB_ROOT_PASSWORD=my-secret-pw -d mariadb:tag
```

**2、修改账户密码**

```bash
root/my-secret-pw
# 同 mysql
```

**3、连接到server**

```bash
docker exec -it some-mariadb bash
```

**4、创建数据库**

```bash

# Creating database dumps
$ docker exec some-mariadb sh -c 'exec mysqldump --all-databases -uroot -p"$MARIADB_ROOT_PASSWORD"' > /some/path/on/your/host/all-databases.sql

# Restoring data from dump files
$ docker exec -i some-mariadb sh -c 'exec mysql -uroot -p"$MARIADB_ROOT_PASSWORD"' < /some/path/on/your/host/all-databases.sql
```

**5、创建用户并授权**

```bash
create database testDB default charset utf8mb4 collate utf8mb4_unicode_ci;
# 本地授权
grant all privileges on testDB.* to 'myuser'@'localhost' identified by 'mypassword';
# 远程授权
grant all privileges on testDB.* to 'myuser'@'%' identified by 'mypassword'; 
# 刷新系统权限表
flush privileges;
```

**6、卸载**

```bash
sudo docker stop some-mariadb
sudo docker rm some-mariadb
```



### 六：elasticsearch

**1、安装**

```bash
$ grep vm.max_map_count /etc/sysctl.conf
vm.max_map_count=262144

$ sysctl -w vm.max_map_count=262144

docker run -p 9200:9200 -p 9300:9300 --restart=always --name elasticsearch \
-e "discovery.type=single-node" \
-e "cluster.name=elasticsearch" \
-v /data/esdata/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
-v /data/esdata/elasticsearch/data:/usr/share/elasticsearch/data \
-d elasticsearch:7.6.2

```

**2、修改账户密码**

```bash

```

**3、连接到server**

```bash

```

**4、创建数据库**

```bash

```

**5、创建用户并授权**

```bash

```

**6、卸载**

```bash

```



### 七：redis

**1、安装**

```bash
docker run --name some-redis -v redis_data:/data -d redis redis-server --appendonly yes
```

**2、修改sa账户密码**

```bash

```

**3、连接到sqlserver**

```bash

```

**4、创建数据库**

```bash

```

**5、创建用户并授权**

```bash

```

**6、卸载**

```bash

```



### 八：mongodb

**1、安装**

```bash
docker run --name some-mongo -v /my/own/datadir:/data/db -d mongo
```

**2、修改账户密码**

```bash

```

**3、连接到server**

```bash

```

**4、创建数据库**

```bash

```

**5、创建用户并授权**

```bash

```

**6、卸载**

```bash

```

### 九：rabbitmq

**1、安装**

```bash

```



### 十：Neo4j

**1、安装**

```bash
docker run -d --restart=always --name neo4j --publish=7474:7474 --publish=7687:7687 -e 'NEO4J_AUTH=neo4j/secret'  neo4j:4.1.5
```

**2、修改账户密码**

```bash
neo4j/secret
```

**3、添加用户**

neo4j退出页面重新登录

```bash
:server disconnect
```

用户创建;

```bash
CALL dbms.security.createUser(name,password,requridchangepassword) 
```

> 其中nam参数是你的用户名，password是密码，requridchangepassword是表示是否需要修改密码，布尔类型。如下创建一个test1用户，密码是test1,不需要修改密码。

```bash
CALL dbms.security.createUser('test','test',false) 
```


查看当前用户

```bash
CALL dbms.security.showCurrentUser()
```


查看所有用户

查看所有用户

```bash
CALL dbms.security.listUsers()
```

删除用户

```bash
# username参数表示你要删除的用户名。
CALL dbms.security.deleteUser('username')  
```

修改密码

```bash
CALL dbms.security.changePassword('password')
```

> 注意：password参数不能为空，或者跟原密码相同，不然会报错





## 常用命令操作

### 一、设置主机名

```bash
# 设置主机名
vi  /etc/hostname
# 设置DNS host解析
vi  /etc/hosts
```

### 二、配置网关

```bash
vi  /etc/sysconfig/network
```

### 三、配置静态IP地址 

```bash
# 查看本机网络配置
ifconfig
# 选择网卡并配置ip地址
vi/etc/sysconfig/network-scripts/ifcfg-enp2s0
# 将ONBOOT=no改为yes，将BOOTPROTO=dhcp改为BOOTPROTO=static，并添加下面四个配置
IPADDR=192.168.140.21
NETMASK=255.255.255.0
GATEWAY=192.168.140.254
DNS1=8.8.8.8
# 重启网关
systemctl restart network.service
systemctl restart network
service network restart
/etc/init.d/network restart
```

### 四、管理员root密码修改

**1：记得root密码修改方式**

```bash
# 1.修改系统用户root密码
[root@ITCATS-01 ~]# passwd
更改用户 root 的密码 。
新的密码：

# 2.修改系统非root用户密码：huazi
[root@ITCATS-01 ~]# passwd huazi
更改用户 huazi 的密码 。
新的密码
```

**2：忘记root密码修改方式**

重启开机，看到下图按e

 ![img](https://img2018.cnblogs.com/blog/1573482/201812/1573482-20181227102200081-552009423.png)

编辑修改两处：ro改为rw,在LANG=en_US.UFT-8后面添加init=/bin/sh

 ![img](https://img2018.cnblogs.com/blog/1573482/201812/1573482-20181227102212849-346361938.png)

然后按【Ctrl+X】进入“单用户模式”，依次执行下列命令

![img](https://img2018.cnblogs.com/blog/1573482/201812/1573482-20181227102230637-1971153107.png)

![img](https://img2018.cnblogs.com/blog/1573482/201812/1573482-20181227102238053-562191328.png)

> 重新挂载文件系统为可写入    **mount -o remount,rw /**
>
> **修改密码** **echo 密码 | passwd –stdin root**
>
> **如果之前系统启用了selinux，必须执行以下命令，否则将无法正常启动系统：****touch  /.autorelabel**
>
> 然后执行命令**exec /sbin/init**来正常启动



### 五、查看内核版本命令

```bash
# 两种方法
1、cat /proc/version
2、uname -a
```

### 六、查看系统版本的命令

```bash
# 3种方法
1、lsb_release -a，这个命令适用于所有的Linux发行版，包括RedHat、SUSE、Debian…等发行版。
2、cat /etc/redhat-release，这种方法只适合Redhat系的Linux：
3、cat /etc/issue，此命令也适用于所有的Linux发行版。
```

### 七、查看网卡信息

```bash
# 查看网卡信息
ifconfig
ip addr
# 查看网卡的信息
ethtool eno2
# 监控网卡实时流量信息
nethogs eno2
iftop -i eno2 -n  -P
```

### 八、查看硬件信息

```bash
# 1.查看CPU信息命令
cat /proc/cpuinfo
# 2.查看内存信息命令
cat /proc/meminfo
# 3.查看硬盘信息命令
fdisk -l
# 4.查看CPU信息（型号） ，看到有8个逻辑CPU, 也知道了CPU型号
cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c 
8  Intel(R) Xeon(R) CPU            E5410   @ 2.33GHz 
# 5.查看物理CPU的个数
cat /proc/cpuinfo | grep "physical id" | sort | uniq | wc -l
# 输出结果：
# 2
# 表示Linux服务器上面实际安装了2个物理CPU芯片。

# 6.查看物理CPU内核的个数
cat /proc/cpuinfo | grep "cpu cores" | uniq
# 输出结果：
# cpu cores : 8
# 表示1个物理CPU里面有8个物理内核。

# 6.查看所有逻辑CPU的个数
cat /proc/cpuinfo | grep "processor" | wc -l
# 输出结果：
# 32
# 表示Linux服务器一共有32个逻辑CPU。

# 8.查看每个物理CPU中逻辑CPU的个数
cat /proc/cpuinfo | grep 'siblings' | uniq
# 输出结果：
# siblings : 16
# 表示每个物理CPU中有16个逻辑CPU，
# 一共有2个物理CPU，
# 所以总共有32个逻辑CPU，
# 和第5步中查看的结果一致。

# 9.查询CPU是否启用超线程
cat /proc/cpuinfo | grep -e "cpu cores" -e "siblings" | sort | uniq
# 输出结果：
# cpu cores : 8
# siblings : 16
# 看到cpu cores数量是siblings数量一半，说明启动了超线程。
# 如果cpu cores数量和siblings数量一致，则没有启用超线程。


# 查看硬盘及分区信息
lsblk
# 查看硬盘
df -h
# 查看内存
free -m
```

### 九、防火墙操作

```bash
# 1.防火墙操作
启动： systemctl start firewalld
查看状态： systemctl status firewalld 
停止： systemctl disable firewalld
禁用： systemctl stop firewalld
# 2.开放指定端口
firewall-cmd --zone=public --add-port=80/tcp --permanent   # 开放端口
firewall-cmd --reload   # //重新载入，使其生效
# 3.关闭指定端口
firewall-cmd --zone=public --remove-port=80/tcp --permanent # 关闭端口
firewall-cmd --reload # 重新载入，使其生效
# 4.查看端口状态
firewall-cmd --zone=public --query-port=80/tcp       # 查看端口状态
# 5.查看已开放得所有端口
firewall-cmd --list-ports
```

### 十、CentOS 7.x 升级内核

CentOS 7.x 系统自带的3.10.x内核存在一些Bugs，导致运行的Docker、Kubernetes不稳定。

> 解决方案如下：
>
> 1. 升级内核到 4.4.X 以上；
> 2. 或者，手动编译内核，disable CONFIG_MEMCG_KMEM 特性；
> 3. 或者，安装修复了该问题的 Docker 18.09.1 及以上的版本。但由于 kubelet 也会设置 kmem（它 vendor 了 runc），所以需要重新编译 kubelet 并指定 GOFLAGS="-tags=nokmem"；

升级内核方式解决

**一：在线安装**

1. 获取源

```bash
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-4.el7.elrepo.noarch.rpm
```

2. 安装完成后检查 /boot/grub2/grub.cfg中对应内核menuentry中是否包含 initrd16 配置，没有再安装一次！

```bash
yum --enablerepo=elrepo-kernel install -y kernel-lt 
```

3. 查看系统中的全部内核【可忽略】

```bash
rpm -qa | grep kernel
```

4. 设置开机从新内核启动 

```bash
grub2-set-default 'CentoS Linux(4.4.202-1.el7.elrepo.×86_64) 7 (Core)'
```

5. 重启启动使配置生效

```bash
reboot
```

6. 查看正在使用的内核【可忽略】

```bash
uname -a
```

**二：离线安装**

升级前内核版本：

```bash
[root@vrgv252 ~]# uname -a

Linux vrgv252.com 3.10.0-957.el7.x86_64 #1 SMP Thu Nov 8 23:39:32 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
```

下载内核文件：

```bash
# 下载指定版本 kernel： 
http://rpm.pbone.net/index.php3?stat=3&limit=1&srodzaj=3&dl=40&search=kernel
# 下载指定版本 kernel-devel：
http://rpm.pbone.net/index.php3?stat=3&limit=1&srodzaj=3&dl=40&search=kernel-devel

官方 Centos 6: http://elrepo.org/linux/kernel/el6/x86_64/RPMS/
官方 Centos 7: http://elrepo.org/linux/kernel/el7/x86_64/RPMS/
```

https://elrepo.org/linux/kernel/el7/x86_64/RPMS/

> 进入后可以看到很多版本文件，一般选择次lt版本文件即可
>
> 说明：lt长期维护版 ml最新稳定版
>
> 我在这里使用两个包即可升级成功
>
> kernel-lt-5.4.91-1.el7.elrepo.x86_64.rpm
>
> kernel-lt-devel-5.4.91-1.el7.elrepo.x86_64.rpm
>

1、把两个文件放到/rpm目录下

```bash
[root@vrgv252 ~]# mkdir /rpm
[root@vrgv252 rpm]# ls
kernel-lt-5.4.91-1.el7.elrepo.x86_64.rpm  kernel-lt-devel-5.4.91-1.el7.elrepo.x86_64.rpm
```

使用rpm命令安装

```bash
[root@vrgv252 rpm]# rpm -ivh *
```

> 注：/rpm目录下只有两个rpm包，所以才可以这么写。如果报错，如perl错误等问题，先升级perl依赖包。

注：这个/rpm目录下只有两个rpm包，所以才可以这么写。如果有报错，如perl错误等问题，先升级perl依赖包。

```
yum -y install kernel-lt-5.4.91-1.el7.elrepo.x86_64.rpm
yum -y install kernel-lt-devel-5.4.91-1.el7.elrepo.x86_64.rpm
```

2、看下现在的内核排序

```bash
[root@vrgv252 ~]# awk -F\' '$1=="menuentry " {print $2}' /etc/grub2.cfg
```

修改内核启动参数为：0，将GRUB_DEFAULT=saved修改为0，此处为0，则改为GRUB_DEFAULT=0

```
[root@vrgv252 ~]# vim /etc/default/grub
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210207112110723.png)

3、使用grub2-mkconfig命令来重新创建内核配置

```bash
[root@localhost ~]# grub2-mkconfig -o /boot/grub2/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-5.4.91-1.el7.elrepo.x86_64
Found initrd image: /boot/initramfs-5.4.91-1.el7.elrepo.x86_64.img
Found linux image: /boot/vmlinuz-3.10.0-957.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-957.el7.x86_64.img
Found linux image: /boot/vmlinuz-0-rescue-86044f9e52ab45b886b99d6bd1729b08
Found initrd image: /boot/initramfs-0-rescue-86044f9e52ab45b886b99d6bd1729b08.img
done
```

4、重启启动服务器

```bash
[root@vrgv252 rpm]# init 6
```

5、验证

```bash
[root@localhost ~]# uname -a
Linux localhost.localdomain 5.4.91-1.el7.elrepo.x86_64 #1 SMP Tue Jan 19 07:32:36 EST 2021 x86_64 x86_64 x86_64 GNU/Linux
```

### 十一：linux修改文件

方法一：echo 实现

> echo 'hello linux' >> /data/hello.txt  这个在企业里很常用：单行内容追加到文件结尾。
>
> 一个大于号**>，是覆盖重定向**，会清除文件里的所有以前数据，增加新数据。
> 两个大于号**>>，是追加重定向**，文件结尾加入内容，不会删除已有文件的内容。

方法二：sed实现

> shell在文本第一行和最后一行添加字符串
>
> sed -i '1 i\ApiInterfaceName ResposeTime' /tmp/apiLog/apiLogFromatSecond.log
> sed -i '1 i\chongfucishu ApiInterfaceName' /tmp/apiLog/apiLogFromatNumber.log
>
> sed '1i 添加的内容' file 　　 #这是在第一行前添加字符串
> sed '$i 添加的内容' file 　　 #这是在最后一行行前添加字符串
> sed '$a添加的内容' file 　　 #这是在最后一行行后添加字符串
>
> sed '1 a\string1\n\string2\n' /etc/passwd 　　#在第1行后插入两行字符串。
>
> sed '1 i\string1\n\string2\n' /etc/passwd 　　#在第1行前插入两行字符串



### 十二：卸载k8s

kubeadm安装的k8s集群卸载方式

```bash
# 卸载服务
kubeadm reset

# 删除rpm包
rpm -qa|grep kube*|xargs rpm --nodeps -e

# 删除容器及镜像
docker images -qa|xargs docker rmi -f
```

```bash
kubeadm reset -f
modprobe -r ipip
lsmod
rm -rf ~/.kube/
rm -rf /etc/kubernetes/
rm -rf /etc/systemd/system/kubelet.service.d
rm -rf /etc/systemd/system/kubelet.service
rm -rf /usr/bin/kube*
rm -rf /etc/cni
rm -rf /opt/cni
rm -rf /var/lib/etcd
rm -rf /var/etcd
yum clean all
yum remove kube*
```

```bash
# 首先清理运行到k8s群集中的pod
$ kubectl delete node --all

# 然后从主机系统中删除数据卷和备份（如果不需要）。最后，可以使用脚本停止所有k8s服务，
$ for service in kube-apiserver kube-controller-manager kubectl kubelet kube-proxy kube-scheduler; do systemctl stop $service done
$ yum -y remove kubernetes #if it's registered as a service
```



## docker 常用操作

### **一：在线安装**

安装操作系统是 CentOS Linux release 7.6.1810 (Core)

**01、添加docker-ce yum源**

```bash
# 扩展yum功能
yum install -y yum-utils
# 添加软件源信息
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# 自动选择最快yum仓库源
yum makecache fast
```

**02、查看当前可以安装docker-ce版本**

yum list docker-ce --showduplicates | sort -r

[![img](https://img2018.cnblogs.com/blog/790307/201909/790307-20190913234443667-687730377.png)](https://img2018.cnblogs.com/blog/790307/201909/790307-20190913234443667-687730377.png)

```bash
# 安装指定版本的格式 ,**注意3:xxx 请移除3:**
yum -y install docker-ce-[VERSION]
yum install -y docker-ce-18.06.3.ce-3.el7
yum install -y docker-ce-19.03.2-3.el7
```

**03、启动测试**

```bash
systemctl start docker
docker info
docker version
```

### 二：离线安装

1、下载docker-19.03.9离线包，
可以官网下载:
https://download.docker.com/linux/static/stable/x86_64/
也可通过以下百度云盘下载：
链接: https://pan.baidu.com/s/1BkzLI9jphQ-4a4lBQK3HYg 提取码: 63cx

2、上传到服务器/home/images目录下，解压

```bash
tar -zxvf  docker-19.03.9.tgz
```


3、授权docker文件目录为可执行文件

```bash
chmod -R 777 docker
```

4、将docker文件夹复制到/usr/bin目录下

```bash
cp -r docker/* /usr/bin/
```

5、在/usr/lib/systemd/system/docker.service文件（没有目录新增）中添加以下内容，然后保存

```bash
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target
[Service]
Type=notify
ExecStart=/usr/bin/dockerd
ExecReload=/bin/kill -s HUP $MAINPID
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
TimeoutStartSec=0
Delegate=yes
KillMode=process
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s
[Install]
WantedBy=multi-user.target
```

6、创建`/etc/docker`目录，新增配置`/etc/docker/daemon.json`文件：

```bash
mkdir /etc/docker
```

编辑daemon.json文件：

```bash
vim /etc/docker/daemon.json
```

文件内容为（data-root为docker数据卷挂载路劲，根据自己的实际情况配置路径）：

```json
{"registry-mirrors":["https://registry.docker-cn.com"],"data-root":"/data/docker"}
```


7、重新加载配置

```
systemctl daemon-reload
```

8、启动docker

```
systemctl start docker
```

9、设置开机启动docker

```bash
systemctl enable docker
```

10、检查docker版本

```bash
docker -v
```



### 三：卸载 Docker Engine**

1. Uninstall the Docker Engine, CLI, and Containerd packages:

   ```bash
   $ sudo yum remove docker-ce docker-ce-cli containerd.io
   ```

2. Images, containers, volumes, or customized configuration files on your host are not automatically removed. To delete all images, containers, and volumes:

   ```bash
   $ sudo rm -rf /var/lib/docker
   $ sudo rm -rf /var/lib/containerd
   ```

You must delete any edited configuration files manually.

### 四：调整镜像挂载目录

```bash
1、systemctl stop docker
2、mkdir -p /data/var/lib/docker
3、vim /etc/docker/daemon.json
4、{
	"graph": "/data/var/lib/docker"
   }

systemctl stop docker
rm -rf /data/var/lib/docker
cp -r /var/lib/docker /data/var/lib/docker
Systemctl restart docker
```

### 五：硬盘挂载和分区

```bash
# 查看系统中的新硬盘
ls /dev/sd*
# 查看分区情况
fdisk -l
# 查看分区及挂载情况
lsblk
# 查看磁盘格式
df -T
df -h
# 新增硬盘分区
fdisk /dev/sdb # mbr分区步骤：输入n,输入p,输入1,回车,回车,输入w
fdisk /dev/sdb # gpt分区步骤：输入g,输入n,输入p,输入1,回车,回车,输入w
# 删除硬盘分区
fdisk /dev/sdb # mbr分区步骤：输入d,输入1,输入p,输入q退出
# 格式化分区
mkfs.ext4 /dev/sdb1   # 格式化为ext4
mkfs -t xfs /dev/sdb1 # 格式化为xfs
# 挂载分区
mkdir /data
mount /dev/sdb1 /data
unmount /data
# 设置开机自动挂载
echo /dev/sdb1 /data ext4 defaults 0 0 >> /etc/fstab
mount -a   # 检测挂载是否正确
```

### 六：docker用户组权限

```bash
sudo groupadd docker #添加docker 用户组
sudo usermod -aG docker $USER #将当前用户添加至docker 用户组
newgrp docker
Systemctl restart docker # 可不执行
```

### 七：docker 镜像导入导出

```bash
docker save hub4rpi64/sonarqube:8.3.1.34397 > sonarqube.tar
docker load < sonarqube.tar

docker save postgres:12 > postgres12.tar 
docker load < postgres12.tar 
```



### 八：docker compose 安装

**1、在线安装**

```bash
# 1. Run this command to download the current stable release of Docker Compose:
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# 2. Apply executable permissions to the binary:
sudo chmod +x /usr/local/bin/docker-compose

# 3. Test the installation
$ docker-compose --version
docker-compose version 1.29.2, build 1110ad01

```

**Note**: If the command `docker-compose` fails after installation, check your path. You can also create a symbolic link to `/usr/bin` or any other directory in your path.

```bash
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

**2、离线安装**

```bash
# 下载docker-compose-Linux-x86_64,地址 https://github.com/docker/compose/releases
mv docker-compose-Linux-x86_64 /usr/local/bin/docker-compose
# Apply executable permissions to the binary:
sudo chmod +x /usr/local/bin/docker-compose
```

### 九：docker 异常排查

nethogs 安装：

```bash
wget https://github.com/raboof/nethogs/archive/v0.8.1.tar.gz
yum install libpcap-devel
tar zxvf v0.8.1.tar.gz
cd nethogs-0.8.1/
make && make install
nethogs eno1
```

net-tools 安装：

```
tar xvf  +文件名（tar.xz）
```

查看 docker 监控信息：

```bash
# 查看监控的cpu、net等信息
docker stats 容器id或者名称
# 查看监控的进程信息
docker top 容器id或者名称
```





## DISM命令离线安装.NET Framework 3.5

首先，我们需要下载Win10 ISO镜像。

参见《使用微软媒体创建工具下载原版Win10 ISO镜像》。

在Win10 ISO镜像文件上点击右键，选择“装载”。如图：

![img](https://pics4.baidu.com/feed/34fae6cd7b899e513e1a409ad2da0c35c9950d2f.jpeg?token=e3db9e6c47fef596089ebf492ffa21e4)

这时“此电脑”中就会显示虚拟光驱“DVD驱动器(X:)”，记下盘符，MS酋长这里显示的盘符是“J:”。如图：

![img](https://pics5.baidu.com/feed/e850352ac65c103866ec331a3d6c4615b27e89d8.jpeg?token=541d33ca39a41646d0f12360f0898a0c)

然后以管理员身份运行命令提示符。方法是：

在Win10任务栏搜索框中输入 cmd ，在搜索结果中选择“以管理员身份运行”。

或者以管理员身份运行Windows PowerShell。方法是：

右键点击Win10开始按钮，选择“Windows PowerShell(管理员)”）。

在打开的“管理员:命令提示符”或“管理员: Windows PowerShell”窗口中输入以下命令：

dism.exe /online /enable-feature /featurename:netfx3 /Source:J:\sources\sxs

注：其中的盘符 J: 要改成你实际的虚拟光驱盘符

![img](https://pics6.baidu.com/feed/a08b87d6277f9e2f2af7ac038e4d3c22b999f37b.jpeg?token=98fea27de748c45728f151172e53a8b1)

![img](https://pics0.baidu.com/feed/6159252dd42a2834e4ec1dd3cbc81cec14cebf20.jpeg?token=2d88794b40f92d0688530e4289affbc5)

按回车键运行命令。等待部署完毕，进度100%，提示“操作成功完成”。

这样Win10就成功安装了.NET Framework 3.5。
