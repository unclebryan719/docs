#### Docker网络

##### 网络模式

```shell
bridge: 桥接 docker默认
none： 不配置网络
host: 和宿主机共享网络
container: 容器网络连通 （局限性大）
```



##### 测试

```shell
docker run -d -P --name tomcat01 (缺省--net bridge) tomcat

# bridge相当于docker0， 默认，域名不能访问，可以通过--link打通！使用比较麻烦


# 自定义网络
docker network create --driver bridge --subnet 192.168.0.0/16 --gateway  192.168.0.1 mynet

# 查看网络
docker network ls

# 使用自己创建的网络启动容器
docker run -d -P --name tomcat01 --net mynet tomcat

# docker自定义网络可以通过域名ping通，不需要使用--link
```

##### docker网络连通

```shell
# 不同集群之间用不同的网络，不同的网段 例如redis集群和mysql集群
# 网卡与网卡是无法打通，但是网卡和容器是可以打通
docker network connect 网络名 容器名
```



