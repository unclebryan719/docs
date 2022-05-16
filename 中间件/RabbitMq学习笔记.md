---
typora-copy-images-to: upload
---

### RabbitMq学习笔记

#### Erlang安装

https://www.erlang-solutions.com/downloads/



```bash
#运行Package Cloud提供的RabbitMQ Server快速安装脚本
curl -s https://packagecloud.io/install/repositories/rabbitmq/rabbitmq-server/script.rpm.sh | sudo bash

#运行Package Cloud提供Erlang环境快速安装脚本
curl -s https://packagecloud.io/install/repositories/rabbitmq/erlang/script.rpm.sh | sudo bash

#使用yum安装Erlang环境
yum  -y install erlang

#安装socat, logrotate依赖
  yum install socat logrotate -y

#使用yum安装RabbitMQ Server
yum install -y rabbitmq-server

```





为什么Rabbitmq是基于channel发送消息而不是基 于连接？

因为连接需要三次握手和四次挥手，会很慢





#### 消息模式

> 所有的模式队列与交换机一定要建立绑定关系，如果没有指明交换机，会走一个默认的交换机，默认交换机是direct模式

##### 1. 简单模式

> 默认交换机

![img](https://tva1.sinaimg.cn/large/e6c9d24ely1h1mead9wbsj208002sq2t.jpg)

##### 2. 工作队列模式

> 默认交换机
>
> - 轮训模式，轮训分发，可以自动ack，但是建议使用手动ack
> - 公平分发，能者多劳，必须手动ack

![img](https://tva1.sinaimg.cn/large/e6c9d24ely1h1meb8nul3j2098033aa0.jpg)

##### 3. 发布订阅模式 fanout

> 需要指定交换机

![img](https://tva1.sinaimg.cn/large/e6c9d24ely1h1mebn8l9rj2098033mx4.jpg)

##### 4. Routing模式（Direct模式）

> 需要指定交换机和路由key

![img](https://tva1.sinaimg.cn/large/e6c9d24ely1h1mecaklb6j20bc04r74c.jpg)

##### 5. Topic模式

> 需要指定交换机和路由key

![img](https://tva1.sinaimg.cn/large/e6c9d24ely1h1meeihy1lj20bs04rq30.jpg)

> '#' : 匹配一个或多个词
>
> '*' : 匹配不多不少恰好1个词
>
> item.# ：能够匹配 item.insert.abc 或者 item.insert
>
> item.* ：只能匹配 item.insert



##### 6. RPC模式

![img](https://tva1.sinaimg.cn/large/e6c9d24ely1h1mefmxo8ij20g005kdg2.jpg)

#### 交换机类型

direct

fanout

topic

headers
