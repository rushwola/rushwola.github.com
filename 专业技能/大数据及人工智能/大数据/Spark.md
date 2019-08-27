spark2优化了Spark Core和Spark SQL,对spark sql 和dataframe中的算子性能进行了优化
提升了Parquet文件的扫描
提升ORC文件的读写性能
代码层级：
统一了DataFrame与Dataset ,只保留的dataset
flatMapToPair由之前的return list变成return iterator
kafka只提供direct方式
JavaStreamingContextFactory被废弃掉


根据类查找jar包地址:
System.out.println("^^MessageAndMetadata^^^^"+kafka.message.MessageAndMetadata.class.getProtectionDomain().getCodeSource());

错误：
spark　集群有三个节点，只要在node3上运行任务，触发从hive上读数据到driver时,比如collect算子，就会卡住，并且报：
19/04/16 13:38:30 WARN cluster.YarnScheduler: Initial job has not accepted any resources; check your cluster UI to ensure that workers are registered and have sufficient resources

这是因为job没有申请到资源，有可能是内存资源，
找到nodemanager内存参数设置

```
yarn.nodemanager.resource.memory-mb=4G

```
