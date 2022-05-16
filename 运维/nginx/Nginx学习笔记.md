---

typora-copy-images-to: upload
---

### Nginx学习笔记

#### 正向代理与反向代理

正向代理：代理客户端，代理软件安装在客户端，比如vpn翻墙

![Screen Shot 2022-04-10 at 11.27.49](https://tva1.sinaimg.cn/large/e6c9d24egy1h14gt4y4irj20za0mgdhw.jpg)



反向代理：代理服务端，代理软件安装在服务端，比如Nginx做服务器端的负载均衡，请求转发。

![Screen Shot 2022-04-10 at 11.28.08](https://tva1.sinaimg.cn/large/e6c9d24egy1h14gstoslzj21320outb6.jpg)



#### iphash

> iphash对客户端请求的ip就行hash操作，然后根据hash结果将同一个客户端ip的请求分发到同一台服务器进行处理，可以解决session不共享的问题。
>
> 但是一般不用上述方式做session共享，因为一旦某一台服务器挂了，session信息会丢失，通常情况下是通过redis做session共享



#### Nginx安装

> 下载地址：http://nginx.org/en/download.html

```bash
# 将下载好的Nginx上传到服务器并解压
tar -zxvf nginx-1.20.2.tar.gz

# 进入到文件目录编译安装nginx
./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --without-http_rewrite_module

make && make install

#编译Stream模块
./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-stream=dynamic --without-http_rewrite_module

make
不需要执行make install

# 在Nginx的安装目录创建modules目录，并执行下面命令
cp /usr/local/software/nginx-1.20.2/objs/ngx_stream_module.so    /usr/local/nginx/modules/

# 修改nginx.conf配置文件，载入模块
load_module  modules/ngx_stream_module.so;

stream {
   ....
}

# 查看nginx安装目录
whereis nginx

# 进入到安装目录下的sbin目录执行启动命令
cd /usr/local/nginx/sbin
./nginx


# 停止
./nginx -s stop

# 安全退出
./nginx -s quit

# 重新加载配置文件
./nginx -s reload


# 软连接配置
ln -s /usr/local/nginx/sbin/nginx /usr/local/bin/nginx

```



#### nginx配置nacos集群

配置nacos

```properties
```



nginx.conf

```properties
load_module  modules/ngx_stream_module.so;
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    upstream nacos-server{
            server unclebryan01:8848 weight=1;
            server unclebryan02:8848 weight=1;
            server unclebryan03:8848 weight=1;
    }
    server {
        listen       80;
        server_name  localhost;
        location /nacos {
                proxy_pass  http://nacos-server;
                proxy_set_header Host $host;
                proxy_set_header X-Real-Ip $remote_addr;
                proxy_set_header X-Forwarded-For $remote_addr;
     		}
 		}
}
stream {
 include /usr/local/nginx/tcp.d/*.conf;
}
```

  

tcp.d/nacos.conf  

```properties
upstream erp-nacos-grpc{
          hash $remote_addr consistent;
          server unclebryan01:9848 weight=1;
          server unclebryan02:9848 weight=1;
          server unclebryan03:9848 weight=1;
  }
  upstream erp-nacos-grpc9{
          hash $remote_addr consistent;
          server unclebryan01:9849 weight=1;
          server unclebryan02:9849 weight=1;
          server unclebryan03:9849 weight=1;
  }
  server {
      listen 1080; # grpc方式对外暴露端口
      proxy_connect_timeout 10s;
      proxy_timeout 10s;
      proxy_pass erp-nacos-grpc;
  }

  server {
      listen 1081; # grpc方式对外暴露端口
      proxy_connect_timeout 10s;
      proxy_timeout 10s;
      proxy_pass erp-nacos-grpc9;
  }

```

```bash
#!/bin/bash

case $1 in
"start"){
	for i in unclebryan01 unclebryan02 unclebryan03
	do
		echo -----------nacos $i 启动----------------
		ssh $i "sh /usr/local/software/nacos/bin/startup.sh"
	done
}
;;

"stop"){
	for i in unclebryan01 unclebryan02 unclebryan03
	do
		echo -----------nacos $i 停止----------------
		ssh $i "sh /usr/local/software/nacos/bin/shutdown.sh"
	done
}
;;


"status"){
	for i in unclebryan01 unclebryan02 unclebryan03
	do
		echo -----------nacos $i 启动中----------------
		ssh $i "tail -f /usr/local/software/nacos/logs/start.out"
	done
}
;;
esac
```



PS. 如果出现JAVA_HOME is not set and java could not be found in PATH，请修改startup.sh

```bash
# 在文件的最前面增加javahome路径
export JAVA_HOME="/usr/local/software/jdk1.8.0_311"
```

