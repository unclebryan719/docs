### Dubbo使用技巧

#### 1. 只订阅不注册

```properties
# 在开发过程中，避免互相影响
dubbo.registry.register=false
dubbo.registry.subscribe=true
```

#### 2. 直连提供者的方式

- JVM启动参数中指定提供者的URL

  ```bash
  java -D com.alibaba.xxx.XxxService=dubbo://localhost:20890
  ```

- 在reference调用位置指定提供者的URL

  ```xml
  <debbo.reference url="dubbo://localhost:20890" interfaceClass="com.alibaba.xxx.XxxService"/>
  ```

  或

  ```java
  @DubboReference(url="dubbo://localhost:20890")
  private DubboDemoService demoService;
  ```

#### 3.指定group分组

> 目的是隔离不同的环境，比如测试、开发、或者自己本地的环境
>
> 使用：分别在`<dubbo:consumer/>` 和`<dubbo:provider/>` 添加`group`属性并自定义组名

#### 4. version

> 当一个接口的实现，出现不兼容升级时，可以用版本号过渡，版本号不同的服务相互间不引用

```xml
<!-- 机器A提供1.0.0版本服务 -->
<dubbo:service interface=“com.foo.BarService” version=“1.0.0” />
 
<!-- 机器B提供2.0.0版本服务 -->
<dubbo:service interface=“com.foo.BarService” version=“2.0.0” />
  
<!-- 机器C消费1.0.0版本服务 -->
<dubbo:reference id=“barService” interface=“com.foo.BarService” version=“1.0.0” />
  
<!-- 机器D消费2.0.0版本服务 -->
<dubbo:reference id=“barService” interface=“com.foo.BarService” version=“2.0.0” />

<!-- 消费者也可以消费任意版本的服务 -->
 <dubbo:reference id=“barService” interface=“com.foo.BarService” version="*" />
```