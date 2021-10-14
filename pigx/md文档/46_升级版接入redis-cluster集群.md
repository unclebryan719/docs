# docker 部署集群
docker run --name redis-cluster -d -e CLUSTER_ANNOUNCE_IP=192.168.0.31 -p 7000-7005:7000-7005 -p 17000-17005:17000-17005  pig4cloud/redis-cluster:4.0

# 客户端连接测试
⋊> ~/e/r/src ./redis-cli -c -h 192.168.0.31 -p 7000
06:56:33
192.168.0.31:7000> cluster nodes
4bafce56dcf021b3f89c2b7359498bbd760d3fd6 192.168.0.31:7002@17002 master - 0 1570834599565 3 connected 10923-16383   
ab7fb748d33c7fb7e106ca326f125a5b44c8689e 192.168.0.31:7001@17001 master - 0 1570834598000 2 connected 5461-10922
3957bae776b8cd07f534bfd6028c4b4852be7372 192.168.0.31:7005@17005 slave ab7fb748d33c7fb7e106ca326f125a5b44c8689e 0 1570834598663 6 connected
1b9e0862e5b75b8bda01208facbca5b7729e7a69 192.168.0.31:7003@17003 slave 4bafce56dcf021b3f89c2b7359498bbd760d3fd6 0 1570834599666 4 connected
702d38c5055746cbbef15cf30326d2a4228040ba 192.168.0.31:7000@17000 myself,master - 0 1570834599000 1 connected 0-5460
4814a52af428e614bc8482c7b817ec1820ed58a7 192.168.0.31:7004@17004 slave 702d38c5055746cbbef15cf30326d2a4228040ba 0 1570834598663 5 connected
# pigx 应用配置集群节点
nacos. application-dev.yml
spring:
  redis:
    cluster:
      nodes:
        - pigx-redis:7000
        - pigx-redis:7001
        - pigx-redis:7002
        - pigx-redis:7003
        - pigx-redis:7004
        - pigx-redis:7005
启动应用 完成

# 扩展部分
针对 3.4 及其以下版本 3.5+ 默认支持

在redis-cluster模式下停用任何一个master或slave节点导致应用不能登录问题

原因
spring boot 2.x 使用的 lettuce 作为默认的redis连接， 自适应拓扑刷新（Adaptive updates）与定时拓扑刷新（Periodic updates） 是默认情况下关闭。
通俗讲 redis-cluster sentinel的变化并不会通知到 lettuce 连接中，会出现以上问题。
Refreshing the cluster topology view

# 代码修改
RedisTemplateConfigDynamicRouteAutoConfiguration 增加如下bean
意思很简单自己创建一个LettuceConnectionFactory 开启轮询刷新 可以设置时间，值不要太小。

@Bean
@ConditionalOnProperty(value = "spring.redis.cluster.nodes",matchIfMissing = true)
public LettuceConnectionFactory redisConnectionFactory(RedisProperties redisProperties) {
    RedisClusterConfiguration redisClusterConfiguration = new RedisClusterConfiguration(redisProperties.getCluster().getNodes());

    // https://github.com/lettuce-io/lettuce-core/wiki/Redis-Cluster#user-content-refreshing-the-cluster-topology-view
    ClusterTopologyRefreshOptions clusterTopologyRefreshOptions = ClusterTopologyRefreshOptions.builder()
            .enablePeriodicRefresh()
            .enableAllAdaptiveRefreshTriggers()
            .refreshPeriod(Duration.ofSeconds(5))
            .build();

    ClusterClientOptions clusterClientOptions = ClusterClientOptions.builder()
            .topologyRefreshOptions(clusterTopologyRefreshOptions).build();

    // https://github.com/lettuce-io/lettuce-core/wiki/ReadFrom-Settings
    LettuceClientConfiguration lettuceClientConfiguration = LettuceClientConfiguration.builder()
            .readFrom(ReadFrom.SLAVE_PREFERRED)
            .clientOptions(clusterClientOptions).build();

    return new LettuceConnectionFactory(redisClusterConfiguration, lettuceClientConfiguration);
}
RedisTemplateConfig 微调

![avator](http://pigx.vip/20191012103619_kiGFVv_Screenshot.jpeg)

当然也有更简单粗暴的解决方案 使用 jedis 代替 lettuce
所有的 data-redis 排除 Add configuration to enable Redis Cluster topology refresh #15630

![avator](http://pigx.vip/20191012103951_bH1NyM_Screenshot.jpeg)