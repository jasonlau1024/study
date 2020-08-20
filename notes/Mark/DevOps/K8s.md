

# 初始K8s
## 概述
### K8s架构
**master节点：**
  Master是Kubernetes Cluster的大脑，运行着的Daemon服务包括 kube-apiserver、kube-scheduler、kube-controller-manager、etcd和Pod 网络(如Flannel)
- API Server（kube-apiserver）：
  API Server提供HTTP/HTTPS RESTful API，即Kubernetes API。 API Server是Kubernetes Cluster的前端接口，各种客户端工具（CLI或 UI）以及Kubernetes其他组件可以通过它管理Cluster的各种资源。
- Scheduler（kube-scheduler）：
  Scheduler负责决定将Pod放在哪个Node上运行。Scheduler在调度 时会充分考虑Cluster的拓扑结构，当前各个节点的负载，以及应用对 高可用、性能、数据亲和性的需求。
- Controller Manager（kube-controller-manager）：
  Controller Manager负责管理Cluster各种资源，保证资源处于预期的状态。Controller Manager由多种controller组成，包括replication controller、endpoints controller、namespace controller、serviceaccounts controller等。不同的controller管理不同的资源。
- etcd
  etcd负责保存Kubernetes Cluster的配置信息和各种资源的状态信息。当数据发生变化时，etcd会快速地通知Kubernetes相关组件。
- Pod 网络
  Pod要能够相互通信，Kubernetes Cluster必须部署Pod网络， flannel是其中一个可选方案。

**node节点：**
  Node是Pod运行的地方，Kubernetes支持Docker、rkt等容器 Runtime。Node上运行的Kubernetes组件有kubelet、kube-proxy和Pod 网络（如flannel）
- kubelet
  kubelet是Node的agent，当Scheduler确定在某个Node上运行Pod 后，会将Pod的具体配置信息（image、volume等）发送给该节点的kubelet，kubelet根据这些信息创建和运行容器，并向Master报告运行状态。
- kube-proxy
  service在逻辑上代表了后端的多个Pod，外界通过service访问 Pod。service接收到的请求是如何转发到Pod的呢？这就是kube-proxy 要完成的工作。每个Node都会运行kube-proxy服务，它负责将访问service的 TCP/UPD数据流转发到后端的容器。如果有多个副本，kube-proxy会 实现负载均衡。
- Pod 网络
  同 master 节点。

![kubernetes从入门到放弃--k8s基本架构- 朗月清风](https://lh3.googleusercontent.com/proxy/9XeTRZc2GFO902TLKZfNlPb1dZ-20NyhuUMOMX3sHcJ0nHjzBqu0b3YLxkQbmCneFUFQL3UbJ27ZKdZb)

## 快速体验

Kubernetes 官网提供了一个在线交互式教程，快速体验Kubernetes的功能和应用场景。
地址：https://kubernetes.io/docs/tutorials/kubernetes-basics/

### 创建K8S集群
```shell
# 1.查看minikube版本信息
$ minikube version
minikube version: v1.8.1
# 2. 启动创建一个集群
$ minikube start
# 3. 查看kubectl版本信息
$ kubectl version
Client Version: ......  # 本地kubectl版本
Server Version：......  # 集群master kubernetes版本信息
# 4. 查看集群信息
$ kubectl cluster-info
Kubernetes master is running at https://172.17.0.37:8443
KubeDNS is running at https://172.17.0.37:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
# 5. 查看集群中的节点
$ kubectl get nodes
NAME       STATUS   ROLES    AGE     VERSION
minikube   Ready    master   9m46s   v1.17.3
```

### 应用部署
Deployment：Deployment定义了K8S如何创建和更新应用程序的实例。K8s将应用程序实例调度到集群中的各个节点上
kubectl：Kubectl 使用 Kubernetes API 与集群进行交互。

```shell
# 1. 创建部署一个应用程序
kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1
# 2. 查看应用程序
$ kubectl get deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   1/1     1            1           58s
```

### 快速部署一个集群
#### 部署环境
三台主机：(RAM:2GB;CPU:2;Disk:30GB)

| HostName(IP)               | 角色   | 组件 |
| -------------------------- | ------ | ---- |
| K8sMaster01(192.168.36.50) | Master |      |
| K8snode01(192.168.36.51)   | worker |      |
| K8snode02(192.168.36.52)   | worker |      |

操作系统：Centos7.x-86_x64

![img](https://mmbiz.qpic.cn/mmbiz_jpg/d5patQGz8KdUBcI6TGe6DF7wL4zcnlibHERLF2Nibk38hccOdwyKxjc2txhSznW0DhXlugoNZ5YjPicS8nFghcTYA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**系统准备：**

```shell
# 1. 关闭防火墙
systemctl stop firewalld
systemctl disable firewalld

# 2. 关闭Selinux
sed -i 's/enforcing/disabled/'/etc/selinux/config
setenforce 0

# 3. 关闭Swap
swapoff -a
vim /etc/fstab

# 4. 添加Hosts
cat >> /etc/hosts << EOF
192.168.36.50   K8sMaster01
192.168.36.51   K8snode01
192.168.36.52   K8snode02
EOF

# 5. 将桥接的IPv4流量传递到iptables的链
cat >/etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables =1
net.bridge.bridge-nf-call-iptables =1
EOF
sysctl --system

# 6. 时间同步
yum install ntpdate -y
ntpdate time.windows.com
timedatectl set-timezone 'Asia/Shanghai'
```

#### 集群环境
Kubernetes默认CRI（容器运行时）为Docker，因此先安装Docker。
**安装docker：**
```shell
# 1. 添加docker Yum源
sudo wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
# 2. 安装docker-ce
sudo yum -y install docker-ce
sudo systemctl enable docker && sudo systemctl start docker
# 3. 配置镜像下载加速器
sudo sh -c 'cat >/etc/docker/daemon.json << EOF
{
  "registry-mirrors": ["https://b9pmyelo.mirror.aliyuncs.com"]
}
EOF'
systemctl restart docker
```
**安装K8s相关组件：**
kubeadm/kubelet/kubectl(由于版本更新频繁，这里指定版本号部署)
- kubelet：systemd守护进程管理
- kubeadm: 部署工具
- kubectl: k8s命令行管理工具
```shell
# 1. 添加 K8s 阿里云 Yum 软件源
sudo sh -c 'cat >/etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF'

# 2. 安装 kubeadm，kubelet和kubectl
sudo yum info kubelet kubeadm kubectl
sudo yum install -y kubelet-1.18.6 kubeadm-1.18.6 kubectl-1.18.6
sudo systemctl enable kubelet
```
#### 集群部署
**`kubeadm init` 初始化一个集群：**(Master)
```shell
kubeadm init \
--apiserver-advertise-address=192.168.36.50 \
--image-repository registry.aliyuncs.com/google_containers \
--kubernetes-version v1.18.0 \
--service-cidr=10.96.0.0/12 \
--pod-network-cidr=10.244.0.0/16
####  说明  ####
--apiserver-advertise-address   集群通告地址（master）
--image-repository  指定镜像仓库地址。默认k8s.gcr.io国内无法访问，这里指定阿里云的镜像仓库。
--kubernetes-version    指定K8s版本
--service-cidr  集群内部虚拟网络，Pod统一访问入口。
--pod-network-cidr  Pod网络，与下面部署的CNI网络组件yaml中保持一致。

# 初始化完成输出集群token
To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.36.50:6443 --token c480ph.ed0y6szt2spema1a \
    --discovery-token-ca-cert-hash sha256:32a699e1fdbcdae6ffaa27bca748b3c05d2ed56d5b5bb8cf1d757629f203bfe3
####  补充  ####
# 默认token有效期为24小时，当过期之后，该token就不可用了。这时就需要重新创建token。
kubeadm token create --print-join-command
```
**拷贝kubectl连接k8s认证文件到默认路径：**
```shell
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
$ kubectl get nodes
NAME          STATUS     ROLES    AGE     VERSION
k8smaster01   NotReady   master   5m53s   v1.18.6
```
**加入K8s Node：**
向集群添加新节点，执行在kubeadm init输出的kubeadm join命令。
这里在两台node机器执行，加如k8s集群。
```shell
kubeadm join 192.168.36.50:6443 --token c480ph.ed0y6szt2spema1a \
    --discovery-token-ca-cert-hash sha256:32a699e1fdbcdae6ffaa27bca748b3c05d2ed56d5b5bb8cf1d757629f203bfe3
$ kubectl get nodes
NAME          STATUS     ROLES    AGE     VERSION
k8smaster01   NotReady   master   16m     v1.18.6
k8snode01     NotReady   <none>   3m54s   v1.18.6
k8snode02     NotReady   <none>   3m20s   v1.18.6
####  说明  ####
这里查看到的STATUS都是NotReady，sudo journalctl -f -u kubelet 查看日志ready: NetworkReady=false。需要部署 集群网络。
```
#### 部署容器网络(CNI)
这里使用Flannel作为Kubernetes容器网络方案，解决容器跨主机网络通信。
Flannel是CoreOS维护的一个网络组件，Flannel为每个Pod提供全局唯一的IP，Flannel使用ETCD来存储Pod子网与Node IP之间的关系。flanneld守护进程在每台主机上运行，并负责维护ETCD信息和路由数据包。
```shell
$ wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
# 修改国内镜像地址
$ sed -i -r "s#quay.io/coreos/flannel:.*-amd64#lizhenliang/flannel:v0.11.0-amd64#g" kube-flannel.yml
kubectl apply -f kube-flannel.yml
$ kubectl get pods -n kube-system
$ kubectl get pod -A
```

#### 部署官方Dashborad(UI)
```shell
wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.3/aio/deploy/recommended.yaml
# 默认Dashboard只能集群内部访问，修改Service为NodePort类型，暴露到外部：
vi recommended.yaml
...
spec:
  ports:
- port:443
      targetPort:8443
      nodePort:30001
  selector:
    k8s-app: kubernetes-dashboard
  type: NodePort
...
# 部署
kubectl apply -f recommended.yaml
# 查看
kubectl get pods -n kubernetes-dashboard

## 创建service account并绑定默认cluster-admin管理员集群角色
# 创建用户
kubectl create serviceaccount dashboard-admin -n kube-system
# 用户授权
kubectl create clusterrolebinding dashboard-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin
# 获取用户Token
kubectl describe secrets -n kube-system $(kubectl -n kube-system get secret | awk '/dashboard-admin/{print $1}')

## 浏览器访问https://192.168.36.51:30001，输入上一步生成的token登录

```
#### 实例部署nginx应用
准备deployment文件：
```shell
cat nginx.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  # 创建2个nginx容器
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.19.1
        ports:
        - containerPort: 80
```
部署应用：
```shell
kubectl apply -f nginx-deployment.yaml
# 查看 pods
kubectl get pods # 或 kubectl get deployment

# 暴露出服务
kubectl expose deployment nginx-deployment --port=80 --type=LoadBalancer
# 查看服务状态
kubectl get services

####  说明  ####
部署过程：
1. kubectl发送部署请求到API Server。
2. API Server通知Controller Manager创建一个deployment资源。
3. Scheduler执行调度任务，将两个副本Pod分发到k8s-node1和 k8s-node2。
4. k8s-node1和k8s-node2上的kubectl在各自的节点上创建并运行 Pod。
```


## 运行应用



## Kubectl 命令详解
### kubectl describe
输出指定的一个或多个资源的详细信息。
*语法：*`kubectl describe (-f FILENAME | TYPE [NAME_PREFIX | -l label] | TYPE/NAME) [options]
`
**示例：**
1)查看一个node
`kubectl describe nodes k8smaster01`
2)查看一个pod
`kubectl describe pods/nginx`
3)查看pod.json中的资源类型和名称指定的pod
`kubectl describe -f pod.json`
4)查看所有的pod
`kubectl describe pods`
5)查看所有包含label app=nginx的pod
`kubectl describe po -l app=nginx`
6)指定命名空间查看pods
`kubectl describe pods nginx -n default`
7)指定命名空间查看指定service
`kubectl describe svc/nginx-deployment -n default`

### kubectl run
- 创建并运行一个或多个容器镜像。
- 创建一个deployment 或job 来管理容器。
*语法：*`kubectl run NAME --image=image [--env="key=value"] [--port=port] [--replicas=replicas] [--dry-run=bool] [--overrides=inline-json] [--command] -- [COMMAND] [args...] [options]`

**示例：**
1)启动nginx实例（默认是创建在default名称空间）
`kubectl run nginx --image=nginx`
2)创建到指定的kube-system空间中
`kubectl run nginx --image=nginx:1.8 --replicas=5 -l name=nginx -n kube-system`
3)启动hazelcast实例，暴露容器端口5701
`kubectl run hazelcast --image=hazelcast --port=5701`
4)启动实例，并进入容器内
`kubectl run -i -t busybox --image=busybox --restart=Never`

### kubectl delete
通过文件名、控制台输入、资源名或者label selector删除资源。接受JSON和YAML格式的描述文件。
*注：*
1. 只能指定以下参数类型中的一种：文件名、资源类型和名称、资源类型和label selector。
2. delete命令不检查资源版本。
*语法：*`kubectl delete ([-f FILENAME] | TYPE [(NAME | -l label | --all)]) [options]`

**实例：**
1)删除 pod.json 中指定资源类型和名称的pod
`kubectl delete -f ./pod.json 或 cat pod.json |kubectl delete -f -`
2)删除名为"baz"和"foo"的Pod和Service
`kubectl delete pod,service baz foo`
3)删除 Lable name = nginx 的Pod和Service
`kubectl delete pods,services -l name=nginx`
4)强制删除 dead node 上的pod
`kubectl delete pod foo --grace-period=0 --force`
5)删除所有pod
`kubectl delete pods --all`
6)删除deployment控制器中名为nginx
`kubectl delete deployments nginx`
7)删除kube-system名称空间中的nginx
`kubectl delete deployments nginx -n kube-system`

### kubectl expose
将资源暴露为新的Kubernetes Service。
*语法：*`kubectl expose (-f FILENAME | TYPE NAME) [--port=port] [--protocol=TCP|UDP] [--target-port=number-or-name] [--name=name] [--external-ip=external-ip-of-service] [--type=type] [options]`

**实例：**
1)为deployments的nginx创建service，并通过service的80端口转发至容器的8000端口上。
`kubectl expose rc nginx --port=80 --target-port=8000`
2)由“nginx-controller.yaml”中指定的type和name标识的RC创建Service，并通过Service的80端口转发至容器的8000端口上。
`kubectl expose -f nginx-controller.yaml --port=80 --target-port=8000`
3)暴露指定空间中的资源，例如下面指定暴露kube-system中的nginx资源暴露service名称为ye-nginx-service暴露的端口是80，容器的端口也是80（target-port）。
`kubectl expose deployments nginx  --port=80 --protocol=TCP --target-port=80  --name=ye-nginx-service -n kube-system`


### kubectl get
获取列出一个或多个资源的信息。

*语法：*`kubectl get [(-o|--output=)json|yaml|wide|custom-columns=...|custom-columns-file=...|go-template=...|go-template-file=...|jsonpath=...|jsonpath-file=...] (TYPE[.VERSION][.GROUP] [NAME | -l label] | TYPE[.VERSION][.GROUP]/NAME ...) [flags] [options]`

**可以使用的资源包括：**
componentstatuses (缩写 cs)
deployments (缩写 deploy)
namespaces (缩写 ns)
nodes (缩写 no)
pods (缩写 po)
replicasets (缩写 rs)
services (缩写 svc)



### kubectl scale
扩容或缩容 Deployment、ReplicaSet、Replication Controller或 Job 中Pod数量。
scale也可以指定多个前提条件，如：当前副本数量或 –resource-version ，进行伸缩比例设置前，系统会先验证前提条件是否成立。

*语法：*`kubectl scale [--resource-version=version] [--current-replicas=count] --replicas=COUNT (-f FILENAME | TYPE NAME) [options]`

**实例：**


## 运行应用
### Deployment
#### Deployment配置文件简介

Kubernetes支持两种创建资源的方式：
- 用kubectl命令直接创建：`kubectl run`
  适合临时测试或实验。
- 基于配置文件方式：`kubectl apply -f`
  - 配置文件提供了创建资源的模板，能够重复部署。
  - 可以像管理代码一样管理部署。
  - 适合正式的、跨环境的、规模化部署。

示例配置文件描述：
```shell
cat nginx.yaml
apiVersion: apps/v1 # 当前配置格式的版本。
kind: Deployment    # 创建的资源类型，
metadata:           # 资源的元数据
  name: nginx-deployment
  labels:
    app: nginx
spec:               # 规格说明
  replicas: 2       # 副本数量
  selector:
    matchLabels:
      app: nginx
  template:         # 定义Pod的模板
    metadata:       # 定义Pod的元数据
      labels:       # 至少要定义一个label。
        app: nginx
    spec:           # 描述Pod的规格
      containers:
      - name: nginx # name和image是必需的
        image: nginx:1.19.1
        ports:
        - containerPort: 80

```
#### 伸缩
修改YAML Depolyment文件，指定`replicas`数，再次执行`kubectl apply`即可。
```shell
$ kubectl get pods -o wide
NAME                               READY   STATUS    RESTARTS   AGE     IP            NODE        NOMINATED NODE   READINESS GATES
nginx-deployment-d4544f9cb-86cft   1/1     Running   0          19s     10.244.1.10   k8snode01   <none>           <none>
nginx-deployment-d4544f9cb-m765t   1/1     Running   2          3d21h   10.244.1.8    k8snode01   <none>           <none>
nginx-deployment-d4544f9cb-mw9qn   1/1     Running   2          3d21h   10.244.2.10   k8snode02   <none>           <none>
nginx-deployment-d4544f9cb-sv9x8   1/1     Running   0          19s     10.244.1.9    k8snode01   <none>           <none>
nginx-deployment-d4544f9cb-vk5dl   1/1     Running   0          19s     10.244.2.14   k8snode02   <none>           <none>
```
*补充：*
出于安全考虑，默认配置下Kubernetes不会将Pod调度到Master节 点。
- 将k8s-master也当作Node使用：
  `kubectl taint node k8s-master node-role.kubernetes.io/master-`
- 恢复Master Only状态：
  `kubectl taint node k8s-master node-role.kubernetes.io/master="":`

#### Failover
停止node2，一段时间后，Kubernetes会检查到node2不可用，将node2上的Pod标记为Unknown状态，并在node1上新创建两个 Pod，维持总副本数为3；
当node2恢复后，Unknown的Pod会被删除，不过已经运行的 Pod不会重新调度回node2。

#### 用label控制Pod的位置
*说明：*默认配置下，Scheduler会将Pod调度到所有可用的Node。不过有 些情况我们希望将Pod部署到指定的Node，比如将有大量磁盘I/O的 Pod部署到配置了SSD的Node；或者Pod需要GPU，需要运行在配置 了GPU的节点上。

1) 打标签
```shell
kubectl label node k8snode02 --show-labels
kubectl label node k8snode02 disktype=ssd
kubectl label node k8snode02 --show-labels
```
2）修改YAML文件

在Pod模板的`spec`里通过`nodeSelector`指定将此Pod部署到具有label disktype=ssd的Node上。
```SHELL
... ...
spec:
  ... ...
  nodeSelector:
    disktype: ssd
$ kubectl apply -f nginx.yaml
$ kubectl get pods -o wide
# 可以看到 nginx 的 pod 都运行在k8snode02
```

3）删除标签
`kubectl label node k8snode02 disktype-`


### DaemonSet
Deployment部署的副本Pod会分布在各个Node上，每个Node都可 能运行好几个副本。DaemonSet的不同之处在于：每个Node上最多只 能运行一个副本。

**典型应用场景：**
- 在集群的每个节点上运行存储Daemon。如：glusterd或ceph
- 在每个节点上运行日志收集Daemon。如：flunentd或 logstash。
- 在每个节点上运行监控Daemon。如：Prometheus Node Exporter或collectd

k8s使用DaemonSet运行系统组件
```SHELL
$ kubectl get daemonset --namespace=kube-system
NAME                      DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-flannel-ds-amd64     3         3         3       3            3           <none>                   7d23h
kube-flannel-ds-arm       0         0         0       0            0           <none>                   7d23h
kube-flannel-ds-arm64     0         0         0       0            0           <none>                   7d23h
kube-flannel-ds-ppc64le   0         0         0       0            0           <none>                   7d23h
kube-flannel-ds-s390x     0         0         0       0            0           <none>                   7d23h
kube-proxy                3         3         3       3            3           kubernetes.io/os=linux   8d
## DaemonSet kube-flannel-ds和kube-proxy分别负责在每个节点上运 行flannel和kube-proxy组件。
```
#### DaemonSet:kube-proxy
```shell
$ kubectl edit daemonset kube-proxy --namespace=kube-system
apiVersion: apps/v1
kind: DaemonSet # 指定这是一个DaemonSet类型的资源
... ...
      containers: # kube-proxy的容器定义
      - command:
        - /usr/local/bin/kube-proxy
        - --config=/var/lib/kube-proxy/config.conf
        - --hostname-override=$(NODE_NAME)
... ...
status: # 当前DaemonSet的运行时状态
  currentNumberScheduled: 3
  desiredNumberScheduled: 3
  numberAvailable: 3
  numberMisscheduled: 0
  numberReady: 3
  observedGeneration: 1
  updatedNumberScheduled: 3
```
#### 运行自己的DaemonSet
以 Prometheus Node Exporter 示例：Node Exporter是Prometheus 的agent，以Daemon的形式运行在每个被监控节点上。
```shell
cat node-exporter.yaml
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: node-exporter
  namespace: kube-system
  labels:
    k8s-app: node-exporter
spec:
  template:
    metadata:
      labels:
        k8s-app: node-exporter
    spec:
      containers:
      - image: prom/node-exporter
        name: node-exporter
        ports:
        - containerPort: 9100
          protocol: TCP
          name: http
---
apiVersion: v1
kind: Service
metadata:
  labels:
    k8s-app: node-exporter
  name: node-exporter
  namespace: kube-system
spec:
  ports:
  - name: http
    port: 9100
    nodePort: 31672
    protocol: TCP
  type: NodePort
  selector:
    k8s-app: node-exporter

```




# 应用实例
## Prometheus 监控系统

### Prometheus简介
Prometheus是一套开源的系统监控报警框架。2016年，Prometheus正式加入CNCF（Cloud Native Computing Foundation），是仅次于Kubernetes的第二个项目，目前已经全面接管了 Kubernetes项目的整套监控体系。

Prometheus的监控是基于时序数据的，即通过采样数据(metrics)，不断获取监控目标的状态信息，即时地记录与展示，并根据设定的门限和方式及时发布告警。

作为应用与Kubernetes的监控体系，Prometheus具备诸多的优势，如：
- Kubernetes默认支持,非常适合容器和微服务
- 无依赖，安装方便，上手容易
- 社区活跃，它不仅仅是个工具，而是生态
- 已有很多插件或者exporter，可以适应多种应用场景的数据收集需要
- Grafana默认支持,提供良好的可视化
- 高效，单一Prometheus可以处理百万级的监控指标，每秒处理数十万的数据点
而当前Prometheus最大的缺点就是暂时还不支持集群化。

**基本架构图：**
![img](https://www.kubernetes.org.cn/img/2020/03/1.png)

**相关组件：**
- `Prometheus Server`：是Prometheus架构中的核心部分，负责实现对监控数据的获取、存储及查询。Prometheus Server可以通过静态配置管理监控目标，也可以配合使用Service Discovery的方式动态管理监控目标，并从这些监控目标中获取数据。其次Prometheus Server本身也是一个时序数据库，将采集到的监控数据按照时间序列的方式存储在本地磁盘当中。Prometheus Server对外提供了自定义的PromQL，实现对数据的查询以及分析。
- `Exporter`：是提供监控数据的来源。Exporter分为两类：一类Exporter直接内置了对Prometheus监控的支持，如Kubernetes、etcd等；另一类是因为原有监控目标并不直接支持Prometheus，需要通过Prometheus提供的Client Library编写该监控目标的监控采集程序，如Mysql、JMX等。对于Exporter，Prometheus Server采用pull的方式来采集数据。
- `PushGateway`：同样是监控数据的来源。对于由于特定原因，如网络环境不允许等，Prometheus Server不能直接与Exporter进行通信时，可以使用PushGateway来进行中转。内部网络的监控数据主动Push到Gateway中，而和对Exporter一样，Prometheus Server也利用pull的方式从PushGateway采集数据。
- `Alertmanager`：是Prometheus体系中的告警组件。在Prometheus Server中可以设定门限与警报规则。当采集到的数据满足相关规则后，就会产生一条告警。Alertmanager从 Prometheus Server接收到告警后，会根据事先设定的路径，向外发出告警。常见的告警发送路径有：电子邮件、PagerDuty、Webhook、Slack等。
- 数据展示与输出：Prometheus Server有内置的UI用于展示采集到的监控数据。在该UI上，可以通过各种内置的数学公式对原始数据进行加工，并通过图形化的方式展现出来。当然，Prometheus Server原生UI的展示方式还是比较基础和单薄，所以目前更多的是通过对接Grafana来进行数据的展示，可以得到更好的展示效果。此外，Prometheus Server也提供API的方式来实现对监控数据的访问。


### prometheus-operator与kube-prometheus

在最新版本中，kubernetes的prometheus-operator部署内容已经从prometheus-operator的github工程中拆分出独立工程kube-prometheus。

kube-prometheus即是通过operator方式部署的kubernetes集群监控，所以我们直接容器化部署kube-prometheus即可。

kube-prometheus Github地址：https://github.com/coreos/kube-prometheus/releases


## Prometheus 持久化安装

参考资料：
kubernetes1.18安装kube-prometheus<https://blog.csdn.net/guoxiaobo2010/article/details/106532357>
grafana&prometheus生产级容器化监控(重点)<https://cloud.tencent.com/developer/article/1559844>

阿里云OSS作为k8s存储(pv/pvc)实践<https://blog.csdn.net/weixin_40449300/article/details/106938845>


镜像下载失败：`docker pull quay.mirrors.ustc.edu.cn/coreos/kube-rbac-proxy:v0.4.1`
从其它机器获取镜像包：`docker save quay.io/coreos/kube-rbac-proxy:v0.4.1 -o /opt/kube-rbac-proxy.tar`
加载到服务器镜像仓库：`docker load -i kube-rbac-proxy.tar`
*注：*镜像名时候与YAML配置文件定义的镜像保持一致，`docker tag IMAGEID(镜像id) REPOSITORY:TAG`








查看Pod运行信息：
```shell
$ kubectl get pod -n monitoring -o wide
NAME                                   READY   STATUS    RESTARTS   AGE     IP              NODE
alertmanager-main-0                    2/2     Running   0          33m     10.244.2.22     k8snode02
alertmanager-main-1                    2/2     Running   0          33m     10.244.1.24     k8snode01
alertmanager-main-2                    2/2     Running   0          33m     10.244.0.5      k8smaster01
grafana-67dfc5f687-bpsvn               1/1     Running   0          33m     10.244.2.23     k8snode02
kube-state-metrics-c7f9bdccb-72ccc     3/3     Running   0          33m     10.244.2.24     k8snode02
node-exporter-22584                    2/2     Running   0          33m     192.168.36.50   k8smaster01
node-exporter-9xtml                    2/2     Running   0          33m     192.168.36.51   k8snode01
node-exporter-l25qq                    2/2     Running   0          33m     192.168.36.52   k8snode02
prometheus-adapter-66b855f564-rfcqd    1/1     Running   0          33m     10.244.1.25     k8snode01
prometheus-k8s-0                       3/3     Running   1          33m     10.244.2.25     k8snode02
prometheus-k8s-1                       3/3     Running   3          33m     10.244.1.26     k8snode01
prometheus-operator-7bf55877f5-2svm9   2/2     Running   0          3h20m   10.244.1.23     k8snode01
```

kube-prometheus主要组件概述

| 组件                | 个数 | 原生 | 作用                                                         |
| ------------------- | ---- | ---- | ------------------------------------------------------------ |
| alertmanager-main   | 3    | 是   | 提供报警插件的支持，可集成钉钉、微信等报警插件               |
| grafana             | 1    | 是   | 提供可视化Web界面                                            |
| kube-state-metrics  | 1    | 是   | 监听Kubernetes API服务器获取对象状态度量。Grafana和prometheus的数据来源 |
| prometheus-adapter  | 1    | 是   | prometheus属于第三方解决方案，原生的k8s系统并不能对Prometheus的自定义指标进行解析，需要借助于k8s-prometheus-adapter将这些指标数据查询接口转换为标准的Kubernetes自定义指标。 |
| prometheus-k8s      | 2    | 是   | prometheus,存放集群度量数据。                                |
| prometheus-operator | 1    | 是   | Operator 是最核心的部分，作为一个控制器，他会去创建 Prometheus 、 ServiceMonitor 、 AlertManager 以及 PrometheusRule 4个 CRD 资源对象，然后会一直监控并维持这4个资源对象的状态。可以这样类比理解,相当于statefulset/deployment与POD的关系。 |
| webhook-dingtalk    | 1    | 否   | 集成钉钉报警机器人                                           |




### 创建命令空间
为管理需要，所有Prometheus组件都应运行在一个独立的命名空间当中。因此安装的第一步，就是要创建一个新的Namespace，此处为“monitoring”。
```shell
apiVersion: v1
kind: Namespace
metadata:
  name: monitoring
```
### 部署node-exporter
node-exporter是以DaemonSet对象的方式进行部署的，可以确保每个Kubernetes Node的数据都会被采集到Prometheus。
```YAML
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: prometheus-node-exporter
  namespace: monitoring
    - image: prom/node-exporter:v1.0.1
      name: prometheus-node-exporter
        containerPort: 9100
        hostPort: 9100
    hostNetwork: true
```
*注：*`node-exporter`开放了hostPort：9100，所以可以通过直接访问<Node_IP>:9100来访问`node-exporter`采集到的数据。
除DaemonSet外，还需要部署对应的Service，供Prometheus Server对接使用。需要注意的是，该Service只开放了Cluster内部端口，不能直接从外部访问。
```YAML
apiVersion: v1
kind: Service
metadata:
  annotations:
    prometheus.io/scrape: 'true'
  name: prometheus-node-exporter
  namespace: monitoring
```
### 部署kube-state-metrics
除了node-exporter，还可以部署另一个数据来源，kube-state-metrics。kube-state-metrics关注于获取kubernetes各种资源的最新状态，如deployment或者daemonset等。kube-state-metrics轮询Kubernetes API，并将Kubernetes的结构化信息转换为metrics，将kubernetes的运行状况在内存中做个快照。
*说明：*因为kube-state-metrics要访问API，所以要先创建ServiceAccount来提供权限。之后再部署相应的Deployment:
```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kube-state-metrics
  namespace: monitoring
spec:
  replicas: 1
    containers:
    - name: kube-state-metrics
      image: gcr.io/goole_containers/kube-state-metrics:v0.5.0
      ports:
      - containerPort: 8080

```
为了和Prometheus Server对接，也要部署对应的Service。和node-exporter一样，这个Service也只开放了Cluster内部端口，不能直接从外部访问。
```YAML
apiVersion: v1
kind: Service
metadata:
  annotations:
    prometheus.io/scrape: 'true'
  name: kube-state-metrics
  namespace: monitoring
```



