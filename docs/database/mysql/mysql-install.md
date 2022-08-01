# Linux-mysql安装

**1.下载Linux版MySQL安装包**

https://downloads.mysql.com/archives/community/

**2 上传MySQL安装包**

**3.创建目录,并解压**

```
mkdir mysql
 
tar -xvf mysql-8.0.26-1.el7.x86_64.rpm-bundle.tar -C mysql
```

 

**4. 安装mysql的安装包**

```
cd mysql
 
rpm -ivh mysql-community-common-8.0.26-1.el7.x86_64.rpm 
 
rpm -ivh mysql-community-client-plugins-8.0.26-1.el7.x86_64.rpm 
 
rpm -ivh mysql-community-libs-8.0.26-1.el7.x86_64.rpm 
 
rpm -ivh mysql-community-libs-compat-8.0.26-1.el7.x86_64.rpm
 
yum install openssl-devel
 
rpm -ivh mysql-community-devel-8.0.26-1.el7.x86_64.rpm
 
rpm -ivh mysql-community-client-8.0.26-1.el7.x86_64.rpm
 
rpm -ivh mysql-community-server-8.0.26-1.el7.x86_64.rpm
```

**5.启动MySQL服务**

```
systemctl start mysqld
systemctl restart mysqld
systemctl stop mysqld
```

**6. 查询自动生成的root用户密码**

```
grep 'temporary password' /var/log/mysqld.log
命令行执行指令 :
mysql -u root -p
然后输入上述查询到的自动生成的密码, 完成登录 .
```

**7. 修改root用户密码**

登录到MySQL之后，需要将自动生成的不便记忆的密码修改了，修改成自己熟悉的便于记忆的密码。

```
ALTER USER 'root'@'localhost' IDENTIFIED BY '1234';
```

执行上述的SQL会报错，原因是因为设置的密码太简单，密码复杂度不够。我们可以设置密码的复杂度为简单类型，密码长度为4。

```
set global validate_password.policy = 0;
set global validate_password.length = 4;
```

降低密码的校验规则之后，再次执行上述修改密码的指令。

**8. 创建用户**

默认的root用户只能当前节点localhost访问，是无法远程访问的，我们还需要创建一个root账户，用户远程访问

```
create user 'root'@'%' IDENTIFIED WITH mysql_native_password BY '1234';
```

**9. 并给root用户分配权限**

```
grant all on *.* to 'root'@'%';
```

**10. 重新连接MySQL**

```
mysql -u root -p
然后输入密码
```

# Docker MySQL安装

## mysql5.7

（1）拉取mysql镜像

docker pull centos/mysql-57-centos7

（2）创建容器

docker run -di --name=mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql

-p 代表端口映射，格式为 宿主机映射端口:容器运行端口

-e 代表添加环境变量 MYSQL_ROOT_PASSWORD 是root用户的登陆密码

（3）远程登录mysql

连接宿主机的IP ,指定端口为3306

## mysql8.0

### CentOS7.x：

1.准备目录

为了方便后期配置MySQL，我们先准备两个目录，用于挂载容器的数据和配置文件目录：

```
# 进入/tmp目录
cd /tmp
# 创建文件夹
mkdir mysql
# 进入mysql目录
cd mysql
```

2.运行命令

进入mysql目录后，执行下面的Docker命令：

```
docker run \
 -p 3306:3306 \
 --name mysql \
 -v $PWD/conf:/etc/mysql/conf.d \
 -v $PWD/logs:/logs \
 -v $PWD/data:/var/lib/mysql \
 -e MYSQL_ROOT_PASSWORD=123456 \
 --privileged \
 -d \
 mysql:5.7.25
```

3.修改配置

在/tmp/mysql/conf目录添加一个my.cnf文件，作为mysql的配置文件：

```
# 创建文件
touch /tmp/mysql/conf/my.cnf
```

文件的内容如下：

```
[mysqld]
skip-name-resolve
character_set_server=utf8
datadir=/var/lib/mysql
server-id=1000
```

4.重启

配置修改后，必须重启容器：

```
docker restart mysql
```

### MacBook M1：

**arm docker mysql安装：**

当前docker的镜像源为华为云 (原版镜像自带)
 docker pull mysql/mysql-server:8.0.24

 

此步骤为建立一个docker网卡 主要是多容器之前内网ip通讯(重启后内网ip可能会变 之前遇到过)
[此步骤非必须]
docker network create --subnet=172.10.0.0/16 dockerNetWork

 

镜像创建一个容器 将容器的3306端口映射到宿主机的9091端口(看需要选择)
docker run -itd --name mysql -p 3306:3306 --net dockerNetWork --ip 127.0.0.1 -e MYSQL_ROOT_PASSWORD=123456 mysql/mysql-server:8.0.24

**Mac book M1本机docker安装MySQL：**

docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql/mysql-server

执行docker exec 进入容器内部 容器名为上一步所填
 docker exec -it mysql /bin/bash

执行mysql -uroot -p 进入mysql进行设置

退出mysql命令 quit
退出容器命令 exit

docker run -itd --name=umysql -p 3306:3306 --ip 127.0.0.1 -e MYSQL_ROOT_PASSWORD=123456 mysql/mysql-server:8.0.24