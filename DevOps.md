[DevOps]

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