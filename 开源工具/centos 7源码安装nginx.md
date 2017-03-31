---
title: centos 7源码安装nginx
tags: centos 源码安装 nginx
grammar_cjkRuby: true
---
# 源码获取
wget http://nginx.org/download/nginx-1.11.12.tar.gz
# 检查安装依赖项
yum -y install gcc pcre pcre-devel zlib zlib-devel openssl openssl-devel

# 配置Nginx安装选项
这查额外安装了ssh模块
./configure --prefix=/opt/nginx --sbin-path=/usr/bin/nginx  --with-http_ssl_module

# 编译并安装

make && make install

# 启动,停止,重启
shell> nginx
# 可通过ps -ef | grep nginx查看nginx是否已启动成功
# 2.停止nginx
shell> nginx -s stop
# 3. 重新启动
shell> nginx -s reload
![enter description here][1]
nginx启动成功
nginx默认配置启动成功后，会有两个进程，一个主进程（守护进程），一个工作进程。主进程负责管理工作进程，工作进程负责处理用户的http请求。

以上引用http://www.linuxidc.com/Linux/2017-02/140418.htm  地址的文章

  [1]: ./images/1490952140311.jpg "1490952140311"