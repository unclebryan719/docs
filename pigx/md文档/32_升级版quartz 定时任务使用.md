目前Pigx支持任务类型有4类
spring bean 类型
rest 类型
java 类型 （反射）
jar 类型 （java -jar）
核心逻辑代码如下:
![avator](http://pigx.vip/20191011095503_pALQVd_Screenshot.jpeg)


# Spring Bean类型任务
新增任务，参数说明
参数	说明
类型	spring bean
执行路径	留空
执行文件	demo ,对应代码的spring bean name
执行方法	执行bean 的指定方法名称
执行参数	对应执行方法的入参
![avator](http://pigx.vip/20191011091956_spixL4_Screenshot.jpeg)
![avator](http://pigx.vip/20191010174803_77ALbi_Screenshot.jpeg)




# Rest 调用
新增任务，如图

3.5- 执行路径只支持应用内调用 http://service_name/xxx
3.5+ 执行路径支持 应用外调用 http://ip:name/xxx
应用内调用注意 目标接口 直接对外暴露，不然401 
![avator](http://pigx.vip/20191011095654_qyybBV_Screenshot.jpeg)
# jar 类型
jar 类型就是定时调用 jar -jar 执行路径 执行参数

参数	说明
类型	jar
执行路径	服务器jar包所在路径
执行文件	空
执行方法	空
执行参数	java -jar 执行时额外的参数


# java 类型
反射机制调用应用

参数	说明
类型	java
执行路径	空
执行文件	类的全类名
执行方法	目标类方法
执行参数	一个string参数
![avator](http://pigx.vip/20191011103908_swuAOa_Screenshot.jpeg)