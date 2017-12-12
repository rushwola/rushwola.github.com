# 书籍

Spark快速大数据分析
 图解Spark：核心技术与案例实战
 Hadoop + Spark 大数据巨量分析与机器学习整合开发实战

 https://www.cnblogs.com/donaldlee2008/p/5836146.html

# spark 单机部署

１、安装scala

```
wget http://www.scala-lang.org/files/archive/scala-2.10.1.tgz
tar zxf scala-2.10.1.tgz
sudo mv scala-2.10.1 /usr/local

```

在/etc/profile.d/下创建文件scala.sh，内容如下

```
#!usr/bin/env bash
export SCALA_HOME=/usr/local/scala-2.10.1
export PATH=$PATH:$SCALA_HOME/bin

```

使环境变量生效

```

cd /etc
source profile

```

2、安装SBT(scala构建工具)

下载sbt 0.13.16 ，由于sbt的官网上下不了，找到了csdn上面有它的资源，下载完毕后再上传到服务器上。

配置PATH环境变量
之后在/etc/profile文件中添加
export PATH=$PATH:/application/sbt/sbt/bin/

生效：
source /etc/profile

验证：
```
[root@hidden sbt]# sbt sbt-version
[info] Set current project to sbt (in build file:/root/util/sbt/)
[info] 0.13.13

```

3、安装spark

```
wget https://archive.apache.org/dist/spark/spark-1.6.1/spark-1.6.1-bin-hadoop2.6.tgz

http://www-eu.apache.org/dist/spark/spark-1.6.3/spark-1.6.3-bin-hadoop2.6.tgz

tar zxf spark-1.6.1-bin-hadoop2.6.tgz

```

启动SPARK

```

./sbin/start-master.sh

```

登录localhost:8080可以看到UI界面，上面会显示当前节点的信息
其中在标题处会显示当前master所在的URL信息，这个参数需要作为入参在创建worker时候使用(spark://IP:PORT)

启动一个worker
```

./bin/spark-class org.apache.spark.deploy.worker.Worker spark://IP:PORT

```

验证：


此时localhost:8080的页面可以看到worker的信息

# spark 集群部署

集群安装条件是要：安装了java、安装了scala。

本次安富有版本：
jdk :1.8
scala-2.12.4
spark-1.6.3-bin-hadoop2.6

##　修改主机名和hosts

```
vi /etc/host
10.39.251.136 master136
10.39.251.101 node101
```

##　配置集群免密码登录
添加用户
```
useradd spark
passwd spark
```
修改ssh配置,
```
vi /etc/ssh/sshd_config
修改以下内容
RSAAuthentication yes                           #启用RSA
PubkeyAuthentication yes                        #启用公钥私钥配对认证方式
AuthorizedKeysFile      %h/.ssh/authorized_keys #公钥文件路径

```
重启ssh

```

systemctl restart sshd.service

```

首先切换到spark用户

集群所有节点互相免密（集群中任意服务器都有其他服务器公钥【锁】）
置所有各自服务器本身公钥和免密

```
ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys

```
现在每台节点本身实现免密
将除master服务器之外所有节点的公钥复制到master上，这里是将node101的公钥拷贝到master的authorized_keys，目的是实现node101登录A实现免密，而反过来不行

```
ssh-copy-id -i ~/.ssh/id_dsa.pub master136

```
运行 ssh master136　如果还要输入密码则执行以下命令：
```
chmod 700 ~/.ssh/
chmod 700 /home/spark
chmod 600 ~/.ssh/authorized_keys
```

## 对master进行相应的配置

修改环境变量文件 .bashrc , 添加以下内容
```
# Spark Env
export SPARK_HOME=/home/spark/spark-1.6.3-bin-hadoop2.6
export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin

```
source .bashrc

进入 Spark 安装目录下的 conf 目录， 拷贝 spark-env.sh.template 到 spark-env.sh。

编辑 spark-env.sh，在其中添加以下配置信息：
```
export SCALA_HOME=/opt/scala-2.11.8
export JAVA_HOME=/opt/java/jdk1.7.0_80
export SPARK_MASTER_IP=192.168.109.137
export SPARK_WORKER_MEMORY=1g
export HADOOP_CONF_DIR=/opt/hadoop-2.6.4/etc/hadoop

```
JAVA_HOME 指定 Java 安装目录；
SCALA_HOME 指定 Scala 安装目录；
SPARK_MASTER_IP 指定 Spark 集群 Master 节点的 IP 地址；
SPARK_WORKER_MEMORY 指定的是 Worker 节点能够分配给 Executors 的最大内存大小；
HADOOP_CONF_DIR 指定 Hadoop 集群配置文件目录。
将 slaves.template 拷贝到 slaves， 编辑其内容为：
```
master136
node101

```
即 master136 既是 Master 节点又是 Worker 节点。

slave机器参照master机器配置即可。

# 解决master高可用问题

http://blog.csdn.net/zhongguozhichuang/article/details/56667137

# eclipse spark环境

见：
http://www.linuxidc.com/Linux/2015-08/120946.htm
http://dongxicheng.org/framework-on-yarn/spark-eclipse-ide/

http://archive.apache.org/dist/hadoop/core/hadoop-2.6.0/
第一步安装 :jdk

第二步：安装Scala

http://www.scala-lang.org/files/archive/scala-2.9.3.msi

下载完成后解压，将安装目录的bin目录增加到PATH环境变量中。


第三步： 配置Spark环境变量


# 本地模式运行spark

写了很简单的一段spark代码，将结果保存为windows本地文件，执行之后总是报错NullPointerException
查询之后 发现是本地缺少hadoop需要的一个文件所致
如果本地已经安装了hadoop 一般不会有此问题 如果不愿安装 可按照下述方法解决
1）下载需要的文件 winutils.exe
http://social.msdn.microsoft.com/Forums/windowsazure/en-US/28a57efb-082b-424b-8d9e-731b1fe135de/please-read-if-experiencing-job-failures?forum=hdinsight
2) 将此文件放置在某个目录下，比如C:\winutils\bin\中。
3）在程序的一开始声明：System.setProperty("hadoop.home.dir", "c:\\winutil\\")
