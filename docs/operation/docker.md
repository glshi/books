# ubuntu docker 安装

## 1：基于仓库安装

    $ sudo apt-get update
    
    $ sudo apt-get install \
        apt-transport-https \
        ca-certificates \
        curl \
        gnupg-agent \
        software-properties-common
    
    $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    
    $ sudo apt-key fingerprint 0EBFCD88
    
    $ sudo add-apt-repository \
       "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
       $(lsb_release -cs) \
       stable"
    
    $ sudo apt-get update
    $ sudo apt-get install docker-ce docker-ce-cli containerd.io
    
    $ sudo docker run hello-world

## 2：添加镜像加速

	sudo tee /etc/docker/daemon.json <<-'EOF'
	{
	  "registry-mirrors": ["https://3y48jgm1.mirror.aliyuncs.com","https://docker.mirrors.ustc.edu.cn"]
	}
	EOF

## 3：添加用户docker权限

	sudo groupadd docker
	sudo gpasswd -a glshi docker
	sudo service docker restart

## 3：重启docker

```
sudo systemctl daemon-reload
sudo systemctl restart docker
```



# docker win10 home 安装

## 1：安装 WSL 2

参考：https://docs.microsoft.com/zh-cn/windows/wsl/install-win10

## 2：安装 docker desktop

下载地址：https://docs.docker.com/docker-for-windows/install-windows-home/

## 3：更改镜像存储位置

删除所有的image/container/wsl/hyperv数据：
![img](https://img-blog.csdnimg.cn/20201119145457556.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZsZWF4aW4=,size_16,color_FFFFFF,t_70#pic_center)
导出wsl子系统镜像:

```
wsl --export docker-desktop docker-desktop.tar
wsl --export docker-desktop-data docker-desktop-data.tar
```

删除现有的wsl子系统：

```
wsl --unregister docker-desktop
wsl --unregister docker-desktop-data
```

重新创建wsl子系统：

```
wsl --import docker-desktop d:\wsl docker-desktop.tar
wsl --import docker-desktop-data d:\wsl\data docker-desktop-data.tar
```

重新运行docker desktop，然后pull一个镜像，C盘不会变大了。

## 4：启用 kubernets

参考：https://github.com/AliyunContainerService/k8s-for-docker-desktop

**dashboard：**

nohup kubectl proxy >/dev/null &

 通过如下 URL 访问 Kubernetes dashboard

http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/

对于Windows环境

```
$TOKEN=((kubectl -n kube-system describe secret default | Select-String "token:") -split " +")[1]

kubectl config set-credentials docker-for-desktop --token="${TOKEN}"

echo $TOKEN
```

# docker常用命令

```bash
systemctl status docker        		查看docker状态

sudo systemctl start docker      	启动docker

sudo docker logs 容器id/name   	   查看容器日志

sudo docker stop 容器id/name  	   停止容器

sudo docker rm 容器id/name   		   移除容器

docker ps 和docker ps -a    			前者查看运行中的容器，后者查看所有容器

# docker一键删除所有none镜像
docker rmi `docker images | grep  "<none>" | awk '{print $3}'`
# 如果确定所有none镜像确实没用，直接加个-f强制删除，谨慎
docker rmi -f `docker images | grep  "<none>" | awk '{print $3}'`

sudo docker run -itd --rm --name tensorflow_ssh_0 -v /data/docker/tensorflow/notebooks:/notebooks -p 8888:8888 -p 2022:22 1001/tensorflow_ssh:latest

docker exec -it mysql1 bash

docker cp  /home/glshi/ltp ltp3.4.0:/home
```



# Docker 镜像批量备份

## **脚本作用**

```
1.批量导出Docker Images;
2.部分导出，通过指定Docker Images ID 到脚本“LIST”变量；
3.支持相同ID，不同REPOSITORY名称备份；
```

例如：

```
docker images 
kry1702/coredns           1.3.1       eb516548c180        5 months ago        40.3MB
k8s.gcr.io/coredns        1.3.1       eb516548c180        5 months ago        40.3MB
注意：备份文件名称格式为：kry1702_coredns:1.3.1.tar
主要是解决相同ID，不同REPOSITORY名称，如果提取“/”最右边为备份文件名称格式导致备份文件冲突，以上为例备份文件名称格式为：coredns:1.3.1.tar 
```

## **运行实例**

```bash
 # 导出全部的镜像；
 sh ExportImg.sh

 # 导出部分镜像
 LIST=“ d235b 201c7a  201c7a”
 sh ExportImg.sh
 # 注意：LIST赋值Docker Images ID ,多个镜像ID通过空格隔离；执行脚本是只会导出定义ID的镜像；
```

## **镜像还原(任选一种)**

```bash
docker load --input xxx.tar
docker load < xx.tar
docker load -i xx.tar
```

## **脚本内容**

```bash
LIST=""
TXT=/root/tmp.txt
BAKDIR=/usr/local/bak
LOGDIR=/usr/local/bak/log
LOGFILE=$LOGDIR/bak.`date +%Y%m%d`.log

[ ! -d $BAKDIR ] && mkdir -p $BAKDIR
[ ! -d $LOGDIR ] && mkdir -p $LOGDIR

if [ -n "$LIST" ]
then
        for list in $LIST
        do
                RESLIST=`docker images |grep $list | awk '{print $1}'`
                for reslist in $RESLIST
                do
                RESTAG=`docker images |grep "$reslist" |awk '{a=$1":"$2;print a }'`
                BAKNAME=`docker images |grep "$reslist" |awk '{a=$1":"$2;print a }'|sed 's/\//_/g'`
                /usr/bin/docker save $RESTAG -o $BAKDIR/$BAKNAME.tar  >> $LOGFILE 2>&1
                done
        done
else
        REC=`docker images |awk '{print $1,$2,$3}'|sed 1d >> $TXT`
        RESLIST=`cat $TXT|awk '{print $1}'`
        for reslist in $RESLIST
        do
                RESTAG=`docker images |grep "$reslist" |awk '{a=$1":"$2;print a }'`
                BAKNAME=`docker images |grep "$reslist" |awk '{a=$1":"$2;print a }'|sed 's/\//_/g'`
                /usr/bin/docker save $RESTAG -o $BAKDIR/$BAKNAME.tar  >> $LOGFILE 2>&1
        done
        /usr/bin/rm -f $TXT
fi

if [ -s $LOGFILE ]
then
        echo -e "\033[31mERROR:Images Backup Failed!\033[0m"
        echo -e "\033[31mPlease View The Log Lile : $LOGFILE\033[0m"
else
        /usr/bin/rm -f $LOGFILE
fi
```

