#官方文档
https://docs.imply.io/on-premise/quickstart

http://lxw1234.com/archives/2015/11/554.htm 海量数据实时OLAP分析系统-Druid.io安装配置和体验
http://druid.io/docs/0.9.2/design/design.html Druid官网搭建
http://blog.csdn.net/u010003835/article/details/52947743
# 概述
Druid.io（以下简称Druid）是面向海量数据的、用于实时查询与分析的OLAP存储系统。Druid的四大关键特性总结如下：

1.亚秒级的OLAP查询分析。Druid采用了列式存储、倒排索引、位图索引等关键技术，能够在亚秒级别内完成海量数据的过滤、聚合以及多维分析等操作

2.实时流数据分析。区别于传统分析型数据库采用的批量导入数据进行分析的方式，Druid提供了实时流数据分析，采用LSM(Long structure merge)-Tree结构使Druid拥有极高的实时写入性能；同时实现了实时数据在亚秒级内的可视化。

3.丰富的数据分析功能。针对不同用户群体，Druid提供了友好的可视化界面、类SQL查询语言以及REST 查询接口。

4.高可用性与高可拓展性。Druid采用分布式、SN(share-nothing)架构，管理类节点可配置HA，工作节点功能单一，不相互依赖，这些特性都使得Druid集群在管理、容错、灾备、扩容等方面变得十分简单。

# 为什么会有Druid
大数据技术从最早的Hadoop项目开始已经有十多年的历史了，而Druid是在2013年年底才开源的，虽然目前还不是Apache顶级项目，但是作为后起之秀，依然吸引了大量用户的目光，社区也非常活跃。那么，为什么会有Druid，而Druid又解决了传统大数据处理框架下的哪些“痛点”问题，下面我们来一一解答。

大数据时代，如何从海量数据中提取有价值的信息，是一个亟待解决的难题。针对这个问题，IT巨头们已经开发了大量的数据存储与分析类产品，比如IBM Netezza、HP Vertica、EMC GreenPlum等，但是他们大多是昂贵的商业付费类产品，业内使用者寥寥。

而受益于近年来高涨的开源精神，业内出现了众多优秀的开源项目，其中最有名的当属Apache Hadoop生态圈。时至今日，Hadoop已经成为了大数据的“标准”解决方案，但是，人们在享受Hadoop便捷数据分析的同时，也必须要忍受Hadoop在设计上的许多“痛点”，下面就罗列三方面的问题：

何时能进行数据查询？对于Hadoop使用的Map/Reduce批处理框架，数据何时能够查询没有性能保证。

随机IO问题。Map/Reduce批处理框架所处理的数据需要存储在HDFS上，而HDFS是一个以集群硬盘作为存储资源池的分布式文件系统，那么在海量数据的处理过程中，必然会引起大量的读写操作，此时随机IO就成为了高并发场景下的性能瓶颈。

数据可视化问题。HDFS是一个优秀的分布式文件系统，但是对于数据分析以及数据的即席查询，HDFS并不是最优的选择。

传统的大数据处理架构Hadoop更倾向于一种“后台批处理的数据仓库系统”，其作为海量历史数据保存、冷数据分析，确实是一个优秀的通用解决方案，但是如何保证高并发环境下海量数据的查询分析性能，以及如何实现海量实时数据的查询分析与可视化，Hadoop确实显得有些无能为力。

# 安装
https://imply.io/
到官网下载安装包。

```
imply-2.3.9.tar

```

```
tar -xzf imply-2.3.9.tar.gz
cd imply-2.3.9

```

在这个包中，你会发现：
bin/* - 为包含的软件运行脚本。
conf/* - 群集设置的模板配置。
conf-quickstart/* - 这个快速入门的配置。
dist/* - 所有包括的软件。
quickstart/* - 这个快速入门有用的文件。

# 启动服务

```
bin/supervise -c conf/supervise/quickstart.conf

```

发现报错:
```
Please install nodejs version 4.5.0 or better!

```

所以要安装node 4.5.0或以上版本

安装nodejs

```
wget https://nodejs.org/dist/v8.9.3/node-v8.9.3-linux-x64.tar.xz
tar -xvf node-v8.9.3-linux-x64.tar.xz

vi /etc/profile

在最后一行添加（设置环境变量）
export NODE_HOME=/mnt/software/node-v4.5.0-linux-x64
export PATH=$PATH:$NODE_HOME/bin
export NODE_PATH=$NODE_HOME/lib/node_modules


source /etc/profile
```


grep "8081"  ./bin/
