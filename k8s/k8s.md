# K8S

## 前世今生



## 架构介绍

### 架构

* master
  * apiServer:  所有服务访问的入口。接收用户输入的命令，提供认证、授权、API注册和发现等机制
  * schedule: 调度器负责接受任务，选择合适的节点分配任务
  * replication controller:  负责维护集群的状态，比如程序部署安排、故障检测、自动扩展、滚动更新等
  * etcd：可信赖分布式键值存储服务，保存分布式需要持久化相关信息
* node
  * kubele：负责维护容器的生命周期，即通过控制docker，来创建、更新、销毁容器
  * kubproxy： 负责提供集群内部的服务发现和负载均衡

### 其他组件

* CoreDNS： 为集群创建一个域名ip对应关系的解析
* DashBoard： 给k8s提供一个web ui
* ingress controller：官网只能实现四层代理，ingress可以实现七层代理
* fedetation：提供一个可以跨集群中心多k8s的统一管理功能
* prometheus：提供一个k8s集群的监控能力
* elk：提供k8s集群日志统一分析接入平台



## 其他组件

### pod的概念

kubernetes的最小控制单元，容器都是运行在pod中的，一个pod中可以有1个或者多个容器。可以理解为pod就是一个隔离的独立空间，用于承载容器运行。

### replicationConteoller (RC)

保证容器应用的副本数始终保持在用户定义的副本数。

### replicaSet (RS)

与rc没有本质的不同

### deployment

建议使用deployment管理，部分特性只有deployment支持

### statufulSet

解决有状态服务的问题（对应的deployments和replicaSets是为了无状态服务而设定的）。使用场景：

* 稳定的持久化存储
* 稳定的网络标志
* 有序部署有序拓展
* 有序收缩有序删除

### DaemonSet

确保全部或者一些node上面运行一个pod的副本。

典型的用法：

* 运行集群存储
* 在每个node上运行日志收集
* 在每个node上运行监控daemon,比如prometheus

### job

负责处理批处理任务，即仅执行一次的任务。保证批处理任务的一个或者多个成功结束。

cronjob管理基于时间的job: 在给定的时间运行或者周期性运行







### 网络通信方式

k8s假定所有的pod都在一个可以直接连通的扁平网络空间。这在`gce goolge cumpute engine` 中是现成的网络模型，k8s假定已经存在。所以需要使用到开源的解决方案，将不同节点的容器打通，然后运行。

目前通用的解决方案是Fannel。 coreos公司开发。





## 集群工具

### kubectl 

k8s集群的命令行管理工具，通过kubectl能够对集群本身进行管理，以及对容器化应用进行安装和管理。

语法：

```shell
kubectl  [command]  [type]  [name]  [flags]

# command 对资源执行的操作 create get delete 
# type 执行资源类型,单数 复数 等
# name 资源名称
# flags 指定可选参数，端口 地址等

kubectl get nodes
kebuctl get nodes   具体的node名称
```

部分command

| 命令             | 说明     |
| ---------------- | -------- |
| kubectl  create  | 创建资源 |
| kubcectl  delete | 删除资源 |
| kubectl  apply   |          |



## k8s 搭建
### 取消swap

```shell
nano /etc/fstab
```

### 安装docker

```shell
sudo apt install docker.io

# 添加源
 cat >/etc/docker/daemon.json <<EOF
{
    "registry-mirrors": ["https://g2djyyu3.mirror.aliyuncs.com"],
     "exec-opts": [ "native.cgroupdriver=systemd" ]
}
EOF

# 重启docker服务
sudo systemctl restart docker
```

### 添加k8s源

```shell
curl -s https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add -

cat >/etc/apt/sources.list.d/kubernetes.list<<EOF
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF
```

### 安装集群工具

```shell
sudo apt install  kubelet=1.18.5-00 kubeadm=1.18.5-00 kubectl=1.18.5-00  -y
```

### 集群初始化

```shell
kubeadm  init --pod-network-cidr=10.244.0.0/16 --image-repository=registry.aliyuncs.com/google_containers  --kubernetes-version=1.18.5 --apiserver-advertise-address=192.168.56.90
```

执行后部分结果

```shell
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.56.90:6443 --token s4wm92.gnonupgepvvqgpso \
    --discovery-token-ca-cert-hash sha256:5e23dd9d3ba1ef3442fc27f7f5b1077e0dee30b51629c2a2db27d46396cab53d 
```

根据提示执行

```shell
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```



### 安装网络插件

```shell
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

kubectl apply -f kube-flannel.yml 
```

稍等片刻之后，node节点会是read状态

```shell
root@master:/home/xnj# kubectl  get nodes
NAME     STATUS   ROLES    AGE   VERSION
master   Ready    master   15m   v1.18.5
node1    Ready    <none>   12m   v1.18.5
```

## 二进制集群搭建


## 资源清单 

k8s对资源管理和资源对象的管理都是采用的yaml文件。





## pod详解

### 容器和pod的区别

* pod是k8s的最小单元

* pod里面可以包括多个容器，一组容器的集合
* pod中的容器是共享网络命名空间
* pod是一个短暂存在的

### 镜像拉取策略

```shell
imagePullPolicy:
- IfNotPresent  默认值，如果本地没有就从互联网进行下载
- Always  每次创建pod都会重新拉取一次镜像
- Nerver  pod永远不会主动拉取这个镜像
```



### 资源限制

```shell
resources:
- requests:  # 请求的资源（最小资源）
	memory: "64M"
	cpu: "250M"
- limits:  # 最大的资源
	memory:
	cpu:
```



































