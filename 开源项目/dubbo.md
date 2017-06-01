---
title: dubbo
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---


官方资源文档地址:
http://dubbo.io/Download-zh.htms


说明:
安装注册中心和,监控中心就可以用于测试环境了.


1. 获取源码
https://github.com/alibaba/dubbo

2. 编译源码
下载源码,导入eclipse,运行:mvn clean install -Dmaven.test.skip

3. 注册中心
用zookeeper当注册中心

4. 监控中心
用dubbo-monitor-simple当监控中心.

