---
title: j2ee-listener
tags:  j2ee-listener
grammar_cjkRuby: true
---
# j2EE监听器-listener
当Web应用在Web容器中运行时，Web应用内部会不断地发生各种事件：如Web应用被启动、Web应用被停止、用户session开始、用户session结束、用户请求到达等，可以用Servlet API提供的监听器来监听Web应用的内部事件，从而允许当Web内部事件发生时回调事件监听器内的方法。应用程序完全可以采用一个监听器类来监听多种事件，只要让该监听器实现类同时实现多个监听器接口即可。

# 常用的Web事件监听器接口

 1. ServletContextListener
 监听Web应用的启动和关闭
 包含的方法：
 contextInitialized(ServletContextEvent sce)  启动Web应用时系统调用Listener的该方法,可通过ServletContextEvent取得该应用的ServletContext实例（sce.getServletContext())
 
 contextDestroyed(ServletContextEvent sce)：关闭Web应用时系统调用Listener的该方法
 
 2. ServletContextAttributeListener：监听ServletContext范围(application)内属性的改变
 包含的方法：
attributeAdded(ServletContextAttributeEvent event)：当把一个属性存入application范围时触发该方法，可通过形参event.getName和event.getValue获取添加的属性名和属性值
attributeRemoved(ServletContextAttributeEvent event)：当把一个属性从application范围删除时将触发该方法，可通过形参event.getName和event.getValue获取删除的属性名和属性值
attributeReplaced(ServletContextAttributeEvent event)：当程序替换application范围内的属性时将触发该方法，可通过形参event.getName和event.getValue获取替换后的属性名和属性值
 
 
 3. 

