# 部署zookpeer
docker run --name testzookeeper --restart always -p 2181:2181 -d zookeeper:3.5.4
# 依赖
<dependency>
	<groupId>com.pig4cloud</groupId>
	<artifactId>pigx-common-job</artifactId>
</dependency>
# 增加配置文件
spring:
  elasticjob:
    # 分布式任务协调依赖zookeeper
    zookeeper:
      server-lists: ${ZOOKEEPER-HOST:pigx-zookeeper}:${ZOOKEEPER-PORT:2181}
      namespace: pigx-daemon
    # 普通任务
    simples:
      spring-simple-job:
        job-class: com.pig4cloud.pigx.daemon.elastic.job.PigxSimpleJob
        cron: 0 0 0/1 * * ?
        sharding-total-count: 3
        sharding-item-parameters: 0=service1,1=service2,2=service3
        eventTraceRdbDataSource: 'dataSource'
        listener:
          listener-class: com.pig4cloud.pigx.daemon.elastic.listener.PigxElasticJobListener

# 开启配置
@EnablePigxElasticJob
main 方法启动应用
# 监控控制台
docker run -d -p 8899:8899 -e ROOT_PASSWD=12345678 -e GUEST_PASSWD=12345678 pig4cloud/elastic-job-console:2.1.5
basic 认证 root/12345678
![avator](http://pigx.vip/20191006164442_ZgvovA_Screenshot.jpeg)