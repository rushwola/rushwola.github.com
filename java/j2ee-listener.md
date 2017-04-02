---
title: j2ee-listener
tags:  j2ee-listener
grammar_cjkRuby: true
---
# j2EE监听器-listener
当Web应用在Web容器中运行时，Web应用内部会不断地发生各种事件：如Web应用被启动、Web应用被停止、用户session开始、用户session结束、用户请求到达等，可以用Servlet API提供的监听器来监听Web应用的内部事件，从而允许当Web内部事件发生时回调事件监听器内的方法。应用程序完全可以采用一个监听器类来监听多种事件，只要让该监听器实现类同时实现多个监听器接口即可。

# 常用的Web事件监听器接口
