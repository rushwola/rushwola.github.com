# 官网
https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html
https://www.elastic.co/downloads/past-releases
# Elasticsearch 安装(单节点)
下载相应的版本包。

运用以下命令：tar -zxvf elasticsearch-2.4.1.tar.gz  解压。
创建用户：
sudo useradd elsearch  

目录文件授权：
chown -R elsearch:elsearch /usr/local/elasticsearch-2.4.1

启动:
su elsearch
./bin/elasticsearch
后台启动加上 -d 参数即可。

# 验证

验证 本机通过 curl localhost:9200 来验证

```
[root@admin ~]# curl localhost:9200  
{  
  "name" : "Jonas Harrow",  
  "cluster_name" : "elasticsearch",  
  "cluster_uuid" : "ygyOJnXmRjWhYjXjRqYl_g",  
  "version" : {  
    "number" : "2.4.1",  
    "build_hash" : "c67dc32e24162035d18d6fe1e952c4cbcbe79d16",  
    "build_timestamp" : "2016-09-27T18:57:55Z",  
    "build_snapshot" : false,  
    "lucene_version" : "5.5.2"  
  },  
  "tagline" : "You Know, for Search"  
}  

```
如果想其他机器也能访问 http://ip:9200  的地址，那么需要修改es目录下 config 中的配置文件：

```

[elsearch@admin elasticsearch-2.4.1]# vi config/elasticsearch.yml

```

其中将 network.host 这行注释打开，ip改为 0.0.0.0 即可。端口可以不管。然后其他机器就可以通过http访问9200端口了

```
# ---------------------------------- Network -----------------------------------  
#  
# Set the bind address to a specific IP (IPv4 or IPv6):  
#  
network.host: 0.0.0.0  
#  
# Set a custom port for HTTP:  
#  
# http.port: 9200  
#  
# For more information, see the documentation at:  
# <http://www.elastic.co/guide/en/elasticsearch/reference/current/modules-network.html>  
#  

```

# 重启
没有重启的办法，只有先kill再启动的方法。

1.查找进程命令 ps -ef | grep elastic

2.杀掉进程 kill -9 2382（进程号）

3.重启命令 sh elasticsearch -d

# 概念名词

Elasticsearch底层是基于Lucene的，Lucene是一款优秀的搜索lib。
## Lucene关键概念：
Document：用来索引和搜索的主要数据源，包含一个或者多个Field，而这些Field则包含我们跟Lucene交互的数据。
Field：Document的一个组成部分，有两个部分组成，name和value。

Term：不可分割的单词，搜索最小单。
Token：一个Term呈现方式，包含这个Term的内容，在文档中的起始位置，以及类型。

## Elasticsearch关键概念：
Index：Elasticsearch用来存储数据的逻辑区域，它类似于关系型数据库中的db概念。一个index可以在一个或者多个shard上面，同时一个shard也可能会有多个replicas。

Document：Elasticsearch里面存储的实体数据，类似于关系数据中一个table里面的一行数据。
document由多个field组成，不同的document里面同名的field一定具有相同的类型。document里面field可以重复出现，也就是一个field会有多个值，即multivalued。

Document type：为了查询需要，一个index可能会有多种document，也就是document type，但需要注意，不同document里面同名的field一定要是相同类型的。

Mapping：存储field的相关映射信息，不同document type会有不同的mapping。

对于熟悉MySQL的童鞋，我们只需要大概认为Index就是一个db，document就是一行数据，field就是table的column，mapping就是table的定义，而document type就是一个table就可以了。

Document type这个概念其实最开始也把笔者给弄糊涂了，其实它就是为了更好的查询，举个简单的例子，一个index，可能一部分数据我们想使用一种查询方式，而另一部分数据我们想使用另一种查询方式，于是就有了两种type了。不过这种情况应该在我们的项目中不会出现，所以通常一个index下面仅会有一个type。

在服务层面，主要有：
Node: 一个server实例。
Cluster：多个node组成cluster。
Shard：数据分片，一个index可能会存在于多个shards，不同shards可能在不同nodes。
Replica：shard的备份，有一个primary shard，其余的叫做replica shards。
Elasticsearch之所以能动态resharding，主要在于它最开始就预先分配了多个shards（貌似是1024），然后以shard为单位进行数据迁移。这个做法其实在分布式领域非常的普遍，codis就是使用了1024个slot来进行数据迁移。

因为任意一个index都可配置多个replica，通过冗余备份的方式保证了数据的安全性，同时replica也能分担读压力，类似于MySQL中的slave。




# 参考资料

https://es.xiaoleilu.com/
https://www.zhihu.com/question/55686136
http://www.cnblogs.com/huangfox/p/3541300.html
http://blog.csdn.net/laoyang360/article/details/52244917
