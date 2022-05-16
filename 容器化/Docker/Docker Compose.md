### Docker Compose

#### 简介

容器编排，管理容器

#### 安装

1、下载

```shell
# 国内镜像
curl -L https://get.daocloud.io/docker/compose/releases/download/1.28.5/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
```

2、授权

```shell
chmod +x /usr/local/bin/docker-compose
```

3、检查安装是否成功

```shell
docker-compose version
```

#### 使用

1、准备应用，例如，xxx.jar

2、Dockerfile应用打包为镜像

3、Docker-compose yaml文件（定义整个服务，需要的环境）完整的上线服务！

4、启动compose项目（docker-compose up）

5、停止docker-compose down

#### Docker Yaml规则

docker-compose.yaml

```yaml
# 3层
version: '' #版本
services: #服务
	service1: xxx
		build: .
		container_name: ''
		ports: 
	service2: xxx
# 其他配置
volumes: 
```

#### 一键搭建博客

```shell
docker-compose up #前台启动
docker-compose -d up # 后台启动
docker-compose up --build # 重新构建
```

1、创建一个目录

```shell
mkdir my_wordpress
```

2、进入文件夹并创建docker-compose.yml

```shell
cd my_wordpress
vim docker-compose.yml
```

3、docker-compose内容

```yaml
version: "3.9"
    
services:
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
    
  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    volumes:
      - wordpress_data:/var/www/html
    ports:
      - "8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
volumes:
  db_data: {}
  wordpress_data: {}
```

4、创建并启动项目

```shell
docker-compose up -d
```

