## nacos 安装

### 1：docker 安装

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

### 2：Kubernetes 安装

https://nacos.io/en-us/docs/use-nacos-with-kubernetes.html



## 常用命令

### 1：常见maven、linux命令

```bash
mvn clean package -P prod

mvn clean install package -Dmaven.test.skip
mvn package docker:build -Dmaven.test.skip

chown -R glshi:glshi /mydata/

# 立刻关机
shutdown -h now 
init 0
```

### 2：数据库命令

**新增用户**

- 用户名`myuser` 密码`mypassword`

```bash
mysql -u root -p 
CREATE USER 'myuser'@'localhost' IDENTIFIED BY 'mypassword'; #本地登录 
CREATE USER 'myuser'@'%' IDENTIFIED BY 'mypassword'; #远程登录 
quit 
mysql -u myuser -p #测试是否创建成功
```

如果出现以下问题,可以参考这个链接

- [ERROR 1819 (HY000): Your password does not satisfy the current policy requirements](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.cnblogs.com%2Fivictor%2Fp%2F5142809.html)

**为用户授权**

- a.授权格式：grant 权限 on 数据库.* to 用户名@登录主机 identified by '密码';
- b.登录MYSQL，这里以ROOT身份登录：
- c.为用户创建一个数据库(testDB)：

```mysql
create database testDB default charset utf8mb4 collate utf8mb4_unicode_ci;

create database lank default charset utf8mb4 collate utf8mb4_unicode_ci;

create database mall_tiny default charset utf8mb4 collate utf8mb4_unicode_ci;
```

- d.授权test用户拥有`testDB`数据库的所有权限：

```mysql
grant all privileges on testDB.* to 'myuser'@'localhost' identified by 'mypassword'; 本地授权
grant all privileges on testDB.* to 'myuser'@'%' identified by 'mypassword'; 远程授权
flush privileges; #刷新系统权限表
```

- e.指定部分权限给用户:

```csharp
grant select,update on testDB.* to 'myuser'@'localhost' identified by 'mypassword'; 
grant select,delete,update,create,drop on *.* to 'test'@'%' identified by 'test123';
flush privileges; #刷新系统权限表
```

**创建常用库**

```sql
create database u_member default charset utf8mb4 collate utf8mb4_unicode_ci;
create database u_order default charset utf8mb4 collate utf8mb4_unicode_ci;
create database u_product default charset utf8mb4 collate utf8mb4_unicode_ci;
```

## docker 相关

### 1：系统环境启动和停止

```bash
cd project/config/
docker-compose -f docker-compose-env.yml up -d
docker-compose -f docker-compose-env.yml stop

docker-compose -f docker-compose-app-bank.yml up -d
docker-compose -f docker-compose-app-lank.yml up -d

# 启动或停止一组docker-compose环境
docker-compose -f docker-compose-env.yml up -d
docker-compose -f docker-compose-env.yml stop

docker-compose -f docker-compose-app.yml up -d
docker-compose -f docker-compose-app.yml stop

# redis windows 启动
cd .\Redis-x64-5.0.10\
redis-server.exe redis.windows.conf
```

### 2：删除容器和镜像

```bash
# 删除 none 和 mall 的镜像
docker rmi `docker images | grep  "<none>" | awk '{print $3}'`
docker rmi `docker images | grep  "mall" | awk '{print $3}'`
# 删除含有 mall 的相关容器
docker rm -f $(docker ps -a |  grep "mall*"  | awk '{print $1}')
# 停止相关容器
docker stop mall-auth
docker stop mall-gateway
docker stop mall-monitor
docker stop mall-admin
docker stop mall-search
docker stop mall-portal
# 删除相关容器
docker rm mall-auth
docker rm mall-gateway
docker rm mall-monitor
docker rm mall-admin
docker rm mall-search
docker rm mall-portal
# 删除多个容器
docker stop $(docker ps -a |  grep "mall*"  | awk '{print $1}')
docker rm -f $(docker ps -a |  grep "mall*"  | awk '{print $1}')

docker rm `docker ps -a -q`
```

### 3：其他常见docker命令



```bash
# 我们可以用这个命令查看所有的docker网络：
docker network ls
docker exec -it django_web_1 cat /etc/hosts 

# 查看IP地址
docker inspect --format='{{.NetworkSettings.IPAddress}}' mall-admin

# 复制文件到容器里
docker cp settings.xml jenkins:/
```

### 4：docker部署spring-boot工程

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



### 5：wsl2 docker部署问题

在win10下使用Docker有时有点坑，网上很多教程都是基于liunx的操作步骤。

现在碰到的主要有两个问题：

1、docker search 没有反应
2、系统没授权 Operation not permitted

```bash
# 创建数据卷
D:\>docker volume create --name mongodata
mongodata

D:\>docker volume ls
DRIVER              VOLUME NAME
... ...
local               f8abd230256df3e227fc2c7605b615e4cc9e31e47ed4afc663e698e59ec31071
local               mongodata
# 挂接运行mongo数据库
D:\>docker run --name mongodb -v mongodata:/data/db -p 27017:27017 -d mongo:latest --auth
a0585181d6102c6c4e1ebd7686fc8d08827632b5b279fb4eae7bf746e8ea49a9

D:\>docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                      NAMES
a0585181d610        mongo:latest        "docker-entrypoint.s…"   11 seconds ago      Up 9 seconds        0.0.0.0:27017->27017/tcp   mongodb

D:\>
```

**单独删除：** docker volume rm <名称>



解决权限问题：

```
docker volume create --name mongodata
docker volume create --name rabbitmqdata
```



## lank 系统相关

### 1：各个系统访问地址

rabbitmq访问地址查看是否安装成功：[http://192.168.3.101:15672](http://192.168.3.101:15672/)

- 输入账号密码并登录：guest guest
- 创建帐号并设置其角色为管理员：mall mall

监控中心应用信息，访问地址：[http://192.168.3.101:8101](http://192.168.3.101:8101/)

日志收集系统信息，访问地址：[http://192.168.3.101:5601](http://192.168.3.101:5601/)

查看注册中心注册服务信息，访问地址：http://192.168.3.101:8848/nacos/

启动成功后，可以查看API文档信息，访问地址：[http://192.168.3.101:8201](http://192.168.3.101:8201/)

查看Portainer的DashBoard信息，访问地址：[http://192.168.3.101:9000](http://192.168.3.101:9000/)

portainer: admin/12345678



### 2：oauth2 授权访问流程

**未登录前访问接口**

![img](http://www.macrozheng.com/images/arch_screen_15.png)

![img](http://www.macrozheng.com/images/arch_screen_16.png)

**登录后访问接口**

- 进行登录操作：登录帐号test 123456

![img](http://www.macrozheng.com/images/arch_screen_17.png)

![img](http://www.macrozheng.com/images/arch_screen_18.png)

- 点击Authorize按钮，在弹框中输入登录接口中获取到的token信息

![img](http://www.macrozheng.com/images/arch_screen_19.png)

![img](http://www.macrozheng.com/images/arch_screen_20.png)

- 登录后访问获取权限列表接口，发现已经可以正常访问

![img](http://www.macrozheng.com/images/arch_screen_15.png)

![img](http://www.macrozheng.com/images/arch_screen_21.png)

### 3：权限管理功能详解

**项目中的权限管理功能是如何实现的？**

之前写过几篇文章，包括了权限管理功能介绍、后端实现和前端实现，看一下基本就清楚了！

- [《大家心心念念的权限管理功能，这次安排上了！》](https://mp.weixin.qq.com/s/3TNrPNmxHpFTcAhfjnuP0g)
- [《手把手教你搞定权限管理，结合Spring Security实现接口的动态权限控制！》](https://mp.weixin.qq.com/s/nvKKNSJuIrGuHeJkUeO7rw)
- [《手把手教你搞定权限管理，结合Vue实现菜单的动态权限控制！》](https://mp.weixin.qq.com/s/UXPeJtx-mIvCPjJW3__c6g)

**`ums_permission`表还在使用么？**

`ums_permission`表已经不再使用了，不再使用的表还包括`ums_admin_permission_relation`和`ums_role_permission_relation`表，最新版已经移除了相关使用代码。

**项目升级代码后`ums_resource`表找不到？**

升级代码以后需要同时导入最新版本的SQL脚本，否则会找不到新创建的表，SQL脚本在项目的`document\sql`文件夹下面。

**只实现了管理后台的权限，移动端权限如何处理的？**

移动端只实现了登录认证，暂时不做权限处理。

**在管理后台添加了一个菜单，为什么前端没有显示？**

只有在前端路由中配置了的菜单，在管理后台添加后才会显示，否则没有效果。

![img](http://www.macrozheng.com/images/mall_permission_question_01.png)

**前端路由中修改了菜单名称，为什么还是原来的名称？**

菜单名称、图标、是否隐藏都是由管理后台配置的，当管理后台配置好后，前端修改是无效的。

![img](http://www.macrozheng.com/images/mall_permission_question_02.png)

**mall-swarm项目中的权限管理功能是如何实现的？**

采用了基于Oauth2的的统一认证鉴权方式，通过认证服务进行统一认证，然后通过网关来统一校验认证和鉴权。具体可以参考：[《微服务权限终极解决方案，Spring Cloud Gateway + Oauth2 实现统一认证和鉴权！》](https://mp.weixin.qq.com/s/npyZsa4p30PLULxjskxKSA)

**mall和mall-swarm项目中权限管理的实现有何不同？**

mall项目实现方式是Spring Security，相当于把安全功能封装成了一个工具包`mall-security`，然后其他模块通过依赖该工具包来实现权限管理，比如`mall-admin`模块。 mall-swarm项目实现方式是Oauth2+Gateway，采用的是统一的认证和鉴权，mall-swarm中的其他模块只需关注自己的相关业务，而无需依赖任何安全工具包，更加适合微服务架构。

**mall-swarm项目对接前端项目时为什么会提示你已经被登出？**

一种情况是前端访问后端接口没有走网关，首先需要修改`mall-admin-web`项目的后端接口访问基础路径，修改文件为项目`config`目录下的`dev.env.js`，将`BASE_API`改为从网关访问`mall-admin`服务的路径。

```javascript
'use strict'
const merge = require('webpack-merge')
const prodEnv = require('./prod.env')

module.exports = merge(prodEnv, {
  NODE_ENV: '"development"',
  BASE_API: '"http://localhost:8201/mall-admin"'
})Copy to clipboardErrorCopied
```

另一种情况是没有更新到最新代码，需要更新到最新代码并导入最新版本的SQL。之前在`mall-gateway`项目中配置白名单的时候把`/mall-admin/admin/info`接口配置进去了，会导致无法登录的情况，需要去除掉，目前已经修复了。

```yaml
secure:
  ignore:
    urls: #配置白名单路径
     - "/mall-admin/admin/info"Copy to clipboardErrorCopied
```

还有一点需要注意的是由于`/mall-admin/admin/info`接口配置已经不在白名单中了，所以需要登录的用户需要配置该接口的相应资源，否则会无法登录。

![img](http://www.macrozheng.com/images/mall_permission_question_03.png)

**mall-swarm项目如何访问需要登录认证的接口？**

由于项目中存在两套不同的用户体系，后台用户和前台用户，认证中心对多用户体系也有所支持。如何访问需要登录的接口，先调用认证中心接口获取token，不同体系下的用户需要传入不同的`client_id`和`client_secret`，后台用户为`admin-app:123456`，前台用户为`portal-app:123456`。

![img](http://www.macrozheng.com/images/mall_permission_question_04.png)

然后将token添加到请求头中，即可访问需要权限的接口了。

![img](http://www.macrozheng.com/images/mall_permission_question_05.png)

当然对原来的登录接口也做了兼容处理，分别会从内部调用认证中心获取Token，依然可以使用。

- 后台用户登录接口：http://localhost:8201/mall-admin/admin/login
- 前台用户登录接口：http://localhost:8201/mall-portal/sso/login

**只想学习权限管理功能，有没有什么简单的项目可以学习下？**

可以直接学习下`mall-tiny`项目，`mall-tiny`是一款基于SpringBoot+MyBatis-Plus的快速开发脚手架，拥有完整的权限管理功能，可对接Vue前端，开箱即用。具体参考[《还在从零开始搭建项目？手撸了款快速开发脚手架！》](https://mp.weixin.qq.com/s/tN3zjoKQxg1U19D4Slih8w)

前端部分搭建

### 4：前端web搭建步骤

- 下载node并安装：[https://nodejs.org/dist/v12.14.0/node-v12.14.0-x64.msi](https://nodejs.org/dist/v12.14.0/node-v12.14.0-x64.msi);
- 该项目为前后端分离项目，访问本地访问接口需搭建后台环境;
- 访问在线接口无需搭建后台环境，只需将`config/dev.env.js`文件中的`BASE_API`改为[http://admin-api.macrozheng.com](http://admin-api.macrozheng.com)即可;
- 如果你对接的是[mall-swarm](https://github.com/macrozheng/mall-swarm)微服务后台的话，所有接口都需要通过网关访问，需要将`config/dev.env.js`文件中的`BASE_API`改为[http://localhost:8201/mall-admin](http://localhost:8201/mall-admin)；
- 克隆源代码到本地，使用IDEA打开，并完成编译;
- 在IDEA命令行中运行命令：npm install,下载相关依赖;
- 在IDEA命令行中运行命令：npm run dev,运行项目;
- 访问地址：[http://localhost:8090](http://localhost:8090) 即可打开后台管理系统页面;
- 具体部署过程请参考：[mall前端项目的安装与部署](http://www.macrozheng.com/#/deploy/mall_deploy_web)
- `注意`：如果遇到无法运行npm命令，需要配置npm的环境变量，如在path变量中添加：C:\Users\zhenghong\AppData\Roaming\npm;
- `注意`：如果遇到npm install无法成功下载依赖，请参考[使用Jenkins一键打包部署前端应用，就是这么6！](http://www.macrozheng.com/#/reference/jenkins_vue) 中`遇到的坑`部分。