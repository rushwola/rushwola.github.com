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