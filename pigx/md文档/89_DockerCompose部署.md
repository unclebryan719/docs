# 安装Docker （centos7）
# 更新yum 源
yum update

#安装 Docker
yum -y install docker

#启动 Docker 后台服务
service docker start

#测试运行 hello-world,由于本地没有hello-world这个镜像，所以会下载一个hello-world的镜像，并在容器内运行。
docker run hello-world
# 安装docker-compose
$ sudo curl -L https://github.com/docker/compose/releases/download/1.20.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
# pig 打包
pig 根目录
mvn clean install -Dmaven.test.skip=true

![avator](http://pic.pig4cloud.com/20190221163545_ykCjD4_Screenshot.jpeg)

压缩pig 整个工程上传到docker 宿主机
执行 docker-compose 命令
# 构建镜像
docker-compose build

# 启动容器 （-d 后台启动，建议第一次不要加，方便看错误）
docker-compose up -d
# 等待3分钟
访问Centos7 IP:8761 查看eureka状态，确定所有服务全部启动。

# 总结
服务端已启动完毕，前端请参考下一章节《前端部署》
不要和开发环境一样，修改容器hosts,docker-compose 会根据容器名称自动处理