# 下载 SkyWalking
SkyWalking 个人建议直接下载官方编译好的，下载地址

# 启动 SkyWalking
sudo bin/startup.sh
# 关闭 SkyWalking
ps -ef | grep sky | awk '{print $2}' | sudo  xargs kill -9
# Java Agent
所有服务都加agent

-javaagent:/agent/skywalking-agent.jar=agent.service_name=pigx-upms
![avator](http://pic.pig4cloud.com/20190722180412_ZsYB7V_Screenshot.jpeg)

# 问题 Redis/ Spring Cloud Gateway 监控不到
![avator](http://pic.pig4cloud.com/20190722180827_YUGKqq_Screenshot.jpeg)