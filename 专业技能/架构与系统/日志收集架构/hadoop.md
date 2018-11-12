#一些资料
Hadoop权威指南、数据算法Hadoop Spark大数据处理技巧、Spark高级数据分析。hadoop应用架构。

https://zhuanlan.zhihu.com/p/24231891

https://www.iteblog.com/archives/851.html

http://www.iteye.com/blogs/subjects/zy19982004

hive :
http://blog.csdn.net/hguisu/article/details/7256833

impala:
http://blog.csdn.net/lijixianglijixiang/article/details/72722335

# 软件版本

Apache Hadoop的2.7及更高版本需要Java 7.它是在OpenJDK和Oracle（HotSpot）的JDK / JRE 上构建和测试的。早期版本（2.6及更早版本）支持Java 6。

java:jdk1.7。
haddop:Hadoop 2.6.0-cdh5.12.0
CDH-5.12.0

关于haddop版本：　

CDH：全称Cloudera’s Distribution Including Apache Hadoop
　　CDH版本衍化
　　hadoop是一个开源项目，所以很多公司在这个基础进行商业化，Cloudera对hadoop做了相应的改变。
　　Cloudera公司的发行版，我们将该版本称为CDH(Cloudera Distribution
Hadoop)。截至目前为止，CDH共有5个版本，其中，前两个已经不再更新，最近的两个，分别是CDH4，在Apache Hadoop
2.0.0版本基础上演化而来的，CDH5，它们每隔一段时间便会更新一次。

信息：http://dongxicheng.org/mapreduce-nextgen/how-to-select-hadoop-versions/

版本下载地址：http://archive.cloudera.com/cdh5/cdh/5/

# 安装hadoop伪分布式部署


下载：

```
wget http://archive.cloudera.com/cdh5/cdh/5/hadoop-2.6.0-cdh5.12.0.tar.gz

```


* 新建hadoop目录，并创建source文件夹用来存放相关的软件。
```
mkdir /opt/hadoop
mkdir /opt/hadoop/source

```

```
# 解压安装
tar -xzvf  hadoop-2.6.0-cdh5.12.0.tar.gz
#将解压后的文件夹移动到 hadoop目录下
mv hadoop-2.7.3 /opt/hadoop

```

# 创建hadoop用户
```
sudo useradd -m hadoop -s /bin/bash
sudo passwd hadoop


```
可为 hadoop 用户增加管理员权限，方便部署，避免一些对新手来说比较棘手的权限问题：
```
sudo adduser hadoop sudo

```

* 配置无密码登录
利用 ssh-keygen 生成密钥，并将密钥加入到授权中：
```
cd ~/.ssh/                   # 若没有该目录，请先执行一次ssh localhost
ssh-keygen -t rsa              # 会有提示，都按回车就可以
cat ./id_rsa.pub >> ./authorized_keys  # 加入授权

chmod 700 ~/.ssh                       # 目录授权
chmod 600 ~/.ssh/authorized_keys       # 目录授权
```


* 配置环境变量

vim /etc/profile
```
#添加如下内容

#HADOOP
export HADOOP_HOME=/home/hadoop/hadoop-2.6.0-cdh5.12.0
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
# hadoop data direcotory
export HADOOP_DATA_DIR=/home/hadoop/mnt/hdfs
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native

#更改文件所有者
chown hadoop $HADOOP_HOME

```

* 建立数据存储目录namenode和datanode

环境变量HADOOP_DATA_DIR=/opt/hadoop/mnt/hdfs
更改权限：
```

#建立目录
mkdir $HADOOP_DATA_DIR/namenode
mkdir $HADOOP_DATA_DIR/datanode

#更改文件所有者
chown hadoop /home/hadoop/mnt/hdfs/namenode
chown hadoop /home/hadoop/mnt/hdfs/datanode


```

* 修改hadoop配置文件

进入hadoop的etc/hadoop目录

对要修改的文件进行备份
```
cp core-site.xml core-site.xml.bak
cp hdfs-site.xml hdfs-site.xml.bak
cp yarn-site.xml yarn-site.xml.bak
cp hadoop-env.sh hadoop-env.sh.bak
cp mapred-site.xml.template mapred-site.xml

```

修改hadoop-env.sh:
```
# 将变量改成常量即路径！！！不然会找不到JAVA_HOME
export JAVA_HOME=$JAVA_HOME //默认
export JAVA_HOME=/usr/local/jdk1.7.0_80 //改成java安装路径

```

修改core-site.xml
在<configuration></configuration>增加hdfs的端口信息
增加的内容如下：
```
<property>
  <name>fs.default.name</name>
  <value>hdfs://localhost:9000</value>
</property>

<property>
<!-- 设置每个节点临时文件目录 -->
  <name>hadoop.tmp.dir</name>
<!-- 当前用户须要对此目录有读写权限，启动集群时自动创建 -->
  <value>/home/hadoop/tmp</value> 　　
</property>
```

修改hdfs-site.xml
在<configuration></configuration>增加以下内容：
```
<property>
<!-- 设置数据的备份数为1-->
 <name>dfs.replication</name>
 <value>1</value>
</property>

<property>
 <name>dfs.namenode.dir</name>
 <value>file:/home/hadoop/mnt/hdfs/namenode</value>
</property>

<property>
 <name>dfs.datanode.dir</name>
 <value>file:/home/hadoop/mnt/hdfs/datanode</value>
</property>


```

修改mapred-site.xml文件:
```
<property>
 <name>mapreduce.framework.name</name>
 <value>yarn</value>
</property>

```

修改yarn-site.xml文件
在<configuration></configuration>增加以下内容：
```
<property>
 <name>yarn.nodemanager.aux-services</name>
 <value>mapreduce_shuffle</value>
</property>

<property>
 <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>           
 <value>org.apache.hadoop.mapred.ShuffleHandler</value>
</property>


```

# 启动

* 配置完成后，执行 NameNode 的格式化:
```
./bin/hdfs namenode -format

```


检验是否成空：Exitting with status 0

* 接着开启 NameNode 和 DataNode 守护进程。
```
./sbin/start-dfs.sh

```
验证：通过jps命令可以看到以下进程：
DataNode, NameNode， SecondaryNameNode


* 然后就可以启动 YARN 了
```
./sbin/start-yarn.sh      # 启动YARN
./sbin/mr-jobhistory-daemon.sh start historyserver  # 开启历史服务器，才能在Web中查看任务运行情况

```

当启动了启动YARN时会多出ResourceManager、NodeManager进程。
当启动了  mr-jobhistory-daemon.sh  会多出JobHistoryServer进程。


# 其它的一些帮助指令

测试集群工作状态的一些指令 ：  
bin/hdfs dfsadmin -report 查看hdfs的各节点状态信息  
bin/hdfs haadmin -getServiceState nn1 获取一个namenode节点的HA状态  
sbin/hadoop-daemon.sh start namenode  单独启动一个namenode进程  
./hadoop-daemon.sh start zkfc   单独启动一个zkfc进程  

# fdhs简单命令

```
./bin/hdfs dfs -mkdir -p /user/hadoop

```


#cdh安装

版本：CDH-5.12.0,cloudera-manager-5.12.0

网页：
https://www.cnblogs.com/zhangyin6985/p/7357738.html
http://www.cnblogs.com/zlslch/
http://blog.csdn.net/tx_smile/article/details/78338110
https://www.cnblogs.com/CaptainLin/p/7089766.html
https://www.cnblogs.com/zlslch/p/6785207.html?utm_source=itdadao&utm_medium=referral
下载地址
http://archive-primary.cloudera.com/cdh5/parcels/5.12.0/

superset 连impala数据源,hive-cli-1.1.0
impala呢？，我们想在开环境也装一下，以后也会有离线任务，方便测试
impala要装cdh
CDH-5.12.0
然后hive、impala、都是装在cdh上

软件下载：
cdh:
http://archive.cloudera.com/cdh5/parcels/

要下载三个文件：
CDH-5.12.0-1.cdh5.12.0.p0.29-el7.parcel  CDH-5.12.0-1.cdh5.12.0.p0.29-el7.parcel.sha1  
manifest.json

cloudera-manager:
http://archive.cloudera.com/cm5/cm/5/


下载文件：
cloudera-manager-centos7-cm5.12.0_x86_64.tar.gz
