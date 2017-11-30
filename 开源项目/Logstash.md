#Logstash

Logstash使用手册，

# input

文件input测试

```

input {
    file {
     path => "/application/elkb/logdemo/logs/*.log"
   }
}
output {
  stdout { codec => rubydebug}
}

```

# filter

## kv

一个完整的示例：
input { stdin { type => "kv" } }
filter {
    if [type] == "kv" {
        kv {
            source => "message"
            prefix => "ex_"
            field_split => "&? "
            allow_duplicate_values => false
            default_keys => {
                "from" => "A"
                "to" => "B"
            }
            trim => "<>\[\],"
            trimkey => "<>\[\],"
            value_split => "=:"
        }
    }
}
output { stdout { codec => rubydebug } }
配置解释：
source为数据源，需要解析的数据，可以是字段等
prefix给所有解析出来的字段加上一个前缀
field_split解析出键值对的分隔符
allow_duplicate_values布尔类型，是否删除重复的键值对。默认值true，不删除重复键值
default_keys增加默认的键值对到事件中,以防源数据解析后不存在这些键值
trim去除解析后value里面包含的小括号或者中括号等符号
trimkey去除解析后key里面包含的小括号或者中括号等符号
value_split设置键值识别关系的分隔符，默认为＝

# 启动

```

nohup bin/logstash -f config/test.yml >/dev/null 2>&1 &

```
# 配置文件举例

首先在项目安装目录　新建patterns目录并建postfix文件内容如下
```

# contents of ./patterns/postfix:
LOG_DATETIME \d{4}\-\d{2}\-\d{2}\s\d{2}:\d{2}:\d{2},\d{1,3}
LOG_THREAD [\w|-]*
LOG_LEVEL \S{1,5}
LOG_NAME [\w|-]*
LOG_DATA \S+

```

例子一：

```


input {
    file {
     path => "/application/elkb/logdemo/logs/tracer*.log"
     type => "tracer"
   }
}
filter {
  grok {
    patterns_dir => ["./patterns"]
    match => { "message" => "%{LOG_DATETIME:log_time} \[%{LOG_THREAD:log_thread}\] %{LOG_LEVEL:log_level}\s{1,2}\[%{LOG_NAME:log_name}\]\s-\s%{LOG_DATA:log_data}" }
  }
  date {
    match => ["log_time","yyyy-MM-dd HH:mm:ss,SSS"]
    target => "date_object"
  }
  mutate {
    remove_field => "message"
  }
  mutate {
    gsub => [ "log_data","\u0001","#"]
  }
  if [type] == "tracer" {

     mutate {
       split => ["log_data","#"]
       add_field => ["requestId", "%{log_data[0]}"]
       add_field => ["response", "%{log_data[14]}"]
       #remove_field => ["log_data"]
     }
     kv {
       source => "requestId"

     }
   }
  ruby {
    code => "event['filed1']=event.get('requestId')"
  }
}
output {
  stdout { codec => rubydebug}
}

```

例子二：

```

input {
    file {
     path => "/application/elkb/logdemo/logs/tracer*.log"
     type => "tracer"
   }
}
filter {
  grok {
    patterns_dir => ["./patterns"]
    match => { "message" => "%{LOG_DATETIME:log_time} \[%{LOG_THREAD:log_thread}\] %{LOG_LEVEL:log_level}\s{1,2}\[%{LOG_NAME:log_name}\]\s-\s%{LOG_DATA:log_data}" }
  }
  date {
    match => ["log_time","yyyy-MM-dd HH:mm:ss,SSS"]
    target => "date_object"
  }
  mutate {
    remove_field => "message"
  }
  if [type] == "tracer" {
     kv {
       source => "log_data"
       field_split => "\u0001"
       value_split => ":"

     }
   }
  #ruby {
  #  code => "event['log_data'][1]"
  #}
}
output {
  #stdout { codec => rubydebug}
  elasticsearch {
        hosts => ["120.24.181.119:9200"]
        index => "tracer"
        document_type => "record"


  }
}

```

例子三：

```

nput {
    beats {
       port => 5044
    }
}
filter {
  grok {
    patterns_dir => ["./patterns"]
    match => { "message" => "%{LOG_DATETIME:log_time} \[%{LOG_THREAD:log_thread}\] %{LOG_LEVEL:log_level}\s{1,2}\[%{LOG_NAME:log_name}\]\s-\s%{LOG_DATA:log_data}" }
  }
  date {
    match => ["log_time","yyyy-MM-dd HH:mm:ss,SSS"]
    target => "date_object"
  }
  mutate {
    remove_field => "message"
  }
  kv {
      source => "log_data"
      field_split => "\u0001"
      value_split => ":"

  }

  #ruby {
  #  code => "event['log_data'][1]"
  #}
}
output {
  #stdout { codec => rubydebug}
  elasticsearch {
        hosts => ["120.24.181.119:9200"]
        index => "tracer"
        document_type => "record"


  }
}

```
