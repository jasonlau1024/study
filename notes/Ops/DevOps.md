[DevOps]



# Docker





# K8S





## 部署实践

### 使用kebeadm快速部署一个K8s集群

#### 概述

kubeadm是官方社区推出的一个用于快速部署kubernetes集群的工具。

这个工具能通过两条指令完成一个kubernetes集群的部署：

- 创建一个 Master 节点：`kubeadm init`
- 将一个 Node 节点加入当前集群：`kubeadm join <Master 节点的 IP 和端口>`

#### 部署环境



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



**主机环境：**

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