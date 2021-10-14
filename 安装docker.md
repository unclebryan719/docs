### 安装docker

1. 卸载旧版本docker

```shell
 $ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

2. 设置仓库

   ```shell
   $ sudo yum install -y yum-utils
   
   $ sudo yum-config-manager \
       --add-repo \
       https://download.docker.com/linux/centos/docker-ce.repo
    建议换成阿里云仓库
   
   $ sudo yum-config-manager \
       --add-repo \
       http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
   
   ```

3. 更新软件包索引

   ```shell
   $ sudo yum makecache fast
   ```

4. 安装docker引擎

   ```shell
   $ sudo yum install docker-ce docker-ce-cli containerd.io
   ```

5. 启动docker服务

   ```shell
   $ sudo systemctl start docker
   ```

6. 检查docker服务是否启动成功

   ``` shell
   $ sudo docker version
   ```

7. 运行hello world

   ```shell
   $ sudo docker run hello-world
   ```

8. 查看镜像列表

   ```shell
   $ sudo docker images
   ```

9. 卸载docker

   ``` shell
   $ sudo yum remove docker-ce docker-ce-cli containerd.io
   $ sudo rm -rf /var/lib/docker
   $ sudo rm -rf /var/lib/containerd
   ```

10. 配置阿里云镜像加速

    ``` shell
    sudo mkdir -p /etc/docker
    sudo tee /etc/docker/daemon.json <<-'EOF'
    {
      "registry-mirrors": ["https://99ogs5av.mirror.aliyuncs.com"]
    }
    EOF
    sudo systemctl daemon-reload
    sudo systemctl restart docker
    ```

    

