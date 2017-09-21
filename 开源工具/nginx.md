---
title: nginx
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---
# Nginx配置upstream实现负载均衡

Nginx可以配置代理多台服务器，当一台服务器宕机之后，仍能保持系统可用。具体配置过程如下：


1. 在http节点下，添加upstream节点。

upstream linuxidc {
      server 10.0.6.108:7080;
      server 10.0.0.85:8980;
}

  2.  将server节点下的location节点中的proxy_pass配置为：http:// + upstream名称，即“
http://linuxidc”.


location / {
            root  html;
            index  index.html index.htm;
            proxy_pass http://linuxidc;
}

    3.  现在负载均衡初步完成了。upstream按照轮询（默认）方式进行负载，每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。虽然这种方式简便、成本低廉。但缺点是：可靠性低和负载分配不均衡。适用于图片服务器集群和纯静态页面服务器集群。

    除此之外，upstream还有其它的分配策略，分别如下：

weight（权重）

    指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。如下所示，10.0.0.88的访问比率要比10.0.0.77的访问比率高一倍。

upstream linuxidc{
      server 10.0.0.77 weight=5;
      server 10.0.0.88 weight=10;
}

    ip_hash（访问ip）

    每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。

upstream favresin{
      ip_hash;
      server 10.0.0.10:8080;
      server 10.0.0.11:8080;
}

    fair（第三方）

    按后端服务器的响应时间来分配请求，响应时间短的优先分配。与weight分配策略类似。

 upstream favresin{      
      server 10.0.0.10:8080;
      server 10.0.0.11:8080;
      fair;
}

url_hash（第三方）

按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。

注意：在upstream中加入hash语句，server语句中不能写入weight等其他的参数，hash_method是使用的hash算法。

 upstream resinserver{
      server 10.0.0.10:7777;
      server 10.0.0.11:8888;
      hash $request_uri;
      hash_method crc32;
}

upstream还可以为每个设备设置状态值，这些状态值的含义分别如下：

down 表示单前的server暂时不参与负载.

weight 默认为1.weight越大，负载的权重就越大。

max_fails ：允许请求失败的次数默认为1.当超过最大次数时，返回proxy_next_upstream 模块定义的错误.

fail_timeout : max_fails次失败后，暂停的时间。

backup： 其它所有的非backup机器down或者忙的时候，请求backup机器。所以这台机器压力会最轻。

upstream bakend{ #定义负载均衡设备的Ip及设备状态
      ip_hash;
      server 10.0.0.11:9090 down;
      server 10.0.0.11:8080 weight=2;
      server 10.0.0.11:6060;
      server 10.0.0.11:7070 backup;
}

Nginx的配置与部署研究，Upstream负载均衡模块  http://www.linuxidc.com/Linux/2013-04/82526p2.htm

CentOS 6.2实战部署Nginx+MySQL+PHP http://www.linuxidc.com/Linux/2013-09/90020.htm

使用Nginx搭建WEB服务器 http://www.linuxidc.com/Linux/2013-09/89768.htm

搭建基于Linux6.3+Nginx1.2+PHP5+MySQL5.5的Web服务器全过程 http://www.linuxidc.com/Linux/2013-09/89692.htm

CentOS 6.3下Nginx性能调优 http://www.linuxidc.com/Linux/2013-09/89656.htm

CentOS 6.3下配置Nginx加载ngx_pagespeed模块 http://www.linuxidc.com/Linux/2013-09/89657.htm

CentOS 6.4安装配置Nginx+Pcre+php-fpm http://www.linuxidc.com/Linux/2013-08/88984.htm

Nginx安装配置使用详细笔记 http://www.linuxidc.com/Linux/2014-07/104499.htm

Nginx日志过滤 使用ngx_log_if不记录特定日志 http://www.linuxidc.com/Linux/2014-07/104686.htm


# nginx+openssl 搭建https服务
运用openssl创建 证书:
openssl genrsa -des3 -out server.key 1024
openssl req -new -key server.key -out server.csr
openssl rsa -in server.key -out server_nopwd.key
openssl x509 -req -days 365 -in server.csr -signkey server_nopwd.key -out server.crt

至此证书已经生成完毕，下面就是配置nginx

server {
    listen 443;
    ssl on;
    ssl_certificate          server.crt;
    ssl_certificate_key     server_nopwd.key;
}

如果出现
[emerg] 10464#0: unknown directive "ssl" in /usr/local/nginx-0.6.32/conf/nginx.conf:74

则说明没有将ssl模块编译进nginx，在configure的时候加上“–with-http_ssl_module”即可
至此已经完成了https服务器搭建，但如何让浏览器信任自己颁发的证书呢？
只要将之前生成的server.crt文件导入到系统的证书管理器就行了，具体方法：

控制面板 -> Internet选项 -> 内容 -> 发行者 -> 受信任的根证书颁发机构 -> 导入 ->选择server.crt

nginx配置示例：

server {
 listen      192.168.1.*:443;
 server_name  192.168.1.*;

 #为一个server开启ssl支持
 ssl                  on;
  #为虚拟主机指定pem格式的证书文件
 ssl_certificate      /home/wangzhengyi/ssl/wangzhengyi.crt;
 #为虚拟主机指定私钥文件
 ssl_certificate_key  /home/wangzhengyi/ssl/wangzhengyi_nopass.key;
 #客户端能够重复使用存储在缓存中的会话参数时间
 ssl_session_timeout  5m;
 #指定使用的ssl协议
 ssl_protocols  SSLv3 TLSv1;
 #指定许可的密码描述
 ssl_ciphers  ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;
 #SSLv3和TLSv1协议的服务器密码需求优先级高于客户端密码
 ssl_prefer_server_ciphers  on;

 location / {
  root  /home/wangzhengyi/ssl/;
  autoindex on;
         autoindex_exact_size    off;
         autoindex_localtime on;
 }
     # redirect server error pages to the static page /50x.html
     #
     error_page  500 502 503 504  /50x.html;
     error_page  404 /404.html;

 location = /50x.html {
         root  /usr/share/nginx/www;
     }
   location = /404.html {
         root  /usr/share/nginx/www;
     }

     # proxy the PHP scripts to fpm
     location ~ \.php$ {
  access_log  /var/log/nginx/ssl/ssl.access.log  main;
  error_log /var/log/nginx/ssl/ssl.error.log;
  root /home/wangzhengyi/ssl/;
  fastcgi_param HTTPS  on;
         include /etc/nginx/fastcgi_params;  
         fastcgi_pass    sslfpm;
     }
}

# nginx  Too many open files

有一台服务器访问量非常高，使用的是nginx ，错误日志不停报以下错误：

2010/05/26 08:53:49 [alert] 13576#0: accept() failed (24: Too many open files)
2010/05/26 08:53:49 [alert] 13576#0: accept() failed (24: Too many open files)
2010/05/26 08:53:49 [alert] 13576#0: accept() failed (24: Too many open files)
2010/05/26 08:53:49 [alert] 13576#0: accept() failed (24: Too many open files)
2010/05/26 08:53:49 [alert] 13576#0: accept() failed (24: Too many open files)
2010/05/26 08:53:49 [alert] 13576#0: accept() failed (24: Too many open files)
2010/05/26 08:53:49 [alert] 13576#0: accept() failed (24: Too many open files)
2010/05/26 08:53:49 [alert] 13576#0: accept() failed (24: Too many open files)
2010/05/26 08:53:49 [alert] 13576#0: accept() failed (24: Too many open files)
2010/05/26 08:53:49 [alert] 13576#0: accept() failed (24: Too many open files)

解决方法：

centos5.3 中 ulimit -n 为1024， 当Nginx连接数超过1024时，error.log中就出现以下错误：

[alert] 12766#0: accept() failed (24: Too many open files)

使用 ulimit -n 655350 可以把打开文件数设置足够大， 同时修改nginx.conf ， 添加 worker_rlimit_nofile 655350； （与error_log同级别）

这样就可以解决Nginx连接过多的问题，Nginx就可以支持高并发。<还要修改nginx>

另外， ulimit -n 还会影响到mysql 的并发连接数。把他提高，也就提高了mysql并发。

注意： 用ulimit -n 2048 修改只对当前的shell有效，退出后失效。

修改方法

若要令修改ulimits的数值永久生效,则必须修改配置文档,可以给ulimit修改命令放入/etc/profile里面，这个方法实在是不方便,

还有一个方法是修改/etc/security/limits.conf

/etc/security/limits.conf 格式，文件里面有很详细的注释，比如

* soft nofile 655360

* hard nofile 655360

星号代表全局， soft为软件，hard为硬件，nofile为这里指可打开文件数。

把以上两行内容加到 limits.conf文件中即可。

另外，要使 limits.conf 文件配置生效，必须要确保 pam_limits.so 文件被加入到启动文件中。查看 /etc/pam.d/login 文件中有：

session required /lib/security/pam_limits.so

修改完重新登录就可以见到效果，可以通过 ulimit -n 查看。

http://blog.chinaunix.net/uid-25525723-id-363880.html

nginx 报   worker_connections are not enough错原因分析:
http://blog.sina.com.cn/s/blog_7842b12b0102v1md.html



# nginx url 问题
url的/问题
在nginx中配置proxy_pass时，当在后面的url加上了/，相当于是绝对根路径，则nginx不会把location中匹配的路径部分代理走;如果没有/，则会把匹配的路径部分也给代理走。

下面四种情况分别用http://192.168.1.4/proxy/test.html 进行访问。
第一种：
location /proxy/ {
     proxy_pass http://127.0.0.1:81/;
}
会被代理到http://127.0.0.1:81/test.html 这个url

第二咱(相对于第一种，最后少一个 /)
location /proxy/ {
     proxy_pass http://127.0.0.1:81;
}
会被代理到http://127.0.0.1:81/proxy/test.html 这个url

第三种：
location /proxy/ {
     proxy_pass http://127.0.0.1:81/ftlynx/;
}
会被代理到http://127.0.0.1:81/ftlynx/test.html 这个url。

第四种情况(相对于第三种，最后少一个 / )：
location /proxy/ {
     proxy_pass http://127.0.0.1:81/ftlynx;
}
会被代理到http://127.0.0.1:81/ftlynxtest.html 这个url

一个示例:
upstream park-report {
    server 10.37.253.58:9084;
    #check interval=3000 rise=1 fall=2 timeout=1000 type=tcp;
}
upstream park-web {
    server 10.37.253.58:7084;
 #   check interval=3000 rise=1 fall=2 timeout=1000 type=tcp;
}
upstream park-api {
    server 10.37.253.58:8084;
  #  check interval=3000 rise=1 fall=2 timeout=1000 type=tcp;
}
server {
    listen 80;
    listen 443 ssl;
    server_name qaparking.qdingnet.com;
    access_log  /data/nginx_log/qaparking-access.log  main;
    error_log /data/nginx_log/parking-error.log;

    ssl_certificate  ssl_qdingnet/server.pem;
    ssl_certificate_key  ssl_qdingnet/server.key;

    ssl_session_timeout  5m;
    ssl_protocols  TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH:HIGH:!RC4:!MD5:!aNULL:!eNULL:!NULL:!DH:!EDH:!EXP:+MEDIUM;
    ssl_prefer_server_ciphers   on;
    ssl_session_cache shared:SSL:1m;

deny 10.37.5.0/24;
    location ~ .*\.html$ {
        root /data/statics/parking/;
        }
    location ~ /(css|js|libs)/ {
        root /data/statics/parking/;
        }
    location /adapter/ {
        proxy_pass http://park-report/;
        }
    location /web/ {
        proxy_pass http://park-web/;
        }
    location / {
        proxy_pass http://park-api/;
    }
    location ^~ /md/ {
        alias /data/statics/h5/;
    }
}
