# 常用命令

## 端口转发

```bash
$ nohup kubectl proxy --port=8887 --address='192.168.1.180' --accept-hosts='^.*' >/dev/null 2>&1 &

$ kubectl -n rook-ceph port-forward svc/rook-ceph-mgr-dashboard-external-http 7000 --address 192.168.1.180

$ kubectl --namespace monitoring port-forward svc/prometheus-k8s 9090 --address 192.168.1.180
Prometheus  http://localhost:9090

$ kubectl --namespace monitoring port-forward svc/grafana 3000 --address 192.168.1.180
Grafana http://localhost:3000 admin admin

$ kubectl --namespace monitoring port-forward svc/alertmanager-main 9093
Alert Manager http://localhost:9093

```

## 镜像拉取失败

```
eval $(minikube docker-env)

docker pull docker.io/aiotceo/etcd:3.4.3-0
docker tag docker.io/aiotceo/etcd:3.4.3-0 k8s.gcr.io/etcd:3.4.3-0
docker rmi docker.io/aiotceo/etcd:3.4.3-0

```

## k8s 常用命令

```bash
# 查看所有 pod 列表,  -n 后跟 namespace, 查看指定的命名空间
kubectl get pod
kubectl get pod -n kube  
kubectl get pod -o wide
kubectl get pods --all-namespaces

# 查看 RC 和 service 列表， -o wide 查看详细信息
kubectl get rc,svc
kubectl get pod,svc -o wide  
kubectl get pod <pod-name> -o yaml

# 显示 Node 的详细信息
kubectl describe node 192.168.0.212

# 显示 Pod 的详细信息, 特别是查看 pod 无法创建的时候的日志
kubectl describe pod <pod-name>

# 根据 yaml 创建资源, apply 可以重复执行，create 不行
kubectl create -f pod.yaml
kubectl apply -f pod.yaml

# 基于 pod.yaml 定义的名称删除 pod 
kubectl delete -f pod.yaml 

# 查看 endpoint 列表
kubectl get endpoints

# 执行 pod 的 date 命令
kubectl exec <pod-name> -- date
kubectl exec <pod-name> -- bash
kubectl exec <pod-name> -- ping 10.24.51.9

# 通过bash获得 pod 中某个容器的 TTY，相当于登录容器
kubectl exec -it <pod-name> -c <container-name> -- bash
eg: kubectl exec -it redis-master-cln81 -- bash

# 查看容器的日志
kubectl logs <pod-name>
kubectl logs -f <pod-name> # 实时查看日志
kubectl log  <pod-name>  -c <container_name> # 若 pod 只有一个容器，可以不加 -c 
kubectl logs -l app=frontend # 返回所有标记为 app=frontend 的 pod 的合并日志。
```

## k8s 全部命令

```bash
# 查看所有 pod 列表,  -n 后跟 namespace, 查看指定的命名空间
kubectl get pod
kubectl get pod -n kube  
kubectl get pod -o wide

# 查看 RC 和 service 列表， -o wide 查看详细信息
kubectl get rc,svc
kubectl get pod,svc -o wide  
kubectl get pod <pod-name> -o yaml

# 显示 Node 的详细信息
kubectl describe node 192.168.0.212

# 显示 Pod 的详细信息, 特别是查看 pod 无法创建的时候的日志
kubectl describe pod <pod-name>

# 根据 yaml 创建资源, apply 可以重复执行，create 不行
kubectl create -f pod.yaml
kubectl apply -f pod.yaml

# 基于 pod.yaml 定义的名称删除 pod 
kubectl delete -f pod.yaml 

# 删除所有包含某个 label 的 pod 和 service
kubectl delete pod,svc -l name=<label-name>

# 删除所有 Pod
kubectl delete pod --all

# 查看 endpoint 列表
kubectl get endpoints

# 执行 pod 的 date 命令
kubectl exec <pod-name> -- date
kubectl exec <pod-name> -- bash
kubectl exec <pod-name> -- ping 10.24.51.9

# 通过bash获得 pod 中某个容器的 TTY，相当于登录容器
kubectl exec -it <pod-name> -c <container-name> -- bash
eg: kubectl exec -it redis-master-cln81 -- bash

# 查看容器的日志
kubectl logs <pod-name>
kubectl logs -f <pod-name> # 实时查看日志
kubectl log  <pod-name>  -c <container_name> # 若 pod 只有一个容器，可以不加 -c 
kubectl logs -l app=frontend # 返回所有标记为 app=frontend 的 pod 的合并日志。

# 查看注释
kubectl explain pod
kubectl explain pod.apiVersion

# 查看节点 labels
kubectl get node --show-labels

# 重启 pod
kubectl get pod <POD名称> -n <NAMESPACE名称> -o yaml | kubectl replace --force -f -

# 修改网络类型
kubectl patch service istio-ingressgateway -n istio-system -p '{"spec":{"type":"NodePort"}}'

# 创建和暴露部署
kubectl create deployment hello-ngx --image=nginx:1.14.2
kubectl expose deployment hello-ngx --type=NodePort --port=8080

kubectl run nginx --image=nginx --port=8080
kubectl cluster-info dump
```



# minikube

## minikube 安装

检查cpu是否支持虚拟化：

    egrep -q 'vmx|svm' /proc/cpuinfo && echo yes || echo no

Install kubectl ：

    curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
    
    chmod +x ./kubectl
    
    sudo mv ./kubectl /usr/local/bin/kubectl
    
    kubectl version --client

Install Minikube：

    官网地址：
    curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube
    
    sudo install minikube /usr/local/bin/
    
    use this:
    minikube start --driver=virtualbox --registry-mirror=https://registry.docker-cn.com
    
    minikube start --vm-driver=virtualbox --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers



## minikube 卸载

```bash
# 删除minikube集群
minikube delete

# 重置，重置之后会清理所有缓存的镜像，重头开始
rm -rf ~/.minikube
```



## minikube 使用

将本地 docker 与 k8s 中的 docker 绑定：

	eval $(minikube docker-env)
	eval $(minikube docker-env -u) 取消绑定

minikube usage:

```bash
# 启动 minikube 集群
minikube start --driver=none
minikube start --driver=virtualbox
minikube start --cpus=4 --memory=8096mb --disk-size='90000mb'
minikube start --registry-mirror=https://registry.docker-cn.com

# 查看集群状态
minikube status
kubectl cluster-info

# 查看 minkube 主机 ip
minikube ip

# start a new cluster
minikube start -p <name> 

# 增加内存
minikube config set memory 4096
 
# 登录虚拟机
minikube ssh

# 关闭 minikube 集群
minikube stop

# 获取 nodeport 端口
kubectl get service $SERVICE --output='jsonpath="{.spec.ports[0].nodePort}"'

```

minikube 插件使用

```bash
#列出当前支持的插件：
minikube addons list

#启用插件，例如 metrics-server：
minikube addons enable metrics-server

#禁用 metrics-server：
minikube addons disable metrics-server

#查看刚才创建的 Pod 和 Service：
kubectl get pod,svc -n kube-system

```

打开 dashboard 控制台：

```bash
minikube dashboard	
minikube dashboard --url

# 解决dashboard外部IP访问
nohup kubectl proxy --port=8887 --address='192.168.1.180' --accept-hosts='^.*' >/dev/null 2>&1 &
```

## 持久卷（PersistentVolume）

Minikube 支持 `hostPath` 类型的 [持久卷](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)。

这些持久卷会映射为 Minikube VM 内的目录。

Minikube VM 引导到 tmpfs，因此大多数目录不会在重新启动（`minikube stop`）之后保持不变。

但是，Minikube 被配置为保存存储在以下主机目录下的文件：

- `/data`
- `/var/lib/minikube`
- `/var/lib/docker`

下面是一个持久卷配置示例，用于在 `/data` 目录中保存数据：

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0001
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 5Gi
  hostPath:
    path: /data/pv0001/
```

## 挂载宿主机文件夹

一些驱动程序将在 VM 中挂载一个主机文件夹，以便你可以轻松地在 VM 和主机之间共享文件。目前这些都是不可配置的，并且根据你正在使用的驱动程序和操作系统的不同而不同。

> **说明：** KVM 驱动程序中尚未实现主机文件夹共享。

| 驱动          | 操作系统 | 宿主机文件夹 | VM 文件夹 |
| ------------- | -------- | ------------ | --------- |
| VirtualBox    | Linux    | /home        | /hosthome |
| VirtualBox    | macOS    | /Users       | /Users    |
| VirtualBox    | Windows  | C://Users    | /c/Users  |
| VMware Fusion | macOS    | /Users       | /Users    |
| Xhyve         | macOS    | /Users       | /Users    |

------

# rook-ceph

## rook 安装

1：检查硬盘状态

```bash
$ lsblk -f
NAME   FSTYPE FSVER LABEL UUID FSAVAIL FSUSE% MOUNTPOINT
sda                                           
`-sda1                            7.5G    50% /mnt/sda1
sdb                                           
sdc                                           
sr0         
```

sdb和sdc为可用的裸设备。

2：安装rook-ceph

```bash
git clone --single-branch --branch v1.4.5 https://github.com/rook/rook.git
cd rook/cluster/examples/kubernetes/ceph
kubectl create -f common.yaml
kubectl create -f operator.yaml

kubectl create -f cluster.yaml

# 配置 cluster-test.yaml
kubectl create -f cluster-test.yaml # minikube使用此安装ceph集群
```

3：安装toolbox工具

```bash
kubectl create -f toolbox.yaml

kubectl -n rook-ceph get pod -l "app=rook-ceph-tools"
```

连接并进入工具

```
kubectl -n rook-ceph exec -it $(kubectl -n rook-ceph get pod -l "app=rook-ceph-tools" -o jsonpath='{.items[0].metadata.name}') bash
```

常用状态查询

- `ceph status`
- `ceph osd status`
- `ceph df`
- `rados df`
- `ceph osd df`
- `ceph osd utilization`
- `ceph osd pool stats`
- `ceph osd tree`
- `ceph pg stat`

卸载工具

```bash
kubectl -n rook-ceph delete deployment rook-ceph-tools
```

配置 cluster-test.yaml

```bash
spec:
  dataDirHostPath: /data/rook # 配置第一项
    
# 集群级别的存储配置，每个节点都可以覆盖
  storage:
    # 是否所有节点都用于存储。如果指定nodes配置，则必须设置为false
    useAllNodes: true
    # 是否在节点上发现的所有设备，都自动的被OSD消费
    useAllDevices: false # 配置第二项
    # 正则式，指定哪些设备可以被OSD消费，示例：
    # sdb 仅仅使用设备/dev/sdb
    # ^sd. 使用所有/dev/sd*设备
    # ^sd[a-d] 使用sda sdb sdc sdd ^sd[b-d]
    # 可以指定裸设备,Rook会自动分区但不挂载
    deviceFilter: ^sd[b-d]  # 配置第三项
    # 每个节点上用于存储OSD元数据的设备。使用低读取延迟的设备，例如SSD/NVMe存储元数据可以提升性能
    # 可以针对每个节点进行配置
    nodes:
    # 节点A的配置
    - name: "172.17.4.101"
      directories:
      - path: "/rook/storage-dir"
      resources:
        limits:
          cpu: "500m"
          memory: "1024Mi"
        requests:
          cpu: "500m"
          memory: "1024Mi"
    # 节点B的配置      
    - name: "172.17.4.201"
      - name: "sdb"
      - name: "sdc"
      storeConfig:
        storeType: bluestore
    - name: "172.17.4.301"
      deviceFilter: "^sd."
```



## rook 卸载

1：清除实验pod和块存储

```bash
kubectl delete -f ../wordpress.yaml
kubectl delete -f ../mysql.yaml
kubectl delete -n rook-ceph cephblockpool replicapool
kubectl delete storageclass rook-ceph-block
kubectl delete -f csi/cephfs/kube-registry.yaml
kubectl delete storageclass csi-cephfs
```

2：Delete the CephCluster CRD

```bash
kubectl -n rook-ceph delete cephcluster rook-ceph
kubectl -n rook-ceph delete cephcluster my-cluster # minikube使用此项

kubectl -n rook-ceph get cephcluster # 验证集群是否删除成功
```

3：Delete the Operator and related Resources

```
kubectl delete -f operator.yaml
kubectl delete -f common.yaml
```

4：Delete the data on hosts

!>Connect to each machine and delete `/var/lib/rook`, or the path specified by the `dataDirHostPath`.

In the future this step will not be necessary when we build on the K8s local storage feature.

If you modified the demo settings, additional cleanup is up to you for devices, host paths, etc.

5：校验删除是否成功

```bash
kubectl -n rook-ceph get pod
kubectl -n rook-ceph get cephcluster
```

彻底删除

```bash
for CRD in $(kubectl get crd -n rook-ceph | awk '/ceph.rook.io/ {print $1}'); do kubectl patch crd -n rook-ceph $CRD --type merge -p '{"metadata":{"finalizers": [null]}}'; done
```

## Block Storage

Create the storage class.

```bash
kubectl create -f cluster/examples/kubernetes/ceph/csi/rbd/storageclass.yaml

# minikube 使用这个
kubectl create -f cluster/examples/kubernetes/ceph/csi/rbd/storageclass-test.yaml
```

Start mysql and wordpress from the `cluster/examples/kubernetes` folder:

```
kubectl create -f mysql.yaml
kubectl create -f wordpress.yaml
```

Both of these apps create a block volume and mount it to their respective pod. You can see the Kubernetes volume claims by running the following:

```
$ kubectl get pvc
NAME             STATUS    VOLUME                                     CAPACITY   ACCESSMODES   AGE
mysql-pv-claim   Bound     pvc-95402dbc-efc0-11e6-bc9a-0cc47a3459ee   20Gi       RWO           1m
wp-pv-claim      Bound     pvc-39e43169-efc1-11e6-bc9a-0cc47a3459ee   20Gi       RWO           1m
```

Once the wordpress and mysql pods are in the `Running` state, get the cluster IP of the wordpress app and enter it in your browser:

```
$ kubectl get svc wordpress
NAME        CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
wordpress   10.3.0.155   <pending>     80:30841/TCP   2m
```

You should see the wordpress app running.

If you are using Minikube, the Wordpress URL can be retrieved with this one-line command:

```
echo http://$(minikube ip):$(kubectl get service wordpress -o jsonpath='{.spec.ports[0].nodePort}')
```

> **NOTE**: When running in a vagrant environment, there will be no external IP address to reach wordpress with. You will only be able to reach wordpress via the `CLUSTER-IP` from inside the Kubernetes cluster.

**Teardown**

To clean up all the artifacts created by the block demo:

```
kubectl delete -f wordpress.yaml
kubectl delete -f mysql.yaml
kubectl delete -n rook-ceph cephblockpool replicapool
kubectl delete storageclass rook-ceph-block
```

## Object Storage



https://rook.io/docs/rook/v1.4/ceph-object.html



## Shared Filesystem

By default only one shared filesystem can be created with Rook. Multiple filesystem support in Ceph is still considered experimental and can be enabled with the environment variable `ROOK_ALLOW_MULTIPLE_FILESYSTEMS` defined in `operator.yaml`.

**Create the Filesystem**

The Rook operator will create all the pools and other resources necessary to start the service. This may take a minute to complete.

```bash
## Create the filesystem
$ kubectl create -f filesystem.yaml
[...]
## To confirm the filesystem is configured, wait for the mds pods to start
$ kubectl -n rook-ceph get pod -l app=rook-ceph-mds
NAME                                      READY     STATUS    RESTARTS   AGE
rook-ceph-mds-myfs-7d59fdfcf4-h8kw9       1/1       Running   0          12s
rook-ceph-mds-myfs-7d59fdfcf4-kgkjp       1/1       Running   0          12s
```

To see detailed status of the filesystem, start and connect to the [Rook toolbox](https://rook.io/docs/rook/v1.4/ceph-toolbox.html). A new line will be shown with `ceph status` for the `mds` service. In this example, there is one active instance of MDS which is up, with one MDS instance in `standby-replay` mode in case of failover.

```bash
$ ceph status
  ...
  services:
    mds: myfs-1/1/1 up {[myfs:0]=mzw58b=up:active}, 1 up:standby-replay
```

**Provision Storage**

Before Rook can start provisioning storage, a StorageClass needs to be created based on the filesystem. This is needed for Kubernetes to interoperate with the CSI driver to create persistent volumes.

> **NOTE**: This example uses the CSI driver, which is the preferred driver going forward for K8s 1.13 and newer. Examples are found in the [CSI CephFS](https://github.com/rook/rook/tree/release-1.4/cluster/examples/kubernetes/ceph/csi/cephfs) directory. For an example of a volume using the flex driver (required for K8s 1.12 and earlier), see the [Flex Driver](https://rook.io/docs/rook/v1.4/ceph-filesystem.html#flex-driver) section below.

Save this storage class definition as `storageclass.yaml`:

Create the storage class.

```bash
kubectl create -f cluster/examples/kubernetes/ceph/csi/cephfs/storageclass.yaml
```

**Consume the Shared Filesystem: K8s Registry Sample**

As an example, we will start the kube-registry pod with the shared filesystem as the backing store. Save the following spec as `kube-registry.yaml`:

Create the Kube registry deployment:

```
kubectl create -f cluster/examples/kubernetes/ceph/csi/cephfs/kube-registry.yaml
```

You now have a docker registry which is HA with persistent storage.

**Teardown**

To clean up all the artifacts created by the filesystem demo:

```
kubectl delete -f kube-registry.yaml
```

To delete the filesystem components and backing data, delete the Filesystem CRD.

> **WARNING: Data will be deleted if preservePoolsOnDelete=false**.

```
kubectl -n rook-ceph delete cephfilesystem myfs
```

Note: If the “preservePoolsOnDelete” filesystem attribute is set to true, the above command won’t delete the pools. Creating again the filesystem with the same CRD will reuse again the previous pools.



## StorageClass and PVC

ceph minikube StorageClass configuration：

StorageClass-test.yaml

```yaml
apiVersion: ceph.rook.io/v1
kind: CephBlockPool
metadata:
  name: replicapool
  namespace: rook-ceph
spec:
  failureDomain: host
  replicated:
    size: 1
    # Disallow setting pool with replica 1, this could lead to data loss without recovery.
    # Make sure you're *ABSOLUTELY CERTAIN* that is what you want
    requireSafeReplicaSize: false
    # gives a hint (%) to Ceph in terms of expected consumption of the total cluster capacity of a given pool
    # for more info: https://docs.ceph.com/docs/master/rados/operations/placement-groups/#specifying-expected-pool-size
    #targetSizeRatio: .5
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
   name: rook-ceph-block
# Change "rook-ceph" provisioner prefix to match the operator namespace if needed
provisioner: rook-ceph.rbd.csi.ceph.com
parameters:
    # clusterID is the namespace where the rook cluster is running
    # If you change this namespace, also change the namespace below where the secret namespaces are defined
    clusterID: rook-ceph

    # If you want to use erasure coded pool with RBD, you need to create
    # two pools. one erasure coded and one replicated.
    # You need to specify the replicated pool here in the `pool` parameter, it is
    # used for the metadata of the images.
    # The erasure coded pool must be set as the `dataPool` parameter below.
    #dataPool: ec-data-pool
    pool: replicapool

    # RBD image format. Defaults to "2".
    imageFormat: "2"

    # RBD image features. Available for imageFormat: "2". CSI RBD currently supports only `layering` feature.
    imageFeatures: layering

    # The secrets contain Ceph admin credentials. These are generated automatically by the operator
    # in the same namespace as the cluster.
    csi.storage.k8s.io/provisioner-secret-name: rook-csi-rbd-provisioner
    csi.storage.k8s.io/provisioner-secret-namespace: rook-ceph
    csi.storage.k8s.io/controller-expand-secret-name: rook-csi-rbd-provisioner
    csi.storage.k8s.io/controller-expand-secret-namespace: rook-ceph
    csi.storage.k8s.io/node-stage-secret-name: rook-csi-rbd-node
    csi.storage.k8s.io/node-stage-secret-namespace: rook-ceph
    # Specify the filesystem type of the volume. If not specified, csi-provisioner
    # will set default as `ext4`.
    csi.storage.k8s.io/fstype: ext4
# uncomment the following to use rbd-nbd as mounter on supported nodes
#mounter: rbd-nbd
allowVolumeExpansion: true
reclaimPolicy: Delete

```

mysql-pvc.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  ports:
    - port: 3306
  selector:
    app: wordpress
    tier: mysql
  clusterIP: None
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
  labels:
    app: wordpress
spec:
  storageClassName: rook-ceph-block
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
    tier: mysql
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: mysql
    spec:
      containers:
      - image: mysql:5.6
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: changeme
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
```

wordpress-pvc.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  ports:
  - port: 80
  selector:
    app: wordpress
    tier: frontend
  type: LoadBalancer
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wp-pv-claim
  labels:
    app: wordpress
spec:
  storageClassName: rook-ceph-block
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
    tier: frontend
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
      - image: wordpress:4.6.1-apache
        name: wordpress
        env:
        - name: WORDPRESS_DB_HOST
          value: wordpress-mysql
        - name: WORDPRESS_DB_PASSWORD
          value: changeme
        ports:
        - containerPort: 80
          name: wordpress
        volumeMounts:
        - name: wordpress-persistent-storage
          mountPath: /var/www/html
      volumes:
      - name: wordpress-persistent-storage
        persistentVolumeClaim:
          claimName: wp-pv-claim

```

## dashboard 配置

**1：NodePort  https 访问 dashboard**

配置 dashboard-external-https.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: rook-ceph-mgr-dashboard-external-https
  namespace: rook-ceph
  labels:
    app: rook-ceph-mgr
    rook_cluster: rook-ceph
spec:
  ports:
  - name: dashboard
    port: 8443
    protocol: TCP
    targetPort: 8443
  selector:
    app: rook-ceph-mgr
    rook_cluster: rook-ceph
  sessionAffinity: None
  type: NodePort
```

```bash
kubectl create -f dashboard-external-https.yaml
```

```bash
$ kubectl -n rook-ceph get service
NAME                     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
rook-ceph-mgr            ClusterIP   10.108.111.192   <none>        9283/TCP         4h
rook-ceph-mgr-dashboard  ClusterIP   10.110.113.240   <none>        8443/TCP         4h
rook-ceph-mgr-dashboard-external-https  NodePort    10.101.209.6     <none>        8443:31176/TCP   4h
```

In this example, port `31176` will be opened to expose port `8443` from the ceph-mgr pod. Find the ip address of the VM. If using minikube, you can run `minikube ip` to find the ip address. Now you can enter the URL in your browser such as `https://192.168.99.100:31176` and the dashboard will appear.

**2：port-forward  http 访问 dashboard：**

```bash
kubectl create -f dashboard-external-http.yaml

kubectl -n rook-ceph port-forward svc/rook-ceph-mgr-dashboard-external-http 7000 --address 192.168.1.180
```

通过 http://192.168.1.180:7000 访问

默认用户名是admin，默认密码通过以下命令获取

```bash
kubectl -n rook-ceph get secret rook-ceph-dashboard-password -o jsonpath="{['data']['password']}" | base64 --decode && echo
```

## osd 重启异常

```bash
# 查看卷组是否激活
lsblk -f

# 激活所有卷组，重启osd的pod
sudo vgchange -a y
```

常用命令

```bash
mount | grep rbd
kubectl drain minikube --ignore-daemonsets --delete-local-data
kubectl uncordon minikube

# 显示物理卷
sudo pvdisplay
sudo vgdisplay
sudo lvdisplay
# 将所有已知卷组设置为活动状态
sudo vgchange -a y
# 将卷组"vg1000"设置为活动状态
sudo vgchange -ay vg1000 
lsblk -f

# Rook pod status
kubectl get pod -n rook-ceph -o wide
kubectl get service -n rook-ceph -o wide

# Logs for Rook pods
kubectl logs -n rook-ceph -l app=rook-ceph-operator
kubectl logs -n rook-ceph -l mon=a
kubectl logs -n rook-ceph

kubectl -n <cluster-namespace> logs <pod-name> --all-containers
kubectl -n <cluster-namespace> logs <pod-name> -c <container-name>

```

lvm2 使用

**1.PV**

```
pvcreate :根据物理盘,创建pv
pvscan :查询目前系统里的pv
pvdisplay:显示pv的状态
pvremove:将pv属性移除
```

**2.VG**

```
vgcreate:创建vg
vgscan:查找当前系统里面的vg
vgdisplay:显示当前系统vg的状态
vgextend:给vg添加额外的pv
vgreduce:在vg内删除pv
vgchange:设置vg是否是启动状态(active)
vgremove:删除一个vg
```

**3.LV**

```
lvcreate:创建lv
lvscan:查询当前系统的lv
lvdisplay:显示lv的属性
lvextend:给lv添加容量
lvredurce:给lv减少容量
lvremove:删除一个lv
lvresize:对lv大小的容量进行调整
```



# presslabs/mysql-operator

## Getting started

Deploying and using the MySQL operator requires for it to have access to a Kubernetes cluster and to have installed [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) and [helm](https://github.com/helm/helm#install).

## Controller deploy

To deploy this controller, use the provided helm chart by running:

```
helm repo add presslabs https://presslabs.github.io/charts
helm install mysql-operator presslabs/mysql-operator
```

For more information about chart values see chart [README](https://github.com/presslabs/mysql-operator/blob/master/charts/mysql-operator/README.md). This chart will deploy the controller along with an [orchestrator](https://github.com/github/orchestrator) cluster.

**NOTE**: At every deploy a random password is generated for the orchestrator user. When running `helm upgrade` this will change the password on the orchestrator side but not in the clusters and this will break the communication between orchestrator and MySQL clusters. To solve this either use `helm upgrade --reuse-values` or specify the orchestrator password. We recommend specifying the orchestrator password.

## Specify the cluster credentials

Before creating a cluster, you need a secret that contains the ROOT_PASSWORD key. An example for this secret can be found at [examples/example-cluster-secret.yaml](https://github.com/presslabs/mysql-operator/blob/master/examples/example-cluster-secret.yaml).

Create a file named `example-cluster-secret.yaml` and copy into it the following YAML code:

```
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  # root password is required to be specified
  ROOT_PASSWORD: bm90LXNvLXNlY3VyZQ==
  # a user name to be created, not required
  USER: dXNlcm5hbWU=
  # a password for user, not required
  PASSWORD: dXNlcnBhc3Nz
  # a name for database that will be created, not required
  DATABASE: dXNlcmRi
```

This secret contains information about the credentials used to connect to the cluster, like `ROOT_PASSWORD`, `USER`, `PASSWORD`, `DATABASE`. Note that once those fields are set, changing them will not reflect in the MySQL server because they are used only at cluster bootstrap.

> ###### NOTE
>
> All secret fields must be base64 encoded.

Moreover, the controller will add some extra fields into this secret with other internal credentials that are used, such as the orchestrator user, metrics exporter used, and so on.

## Create and deploy the cluster

Now, to create a cluster you need just a simple YAML file that defines it. Create a file named `example-cluster.yaml` and copy into it the following YAML code:

```
apiVersion: mysql.presslabs.org/v1alpha1
kind: MysqlCluster
metadata:
  name: my-cluster
spec:
  replicas: 2
  secretName: my-secret
```

A more comprehensive YAML example can be found at [examples/examples-cluster.yaml](https://github.com/presslabs/mysql-operator/blob/master/examples/example-cluster.yaml).

> ###### NOTE
>
> Make sure that the cluster name is not too long, otherwise the cluster will fail to register with the orchestrator. See issue [#170](https://github.com/presslabs/mysql-operator/issues/170) to learn more.

To deploy the cluster, run the commands below which will generate a secret with credentials and the `MySQLCluster` resources into Kubernetes.

```
git clone https://github.com/presslabs/mysql-operator.git

$ kubectl apply -f example-cluster-secret.yaml
$ kubectl apply -f example-cluster.yaml
```

## Cluster configuration overview

Some important fields of `MySQLCluster` resource from `spec` are described in the following table:

| Field Name                       | Description                                                  | Example                            | Default value           |
| -------------------------------- | ------------------------------------------------------------ | ---------------------------------- | ----------------------- |
| `replicas`                       | The cluster replicas, how many nodes to deploy.              | 2                                  | 1 (a single node)       |
| `secretName`                     | The name of the credentials secret. Should be in the same namespace with the cluster. | `my-secret`                        | *is required*           |
| `initBucketURL`                  | The S3 URL that the cluster initialization backup is taken from | `s3://my-bucket/backup.xbackup.gz` | ""                      |
| `initBucketSecretName`           | The secret that the S3 credentials for the initialization are read from | `backups-secret`                   | ""                      |
| `backupSchedule`                 | Represents the time and frequency of making cluster backups, in a cron format with seconds. | `0 0 0 * * *`                      | ""                      |
| `backupURL`                      | The bucket URL where to put the backup.                      | `gs://bucket/app`                  | ""                      |
| `backupSecretName`               | The name of the secret that contains credentials for connecting to the storage provider. | `backups-secret`                   | ""                      |
| `backupScheduleJobsHistoryLimit` | The number of many backups to keep.                          | `10`                               | inf                     |
| `image`                          | Specify a custom Docker image to be used for the MySQL server container. | `percona:5.7-centos`               | nil                     |
| `mysqlConf`                      | Key-value configs for MySQL that will be set in `my.cnf` under `mysqld` section. | `max_allowed_packet: 128M`         | {}                      |
| `podSpec`                        | This allows to specify pod-related configs. (e.g. `imagePullSecrets`, `labels` ) |                                    | {}                      |
| `volumeSpec`                     | Specifications for PVC, HostPath or EmptyDir, used to store data. |                                    | (a PVC with size = 1GB) |
| `maxSlaveLatency`                | The allowed slave lag until it's removed from read service. (in seconds) | `30`                               | nil                     |
| `queryLimits`                    | Parameters for pt-kill to ensure some query run limits. (e.g. idle time) | `idleTime: 60`                     | nil                     |
| `readOnly`                       | A Boolean value that sets the cluster in read-only state.    | `True`                             | False                   |

For more detailed information about cluster structure and configuration fields can be found in [godoc](https://godoc.org/github.com/presslabs/mysql-operator/pkg/apis/mysql/v1alpha1#MysqlClusterSpec).

## Cluster data volume configuration

Currently, for a cluster generated with the MySQL operator you can set one of the following volume sources for MySQL data:

- Persistent Volume Claim
- Host Path (represents an existing directory on the host)
- Empty Dir (represents a temporary directory that shares a pod's lifetime).

## Persistent volume claim

An example to specify the PVC configuration:

```yaml
apiVersion: mysql.presslabs.org/v1alpha1
kind: MysqlCluster
metadata:
  name: my-cluster
spec:
  secretName: my-secret
  volumeSpec:
    persistentVolumeClaim:
   	  storageClassName: rook-ceph-block
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 20Gi
```

If nothing is specified under the `volumeSpec` field, the cluster is created by default with a PVC of `1Gi` in size, as described above. For more fine tuning of the PVC check the specs [godoc](https://godoc.org/k8s.io/api/core/v1#PersistentVolumeClaimSpec).

Note that the Kubernetes cluster needs to have a default storage class configured, otherwise it will fail to provision PVCs. If you want to change it, visit the [Kubernetes documentation](https://kubernetes.io/docs/tasks/administer-cluster/change-default-storage-class/), or just create a new storage class and specify it under `storageClassName` field in PVC specs.

> ###### NOTE
>
> Some additional information related to this topic can be found on issue [#168](https://github.com/presslabs/mysql-operator/issues/168).

## To list the deployed clusters use

```shell
$ kubectl get mysql
NAME         AGE
my-cluster   1m
```

## To check cluster state use

```shell
$ kubectl describe mysql my-cluster
...
Status:
  Ready Nodes:  2
  Conditions:
    Last Transition Time:  2018-03-28T10:20:23Z
    Message:               Cluster is ready.
    Reason:                statefulset ready
    Status:                True
    Type:                  Ready
...
```

## Access the orchestrator

The service `<release-name>-mysql-operator` exposes port 80. Via this port you will be able to talk to the orchestrator leader. You can either port forward this service to localhost, or use a service of type load balancer or enable the ingress.

```shell
kubectl port-forward service/<release-name>-mysql-operator 8080:80
kubectl port-forward service/mysql-operator 8080:80 --address 192.168.1.180
```

Then type `localhost:8080` in a browser.



# redis install





​    

​    


----------

# Helm 安装： #

----------

1：源码安装

    wget https://storage.googleapis.com/kubernetes-helm/helm-v2.9.1-linux-amd64.tar.gz
    tar -zxvf helm-v2.9.1-linux-amd64.tar.gz
    mv linux-amd64/helm /usr/local/bin/helm

2：脚本安装

	$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
	$ chmod 700 get_helm.sh
	$ ./get_helm.sh

3：安装tiller

	docker pull jessestuart/tiller:v2.16.8
	docker tag jessestuart/tiller:v2.16.8 gcr.io/kubernetes-helm/tiller:v2.16.8
	docker rmi jessestuart/tiller:v2.16.8
	
	minikube addons disable helm-tiller
	minikube addons enable helm-tiller
	kubectl get pod,svc -n kube-system

4：添加aliyun， github 和官方incubator charts repository仓库

	helm repo add gitlab https://charts.gitlab.io/
	helm repo add aliyun https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
	helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
	helm repo update

5：使用示例

	helm install stable/mysql
	helm show chart stable/mysql
	helm ls
	helm uninstall smiling-penguin
	helm status smiling-penguin
	helm get -h




----------

# kube-prometheus 安装： #

----------

快速安装：

```bash
git clone https://github.com/prometheus-operator/kube-prometheus.git
cd kube-prometheus
# Create the namespace and CRDs, and then wait for them to be availble before creating the remaining resources
kubectl create -f manifests/setup && \
until kubectl get servicemonitors --all-namespaces ; do date; sleep 1; echo ""; done && \
kubectl create -f manifests/ 

```

卸载：

	kubectl delete --ignore-not-found=true -f manifests/ -f manifests/setup

访问 dashboards：

	$ kubectl --namespace monitoring port-forward svc/prometheus-k8s 9090 --address 192.168.1.180
	Prometheus  http://localhost:9090
	$ kubectl --namespace monitoring port-forward svc/grafana 3000 --address 192.168.1.180
	Grafana http://localhost:3000 admin admin
	$ kubectl --namespace monitoring port-forward svc/alertmanager-main 9093
	Alert Manager http://localhost:9093

