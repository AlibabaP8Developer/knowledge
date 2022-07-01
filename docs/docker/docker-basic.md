# Docker基本操作

## 镜像操作命令

![](https://2022tiger.oss-cn-beijing.aliyuncs.com/image/1.png?versionId=CAEQIhiBgMCxgKLnjRgiIGQxNjJmZmRiYjAxODQ2ZGRiOTM5Njk3ZjhmZjhhMjMx)

案例：从dockerhub中拉取一个nginx镜像并查看
1.首先去镜像仓库搜索nginx镜像，比如DockerHub

https://hub.docker.com/

2.根据查看到的镜像名称，拉取自己需要的镜像，通过命令：docker pull nginx

3.通过命令：docker images查看镜像

案例：利用docker save将nginx镜像导出磁盘，然后再通过load加载回来 

1.利用docker xxx --help命令查看docker save和docker load语法

![](https://2022tiger.oss-cn-beijing.aliyuncs.com/image/2.png?versionId=CAEQIhiCgICygKLnjRgiIDk3NTFkY2NjMmYwNTRlNGFhNDE2NWU2YWViZmJmZGU4)

2.使用docker tag创建新镜像mynginx1.0

3.使用docker save导出镜像到磁盘

![](https://2022tiger.oss-cn-beijing.aliyuncs.com/image/3.png?versionId=CAEQIhiBgICygKLnjRgiIDJkZTEzYjI4YjlkNDQwNzliMGE0ZWMwNDNiMjVjZDcx)

4.通过load加载镜像

![](https://2022tiger.oss-cn-beijing.aliyuncs.com/image/6.png?versionId=CAEQIhiBgMDMpqfnjRgiIGYxOWI3YjkwMTU3ZjQ2ODViYjU1YjY0MzEwYzg0N2U4)

## 容器相关命令

![](https://2022tiger.oss-cn-beijing.aliyuncs.com/image/5.png?versionId=CAEQIhiBgMDn_6HnjRgiIDQ3MjRjOGYwNzQ3NjQyOTg5ZDJlNjNmNTkwMDAyNTdm)

案例：创建运行一个Nginx容器

1.去docker hub查看nginx的容器运行命令

```java
docker run --name containerName -p 80:80 -d nginx

命令解读：

docker run：创建并运行一个容器

--name：给容器起一个名字，比如叫做mn

-p：将宿主机端口与容器端口映射，冒号左侧是宿主机端口，右侧是容器端口
 
docker run --name mn -p 80:80 -d nginx
```

案例：进入nginx容器，修改html文件内容，添加“大明太祖爷朱元璋”

1.进入容器

```java
docker exec -it mn bash

命令解读：

docker exec：进入容器内部，执行一个命令

-it：给当前进入的容器创建一个标准输入输出终端，允许我们与容器交互
  
mn：要进入的容器的名称

bash：进入容器后执行的命令，bash是一个Linux终端交互命令
```

2.进入nginx的html所在目录 /usr/share/nginx/html

3.修改index.html的内容

```java
sed -i 's#Welcome to nginx#传智播客#g' index.html

sed -i 's#<head>#<head><meta charset="utf-8">#g' index.html
```

# 数据卷

容器与数据耦合的问题

![](https://2022tiger.oss-cn-beijing.aliyuncs.com/image/7.png?versionId=CAEQIhiBgIDV5rPnjRgiIDlmNWUzNjJiMGY5NzRmYTU5ZjkwMjVmOTc5NTQ1ZDk4)

数据卷（volume）是一个虚拟目录，指向宿主机文件系统中的某个目录

![](https://2022tiger.oss-cn-beijing.aliyuncs.com/image/8.png?versionId=CAEQIhiBgICF57PnjRgiIGUzOWIxN2E5ZTVmOTQzZjZiYWE5ZTY1ZGZkZjI4ZjI4)

## 操作数据卷

数据卷操作的基本语法：

docker volume [COMMAND]

docker volume命令是数据卷操作，根据命令后跟随的command来确定下一步的操作：

create  创建一个volume

inspect 显示一个或多个volume的信息

ls    列出所有的volume

prune  删除未使用的volume

rm    删除一个或多个指定的volume

 

案例：创建一个数据卷，并查看数据卷在宿主机的目录位置

1.创建数据卷

docker volume create html

2.查看所有数据卷

docker volume ls

3.查看数据卷详细信息卷

docker volume inspect html

数据卷的作用：

将容器与数据分离，解耦合，方便操作容器内数据，保证数据安全

数据卷操作：

docker volume create/ls/inspect/rm/prune

## 挂载数据卷

在创建容器时，可以通过-v参数来挂载一个数据卷到某个容器目录

![](https://2022tiger.oss-cn-beijing.aliyuncs.com/image/9.png?versionId=CAEQIhiBgID65rPnjRgiIGY0MmUzOGE3NzIxNjRlYTI4Y2YzMDgxNzcxODczM2Y3)

案例：创建一个nginx容器，修改容器内的html目录内的index.html内容

需求说明：上个案例中，我们进入nginx容器内部，已经知道nginx的html目录所在位置/usr/share/nginx/html，我们需要把这个目录挂载到html这个数据卷上，方便操作其中的内容。

提示：运行容器时使用-v参数挂载数据卷

步骤：

1.创建容器并挂载数据卷到容器内的html目录

```java
docker run --name mn -p 8082:8082 -v html:/usr/share/nginx/html -d nginx
```

2.进入html数据卷所在位置，并修改html内容

```java
# 查看html数据卷的位置

docker volume inspect html

cd /var/lib/docker/volumes/html/_data

vim index.html
```

