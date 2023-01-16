# 1.Hadoop运行模式

1.Hadoop 官方网站：http://hadoop.apache.org/

2.Hadoop 运行模式包括：本地模式、伪分布式模式以及完全分布式模式。

​	➢ 本地模式：单机运行，只是用来演示一下官方案例。生产环境不用。 

​	➢ 伪分布式模式：也是单机运行，但是具备 Hadoop 集群的所有功能，一台服务器模拟一个分布式的环境。个别缺钱的公司用来测试，生产环境不用。

​	➢ 完全分布式模式：多台服务器组成分布式环境。生产环境使用。

# 1.1本地运行模式（官方 WordCount）

1）创建在 hadoop-3.1.3 文件下面创建一个 wcinput 文件夹

```sh
[atguigu@hadoop102 hadoop-3.1.3]$ mkdir wcinput
```

2）在 wcinput 文件下创建一个 word.txt 文件

```sh
[atguigu@hadoop102 hadoop-3.1.3]$ cd wcinput
```

3）编辑 word.txt 文件

```sh
[atguigu@hadoop102 wcinput]$ vim word.txt
```

➢ 在文件中输入如下内容

```sh
hadoop yarn
hadoop mapreduce
atguigu
atguigu
```

➢ 保存退出：:wq

4）回到 Hadoop 目录/opt/module/hadoop-3.1.3

5）执行程序

[atguigu@hadoop102 

hadoop-3.1.3]$ 

hadoop jar 

share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar 

wordcount wcinput wcoutput

6）查看结果

```sh
[atguigu@hadoop102 hadoop-3.1.3]$ cat wcoutput/part-r-00000
```

看到如下结果：

```sh
atguigu 2
hadoop 2
mapreduce 1
yarn 1
```

# 1.2 完全分布式运行模式（开发重点）

分析：

- 准备 3 台客户机（关闭防火墙、静态 IP、主机名称）

- 安装 JDK

- 配置环境变量

- 安装 Hadoop

- 配置环境变量

- 配置集群
- 单点启动

- 配置 ssh

- 群起并测试集群


## 1、虚拟机准备

详见《Hadoop 运行环境搭建》。

## 2、编写集群分发脚本 xsync

（1）scp（secure copy）安全拷贝

- scp 定义：scp 可以实现服务器与服务器之间的数据拷贝。（from server1 to server2）


- 基本语法


```sh
scp -r   $pdir/$fname      $user@$host:$pdir/$fname
命令 递归 要拷贝的文件路径/名称 目的地用户@主机:目的地路径/名称
```

- 案例实操


➢ 前提：在 hadoop102、hadoop103、hadoop104 都已经创建好的/opt/module、 /opt/software 两个目录，并且已经把这两个目录修改为 atguigu:atguigu

```sh
[atguigu@hadoop102 ~]$ sudo chown atguigu:atguigu -R /opt/module
```

（a）在 hadoop102 上，将 hadoop102 中/opt/module/jdk1.8.0_212 目录拷贝到hadoop103 上。

```sh
[atguigu@hadoop102 ~]$ scp -r /opt/module/jdk1.8.0_212 atguigu@hadoop103:/opt/module
```

（b）在 hadoop103 上，将 hadoop102 中/opt/module/hadoop-3.1.3 目录拷贝到hadoop103 上。

```sh
[atguigu@hadoop103 ~]$ scp -r atguigu@hadoop102:/opt/module/hadoop-3.1.3 /opt/module/
```

（c）在 hadoop103 上操作，将 hadoop102 中/opt/module 目录下所有目录拷贝到hadoop104 上。

```sh
[atguigu@hadoop103 opt]$ scp -r atguigu@hadoop102:/opt/module/*
atguigu@hadoop104:/opt/module
```

（2）rsync 远程同步工具

rsync 主要用于备份和镜像。具有速度快、避免复制相同内容和支持符号链接的优点。

rsync 和 scp 区别：用 rsync 做文件的复制要比 scp 的速度快，rsync 只对差异文件做更新。scp 是把所有文件都复制过去。

- 基本语法


```sh
rsync -av     $pdir/$fname       $user@$host:$pdir/$fname
命令   选项参数 要拷贝的文件路径/名称 目的地用户@主机:目的地路径/名称 选项参数说明
```

| 选项 | 功能         |
| ---- | ------------ |
| -a   | 归档拷贝     |
| -v   | 显示复制过程 |

- 案例实操

（a）删除 hadoop103 中/opt/module/hadoop-3.1.3/wcinput

```sh
[atguigu@hadoop103 hadoop-3.1.3]$ rm -rf wcinput/
```

（b）同步 hadoop102 中的/opt/module/hadoop-3.1.3 到 hadoop103

```sh
[atguigu@hadoop102 module]$ rsync -av hadoop-3.1.3/ atguigu@hadoop103:/opt/module/hadoop-3.1.3/
```

（3）xsync集群分发脚本

- 需求：循环复制文件到所有节点的相同目录下
- 需求分析：

（a）rsync 命令原始拷贝：

```sh
rsync -av /opt/module atguigu@hadoop103:/opt/
```

（b）期望脚本：xsync 要同步的文件名称

（c）期望脚本在任何路径都能使用（脚本放在声明了全局环境变量的路径）

```sh
[atguigu@hadoop102 ~]$ echo $PATH /usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/atguigu/.local/bin:/home/atguigu/bin:/opt/module/jdk1.8.0_212/bin
```

- 脚本实现

（a）在/home/atguigu/bin 目录下创建 xsync 文件

```sh
[atguigu@hadoop102 opt]$ cd /home/atguigu
[atguigu@hadoop102 ~]$ mkdir bin
[atguigu@hadoop102 ~]$ cd bin
[atguigu@hadoop102 bin]$ vim xsync
```

在该文件中编写如下代码

```sh
#!/bin/bash
#1. 判断参数个数
if [ $# -lt 1 ]
then
 echo Not Enough Arguement!
 exit;
fi
#2. 遍历集群所有机器
for host in hadoop102 hadoop103 hadoop104
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

（b）修改脚本 xsync 具有执行权限

```sh
[atguigu@hadoop102 bin]$ chmod +x xsync
```

（c）测试脚本

```sh
[atguigu@hadoop102 ~]$ xsync /home/atguigu/bin
```

（d）将脚本复制到/bin 中，以便全局调用

```sh
[atguigu@hadoop102 bin]$ sudo cp xsync /bin/
```

（e）同步环境变量配置（root 所有者）

```sh
[atguigu@hadoop102 ~]$ sudo ./bin/xsync 
/etc/profile.d/my_env.sh
```

注意：如果用了 sudo，那么 xsync 一定要给它的路径补全。

让环境变量生效

```sh
[atguigu@hadoop103 bin]$ source /etc/profile
[atguigu@hadoop104 opt]$ source /etc/profile
```

## 3、SSH 无密登录配置

（1）配置ssh

- 基本语法

​	ssh 另一台电脑的 IP 地址

- ssh 连接时出现 Host key verification failed 的解决方法

```sh
[atguigu@hadoop102 ~]$ ssh hadoop103
```

➢ 如果出现如下内容

```sh
Are you sure you want to continue connecting (yes/no)? 
```

➢ 输入 yes，并回车

- 退回到 hadoop102

```sh
[atguigu@hadoop103 ~]$ exit
```

（2）无密钥配置

- 免密登录原理

  ![](./assets/05.png)

- 生成公钥和私钥

  ```sh
  [atguigu@hadoop102 .ssh]$ pwd
  /home/atguigu/.ssh
  [atguigu@hadoop102 .ssh]$ ssh-keygen -t rsa
  ```

  然后敲（三个回车），就会生成两个文件 id_rsa（私钥）、id_rsa.pub（公钥）

- 将公钥拷贝到要免密登录的目标机器上

  ```sh
  [atguigu@hadoop102 .ssh]$ ssh-copy-id hadoop102
  [atguigu@hadoop102 .ssh]$ ssh-copy-id hadoop103
  [atguigu@hadoop102 .ssh]$ ssh-copy-id hadoop104
  ```

  注意：

  还需要在 hadoop103 上采用 atguigu 账号配置一下无密登录到 hadoop102、hadoop103、hadoop104 服务器上。

  还需要在 hadoop104 上采用 atguigu 账号配置一下无密登录到 hadoop102、hadoop103、hadoop104 服务器上。

  还需要在 hadoop102 上采用 root 账号，配置一下无密登录到 hadoop102、hadoop103、hadoop104；

（3）.ssh 文件夹下（~/.ssh）的文件功能解释

| known_hosts     | 记录 ssh 访问过计算机的公钥（public key） |
| --------------- | ----------------------------------------- |
| id_rsa          | 生成的私钥                                |
| id_rsa.pub      | 生成的公钥                                |
| authorized_keys | 存放授权过的无密登录服务器公钥            |

# 4、集群配置

## 1.集群部署规划

注意：

​	➢ NameNode 和 SecondaryNameNode 不要安装在同一台服务器

​	➢ ResourceManager 也很消耗内存，不要和 NameNode、SecondaryNameNode 配置在同一台机器上。

|      | hadoop102          | hadoop103                    | hadoop104                   |
| ---- | ------------------ | ---------------------------- | --------------------------- |
| HDFS | NameNode、DataNode | DataNode                     | SecondaryNameNode、DataNode |
| YARN | NodeManager        | ResourceManager、NodeManager | NodeManager                 |

## 2.配置文件说明

Hadoop 配置文件分两类：默认配置文件和自定义配置文件，只有用户想修改某一默认配置值时，才需要修改自定义配置文件，更改相应属性值。

（1）默认配置文件：

| 要获取的默认文件     | 文件存放在 Hadoop 的 jar 包中的位置                       |
| -------------------- | --------------------------------------------------------- |
| [core-default.xml]   | hadoop-common-3.1.3.jar/core-default.xml                  |
| [hdfs-default.xml]   | hadoop-hdfs-3.1.3.jar/hdfs-default.xml                    |
| [yarn-default.xml]   | hadoop-yarn-common-3.1.3.jar/yarn-default.xml             |
| [mapred-default.xml] | hadoop-mapreduce-client-core-3.1.3.jar/mapred-default.xml |

（2）自定义配置文件：

core-site.xml、hdfs-site.xml、yarn-site.xml、mapred-site.xml 四个配置文件存放在$HADOOP_HOME/etc/hadoop 这个路径上，用户可以根据项目需求重新进行修改配置。

## 3.配置集群

（1）核心配置文件

配置 core-site.xml

```sh
[atguigu@hadoop102 ~]$ cd $HADOOP_HOME/etc/hadoop
[atguigu@hadoop102 hadoop]$ vim core-site.xml
```

文件内容如下：

```sh
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
 <!-- 指定 NameNode 的地址 -->
 <property>
 <name>fs.defaultFS</name>
 <value>hdfs://hadoop102:8020</value>
 </property>
 <!-- 指定 hadoop 数据的存储目录 -->
 <property>
 <name>hadoop.tmp.dir</name>
 <value>/opt/module/hadoop-3.1.3/data</value>
 </property>
 <!-- 配置 HDFS 网页登录使用的静态用户为 atguigu -->
 <property>
 <name>hadoop.http.staticuser.user</name>
 <value>atguigu</value>
 </property>
</configuration>
```

（2）HDFS 配置文件

配置 hdfs-site.xml

```sh
[atguigu@hadoop102 hadoop]$ vim hdfs-site.xml
```

文件内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
<!-- nn web 端访问地址-->
<property>
 <name>dfs.namenode.http-address</name>
 <value>hadoop102:9870</value>
 </property>
<!-- 2nn web 端访问地址-->
 <property>
 <name>dfs.namenode.secondary.http-address</name>
 <value>hadoop104:9868</value>
 </property>
</configuration>
```

（3）YARN 配置文件

配置 yarn-site.xml

```sh
[atguigu@hadoop102 hadoop]$ vim yarn-site.xml
```

文件内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
 <!-- 指定 MR 走 shuffle -->
 <property>
 <name>yarn.nodemanager.aux-services</name>
 <value>mapreduce_shuffle</value>
 </property>
 <!-- 指定 ResourceManager 的地址-->
 <property>
 <name>yarn.resourcemanager.hostname</name>
 <value>hadoop103</value>
 </property>
 <!-- 环境变量的继承 -->
 <property>
 <name>yarn.nodemanager.env-whitelist</name>
 
<value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CO
NF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAP
RED_HOME</value>
 </property>
</configuration>
```

（4）MapReduce 配置文件

配置 mapred-site.xml

```sh
[atguigu@hadoop102 hadoop]$ vim mapred-site.xml
```

文件内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
<!-- 指定 MapReduce 程序运行在 Yarn 上 -->
 <property>
 <name>mapreduce.framework.name</name>
 <value>yarn</value>
 </property>
</configuration>
```

## 4.在集群上分发配置好的 Hadoop 配置文件

```sh
[atguigu@hadoop102 hadoop]$ xsync /opt/module/hadoop-3.1.3/etc/hadoop/
```

## 5.去 103 和 104 上查看文件分发情况

```sh
[atguigu@hadoop103 ~]$ cat /opt/module/hadoop-3.1.3/etc/hadoop/core-site.xml
[atguigu@hadoop104 ~]$ cat /opt/module/hadoop-3.1.3/etc/hadoop/core-site.xml
```

# 5.群起集群

## 1.配置 workers

```sh
[atguigu@hadoop102 hadoop]$ vim /opt/module/hadoop-3.1.3/etc/hadoop/workers
```

在该文件中增加如下内容：

```xml
hadoop102
hadoop103
hadoop104
```

注意：该文件中添加的内容结尾不允许有空格，文件中不允许有空行。

同步所有节点配置文件

```sh
[atguigu@hadoop102 hadoop]$ xsync /opt/module/hadoop-3.1.3/etc
```

## 2.启动集群

（1）如果集群是第一次启动，需要在 hadoop102 节点格式化 NameNode（注意：格式化 NameNode，会产生新的集群 id，导致 NameNode 和 DataNode 的集群 id 不一致，集群找不到已往数据。如果集群在运行过程中报错，需要重新格式化 NameNode 的话，一定要先停止 namenode 和 datanode 进程，并且要删除所有机器的 data 和 logs 目录，然后再进行格式化。）

```sh
[atguigu@hadoop102 hadoop-3.1.3]$ hdfs namenode -format
```

（2）启动 HDFS

```sh
[atguigu@hadoop102 hadoop-3.1.3]$ sbin/start-dfs.sh
```

（3）在配置了 ResourceManager 的节点（hadoop103）启动 YARN

```sh
[atguigu@hadoop103 hadoop-3.1.3]$ sbin/start-yarn.sh
```

（4）Web 端查看 HDFS 的 NameNode

- 浏览器中输入：http://hadoop102:9870
- 查看 HDFS 上存储的数据信息

（5）Web 端查看 YARN 的 ResourceManager

- 浏览器中输入：http://hadoop103:8088
- 查看 YARN 上运行的 Job 信息

## 3.集群基本测试

（1）上传文件到集群

➢ 上传小文件

```sh
[atguigu@hadoop102 ~]$ hadoop fs -mkdir /input
[atguigu@hadoop102 ~]$ hadoop fs -put $HADOOP_HOME/wcinput/word.txt /input
```

➢ 上传大文件

```sh
[atguigu@hadoop102 ~]$ hadoop fs -put /opt/software/jdk-8u212-linux-x64.tar.gz /
```

（2）上传文件后查看文件存放在什么位置

➢ 查看 HDFS 文件存储路径

```sh
[atguigu@hadoop102 subdir0]$ pwd
/opt/module/hadoop-3.1.3/data/dfs/data/current/BP-1436128598-192.168.10.102-1610603650062/current/finalized/subdir0/subdir0
```

➢ 查看 HDFS 在磁盘存储文件内容

```sh
[atguigu@hadoop102 subdir0]$ cat blk_1073741825
hadoop yarn
hadoop mapreduce 
atguigu
atguigu
```

（3）拼接

-rw-rw-r--. 1 atguigu atguigu 134217728 5 月 23 16:01 blk_1073741836

-rw-rw-r--. 1 atguigu atguigu 1048583 5 月 23 16:01 blk_1073741836_1012.meta

-rw-rw-r--. 1 atguigu atguigu 63439959 5 月 23 16:01 blk_1073741837

-rw-rw-r--. 1 atguigu atguigu 495635 5 月 23 16:01 blk_1073741837_1013.meta

```sh
[atguigu@hadoop102 subdir0]$ cat blk_1073741836>>tmp.tar.gz
[atguigu@hadoop102 subdir0]$ cat blk_1073741837>>tmp.tar.gz
[atguigu@hadoop102 subdir0]$ tar -zxvf tmp.tar.gz
```

（4）下载

```sh
[atguigu@hadoop104 software]$ hadoop fs -get /jdk-8u212-linuxx64.tar.gz ./
```

（5）执行 wordcount 程序

```sh
[atguigu@hadoop102 hadoop-3.1.3]$ hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar wordcount /input /output
```

# 6.配置历史服务器

为了查看程序的历史运行情况，需要配置一下历史服务器。具体配置步骤如下：

1.配置 mapred-site.xml

```sh
[atguigu@hadoop102 hadoop]$ vim mapred-site.xml
```

在该文件里面增加如下配置。
```xml
<!-- 历史服务器端地址 -->

<property>

 <name>mapreduce.jobhistory.address</name>

 <value>hadoop102:10020</value>

</property>

<!-- 历史服务器 web 端地址 -->

<property>

 <name>mapreduce.jobhistory.webapp.address</name>

 <value>hadoop102:19888</value>

</property>
```

2）分发配置
```sh
[atguigu@hadoop102 hadoop]$ xsync $HADOOP_HOME/etc/hadoop/mapred-site.xml
```
3）在 hadoop102 启动历史服务器

```sh
[atguigu@hadoop102 hadoop]$ mapred --daemon start historyserver
```
4）查看历史服务器是否启动
```sh
[atguigu@hadoop102 hadoop]$ jps
```

5）查看 JobHistory

http://hadoop102:19888/jobhistory

# 7.配置日志的聚集

日志聚集概念：应用运行完成以后，将程序运行日志信息上传到 HDFS 系统上。

![](./assets/06.png)

日志聚集功能好处：可以方便的查看到程序运行详情，方便开发调试。

注意：开启日志聚集功能，需要重新启动 NodeManager 、ResourceManager 和 HistoryServer。

开启日志聚集功能具体步骤如下：

1.配置 yarn-site.xml

```sh
[atguigu@hadoop102 hadoop]$ vim yarn-site.xml
```

在该文件里面增加如下配置。

```xml
<!-- 开启日志聚集功能 -->
<property>
 <name>yarn.log-aggregation-enable</name>
 <value>true</value>
</property>
<!-- 设置日志聚集服务器地址 -->
<property> 
 <name>yarn.log.server.url</name> 
 <value>http://hadoop102:19888/jobhistory/logs</value>
</property>
<!-- 设置日志保留时间为 7 天 -->
<property>
 <name>yarn.log-aggregation.retain-seconds</name>
 <value>604800</value>
</property>
```

2.分发配置

```xml
[atguigu@hadoop102 hadoop]$ xsync $HADOOP_HOME/etc/hadoop/yarn-site.xml
```
3.关闭 NodeManager 、ResourceManager 和 HistoryServer

```xml
[atguigu@hadoop103 hadoop-3.1.3]$ sbin/stop-yarn.sh
[atguigu@hadoop103 hadoop-3.1.3]$ mapred --daemon stop historyserver
```

4.启动 NodeManager 、ResourceManage 和 HistoryServer

```sh
[atguigu@hadoop103 ~]$ start-yarn.sh
[atguigu@hadoop102 ~]$ mapred --daemon start historyserver
```
5.删除 HDFS 上已经存在的输出文件
```sh
[atguigu@hadoop102 ~]$ hadoop fs -rm -r /output
```
6.执行 WordCount 程序

```sh
[atguigu@hadoop102 hadoop-3.1.3]$ hadoop jar 
share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar 
wordcount /input /output
```
7.查看日志

（1）历史服务器地址

http://hadoop102:19888/jobhistory

（2）历史任务列表

![](./assets/07.png)

（3）查看任务运行日志
![](./assets/08.png)

（4）运行日志详情
![](./assets/09.png)

# 8.集群启动/停止方式总结

1.各个模块分开启动/停止（配置 ssh 是前提）常用
（1）整体启动/停止 HDFS
```sh
start-dfs.sh/stop-dfs.sh
```
（2）整体启动/停止 YARN
```sh
start-yarn.sh/stop-yarn.sh
```
2.各个服务组件逐一启动/停止
（1）分别启动/停止 HDFS 组件
```sh
hdfs --daemon start/stop namenode/datanode/secondarynamenode
```
（2）启动/停止 YARN
```sh
yarn --daemon start/stop resourcemanager/nodemanager
```

# 9.编写 Hadoop 集群常用脚本
1.Hadoop 集群启停脚本（包含 HDFS，Yarn，Historyserver）：myhadoop.sh
```sh
[atguigu@hadoop102 ~]$ cd /home/atguigu/bin
[atguigu@hadoop102 bin]$ vim myhadoop.sh
```
➢ 输入如下内容

```sh
#!/bin/bash
if [ $# -lt 1 ]
then
 echo "No Args Input..."
 exit ;
fi
case $1 in
"start")
 echo " =================== 启动 hadoop 集群 ==================="
 echo " --------------- 启动 hdfs ---------------"
 ssh hadoop102 "/opt/module/hadoop-3.1.3/sbin/start-dfs.sh"
 echo " --------------- 启动 yarn ---------------"
  ssh hadoop103 "/opt/module/hadoop-3.1.3/sbin/start-yarn.sh"
 echo " --------------- 启动 historyserver ---------------"
 ssh hadoop102 "/opt/module/hadoop-3.1.3/bin/mapred --daemon start 
historyserver"
;;
"stop")
 echo " =================== 关闭 hadoop 集群 ==================="
 echo " --------------- 关闭 historyserver ---------------"
 ssh hadoop102 "/opt/module/hadoop-3.1.3/bin/mapred --daemon stop 
historyserver"
 echo " --------------- 关闭 yarn ---------------"
 ssh hadoop103 "/opt/module/hadoop-3.1.3/sbin/stop-yarn.sh"
 echo " --------------- 关闭 hdfs ---------------"
 ssh hadoop102 "/opt/module/hadoop-3.1.3/sbin/stop-dfs.sh"
;;
*)
 echo "Input Args Error..."
;;
esac
```
➢ 保存后退出，然后赋予脚本执行权限

```sh
[atguigu@hadoop102 bin]$ chmod +x myhadoop.sh
```
2.查看三台服务器 Java 进程脚本：jpsall
```sh
[atguigu@hadoop102 ~]$ cd /home/atguigu/bin
[atguigu@hadoop102 bin]$ vim jpsall
```
➢ 输入如下内容
```shell
#!/bin/bash
for host in hadoop102 hadoop103 hadoop104
do
 echo =============== $host ===============
 ssh $host jps 
done
```
➢ 保存后退出，然后赋予脚本执行权限
```sh
[atguigu@hadoop102 bin]$ chmod +x jpsall
```
3.分发/home/atguigu/bin 目录，保证自定义脚本在三台机器上都可以使用
```sh
[atguigu@hadoop102 ~]$ xsync /home/atguigu/bin/
```
# 10.常用端口号说明

| 端口名称                   | Hadoop2.x   | Hadoop3.x        |
| -------------------------- | ----------- | ---------------- |
| NameNode 内部通信端口      | 8020 / 9000 | 8020 / 9000/9820 |
| NameNode HTTP UI           | 50070       | 9870             |
| MapReduce 查看执行任务端口 | 8088        | 8088             |
| 历史服务器通信端口         | 19888       | 19888            |

# 11.集群时间同步
​	如果服务器在公网环境（能连接外网），可以不采用集群时间同步，因为服务器会定期
和公网时间进行校准；
​	如果服务器在内网环境，必须要配置集群时间同步，否则时间久了，会产生时间偏差，
导致集群执行任务时间不同步。

1）需求
找一个机器，作为时间服务器，所有的机器与这台集群时间进行定时的同步，生产环境
根据任务对时间的准确程度要求周期同步。测试环境为了尽快看到效果，采用 1 分钟同步一
次。
![](./assets/10.png)
2）时间服务器配置（必须 root 用户）
（1）查看所有节点 ntpd 服务状态和开机自启动状态
```sh
[atguigu@hadoop102 ~]$ sudo systemctl status ntpd
[atguigu@hadoop102 ~]$ sudo systemctl start ntpd
[atguigu@hadoop102 ~]$ sudo systemctl is-enabled ntpd
```
（2）修改 hadoop102 的 ntp.conf 配置文件
```sh
[atguigu@hadoop102 ~]$ sudo vim /etc/ntp.conf
```
修改内容如下
（a）修改 1（授权 192.168.10.0-192.168.10.255 网段上的所有机器可以从这台机器上查
询和同步时间）
```sh
#restrict 192.168.10.0 mask 255.255.255.0 nomodify notrap
为 restrict 192.168.10.0 mask 255.255.255.0 nomodify notrap
```
（b）修改 2（集群在局域网中，不使用其他互联网上的时间）
```sh
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst
```
为
```sh
#server 0.centos.pool.ntp.org iburst
#server 1.centos.pool.ntp.org iburst
#server 2.centos.pool.ntp.org iburst
#server 3.centos.pool.ntp.org iburst
```
（c）添加 3（当该节点丢失网络连接，依然可以采用本地时间作为时间服务器为集群中
的其他节点提供时间同步）
```sh
server 127.127.1.0
fudge 127.127.1.0 stratum 10
```
（3）修改 hadoop102 的/etc/sysconfig/ntpd 文件
```sh
[atguigu@hadoop102 ~]$ sudo vim /etc/sysconfig/ntpd
```
增加内容如下（让硬件时间与系统时间一起同步）
```sh
SYNC_HWCLOCK=yes
```
（4）重新启动 ntpd 服务
```sh
[atguigu@hadoop102 ~]$ sudo systemctl start ntpd
```
（5）设置 ntpd 服务开机启动
```sh
[atguigu@hadoop102 ~]$ sudo systemctl enable ntpd
```
3）其他机器配置（必须 root 用户）
（1）关闭所有节点上 ntp 服务和自启动

```sh
[atguigu@hadoop103 ~]$ sudo systemctl stop ntpd
[atguigu@hadoop103 ~]$ sudo systemctl disable ntpd
[atguigu@hadoop104 ~]$ sudo systemctl stop ntpd
[atguigu@hadoop104 ~]$ sudo systemctl disable ntpd
```
（2）在其他机器配置 1 分钟与时间服务器同步一次
```sh
[atguigu@hadoop103 ~]$ sudo crontab -e
```
编写定时任务如下：
```sh
*/1 * * * * /usr/sbin/ntpdate hadoop102
```
（3）修改任意机器时间
```sh
[atguigu@hadoop103 ~]$ sudo date -s "2021-9-11 11:11:11"
```
（4）1 分钟后查看机器是否与时间服务器同步
```sh
[atguigu@hadoop103 ~]$ sudo date
```

