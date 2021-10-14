[toc]

### OpenFeign

#### 基本使用

添加依赖

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

在启动类开启上添加@EnableFeignClients注解开启OpenFeign

```java
@SpringBootApplication
@EnableFeignClients
public class xxxApplication {
    public static void main(String[] args) {
        SpringApplication.run(xxxApplication.class,args);
    }
}
```

通过@FeignClient注解声明FeignClient并绑定服务

```java
@Component
@FeignClient(value = "serverName")
public interface XxxService {
    @GetMapping("/helloworld")
    void helloworld();

}
```

服务调用

```java
@Resource
XxxService xxxService;
xxxService.helloworld();
```



#### 设置客户端超时时间

> OpenFeign整合了Ribbon，客户端的超时时间也由Ribbon控制

```yaml
ribbon:
	#请求处理的超时时间
	ReadTimeout: 5000
	#ribbon请求连接的超时时间
	ConnectTimeout: 5000
```

#### 开启调用日志

##### 日志级别

- NONE：默认的，不显示任何日志；
- BASIC：仅记录请求方法、URL、相应状态码及执行时间
- HEADERS：除了BASIC中定义的信息之外，还有请求和响应头信息；
- FULL：除了HEADERS中定义的信息外，还有请求和响应的正文及元数据。

##### 开启日志

```java
# 第一步，创建Feign配置类
@Configuration
public class FeignLogConfiguration {
  @Bean
  Logger.Level feignLoggerLevel() {
    return Logger.Level.FULL;
  }
}
```

```yaml
# 第二步，开启feign日志
logging:
	level:
		# 日志以什么级别监控哪个接口
		com.xxx.xxxFeign: debug
```

如果你不是在`application.yml`中配置的日志级别，而是使用`logback-spring.xml`，同理，在`logback-spring.xml`中做相应配置：

```xml
# logback-spring.xml
<logger name="com.xxx.xxx" level="DEBUG" additivity="false">
    <appender-ref ref="Console"/>
</logger>
```

