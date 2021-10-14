# 依赖
<dependency>
	<groupId>com.pig4cloud</groupId>
	<artifactId>pigx-common-job</artifactId>
</dependency>
# 增加配置文件
xxl:
  job:
    admin:
      addresses: http://pigx-xxl:9080/xxl-job-admin  # xxl-job-admin 接口地址
    executor:
      port: 9988   #通讯端口
      appName: test-xxl
# 开启配置
@EnablePigxXxlJob
main 方法启动服务应用
# docker 部署 xxl-job-admin
本文以docker的形式部署 xxl-job-admin，注意镜像是 pig4cloud 定制版 也可以使用官方版本
非docker 运行参考视频 pigx 定时任务xxl-job 使用
# docker 运行
注意链接数据源配置
docker run -e PARAMS="--spring.datasource.url=jdbc:mysql://192.168.0.31:3306/pigxx_job?Unicode=true&characterEncoding=UTF-8&useSSL=false --spring.datasource.username=root --spring.datasource.password=root --xxl.admin.login=false"\
 -p 9080:9080 --name xxl-job-admin\
 -d pig4cloud/xxl-job-admin:2.1.0
# 访问控制台
浏览器访问
http://pigx-xxl:9080/xxl-job-admin
新增执行器
![avator](http://pigx.vip/20191006144506_Ia7THM_Screenshot.jpeg)

# 开发第一个任务“Hello World”
本示例以新建一个 “GLUE模式(Java)” 运行模式的任务为例。更多有关任务的详细配置，请查看详细使用文档参考 xxl-job 官网

前提：请确认“调度中心”和“执行器”项目已经成功部署并启动；

步骤一：新建任务： 登录调度中心，点击下图所示“新建任务”按钮，新建示例任务。然后，参考下面截图中任务的参数配置，点击保存。 image

image
![avator](https://raw.githubusercontent.com/xuxueli/xxl-job/master/doc/images/img_o8HQ.png)

![avator](https://raw.githubusercontent.com/xuxueli/xxl-job/master/doc/images/img_ZAsz.png)
