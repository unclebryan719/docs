---
typora-copy-images-to: upload
---

### Zookeeper学习笔记

![image-20220404211140955](https://tva1.sinaimg.cn/large/e6c9d24ely1h0xzy2xkchj21f70u0ag8.jpg)

```properties
# The number of milliseconds of each tick
# 每次的心跳时间，指的是客户端与服务端或者服务端与服务端，如果超过2秒钟，说明连接断开
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
# 初始化时的心跳个数，指的是第一次Leader与Follower建立连接时的通信心跳个数，即20秒，如果超过20秒则表示建立连接失败
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
# 指的是非第一次通信时，Leader与Follower建立连接时的通信心跳个数
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
# zookeeper数据目录，不能存在tmp目录，Linux会定期回收tmp里的文件
dataDir=/Users/unclebryan/DevTools/apache-zookeeper-3.7.0-bin/data
# 日志文件
dataLogDir=/Users/unclebryan/DevTools/apache-zookeeper-3.7.0-bin/logs
# the port at which the clients will connect
# 客户端与服务端通信端口
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1

## Metrics Providers
#
# https://prometheus.io Metrics Exporter
#metricsProvider.className=org.apache.zookeeper.metrics.prometheus.PrometheusMetricsProvider
#metricsProvider.httpPort=7000
#metricsProvider.exportJvmInfo=true

```



#### 集群配置

> 每一个节点都要修改
>
> 1. 在zk的数据目录新建一个myid文件，输入myid的值
> 2. 在zoo.cfg中加入集群配置信息，如下

```bash
# 集群模式 1代表myid,2888是Follower与Leader的交换端口，3888是选举端口
server.1=unclebryan01:2888:3888
server.2=unclebryan02:2888:3888
server.3=unclebryan03:2888:3888

# ps. 记得关闭防火墙
1. firewall-cmd --reload                  #重启firewall
2. firewall-cmd  --state                  #查看防火墙状态
3. systemctl start firewalld.service      #开启firewall
4. systemctl stop firewalld.service       #停止firewall
5. systemctl disable firewalld.service    #禁止firewall开机启动

# 分别启动zk服务
zkServer.sh start
zkServer.sh status 查看状态
```



#### 客户端操作

```bash
# 启动客户端
zkCli.sh -server unclebryan01:2181
```

##### 常用命令

```bash
ls -s /

[zookeeper]
# 创建节点时的事务id
cZxid = 0x0
ctime = Thu Jan 01 08:00:00 CST 1970
# 最后更新的事务id
mZxid = 0x0
mtime = Thu Jan 01 08:00:00 CST 1970
# 最后更新的子节点的事务id
pZxid = 0x0
# 子节点的版本号，即子节点的修改次数
cversion = -1
# 数据版本号
dataVersion = 0
# 访问控制列表的版本号
aclVersion = 0
# 临时节点的拥有者的sessionid，如果不是临时节点则为0
ephemeralOwner = 0x0
# 数据的长度
dataLength = 0
# 子节点个数
numChildren = 1

# 永久无序
create /node1 node1
# 永久有序，序号由父节点决定 真正的节点名为节点名+00000001....
create -s /node2 node2 

# 临时无序
create -e /node3 node3
# 临时有序，序号由父节点决定 真正的节点名为节点名+00000001....
create -e -s /node4 node4


# 获取节点信息
get -s /node1


# 单节点删除
delete /node1/n1

# 删除整个节点，包括子节点
deleteall /node1
```



#### 节点类型

- 持久的

  > 断开连接不删除

- 临时的

  > 客户端与服务端断开连接就删除临时节点

- 有序的

- 无序的



#### 监听器

> 注意：监听是不能重复监听，需要重复监听就需要重复watch

- 监听节点值的变化

  ```bash
  # 通过某个客户端设置监听的节点
  get -w /node1
  
  #在其他客户端修改节点值
  set /node1 test
  
  
  # 监听客户端收到如下通知
  WATCHER::
  
  WatchedEvent state:SyncConnected type:NodeDataChanged path:/node1
  
  ```

- 监听子节点节点数的变化

  ```bash
  # 通过某个客户端设置监听的节点
  ls -w /node1
  
  # 当有新增或者删除节点时，客户端会收到如下通知
  WATCHER::
  
  WatchedEvent state:SyncConnected type:NodeChildrenChanged path:/node1
  
  ```

  



#### 选举机制

- 首次启动集群

  > 服务器1启动
  >
  > - 投自己一票
  > - 此时集群中只有一个节点，不进行选票传递
  > - 判断自己的选票有没有大于集群中的半数节点，即是否大于3，不大于进入LOOKING状态
  >
  > 服务器2启动
  >
  > - 投自己一票
  > - 进行选票传递，服务器1的myid小于服务器2的myid，服务器1将选票传递给服务器2，此时服务器1的选票为0，服务器2的选票为2
  > - 判断自己的选票有没有大于集群中的半数节点，即是否大于3，不大于进入LOOKING状态
  >
  > 服务器3启动
  >
  > - 投自己一票
  > - 进行选票传递，服务器2的myid小于服务器3的myid，服务器2将选票传递给服务器3，此时服务器1、2的选票均为0，服务器3的选票为3
  > - 判断自己的选票有没有大于集群中的半数节点，即是否大于3，大于3，选举成功，此时服务器3是Leader节点，状态由LOOKING变为LEADING；服务器1/2自动变为Follower节点，状态由LOOKING变为FOLLOWING
  >
  > 服务器4启动
  >
  > - 投自己一票
  > - 此时集群中1/2/3服务器已经不再是LOOKING状态，服务器4自动变为Follower
  >
  > 服务器5启动
  >
  > - 投自己一票
  > - 此时集群中1/2/3/4服务器已经不再是LOOKING状态，服务器5自动变为Follower

- 非第一次启动

  > 当服务器5无法与Leader保持连接时，会发起一次选举
  >
  > - Leader正常，只是服务器5无法与Leader保持连接，则继续尝试连接Leader
  >
  > - Leader确实挂了，则进入选举
  >
  >   假如当前集群中SID分别为1、2、3、4、5，ZXID分别为8、8、8、7、7，并且SID为3的服务器为Leader；此时服务器3与5突然挂了，需要进行Leader重新选举，流程如下：
  >
  >   - 1、2、4进行Leader选举，依次比较Epoch、ZXID、SID，大的成为Leader
  >   - 此时1、2、4的数据分别为1 8 1、1 8 2、1 7 4，所以服务器2为新的Leader



#### 集群启动脚本

```bash
#!/bin/bash

case $1 in
"start"){
	for i in unclebryan01 unclebryan02 unclebryan03
	do
		echo -----------zk $i 启动----------------
		ssh $i "/usr/local/software/zookeeper-3.5.7/bin/zkServer.sh start"
	done
}
;;

"stop"){
	for i in unclebryan01 unclebryan02 unclebryan03
	do
		echo -----------zk $i 停止----------------
		ssh $i "/usr/local/software/zookeeper-3.5.7/bin/zkServer.sh stop"
	done
}
;;


"status"){
	for i in unclebryan01 unclebryan02 unclebryan03
	do
		echo -----------zk $i 状态----------------
		ssh $i "/usr/local/software/zookeeper-3.5.7/bin/zkServer.sh status"
	done
}
;;
esac
```

PS. 如果出现JAVA_HOME is not set and java could not be found in PATH，请修改zkEnv.sh

```bash
# 在文件的最前面增加javahome路径
export JAVA_HOME="/usr/local/software/jdk1.8.0_311"
```



#### 写数据流程

- 客户端连接的是Leader节点

![image-20220405213054290](https://tva1.sinaimg.cn/large/e6c9d24ely1h0z6450qfkj21dv0u0gno.jpg)

- 客户端连接的是Follower节点

  ![image-20220405213243774](https://tva1.sinaimg.cn/large/e6c9d24ely1h0z65zhsnej220r0u00vn.jpg)

#### 场景应用

- 服务的动态上下线

  ![image-20220405213715307](https://tva1.sinaimg.cn/large/e6c9d24ely1h0z6aqsdj8j21nw0u0gqe.jpg)



- 分布式锁

  ![image-20220405214256648](https://tva1.sinaimg.cn/large/e6c9d24ely1h0z6gm3tw6j21sr0u0dka.jpg)

> zk实现的分布式锁请使用curator框架
>
> 官网地址：https://curator.apache.org/



#### 集群zk的节点数

> 1. 安装奇数台
> 2. 生产经验值：
>    1. 10台服务器，安装3台zk
>    2. 20台服务器，安装5台zk
>    3. 100台服务器，安装11台zk
>    4. 200台服务器，安装11台zk
> 3. 服务器台数多，好处是提高稳定性，坏处是通信延迟



#### zookeeper一致性问题

##### Paxos算法

角色：

Proposer（提案者），Acceptor（接收者）、Learner（学习者）；每个节点可以身兼数职

三个阶段：

1. 准备阶段
   - Proposer向Acceptor发出Propose请求Promise（许可），无需携带提案内容
   - Acceptor向Proposer发送同意此提案
2. Accept接收阶段
   - Proposer收到超过半数的Acceptor许可后，向Acceptor发出正式Propose
   - Acceptor接收到提案后进行Accept处理
3. Lean阶段
   - Proposer将最终的提案发送给所有的Learners

存在的问题：多个提案者可能出现迟迟无法达成一致的问题，导致性能降低

##### ZAB算法

借鉴Paxos算法，只有一个提案者

包含两种模式：消息广播、崩溃恢复



#### 源码

1. 找到启动入口

   ```bash
   zkServer.sh start
   ZOOMAIN="org.apache.zookeeper.server.quorum.QuorumPeerMain"
   ```

   