[toc]



### 

Kubernetes

#### 简介

> Go语言开发，消耗资源小，基于Google内部的Borg系统设计思想进行设计并开源；特性伸缩；负载均衡：IPVS

无状态服务：LVS、APACHE，Docker主要适用于无状态服务

有状态服务：DBMS

#### 组件说明



Borg系统

![image-20210823213651275](E:\我的\学习文档\md-pic\image-20210823213651275.png)

K8s系统

![image-20210823213835695](E:\我的\学习文档\md-pic\image-20210823213835695.png)

##### API Server

> 所有服务的统一访问入口

##### Controller Manager:

> 维持副本期望数目

##### Scheduler：

> 负责接收任务，选择合适的节点进行分配任务

##### ETCD

> 键值存储服务，存储K8S集群所有重要信息（持久化）==推荐使用V3版本==
>
> etcd V2版：数据保存在内存中
>
> etcd V3版：数据保存到磁盘中，使用本地卷的持久化操作

##### Kubelet

> 直接跟容器引擎交互实现容器的生命周期管理

##### Kube Proxy

> 负责写入规则至IPTABLES、IPVS实现服务映射访问的

##### CoreDNS

> 可以为集群中的SVC创建一个域名IP的对应关系解析

##### Dashboard

> 给K8S集群提供一个B/S结构访问体系

##### Ingress Controller

> 官方只能实现四层代理，Ingress可以实现七层代理

##### Federation

> 提供一个可以跨集群中心==多K8S统一管理==功能

##### Prometheus

> 提供K8S集群的==监控==能力

##### ELK

> 提供K8S集群==日志==统一分析平台

![image-20210823214336521](E:\我的\学习文档\md-pic\image-20210823214336521.png)



#### K8S的三个核心概念

##### Pod

> Pod是k8s中最小的部署单元，可以有一组容器的集合，一个Pod中的容器是共享网络的，生命周期是短暂的

- 自主式Pod
- 控制器管理的Pod

##### Controller

> 确保预期的pod的副本的数量
>
> 无状态的应用部署：
>
> 有状态的应用部署：
>
> 确保所有的node运行同一个Pod
>
> 一次性任务和定时任务

##### Service

> 定义一组Pod的访问规则

StatefulSet是为了解决有状态服务的问题。

DaemonSet确保全部或者一些Node上运行一个Pod副本。

Job负责批处理任务，即仅执行一次的任务，它保证批处理任务的一个或者多个Pod成功结束。

Cron Job管理基于时间的Job

#### 平台规划

- 单Master集群

  ```mermaid
  graph TD
  Master --- Node1 & Node2 & Node3
  ```

  

- 多Master集群

  ```mermaid
  graph TD 
  Master1 & Master2 & Master3 --- 负载均衡 --- Node1 & Node2 & Node3
  ```

  

#### 服务器配置要求

> 测试环境：
>
> Master节点： 2核 4G 20G 
>
> Node 4核 8G 40G
>
> 生产环境：更高要求
>
> Master：8核 16G 100G
>
> Node: 16核 64G 500G

#### 部署K8S集群的两种常见方式

- kubeadm

  > Kubeadm是一个K8S部署工具， 提供kubeadm init 和 kubeadm join，用于快速部署，技术门槛低，屏蔽了很多细节

- 二进制包

  > 从github下载发行版的二进制包，手动部署每个组件，组成集群，利于排查问题，理解工作原理

#### 虚拟机中实战搭建K8S集群

> 虚拟机配置要求：Centos7 2GB 2CPU 30G
>
> Master节点：1
>
> Node节点：2