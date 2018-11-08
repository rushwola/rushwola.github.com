# tomcat
-Dcatalina.home=E:\project\study\tomcat-study\catalina-home

# tomcat 性能优化
http://ajita.iteye.com/blog/1994974
http://blog.chinaunix.net/uid-7374279-id-4470247.html
http://blog.csdn.net/java_best/article/details/54708690


# tomcat源码分析

tomcat有四种容器：
Engine:表示整个Catalina servlet引擎。
Host:表示包含有一个或多个Context容器的虚拟主机；
Context:表示一个Web应用程序。一个Context可以有多个Wrapper;
Wrapper:表示一个独立的servlet。

容器还包含其它几个模块：如记录器、载入器、管理器。
