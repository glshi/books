## nacos 部署

### docker run部署

- 获取镜像

```powershell
docker pull nacos/nacos-server:1.3.0
```

- 初始化nacos数据库

  > docker cp /mydata/nacos-db.sql mysql:/
  >
  > 
  >
  > docker exec -it mysql /bin/bash
  >
  > mysql -uroot -proot --default-character-set=utf8
  >
  > 
  >
  > create database nacos_config character set utf8;
  >
  > use nacos_config; 
  >
  > source /nacos-db.sql;
  >
  > 
  >
  > 初始化[nacos数据sql](https://github.com/alibaba/nacos/blob/master/config/src/main/resources/META-INF/nacos-db.sql)

- 运行镜像

```bash
docker run --name nacos-registry  -p 8848:8848 \
--link mysql:db \
--env MODE=standalone \
--env MYSQL_SERVICE_HOST=db \
--env SPRING_DATASOURCE_PLATFORM=mysql \
--env MYSQL_SERVICE_DB_NAME=nacos_config \
-d nacos/nacos-server:1.3.0

docker exec -it nacos-registry bash

# 修改对应mysql参数,设置MYSQL_SERVICE_USER、MYSQL_SERVICE_PASSWORD的值
vim conf/application.properties

# 重启 nacos                                              
docker restart nacos-registry                                                            
```

- nacos管理平台（新增配置 ，然后可在数据库查看）

```bash
http://192.168.1.180:8848/nacos/index.html
nacos/nacos(用户名密码)
```

### Nacos Kubernetes 部署

https://nacos.io/en-us/docs/use-nacos-with-kubernetes.html



## maven

```bash
mvn clean package -P prod
```



rabbitmq访问地址查看是否安装成功：[http://192.168.3.101:15672](http://192.168.3.101:15672/)

- 输入账号密码并登录：guest guest
- 创建帐号并设置其角色为管理员：mall mall

监控中心应用信息，访问地址：[http://192.168.3.101:8101](http://192.168.3.101:8101/)

日志收集系统信息，访问地址：[http://192.168.3.101:5601](http://192.168.3.101:5601/)

查看注册中心注册服务信息，访问地址：http://192.168.3.101:8848/nacos/

启动成功后，可以查看API文档信息，访问地址：[http://192.168.3.101:8201](http://192.168.3.101:8201/)

查看Portainer的DashBoard信息，访问地址：[http://192.168.3.101:9000](http://192.168.3.101:9000/)



## Jenkins

```bash
docker run -p 8080:8080 -p 50000:5000 --name jenkins \
-u root \
-v /mydata/jenkins_home:/var/jenkins_home \
-d jenkins/jenkins:lts
```

- 使用管理员密码进行登录，可以使用以下命令从容器启动日志中获取管理密码：

```
docker logs jenkins
```

- 从日志中获取管理员密码：

![img](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwliaEOrAaIx3Wjs4GZBQrpdOPHr2pBTCgMCV7224g3C8lxV0HPXRkhGaOia0ZXmWQqFJk1fS1viauLtg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

admin admin



```
# 只install mall-common,mall-mbg,mall-security三个模块
clean install -pl mall-common,mall-mbg,mall-security -am
${WORKSPACE}/pom.xml


clean package
${WORKSPACE}/mall-admin/pom.xml

/mydata/sh/mall-admin.sh
```

```bash
# 立刻关机
shutdown -h now 
init 0
```

docker rmi `docker images | grep  "<none>" | awk '{print $3}'`
docker rmi `docker images | grep  "mall" | awk '{print $3}'`





```bash
docker stop mall-auth
docker stop mall-gateway
docker stop mall-monitor
docker stop mall-admin
docker stop mall-search
docker stop mall-portal

docker rm mall-auth
docker rm mall-gateway
docker rm mall-monitor
docker rm mall-admin
docker rm mall-search
docker rm mall-portal

docker stop $(docker ps -a |  grep "mall*"  | awk '{print $1}')
docker rm -f $(docker ps -a |  grep "mall*"  | awk '{print $1}')

docker rm `docker ps -a -q`
```

我们可以用这个命令查看所有的docker网络：

```bash
docker network ls
docker exec -it django_web_1 cat /etc/hosts 
```

chown -R glshi:glshi /mydata/

```
docker-compose -f docker-compose-env.yml up -d
docker-compose -f docker-compose-env.yml stop

docker-compose -f docker-compose-app.yml up -d
docker-compose -f docker-compose-app.yml stop

```



```
docker inspect --format='{{.NetworkSettings.IPAddress}}' mall-admin
```



```
docker cp settings.xml jenkins:/

mvn clean install package -Dmaven.test.skip
mvn package docker:build -Dmaven.test.skip
```

四、docker部署spring-boot工程

1、进入父工程pom文件所在目录，打包编译，将依赖包放至本地仓库

```
mvn clean install package -Dmaven.test.skip  
```

2、进入各模块，使用 DockerFile 构建镜像

```
cd dw-demo  
mvn package docker:build -Dmaven.test.skip  
```

3、运行该镜像

```
docker run -p 9882:8082 -t com.rd/dw-demo:2.0.0  
```



mvn package docker:build -Dmaven.test.skip



portainer: admin/12345678



```bash
"Name": "/mysql",
"NetworkSettings": {
            "Bridge": "",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,


"Name": "/mall-gateway",
"NetworkSettings": {
            "Bridge": "",
            "Networks": {
                "docker_default": {
                    "IPAMConfig": null,

glshi@glshi-b550:/var/lib/docker$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
f44b806ac378        bridge              bridge              local
fcd290516578        docker_default      bridge              local
48cb63767bf4        example_default     bridge              local
90eb98f75f53        host                host                local
b27959612ac8        none                null                local

networks:
  default:
    external:
     name: bridge
     
networks:
  default:
    external: true
```

