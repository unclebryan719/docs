#### Docker基本操作

```shell
docker version 
docker info  # 显示docker的系统信息，包括容器和镜像的数量
docker 命令 --help

```



#### 镜像命令

```shell
docker images #查看本机所有的镜像
-a 
-q  #只显示镜像id

docker search
```



##### commit镜像

```shell
docker commit -m="描述信息" -a="作者" 容器id 目标镜像名称:[TAG]
```



#### 什么是容器数据卷

相当于数据挂载，将Docker中的数据同步到本地

#### 使用数据卷

> 方式一：直接使用命令来挂载

```shell
docker run -it -v 主机目录:容器内目录

### 查看容器信息
docker inspect 容器id
```



```shell
### 查看正在运行的容器
docker ps 
```



#### 实战：MySQL数据持久化

```shell
# 获取镜像
docker pull mysql:5.7

# 启动命令参数
-d 后台运行
-p 端口映射
-v 卷挂载
-e 环境配置
--name 容器名字
# 运行容器并挂载
docker run -d -p 3310:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:5.7
```

#### 容器命令

```shell
docker run [参数] image

#参数说明
--name = "容器名字"
-d   后台启动
-p 指定容器端口 主机端口:容器端口
-it 使用交互方式运行，进入容器查看内容

docker ps 查看当前正在运行的容器
-a 带出历史运行过的容器
-n=? 指定显示的数量
-q 显示容器的编号


#退出容器
exit 直接退出并停止容器
Ctrl + P +Q 退出不停止容器

#删除容器
docker rm 容器id
docker rm -f $(docker ps -aq) 删除所有容器
docker ps -a -q | xargs docker rm  删除所有容器

#启动和停止
docker start 容器id
docker restart 容器id
docker stop 容器id
docker kill 容器id

#后台启动容器
docker run -d centos # 通过-d的方式启动，发现centos停止了；docker容器使用后台运行，就必须要有一个前台进程，就会自动停止

#显示日志
docker logs -f -t --tail 10 容器id

# 查看容器中进程信息
docker top 容器id

docker inspect 容器id

# 进入正在运行的容器
docker exec -it 容器id /bin/bash  #进入容器并开启一个新的终端
docker attach 进入容器并  # 进入容器当前正在运行的终端


#容器内文件拷贝到主机
docker cp 容器id:/home/test.java /home/

```



#### DockerFile的指令

```shell
FROM		# 基础镜像，一切从这里开始，99%的镜像都是从scratch来的
MAINTAINER	# 镜像是谁写的，姓名+邮箱
RUN			# 镜像构建的时候需要运行的命令 
ADD 		# 步骤： Tomcat镜像，这个Tomcat压缩包！添加内容
WORKDIR		# 镜像的工作目录
VOLUME 		# 挂载的目录
EXPOSE		# 保留端口配置
CMD			# 指定容器启动的时候要运行的命令，只有最后一个会生效，可以被替代
ENTRYPOINT	# 指定这个容器启动的时候要运行的命令，可以追加命令
ONBUILD		# 当构建一个被继承DockerFile这个时候就会运行ONBIULD的指令。触发指令
COPY		# 类似ADD， 将我们文件拷贝到镜像中
ENV			# 构建的时候设置环境变量！
```



#### Dockerfile构建自己的镜像

```shell
# 1.编写Dockerfile文件
FROM centos
MAINTAINER unclebryan<unclebryan719@gmail.com>

ENV MYPATH /usr/local
WORKDIR $MYPATH

RUN yum -y install vim
RUN yum -y install net-tools

EXPOSE 80

CMD echo $MYPATH
CMD echo "----end----"
CMD /bin/bash

# 2. 通过文件构建镜像
docker build -f dockerfile文件路径 -t 镜像名:[tag]

# 3. 运行
docker run -it --name "mycentos" 镜像名称:[TAG]
```



Tomcat
