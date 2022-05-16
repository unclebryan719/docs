## DUBBO

> **原因：**通过VPN连接公司内网，会产生一个内网ip，比如10.xx.xx.xx，而本机实际ip有可能是168.xx.xx.xx
>
> 1. 启动服务提供者，注册信息显示服务注册在10.xx.xx.xx
> 2. 启动服务消费者，注册信息显示服务注册在168.xx.xx.xx
>
> **现象：**消费者会去10.xx.xx.xx的机器上找对应的服务，导致访问失败
>
> **方案：**
>
> **Comsumer设置注册到注册中心的ip——加入环境变量**
>
> https://dubbo.apache.org/zh/docs/v2.7/user/examples/set-host/
>
> - DUBBO_IP_TO_REGISTRY — 注册到注册中心的ip地址
>
> - ==DUBBO_IP_TO_BIND== — 监听ip地址<font color=red>起作用的就是这个配置</font>
>
> -DDUBBO_IP_TO_BIND=192.168.0.76 -DDUBBO_IP_TO_REGISTRY=192.168.0.76
>
> **Provider设置注册到注册中心的ip——增加host或者加入环境变量**
>
> ```xml
> <dubbo:protocol host="192.168.0.76" port="${dubbo.protocol.port:20880}" name="dubbo" dispatcher="${dubbo.protocol.dispatcher:all}"
>                 threads="${dubbo.protocol.threads:500}" threadpool="${dubbo.protocol.threadPool:limited}"/>
> ```
>
> -DDUBBO_IP_TO_BIND=192.168.0.76 -DDUBBO_IP_TO_REGISTRY=192.168.0.76





```bash
QOS在Dubbo2.7.3以前无法禁用
https://github.com/apache/dubbo/issues/4377
```

