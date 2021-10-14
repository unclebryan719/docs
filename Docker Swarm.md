### Docker Swarm

```shell
#初始化节点
docker swarm init
#加入一个节点
docker swarm join
#在主节点执行如下命令获取令牌，得到一个命令，在需要加入集群的集群上执行此命令
docker swarm join-token manager
docker swarm join-token worker
```

#### Raft协议



#### 灰度发布、动态扩缩容

```shell
docker run #容器启动，不具有扩缩容功能
docker service #启动服务，具有扩缩容功能
docker service ls #查询服务
#扩缩容命令
docker service update --replicas 10 服务名
#扩缩容命令
docker service scale 服务名=num
#移除命令
docker service rm 服务名
```

#### 概念

**swarm**

> 集群的管理和编号。docker可以初始化一个swarm集群，其他节点可以加入。（管理者和工作者）

**Node**

> 就是docker的一个节点

**Service**

> 任务，可以在管理节点或者工作节点来运行。

**Task**

> 容器内的命令，细节任务。

**逻辑**

> 命令->管理->api->调度->工作节点（创建Task容器维护创建！）

#### Swarm网络

> 网络模式
>
> - Overlay:
>
> - ingress：特殊的Overlay网络！具有负载均衡的功能！IPVS VIP技术
>
> 虽然docker在不同的服务器上运行，实际网络却是同一个！使用的是ingress网络模式，ingress网络是一个特殊的Overlay网络！



#### Docker Stack

```shell
# 单机下用docker-compose
docker-compose up -d xxx.yaml
# 集群下用docker stack
docker stack deploy xxx.yaml
```

#### K8S

==云原生时代==

==Go语言==

