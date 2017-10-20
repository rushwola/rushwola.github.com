源码分析:
https://yq.aliyun.com/articles/61765
http://doc.okbase.net/Donald_Draper/archive/252698.html
http://blog.csdn.net/z69183787/article/details/19965497
http://blog.csdn.net/aesop_wubo/article/details/7582266

http://blog.csdn.net/u013291394/article/details/50167359
http://blog.csdn.net/cutesource/article/details/5006062

tomcat8源码分析

# 组件
![http://www.uml.org.cn/j2ee/images/2013062851.jpg]();

Server：Server表示了整个Tomcat容器。
Catalina：与开始/关闭shell脚本交互的主类，因此如果要研究启动和关闭的过程，就从这个类开始看起。
Service：Service是一个在Server中，通过一个或多个Connector与一个Engine相连的中间件。
Container：可以理解为处理某类型请求的容器，处理的方式一般为把处理请求的处理器包装为Valve对象，并按一定顺序放入类型为Pipeline的管道里。Container有多种子类型：Engine、Host、Context和Wrapper，这几种子类型Container依次包含，处理不同粒度的请求。另外Container里包含一些基础服务，如Loader、Manager和Realm。
Engine：Engine用来处理Service的请求并返回结果。
Host：Host与域名相对应，一个Engine可以包含多个Host。
Connector：Connector用来与客户端通信，Tomcat中有多种不同的Connector实现方案。
Context：Context表示一个Web应用，一个Host可以有多个Context，通过不同的路径来区分.
Wrapper：Wrapper是针对每个Servlet的Container，每个Servlet都有相应的Wrapper来管理。

可以看出Server、Service、Connector、Container、Engine、Host、Context和Wrapper这些核心组件的作用范围是逐层递减，并逐层包含。
下面就是些被Container所用的基础组件：
Loader：是被Container用来载入各种所需的Class。
Manager：是被Container用来管理Session池。
Realm：是用来处理安全里授权与认证。
分析完核心类后，再看看Tomcat启动的过程，Tomcat启动的时序图如下所示：

# 主要包

javax：Java中标准的Servlet规范。
catalina：Tomcat中的Servlet容器。
coyote：Tomcat中的连接器。
el：正则表达式解析规范。
jasper：JSP文件解析规范。
juli：Tomcat的日志系统。
naming：JNDI的实现。
tomcat：Tomcat的工具包。

# 启动
与启动相关的两个主要类：Catalina和Bootstrap。

它们都位于org.apachae.catalina.startup包下；Catalina类用于启动或关闭
Server对象，并负责解析server.xml配置文件；Bootstrap类是一个入口点，负责创建Catalina实例，并调用其process()方法。
