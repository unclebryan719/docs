```bash
2022-04-13 15:59:47 ERROR com.alibaba.dubbo.qos.server.Server:102 [main] []  [DUBBO] qos-server can not bind localhost:22222, dubbo version: 2.6.7, current host: 10.60.194.29
java.net.BindException: Address already in use
	at java.base/sun.nio.ch.Net.bind0(Native Method)
	at java.base/sun.nio.ch.Net.bind(Net.java:455)
	at java.base/sun.nio.ch.Net.bind(Net.java:447)
	at java.base/sun.nio.ch.ServerSocketChannelImpl.bind(ServerSocketChannelImpl.java:227)
	at io.netty.channel.socket.nio.NioServerSocketChannel.doBind(NioServerSocketChannel.java:130)
	at io.netty.channel.AbstractChannel$AbstractUnsafe.bind(AbstractChannel.java:562)
	at io.netty.channel.DefaultChannelPipeline$HeadContext.bind(DefaultChannelPipeline.java:1358)
	at io.netty.channel.AbstractChannelHandlerContext.invokeBind(AbstractChannelHandlerContext.java:501)
	at io.netty.channel.AbstractChannelHandlerContext.bind(AbstractChannelHandlerContext.java:486)
	at io.netty.channel.DefaultChannelPipeline.bind(DefaultChannelPipeline.java:1019)
	at io.netty.channel.AbstractChannel.bind(AbstractChannel.java:258)
	at io.netty.bootstrap.AbstractBootstrap$2.run(AbstractBootstrap.java:366)
	at io.netty.util.concurrent.AbstractEventExecutor.safeExecute$$$capture(AbstractEventExecutor.java:163)
	at io.netty.util.concurrent.AbstractEventExecutor.safeExecute(AbstractEventExecutor.java)
	at io.netty.util.concurrent.SingleThreadEventExecutor.runAllTasks(SingleThreadEventExecutor.java:404)
	at io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:474)
	at io.netty.util.concurrent.SingleThreadEventExecutor$5.run(SingleThreadEventExecutor.java:909)
	at io.netty.util.concurrent.FastThreadLocalRunnable.run(FastThreadLocalRunnable.java:30)
	at java.base/java.lang.Thread.run(Thread.java:829)
```



##### 简介：

Quality of Service，qos是Dubbo的在线运维命令，可以对服务进行动态的配置、控制及查询

##### 原因：

consumer启动时qos-server也是使用的默认的22222端口，但是这时候端口已经被provider给占用了，所以才会报错的。

##### 解决：

配置consumer的qos

```properties
dubbo.application.qos.enable=true
dubbo.application.qos.port=33333
dubbo.application.qos.accept.foreign.ip=false
```

