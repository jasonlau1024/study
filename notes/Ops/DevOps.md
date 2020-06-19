[DevOps]



# Docker

## 概述

### Docker是什么

 **使用最广泛的开源容器引擎** 

 **一种操作系统级的虚拟化技术** 

 **依赖于Linux内核特性：Namespace（**资源隔离**）和Cgroups（**资源限制**）** 

 **一个简单的应用程序打包工具**

### Docker设计目标

 **提供简单的应用程序打包工具** 

 **开发人员和运维人员职责逻辑分离** 

 **多环境保持一致性**

### Docker基本组成

 **Docker Client：客户端** 

 **Ddocker Daemon：守护进程** 

 **Docker Images：镜像** 

 **Docker Container：容器** 

 **Docker Registry：镜像仓库**

![1592203621187](C:\Users\jason\AppData\Roaming\Typora\typora-user-images\1592203621187.png)

### 容器 vs 虚拟机



![1592203675352](C:\Users\jason\AppData\Roaming\Typora\typora-user-images\1592203675352.png)







## 安装

**版本说明：**

- **社区版（Community Edition，CE）** 

- **企业版（Enterprise Edition，EE）**

**支持平台：**

- **Linux（CentOS,Debian,Fedora,Oracle Linux,RHEL,SUSE和Ubuntu）** 

- **Mac** 

- **Windows**

### CentOS7.x 安装 Docker

>官方文档：https://docs.docker.com
>
>阿里云源：http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

```shell
# 安装依赖包
yum install -y yum-utils device-mapper-persistent-data lvm2
# 添加Docker软件包源
yum-config-manager \
--add-repo \
https://download.docker.com/linux/centos/docker-ce.repo
# 安装Docker CE
yum install -y docker-ce
# 启动Docker服务并设置开机启动
systemctl start docker
systemctl enable docker
## 更换阿里源
cat > /etc/docker/daemon.json << EOF
{
  "registry-mirrors": ["https://b9pmyelo.mirror.aliyuncs.com"]
}
EOF
```







# K8S





## 部署实践

### 使用kebeadm快速部署一个K8s集群

#### 概述

kubeadm是官方社区推出的一个用于快速部署kubernetes集群的工具。

这个工具能通过两条指令完成一个kubernetes集群的部署：

- 创建一个 Master 节点：`kubeadm init`
- 将一个 Node 节点加入当前集群：`kubeadm join <Master 节点的 IP 和端口>`

#### 部署环境

##### 服务器要求

CPU:2核+

Mem:2GB+

Disk:30GB+

集群中所有机器之间网络互通

禁止Swap分区（`swapoff -a`）

关闭防火墙/Selinux

节点之中不可以有重复的主机名、MAC 地址(`ip link` 或 `ifconfig -a`)或 product_uuid(`cat /sys/class/dmi/id/product_uuid`)

**环境配置：**

```shell
关闭防火墙：
$ systemctl stop firewalld
$ systemctl disable firewalld

关闭selinux：
$ sed -i 's/enforcing/disabled/' /etc/selinux/config  # 永久
$ setenforce 0  # 临时

关闭swap：
$ swapoff -a  # 临时
$ vim /etc/fstab  # 永久

设置主机名：
$ hostnamectl set-hostname <hostname>

在master添加hosts：
$ cat >> /etc/hosts << EOF
172.31.215.246	k8s-master01
172.31.215.248	k8s-node01
172.31.215.241	k8s-node02
172.31.215.240	k8s-node03
EOF

将桥接的IPv4流量传递到iptables的链：
$ cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
$ sysctl --system  # 生效

时间同步：
$ yum install ntpdate -y
$ ntpdate time.windows.com

添加阿里云YUM软件源：
$ cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

##### 检查所需端口

控制面板节点

| 协议 | 方向 | 端口范围  | 作用                    | 使用者                       |
| :--- | :--- | :-------- | :---------------------- | :--------------------------- |
| TCP  | 入站 | 6443*     | Kubernetes API 服务器   | 所有组件                     |
| TCP  | 入站 | 2379-2380 | etcd server client API  | kube-apiserver, etcd         |
| TCP  | 入站 | 10250     | Kubelet API             | kubelet 自身、控制平面组件   |
| TCP  | 入站 | 10251     | kube-scheduler          | kube-scheduler 自身          |
| TCP  | 入站 | 10252     | kube-controller-manager | kube-controller-manager 自身 |

工作节点

| 协议 | 方向 | 端口范围    | 作用            | 使用者                     |
| :--- | :--- | :---------- | :-------------- | :------------------------- |
| TCP  | 入站 | 10250       | Kubelet API     | kubelet 自身、控制平面组件 |
| TCP  | 入站 | 30000-32767 | NodePort 服务** | 所有组件                   |

##### 集群架构

**架构图：**

![kubernetesæ¶æå¾](https://blog-1252881505.cos.ap-beijing.myqcloud.com/k8s/single-master.jpg)

**集群服务列表：**

| 角色         | IP             | 安装组件                               |
| ------------ | -------------- | -------------------------------------- |
| k8s-master01 | 172.31.215.246 | apiserver/controller-manager/scheduler |
| k8s-node01   | 172.31.215.248 | kubeadm/kubelet/docker + etcd          |
| k8s-node02   | 172.31.215.241 | kubeadm/kubelet/docker + etcd          |
| k8s-node03   | 172.31.215.240 | kubeadm/kubelet/docker + etcd          |

**部署步骤：**

1. `work`节点安装`Docker/kubeadm/kubelet`（Kubernetes默认CRI（容器运行时）为Docker，因此先安装Docker）



#### 集群部署

```shell
yum install -y kubelet-1.18.3 kubeadm-1.18.3 kubectl-1.18.3
systemctl enable kubelet
## 部署 Master
kubeadm init \
  --apiserver-advertise-address=172.31.215.246 \
  --image-repository registry.aliyuncs.com/google_containers \
  --kubernetes-version v1.18.3 \
  --service-cidr=10.96.0.0/12 \
  --pod-network-cidr=10.244.0.0/16
输出：
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.31.215.246:6443 --token t41zsd.fjtyorklt1iktgfm \
    --discovery-token-ca-cert-hash sha256:ed3493b2e2cc1c209b252892eca2e68ffb88e4e697920e24d4da31a529e66c57
## 配置kubectl工具
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
## Master 安装Pod网络插件CNI（确保能访问quay.io这个registery）
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
## Work 加入集群
kubeadm join 172.31.215.246:6443 --token t41zsd.fjtyorklt1iktgfm \
    --discovery-token-ca-cert-hash sha256:ed3493b2e2cc1c209b252892eca2e68ffb88e4e697920e24d4da31a529e66c57
## 查看集群节点
kubectl get nodes
```

























# Prometheus

> 相关查看文档：
>
> Prometheus最强篇：https://mp.weixin.qq.com/s/L4xaAeU2JoUx03WaITZQmQ

## 概述

### 介绍

> Prometheus是SoundCloud开源的监控告警系统，使用Golang开发。2012年开始编码，2015年在Github上开源，2016年加入CNCF成为继K8s之后第二名成员。

`Prometheus`已经成为了云原生中指标监控的事实标准，几乎所有`Kubernetes`的核心组件以及其它云原生系统都以`Prometheus`的指标格式输出自己的运行时监控信息。

***主要功能：***

1. 多维数据模型，时序数据由`Metric`和多个`Label`组成。
2. `PromQL`灵活的查询语法。
3. 无依赖存储，支持本地和远程存储。
4. 通过`HTTP`协议采用`PULL`的方式拉取数据。
5. 可以采用服务发现或者静态配置方式，来发现目标服务。
6. 支持多种统计数据模型，UI 优化，`Grafana`也支持`Prometheus`。

### 架构

官方架构图：（Prometheus大部分组件使用`Go`开发，少部分组件使用Java、Python以及Ruby）

![img](https://mmbiz.qpic.cn/mmbiz_png/x63NLUqhL5E4SgsQqEibBiaicRdsxqzFw8h56SnuTbCNN6NcibV6Yibv4bU0ic2ib9GWL44Vd8NVvubuiaJ9BcQcibQbDiaQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

`Prometheus`的6个核心组件：

1. 