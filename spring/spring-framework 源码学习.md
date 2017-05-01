---
title: spring-framework 源码学习
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---
http://blog.csdn.net/gksrxn/article/details/14105245

一. 准备工作

1.下载安装sts(springsource推荐使用), 毕竟人家的框架用他自家的ide是最好的,当然sts也是基本eclipse的,　下载地址: http://www.springsource.org/downloads/sts-ggts

2.下载安装gradle, spring 源码构建加入了gradle支持. gradle下载: http://www.gradle.org/downloads ,下载后设置环境变量: GRADLE_HOME = gradle主目录 , 并在path中加入;%GRADLE_HOME%\bin;

3.下载安装github,　spring源码托管到了github : http://windows.github.com/ (windows) ，当然您压需要注册github账号 , spring github托管地址: https://github.com/SpringSource/spring-framework