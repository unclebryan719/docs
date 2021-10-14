注意 ,如果使用以下方法, (官方Nacos包) pigx-register 模块不需要启动

建议使用端口 （8840 8841 8842 ）只需要修改 hosts pigx-register 的映射即可完成集群接入 不需要修改密码

# 下载Nacos-server 安装包
nacos-server-1.1.3.zip

修改 conf/application.properties 数据源信息
spring.datasource.platform=mysql

db.num=1
db.user=root
db.password=root
db.url.0=jdbc:mysql://${MYSQL-HOST:pigx-mysql}:${MYSQL-PORT:3306}/${MYSQL-DB:pigxx_config}?characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=false&useJDBCCompliantTimezoneShift=true&useLegacyDatetimeCode=false&serverTimezone=GMT%2B8&nullCatalogMeansCurrent=true&allowPublicKeyRetrieval=true
修改 conf/cluster.conf 集群配置
#it is ip
#多网卡 注意此处IP 和nacos控制台输出的保持一致，不然应用报错
127.0.0.1:8840
127.0.0.1:8841
127.0.0.1:8842

复制三份 bin/startup.sh 启动脚本，增加端口配置
cp startup.sh startup8840.sh
cp startup.sh startup8841.sh
cp startup.sh startup8842.sh
vim startup8840.ssh 
# line113  注意端口
JAVA_OPT="${JAVA_OPT} --server.port=8840"

vim startup8841.ssh 
# line113  注意端口
JAVA_OPT="${JAVA_OPT} --server.port=8841"

vim startup8842.ssh 
# line113  注意端口
JAVA_OPT="${JAVA_OPT} --server.port=8842"
启动nacos
sh startup8840.sh
sh startup8841.sh
sh startup8842.sh
配置NGINX 解析
upstream serverList {
	server 127.0.0.1:8840;
	server 127.0.0.1:8841;
	server 127.0.0.1:8842;
}

server {
	listen 8848;
	server_name localhost;
	location / {
		proxy_pass http://serverList;
		index index.html index.htm;
	}
}


注意修改hosts 映射，代码不需要修改
pigx-register 指向nginx 8848
pigx-register  127.0.0.1