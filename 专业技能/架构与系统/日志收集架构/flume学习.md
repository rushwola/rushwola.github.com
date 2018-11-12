# flume

官方网站：http://flume.apache.org/

## flume启动

非后台启动：

```
./bin/flume-ng agent  -c ./conf  -f ./conf/flume-conf.properties -Dflume.root.logger=DEBUG,console -n
 agent

```


后台启动：

```
nohup bin/flume-ng agent --conf ./conf/ -f conf/flume-conf.properties -Dflume.root.logger=DEBUG,console -n agent >>333.out &

```

示例一：

```

# example.conf: A single-node Flume configuration

# 定义agent名字为a1，它的sources名叫r1，sinks名叫k1,channels名叫c1。
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# 配置sources
a1.sources.r1.type = netcat
a1.sources.r1.bind = localhost
a1.sources.r1.port = 44444

# 配置 sink
a1.sinks.k1.type = logger

# 配置channels
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# 将sources、sinks与channel绑定。
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1

```
