# logback-spring.xml 核心配置

![avator](http://pigx.vip/20191006193857_1OKRoq_Screenshot.jpeg)
appender configuration 的子元素。负责写日志的组件,定义日志输出的原则例如输出到控制台、输出到文件 等

logger configuration 的子元素。用来设置某个包或及具体的某个类的日志输出以及指定 appender

root configuration 的子元素。root节点是必选节点，用来指定最基础的日志输出级别，只有一个level属性.

# pigx 配置说明
小技巧: 在根pom里面设置统一存放路径，统一管理方便维护

<properties>
    <log-path>/Users/lengleng</log-path>
</properties>
其他模块加日志输出，直接copy本文件放在resources 目录即可
注意修改 的value模块
![avator](http://pigx.vip/20191006195049_1sJ5hY_Screenshot.jpeg)