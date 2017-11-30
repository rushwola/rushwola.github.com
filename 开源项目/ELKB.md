# 官网

https://www.elastic.co

堆荐资料：ELK+stack权威指南
# ELKB 组件

Elastic Search: 从名称可以看出，Elastic Search 是用来进行搜索的，提供数据以及相应的配置信息（什么字段是什么数据类型，哪些字段可以检索等），然后你就可以自由地使用API搜索你的数据。

Logstash：日志文件基本上都是每行一条，每一条里面有各种信息，这个软件的功能是将每条日志解析为各个字段。

Kibana：提供一套Web界面用来和 Elastic Search 进行交互，这样我们不用使用API来检索数据了，可以直接在 Kibana 中输入关键字，Kibana 会将返回的数据呈现给我们，当然，有很多漂亮的数据可视化图表可供选择。

Beats：安装在每台需要收集日志的服务器上，将日志发送给Logstash进行处理，所以Beats是一个“搬运工”，将你的日志搬运到日志收集服务器上


# Beats 安装
中文资料:https://www.cnblogs.com/kerwinC/p/6227768.html
filebeat是一个本地文件的日志发射器。被当作一个agent安装在服务器上。Filebeat 监控日志目录或具体的日志文件，文件末尾，并转发到Elasticsearch或Logstash索引。
##　下载tar  包并安装依赖包
下载：filebeat-5.6.4-linux-x86_64.tar.gz安装包。
版本5.4.1
安装依赖包
```

sudo yum install libpcap

```

## 配置文件

```

vim filebeat.yml

```

默认监控日志配置:

```

filebeat.prospectors:

- input_type: log
  paths:
    - /home/appdev/soft/nginx/log/access.log
  document_type: nginx_access

- input_type: log
  paths:
    - /home/appdev/elkb/logdemo/logs/tracer.log
  document_type: tracer

- input_type: log
  paths:
    - /home/appdev/elkb/logdemo/logs/test02.log
  document_type: test02

```
按照要求修改为:

```

filebeat:
  prospectors:
  -
      paths:
        - /www/wwwLog/www.lanmps.com_old/*.log
        - /www/wwwLog/www.lanmps.com/*.log
      input_type: log
      document_type: nginx-access-www.lanmps.com
 -
      paths:
        - /www/wwwRUNTIME/www.lanmps.com/order/*.log
      input_type: log
      document_type: order-www.lanmps.com
  -
      paths:
        - /www/wwwRUNTIME/www.lanmps.com/pay/*.log
      input_type: log
      document_type: pay-www.lanmps.com
output:
  #output.elasticsearch:
  #   hosts: ["localhost:9200"]
   output.logstash:
    hosts: ["10.1.5.65:5044"]


```
## 启动停止

启动

```

nohup ./filebeat -e -c filebeat.yml >/dev/null 2>&1 &

```

停止

```

ps -ef |grep filebeat

```


# logstash

http://blog.csdn.net/bingtang5/article/details/53206020
配置文件：

其实它就是一个收集器而已，我们需要为它指定Input和Output（当然Input和Output可以为多个）。由于我们需要把Java代码中Log4j的日志输出到ElasticSearch中，因此这里的Input就是Log4j，而Output就是ElasticSearch。
编写配置文件(名字和位置可以随意，这里我放在config目录下，取名为log4j_to_es.conf)：

```

vi config/logstash_test.conf

```

输入以下内容：

```

input {
  Beats {
    port => 5044
    host => "xxxx"
  }
}


# For detail structure of this file
# Set: https://www.elastic.co/guide/en/logstash/current/configuration-file-structure.html
input {
  # For detail config for log4j as input,
  # See: https://www.elastic.co/guide/en/logstash/current/plugins-inputs-log4j.html
  log4j {
    mode => "server"
    host => "centos2"
    port => 4567
  }
}
filter {
  #Only matched data are send to output.
}
output {
  # For detail config for elasticsearch as output,
  # See: https://www.elastic.co/guide/en/logstash/current/plugins-outputs-elasticsearch.html
  elasticsearch {
    action => "index"          #The operation on ES
    hosts  => "centos2:9200"   #ElasticSearch host, can be array.
    index  => "applog"         #The index to write data to.
  }
}

```

启动：

```


bin/logstash -f config/logstash.conf

nohup bin/logstash -f config/test.yml >/dev/null 2>&1 &

```

测试用output:

```

output {
 stdout { codec => rubydebug}
}

```

# Kibana

配置Kibana:

```

server.port: 5601
server.host: “centos2”
elasticsearch.url: http://centos2:9200
kibana.index: “.kibana”

```

启动：

```

./bin/kibana

```

访问：

```

http://localhost:5601

```


# Elasticsearch

下载版本 5.4.1

ES_JAVA_OPTS="-Des.insecure.allow.root=true"
