# kafka 命令
kafka启动命令:
```
命令一：
nohup  ./bin/kafka-server-start.sh config/server.properties 1>./nohup.out 2>&1 &

命令二：
./bin/kafka-server-start.sh -daemon config/server.properties

```

常用的几个命令如下:
kafka-server-start.sh
kafka-console-consumer.sh
kafka-console-producer.sh
kafka-topics.sh

在这几个命令中，第一个仅用于启动Kafka，后两个console常用于测试，用途最多的是最后一个命令，所以下面命令中主要介绍的就是 kafka-topics.sh。


1. 查看topic列表

```
./bin/kafka-topics.sh  --list --zookeeper 10.39.251.161:2181,10.39.251.162:2181,10.39.251.163:2181

```

2. 创建topic

```
./bin/kafka-topics.sh --create --zookeeper 10.39.251.161:2181,10.39.251.162:2181,10.39.251.163:2181  --replication-factor 3 --partitions 3 --topic lockerstatus

```

3. 查看topic详细信息

```
./bin/kafka-topics.sh --describe --zookeeper 10.39.251.161:2181,10.39.251.162:2181,10.39.251.163:2181 --topic elecoperation
```

4. 生产者操作

```

./bin/kafka-console-producer.sh --broker-list 10.39.251.161:9092,10.39.251.162:9092,10.39.251.163:9092 --topic waterdata

```

5. 消费者操作

```
./bin/kafka-console-consumer.sh --zookeeper 10.39.251.161:2181,10.39.251.162:2181,10.39.251.163:2181 --topic  lockeroperation --consumer-property group.id=group_test --from-beginning

```

6. 删除topic

```
需要设置 auto.create.topics.enable = false
server.properties 设置 delete.topic.enable=true
./bin/kafka-topics.sh --delete --zookeeper 10.39.251.161:2181,10.39.251.162:2181,10.39.251.163:2181 --topic lockerstatus

删除kafka存储目录
登录到zk shell 执行rmr /brokers/topics/【topic name】
```

# kafka排错
server.log日志报如下错误
```

[2019-03-13 17:59:59,756] WARN [ReplicaFetcherThread-0-2], Error in fetch kafka.server.ReplicaFetcherThread$FetchRequest@62bb6b1e (kafka.server.ReplicaFetcherThread)
java.io.IOException: Connection to els3:9092 (id: 2 rack: null) failed
	at kafka.utils.NetworkClientBlockingOps$.awaitReady$1(NetworkClientBlockingOps.scala:84)
	at kafka.utils.NetworkClientBlockingOps$.blockingReady$extension(NetworkClientBlockingOps.scala:94)
	at kafka.server.ReplicaFetcherThread.sendRequest(ReplicaFetcherThread.scala:244)
	at kafka.server.ReplicaFetcherThread.fetch(ReplicaFetcherThread.scala:234)
	at kafka.server.ReplicaFetcherThread.fetch(ReplicaFetcherThread.scala:42)
	at kafka.server.AbstractFetcherThread.processFetchRequest(AbstractFetcherThread.scala:118)
	at kafka.server.AbstractFetcherThread.doWork(AbstractFetcherThread.scala:103)
	at kafka.utils.ShutdownableThread.run(ShutdownableThread.scala:63)

```
这是由于连不上els3:9092,而els3是一个hostname，这个hostname网络上是连不通的。
修改kafka配置文件server.properties,将kafka broker的监听地址改为ip而不用hostname。

```
listeners=PLAINTEXT://10.39.251.161:9092
```
