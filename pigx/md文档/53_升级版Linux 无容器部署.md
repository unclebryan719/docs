注意：开发机使用ubuntu

# 目标
搭建一个可以独立运行的PigX运行环境，以方便进行业务模块开发。

访问地址:

注册中心： http://pigx-dev-server:8848/nacos
后台管理：http://pigx-dev-server/
MinIO：http://minio-server:9199/ (用户名口令都是 lengleng)
虚拟机内存8G起步，16G可以愉快玩耍

# 准备工作
# 软件安装
mysql
redis
zookeeper
MinIO
sudo apt-get install nginx mysql-server redis-server zookeeper 
Redis注意事项

默认安装后，注意以下几处配置项(``)

# 注释掉 允许外网连接
#bind 127.0.0.1
MySQL注意事项:

如果安装过程中没有提示设置root账号的密码,在安装完成后，可以去/etc/mysql/debian.cnf看默认的账号和密码。登陆后再用SET PASSWORD FOR 'root'@'localhost' = PASSWORD('newpass');修改root的密码。

MinIO 需要手动下载

wget https://dl.min.io/server/minio/release/linux-amd64/minio
chmod +x minio
# MySQL配置
修改配置文件/etc/mysql/mysql.conf.d/mysqld.cnf

[mysqld]
# 注释掉，允许外部TCP连接
# bind-address           = 127.0.0.1
# 表名大小写
lower_case_table_names=1
新建自定义配置文件/etc/mysql/mysql-opz.cnf

[client]
default-character-set = utf8mb4

[mysqld]
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci

[mysql]
default-character-set = utf8mb4
# 网络配置
修改虚拟机的host信息(/etc/hosts文件)

# pigx3
192.168.0.165 pigx-dev-server
192.168.0.165 minio-server
192.168.0.165 pigx-register
192.168.0.165 pigx-gateway
192.168.0.165 pigx-redis
192.168.0.165 pigx-zookeeper
192.168.0.165 pigx-mysql
注意:

192.168.0.165 是虚拟机的IP地址，你需要根据实际情况修改(/etc/network/interfaces)

建议外部开发机也添加上面的host信息，省去一些不必要的麻烦

# 服务启停
所有服务均已配置为系统服务，开机自动运行。如有需要通过systemctl命令来启停服务

sudo systemctl [服务选项] [服务名]
服务选项

start：启动
stop：停止
restart：重启
status：查看状态
服务名见后面服务脚本部分。

如果需要列出系统中的服务，使用：

sudo systemctl list-units --type=service
# 服务脚本(systemd)
# MinIO(对象存储)
/etc/systemd/system/minio.service

[Unit]
Description=MinIO
Documentation=https://docs.min.io
Wants=network-online.target
After=network-online.target
AssertFileIsExecutable=/home/jclazz/minio/bin/minio

[Service]
WorkingDirectory=/home/jclazz/minio/bin/

#User=jclazz
#Group=jclazz

EnvironmentFile=/home/jclazz/minio/conf/minio.env
ExecStartPre=/bin/bash -c "if [ -z \"${MINIO_VOLUMES}\" ]; then echo \"Variable MINIO_VOLUMES not set in /home/jclazz/minio/conf/minio.env\"; exit 1; fi"

ExecStart=/home/jclazz/minio/bin/minio server $MINIO_OPTS $MINIO_VOLUMES

# Let systemd restart this service always
#Restart=always
# Never restart this service
Restart=no
RestartSec=30s

# Specifies the maximum file descriptor number that can be opened by this process
LimitNOFILE=65536

# Disable timeout logic and wait until process is stopped
TimeoutStopSec=infinity
SendSIGKILL=no

[Install]
WantedBy=multi-user.target

启用服务

sudo systemctl enable minio.service
配置文件

/home/jclazz/minio/conf/minio.env

# Volume to be used for MinIO server.
MINIO_VOLUMES="/home/jclazz/minio/data"
# Use if you want to run MinIO on a custom port.
MINIO_OPTS="--address :9199"
# Access Key of the server.
MINIO_ACCESS_KEY=lengleng
# Secret key of the server.
MINIO_SECRET_KEY=lengleng
# zookeeper(中间件)
/etc/systemd/system/zookeeper.service

[Unit]
Description=Zookeeper Daemon
Documentation=http://zookeeper.apache.org
Requires=network.target
After=network.target
#ConditionPathExists=/etc/zookeeper/conf/zoo.cfg

[Service]    
Type=forking
Environment=JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
WorkingDirectory=/usr/share/zookeeper/bin
#User=zk
#Group=zk
ExecStart=/usr/share/zookeeper/bin/zkServer.sh start /etc/zookeeper/conf/zoo.cfg
ExecStop=/usr/share/zookeeper/bin/zkServer.sh stop /etc/zookeeper/conf/zoo.cfg
ExecReload=/usr/share/zookeeper/bin/zkServer.sh restart /etc/zookeeper/conf/zoo.cfg
TimeoutSec=120
# Let systemd restart this service always
#Restart=always
# Never restart this service
Restart=no
RestartSec=30s

[Install]
WantedBy=default.target
启用服务

sudo systemctl enable zookeeper.service
# pigx-register(平台服务)
/etc/systemd/system/pigx-register.service

[Unit]
Description=pigx-register
Requires=network.target
After=network.target mysql.service redis.service

[Service]    
Type=simple
WorkingDirectory=/home/jclazz/pigx/bin/pigx-register
ExecStart=/home/jclazz/pigx/bin/pigx-register/run.sh
SuccessExitStatus=143
TimeoutStopSec=120
TimeoutSec=120
# Let systemd restart this service always
#Restart=always
# Never restart this service
Restart=no
RestartSec=30s

[Install]
WantedBy=default.target
启用服务

sudo systemctl enable pigx-register.service
辅助脚本

/home/jclazz/pigx/bin/pigx-register/run.sh

#!/bin/sh

java -Xmx512m -jar pigx-register.jar
# pigx-gateway(平台服务)
/etc/systemd/system/pigx-gateway.service

[Unit]
Description=pigx-gateway
Requires=network.target
After=network.target pigx-register.service

[Service]    
Type=simple
WorkingDirectory=/home/jclazz/pigx/bin/pigx-gateway
ExecStart=/home/jclazz/pigx/bin/pigx-gateway/run.sh
SuccessExitStatus=143
TimeoutStopSec=120
TimeoutSec=120
# Let systemd restart this service always
#Restart=always
# Never restart this service
Restart=no
RestartSec=30s

[Install]
WantedBy=default.target
启用服务

sudo systemctl enable pigx-gateway.service
辅助脚本

/home/jclazz/pigx/bin/pigx-gateway/run.sh

#!/bin/sh

java -Xmx512m -jar pigx-gateway.jar
# pigx-auth(核心业务)
/etc/systemd/system/pigx-auth.service

[Unit]
Description=pigx-auth
Requires=network.target
After=pigx-register.service

[Service]    
Type=simple
WorkingDirectory=/home/jclazz/pigx/bin/pigx-auth
ExecStart=/home/jclazz/pigx/bin/pigx-auth/run.sh
SuccessExitStatus=143
TimeoutStopSec=120
TimeoutSec=120
# Let systemd restart this service always
#Restart=always
# Never restart this service
Restart=no
RestartSec=30s

[Install]
WantedBy=default.target
启用服务

sudo systemctl enable pigx-auth.service
辅助脚本

/home/jclazz/pigx/bin/pigx-auth/run.sh

#!/bin/sh

java -Xmx512m -jar pigx-auth.jar
访问地址:

http://pigx-register:8848/nacos

# pigx-admin(后台管理)
/etc/systemd/system/pigx-admin.service

[Unit]
Description=pigx-admin
Requires=network.target
After=pigx-register.service

[Service]    
Type=simple
WorkingDirectory=/home/jclazz/pigx/bin/pigx-upms-biz
ExecStart=/home/jclazz/pigx/bin/pigx-upms-biz/run.sh
SuccessExitStatus=143
TimeoutStopSec=120
TimeoutSec=120
# Let systemd restart this service always
#Restart=always
# Never restart this service
Restart=no
RestartSec=30s

[Install]
WantedBy=default.target
启用服务

sudo systemctl enable pigx-admin.service
辅助脚本

/home/jclazz/pigx/bin/pigx-upms-biz/run.sh

#!/bin/sh

java -Xmx512m -jar pigx-upms-biz.jar
# pigx-tx-manager
/etc/systemd/system/pigx-tx-manager.service

[Unit]
Description=pigx-tx-manager
Requires=network.target
After=pigx-register.service

[Service]    
Type=simple
WorkingDirectory=/home/jclazz/pigx/bin/pigx-tx-manager
ExecStart=/home/jclazz/pigx/bin/pigx-tx-manager/run.sh
SuccessExitStatus=143
TimeoutStopSec=120
TimeoutSec=120
# Let systemd restart this service always
#Restart=always
# Never restart this service
Restart=no
RestartSec=30s

[Install]
WantedBy=default.target
启用服务

sudo systemctl enable pigx-tx-manager.service
辅助脚本

/home/jclazz/pigx/bin/pigx-tx-manager/run.sh

#!/bin/sh

java -Xmx512m -jar pigx-tx-manager.jar
# pigx-elastic-job
/etc/systemd/system/pigx-elastic-job.service

[Unit]
Description=pigx-elastic-job
Requires=network.target
After=pigx-register.service redis.service

[Service]    
Type=simple
WorkingDirectory=/home/jclazz/pigx/bin/pigx-elastic-job
ExecStart=/home/jclazz/pigx/bin/pigx-elastic-job/run.sh
SuccessExitStatus=143
TimeoutStopSec=120
TimeoutSec=120
# Let systemd restart this service always
#Restart=always
# Never restart this service
Restart=no
RestartSec=30s

[Install]
WantedBy=default.target
启用服务

sudo systemctl enable pigx-elastic-job.service
辅助脚本

/home/jclazz/pigx/bin/pigx-elastic-job/run.sh

#!/bin/sh

java -Xmx512m -jar pigx-daemon-elastic-job.jar
# pigx-quartz
/etc/systemd/system/pigx-quartz.service

[Unit]
Description=pigx-quartz
Requires=network.target
After=pigx-register.service

[Service]    
Type=simple
WorkingDirectory=/home/jclazz/pigx/bin/pigx-quartz
ExecStart=/home/jclazz/pigx/bin/pigx-quartz/run.sh
SuccessExitStatus=143
TimeoutStopSec=120
TimeoutSec=120
# Let systemd restart this service always
#Restart=always
# Never restart this service
Restart=no
RestartSec=30s

[Install]
WantedBy=default.target
启用服务

sudo systemctl enable pigx-quartz.service
辅助脚本

/home/jclazz/pigx/bin/pigx-quartz/run.sh

#!/bin/sh

java -Xmx512m -jar pigx-daemon-quartz.jar
# pigx-activiti
/etc/systemd/system/pigx-activiti.service

[Unit]
Description=pigx-activiti
Requires=network.target
After=pigx-register.service

[Service]    
Type=simple
WorkingDirectory=/home/jclazz/pigx/bin/pigx-activiti
ExecStart=/home/jclazz/pigx/bin/pigx-activiti/run.sh
SuccessExitStatus=143
TimeoutStopSec=120
TimeoutSec=120
# Let systemd restart this service always
#Restart=always
# Never restart this service
Restart=no
RestartSec=30s

[Install]
WantedBy=default.target
启用服务

sudo systemctl enable pigx-activiti.service
辅助脚本

/home/jclazz/pigx/bin/pigx-activiti/run.sh

#!/bin/sh

java -Xmx512m -jar pigx-activiti.jar
# pigx-monitor
/etc/systemd/system/pigx-monitor.service

[Unit]
Description=pigx-monitor
Requires=network.target
After=pigx-register.service

[Service]    
Type=simple
WorkingDirectory=/home/jclazz/pigx/bin/pigx-monitor
ExecStart=/home/jclazz/pigx/bin/pigx-monitor/run.sh
SuccessExitStatus=143
TimeoutStopSec=120
TimeoutSec=120
# Let systemd restart this service always
#Restart=always
# Never restart this service
Restart=no
RestartSec=30s

[Install]
WantedBy=default.target
启用服务

sudo systemctl enable pigx-monitor.service
辅助脚本

/home/jclazz/pigx/bin/pigx-monitor/run.sh

#!/bin/sh

java -Xmx512m -jar pigx-monitor.jar
# pigx-mp-manager
/etc/systemd/system/pigx-mp-manager.service

[Unit]
Description=pigx-admin
Requires=network.target
After=pigx-register.service

[Service]    
Type=simple
WorkingDirectory=/home/jclazz/pigx/bin/pigx-mp-manager
ExecStart=/home/jclazz/pigx/bin/pigx-mp-manager/run.sh
SuccessExitStatus=143
TimeoutStopSec=120
TimeoutSec=120
# Let systemd restart this service always
#Restart=always
# Never restart this service
Restart=no
RestartSec=30s

[Install]
WantedBy=default.target
启用服务

sudo systemctl enable pigx-mp-manager.service
辅助脚本

/home/jclazz/pigx/bin/pigx-mp-manager/run.sh

#!/bin/sh

java -Xmx512m -jar pigx-mp-manager.jar
# pigx-codegen
/etc/systemd/system/pigx-codegen.service

[Unit]
Description=pigx-admin
Requires=network.target
After=pigx-register.service

[Service]    
Type=simple
WorkingDirectory=/home/jclazz/pigx/bin/pigx-codegen
ExecStart=/home/jclazz/pigx/bin/pigx-codegen/run.sh
SuccessExitStatus=143
TimeoutStopSec=120
TimeoutSec=120
# Let systemd restart this service always
#Restart=always
# Never restart this service
Restart=no
RestartSec=30s

[Install]
WantedBy=default.target
启用服务

sudo systemctl enable pigx-codegen.service
辅助脚本

/home/jclazz/pigx/bin/pigx-codegen/run.sh

#!/bin/sh

java -Xmx512m -jar pigx-codegen.jar
# nginx配置
/etc/nginx/conf.d/pigx-ui.conf

server {
    listen 80;
    server_name pigx.cloud-dev.com;

    root /home/jclazz/pigx/www/pigx-ui/;

    #listen      443 ssl;
    #ssl_certificate /root/.acme.sh/pig4cloud.com/fullchain.cer;
    #ssl_certificate_key /root/.acme.sh/pig4cloud.com/pig4cloud.com.key;

          
    location ~* ^/(code|auth|admin|gen|daemon|tx|act|monitor|mp|job|pay) {
       proxy_pass http://127.0.0.1:9999;
       #proxy_set_header Host $http_host;
       proxy_connect_timeout 15s;
       proxy_send_timeout 15s;
       proxy_read_timeout 15s;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
} 
# 常见问题
# 页面打不开
检查对应的后端服务是否运行(见后面的服务状态检查)，虚拟机开机的时候所有服务都在启动，可能有一定延迟。
你的机器没有配置相关的host信息，参考前面的网络配置。
检查菜单配置，比如：有些菜单的URL是127.0.0.1,这样就只能在本机访问，修改为实际IP即可。
# 虚拟机模版
懒人可直接下载现成的虚拟机模版，然后导入VMware等虚拟化软件中，由于所有业务模块、中间件都已经配置为系统服务，因此虚拟机开机即时便会自动启动它们。

链接: https://pan.baidu.com/s/1cQ_F8UBi5zps7LzHa3e9HA 提取码: 9f21.

导入模版后第一件事是修改IP(/etc/network/interfaces)，并更新/etc/hosts

注意事项：

虚拟机里面有业务模块成品jar包，没有源代码和其他资料，如有侵权，请联系 jclazz@outlook.com
PigX不是开源软件，相关代码和资料请移步官网:https://pig4cloud.com/
# 参考资料
# zookeeper 安装信息
#安装路径
/usr/share/zookeeper
#配置文件
/etc/zookeeper/conf/zoo.cfg
# MinIO安装信息
MinIO安装路径为:/home/jclazz/minio/

# PigX 部署位置
PigX部署路径为:/home/jclazz/pigx/

# 系统服务的状态检查
#!/bin/bash
middleware_service=("mysql.service" "redis.service" "zookeeper.service" "minio.service")
core_service=("pigx-register.service" "pigx-gateway.service" "pigx-auth.service" "pigx-admin.service")
ex_service=("pigx-tx-manager.service" "pigx-elastic-job.service" "pigx-quartz.service" "pigx-activiti.service" "pigx-monitor.service" "pigx-mp-manager.service" "pigx-codegen.service")

function check_sudo {
	if [[ $UID != 0 ]]; then
	    echo "Please run this script with sudo:"
	    echo "sudo $0 $*"
	    exit 1
	fi
}
function stop {
	check_sudo
	op_services=("${ex_service[@]}" "${core_service[@]}")
	for e in "${op_services[@]}"
	do
		echo "stop $e"
		systemctl stop $e
	done
}

function start {
	check_sudo
	op_services=("${middleware_service[@]}" "${core_service[@]}")
	for e in "${op_services[@]}"
	do
		echo "start $e"
		systemctl start $e
	done
}

function stop_all {
	check_sudo
	op_services=("${ex_service[@]}" "${core_service[@]}" "${middleware_service[@]}")
	for e in "${op_services[@]}"
	do
		echo "stop $e"
		systemctl stop $e
	done
}

function start_all {
	check_sudo
	op_services=("${middleware_service[@]}" "${core_service[@]}" "${ex_service[@]}")
	for e in "${op_services[@]}"
	do
		echo "start $e"
		systemctl start $e
	done
}

function show_status {
	op_services=("${middleware_service[@]}" "${core_service[@]}" "${ex_service[@]}")
	for e in "${op_services[@]}"
	do
		echo "$e -> $(systemctl show -p ActiveState $e)"
	done
}
if [[ $1 == "status" ]]; then
	show_status
	exit 0
fi
if [[ $1 == "start-all" ]]; then
	start_all
	exit 0
fi
if [[ $1 == "stop-all" ]]; then
	stop_all
	exit 0
fi
if [[ $1 == "start" ]]; then
	start
	exit 0
fi
if [[ $1 == "stop" ]]; then
	stop
	exit 0
fi
echo "args: [status|start-all|stop-all]"

