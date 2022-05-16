---
typora-copy-images-to: upload
---

### Linux配置

#### 1.设置通过主机名访问

> 修改/etc/hosts文件
>
> 在每个主机下分别加入本机与其他主机与ip的映射关系
>
> 10.211.55.4 unclebryan01
> 10.211.55.5 unclebryan02
> 10.211.55.6 unclebryan03

#### 2.免密登录

```bash
# 执行下面命令一路回车
ssh-keygen -t rsa
# 将秘钥copy到想要免密登录的机器上(本机最好也执行一下，自己免密登录自己)
ssh-copy-id ip/主机名
# 免密登录后可以直接通过下面命令连接远程主机
ssh ip/主机名
```



#### 3.编写文件分发脚本

> 将文件分发到集群中的每一台机器上
>
> 在/usr/local/bin路径下新建xsync文件

```bash
#!/bin/bash
#1. 判断参数个数
if [ $# -lt 1 ]
then
    echo Not Enough Arguement!
    exit;
fi

#2. 遍历集群所有机器
for host in unclebryan02 unclebryan03
do
    echo ==================== $host ====================
    #3. 遍历所有目录，挨个发送
    for file in $@
    do
        #4. 判断文件是否存在
        if [ -e $file ]
            then
                #5. 获取父目录
                pdir=$(cd -P $(dirname $file); pwd)
                #6. 获取当前文件的名称
                fname=$(basename $file)
                ssh $host "mkdir -p $pdir"
                rsync -av $pdir/$fname $host:$pdir
            else
                echo $file does not exists!
        fi
    done
done
```

```bash
#设置脚本权限
chmod 777 xsync
# 执行测试
xsync + 文件夹 
```



#### 4.编写批量命令执行脚本

在/usr/local/bin路径下新建xcall文件

```bash
#!/bin/bash
#在集群的所有机器上批量执行同一条命令
if(($#==0))
then
	echo 请输入您要操作的命令！
	exit
fi

echo 要执行的命令是$*

#循环执行此命令
for((i=1;i<=3;i++))
do
	echo ---------------------unclebryan0$i-----------------
	ssh unclebryan0$i $*
done

```

```bash
# 执行命令时一定要带参数，否则会直接退出
xcall # 会退出脚本
xcall ls # 不会退出脚本
```



#### 5.删除自带的jdk

```bash

sudo yum -y remove *openjdk*

# 配置自己安装的jdk
echo 'JAVA_HOME=/usr/local/software/jdk1.8.0_311' >> ~/.bash_profile
echo 'PATH=$PATH:$JAVA_HOME/bin:$JAVA_HOME/lib' >> ~/.bash_profile
source ~/.bash_profile
```



#### 6.安装zshell

```bash
# 查看当前主机的shell类型
echo $SHELL 

# 查看是否已安装zshell
cat /etc/shells 

# zsh安装
#CentOS： 
yum -y install zsh git
#Ubuntu:  
sudo apt -y install zsh git

# 安装ohmyzsh
$ sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
或者
$ sh -c "$(wget https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"

# 切换zshell
chsh -s /bin/zsh

# 切换为bash
chsh -s /bin/bash

# ps. 切换后新开一个终端才会生效
```



```bash
# 如果安装ohmyzsh一直失败，有可能是网络原因，请替换国内镜像
# 下载码云安装包
wget https://gitee.com/mirrors/oh-my-zsh/raw/master/tools/install.sh

# 修改install.sh中的下面两行
REPO=${REPO:-ohmyzsh/ohmyzsh}
REMOTE=${REMOTE:-https://github.com/${REPO}.git}
# 为
REPO=${REPO:-mirrors/oh-my-zsh}
REMOTE=${REMOTE:-https://gitee.com/${REPO}.git}

#运行install.sh
./install.sh

# 修改源码仓库地址
cd ~/.oh-my-zsh
git remote set-url origin https://gitee.com/mirrors/oh-my-zsh.git
git pull
```

#### 7.安装mysql

```bash
# 下载rpm
wget https://repo.mysql.com//mysql80-community-release-el7-1.noarch.rpm

# 进行repo安装，会在/etc/yum.repos.d/目录下生成两个repo文件mysql-community.repo mysql-community-source.repo
rpm -ivh mysql80-community-release-el7-1.noarch.rpm

# 安装
 yum install mysql-server -y


# mysql设置不区分大小写
vim /etc/my.cnf
#让MYSQL大小写敏感(1-不敏感，0-敏感)
lower_case_table_names=1

# 启动服务
systemctl start mysqld.service

# 查看状态
systemctl status mysqld.service

# 获取密码，有时候是空密码
grep "password" /var/log/mysql/mysqld.log 
或者
grep "password" /var/log/mysqld.log

# 登录并修改密码
mysql -uroot -p 
ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';

# 远程登录
use mysql;
update user  set host = '%' where user = 'root'; 

# 设置后远程登录后，记得重启一下服务
systemctl restart mysqld.service
```

安装中的问题

1. 

> 用Navicat连接mysql，报错如下：
>  `Client does not support authentication protocol requested by server；`
>  报错原因：
>  mysql8.0 引入了新特性 `caching_sha2_password`；这种密码加密方式Navicat 12以下客户端不支持；
>  Navicat 12以下客户端支持的是`mysql_native_password`这种加密方式；
>  解决方案：
>  1，用如下语句查看MySQL当前加密方式
>  `select host,user,plugin from user;`
>  查询结果
>
> ```ruby
> +-----------+------------------+-----------------------+
> | host      | user             | plugin                |
> +-----------+------------------+-----------------------+
> | %         | root             | caching_sha2_password |
> | localhost | mysql.infoschema | mysql_native_password |
> | localhost | mysql.session    | mysql_native_password |
> | localhost | mysql.sys        | mysql_native_password |
> +-----------+------------------+-----------------------+
> ```
>
> 看第一行，root加密方式为`caching_sha2_password`。
>  使用命令将他修改成`mysql_native_password`加密模式：
>  `update user set plugin='mysql_native_password' where user='root';`
>  再次连接

2. 

```log
安装mysql时报错：

warning: /var/cache/yum/x86_64/7/mysql80-community/packages/mysql-community-server-8.0.28-1.el7.x86_64.rpm: Header V4 RSA/SHA256 Signature, key ID 3a79bd29: NOKEY
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql


The GPG keys listed for the "MySQL 8.0 Community Server" repository are already installed but they are not correct for this package.
Check that the correct key URLs are configured for this repository.


 Failing package is: mysql-community-server-8.0.28-1.el7.x86_64
 GPG Keys are configured as: file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql


解决方案：
这个key文件RPM-GPG-KEY-mysql不对，要替换成正确的文件。

我安装的是mysql 8，下载最新的key文件地址为：https://repo.mysql.com/

这个地址下的最新key文件是：

RPM-GPG-KEY-mysql-2022
将这个文件下载到本地的key文件夹。

cd /etc/pki/rpm-gpg/

wget https://repo.mysql.com/RPM-GPG-KEY-mysql-2022

然后需要修改yum repo文件来设置安装时，使用这个key文件

vi /etc/yum.repos.d/mysql-community.repo
修改为下面内容后重新安装

```

![Screen Shot 2022-04-10 at 15.48.56](https://tva1.sinaimg.cn/large/e6c9d24egy1h14oddc7suj20ik0e2t9z.jpg)







```properties
http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    #keepalive_timeout  65;

    #gzip  on;


upstream nacos-server{
        server 192.168.64.2:8848 weight=1;
        server 192.168.64.3:8848 weight=1;
        server 192.168.64.4:8848 weight=1;
}
 server {
      #  listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;


        location /nacos {
                proxy_pass  http://nacos-server;
                proxy_set_header Host $host;
                proxy_set_header X-Real-Ip $remote_addr;
                proxy_set_header X-Forwarded-For $remote_addr;
        }
 }
```



#### 8. yum报错

```bash
CentOS Linux 8 - AppStream                                                                                                                                                                                            73  B/s |  38  B     00:00
Error: Failed to download metadata for repo 'appstream': Cannot prepare internal mirrorlist: No URLs in mirrorlist
```

**Step 1:** Go to the `/etc/yum.repos.d/` directory.

```bash
[root@autocontroller ~]# cd /etc/yum.repos.d/
```

**Step 2:** Run the below commands

```bash
[root@autocontroller ~]# sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
[root@autocontroller ~]# sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*
```

**Step 3:** Now run the yum update

```bash
[root@autocontroller ~]# yum update -y
```

#### 9. vim中文乱码

> vim的设置一般放在/etc/vimrc文件中，不过，建议不要修改它。可以修改~/.vimrc文件（默认不存在，可以自己新建一个），写入所希望的设置。~/.vimrc内容如下：

```bash
set fileencodings=utf-8,ucs-bom,gb18030,gbk,gb2312,cp936
set termencoding=utf-8
set encoding=utf-8
```

