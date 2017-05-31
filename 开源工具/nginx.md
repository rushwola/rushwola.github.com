---
title: nginx
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---


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
