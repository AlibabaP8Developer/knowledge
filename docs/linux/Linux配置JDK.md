1.新建目录

建议目录：/home/software/java 【根据情况而定】

上传文件夹到此目录并解压到当前目录

```java
tar -zxvf jdk-8u60-linux-x64.tar.gz
```

2.编辑配置文件，配置环境变量

```java
vim /etc/profile
添加如下内容：JAVA_HOME根据实际目录来

JAVA_HOME=/usr/java/jdk1.8.0_60
CLASSPATH=$JAVA_HOME/lib/
PATH=$PATH:$JAVA_HOME/bin
export PATH JAVA_HOME CLASSPATH
```

3.执行命令

```java
source /etc/profile
```

4.查看安装情况

```java
java -version
```

 



