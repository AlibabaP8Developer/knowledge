# 一、在CentOS上安装Docker

其他系统参照如下文档

https://docs.docker.com/engine/install/centos/

1、移除以前docker相关包

 ```java
 sudo yum remove docker \
 
         docker-client \
 
         docker-client-latest \
 
         docker-common \
 
         docker-latest \
 
         docker-latest-logrotate \
 
         docker-logrotate \
 
         docker-engine
 ```

2.配置yum源

```java
sudo yum install -y yum-utils
sudo yum-config-manager \
--add-repo \
http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

3.安装Docker

```java
sudo yum install -y docker-ce docker-ce-cli containerd.io
#以下是在安装k8s的时候使用
yum install -y docker-ce-20.10.7 docker-ce-cli-20.10.7  containerd.io-1.4.6
```

4.启动

```java
systemctl enable docker --now
```

5.配置加速

```java
这里额外添加了docker的生产环境核心配置cgroup
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://82m9ar63.mirror.aliyuncs.com"],
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

6.Docker命令

```java
systemctl命令是系统服务管理器指令
启动docker：
		systemctl start docker
停止docker：
		systemctl stop docker
重启docker：
		systemctl restart docker
查看docker状态：
		systemctl status docker
开机启动：
		systemctl enable docker
查看docker概要信息
		docker info
查看docker帮助文档
	  docker --help
```

# 二、在Ubuntu上安装Docker

```java
安装docker
		Ubuntu：sudo apt-get install -y docker.io
安装后查看docker版本
	  docker -v
```



