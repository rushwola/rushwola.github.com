---
title: ContextLoaderListener学习
tags: ContextLoaderListener
grammar_cjkRuby: true
---

# ContextLoaderListener详解

ContextLoaderListener监听器的作用就是启动Web容器时，自动装配ApplicationContext的配置信息。因为它实现了ServletContextListener这个接口，在web.xml配置这个监听器，启动容器时，就会默认执行它实现的方法。至于ApplicationContext.xml这个配置文件部署在哪，如何配置多个xml文件，书上都没怎么详细说明。现在的方法就是查看它的API文档。在ContextLoaderListener中关联了ContextLoader这个类，所以整个加载配置过程由ContextLoader来完成。看看它的API说明。

第一段说明ContextLoader可以由 ContextLoaderListener和ContextLoaderServlet生成。如果查看ContextLoaderServlet的API，可以看到它也关联了ContextLoader这个类而且它实现了HttpServlet这个接口。

 第二段，ContextLoader创建的是 XmlWebApplicationContext这样一个类，它实现的接口是WebApplicationContext->ConfigurableWebApplicationContext->ApplicationContext->BeanFactory这样一来spring中的所有bean都由这个类来创建

第三段,讲如何部署applicationContext的xml文件。

如果在web.xml中不写任何参数配置信息，默认的路径是/WEB-INF/applicationContext.xml，在WEB-INF目录下创建的xml文件的名称必须是applicationContext.xml；

如果是要自定义文件名可以在web.xml里加入contextConfigLocation这个context参数：
<context-param> 
        <param-name>contextConfigLocation</param-name> 
        <param-value> 
            /WEB-INF/classes/applicationContext-*.xml  
        </param-value> 
    </context-param> 
在<param-value> </param-value>里指定相应的xml文件名，如果有多个xml文件，可以写在一起并一“,”号分隔。上面的applicationContext-*.xml采用通配符，比如这那个目录下有applicationContext-ibatis-base.xml，applicationContext-action.xml，applicationContext-ibatis-dao.xml等文件，都会一同被载入。

由此可见applicationContext.xml的文件位置就可以有两种默认实现：

第一种：直接将之放到/WEB-INF下，之在web.xml中声明一个listener；

第二种：将之放到classpath下，但是此时要在web.xml中加入<context-param>，用它来指明你的applicationContext.xml的位置以供web容器来加载。按照Struts2 整合spring的官方给出的档案，写成：
<context-param> 
    <param-name>contextConfigLocation</param-name> 
    <param-value>/WEB-INF/applicationContext-*.xml,classpath*:applicationContext-*.xml</param-value> 
</context-param>


# ContextLoaderListener与springmvc

## DispatcherServlet介绍
 DispatcherServlet是Spring前端控制器的实现，提供Spring Web MVC的集中访问点，并且负责职责的分派，与Spring IoC容器无缝集成，从而可以获得Spring的所有好处。

DispatcherServlet主要用作职责调度工作，本身主要用于控制流程，主要职责如下：
1、文件上传解析，如果请求类型是multipart将通过MultipartResolver进行文件上传解析；
2、通过HandlerMapping，将请求映射到处理器（返回一个HandlerExecutionChain，它包括一个处理器、多个HandlerInterceptor拦截器）；
3、通过HandlerAdapter支持多种类型的处理器(HandlerExecutionChain中的处理器)；
4、通过ViewResolver解析逻辑视图名到具体视图实现；
5、本地化解析；
6、渲染具体的视图等；
7、如果执行过程中遇到异常将交给HandlerExceptionResolver来解析。

DispatcherServlet默认使用WebApplicationContext作为上下文，Spring默认配置文件为“/WEB-INF/[servlet名字]-servlet.xml”
DispatcherServlet也可以配置自己的初始化参数，覆盖默认配置：

| 参数                  | 描述                                                                                                                                                                                                                                               |
| --------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| contextClass          | 实现WebApplicationContext接口的类，当前的servlet用它来创建上下文。如果这个参数没有指定， 默认使用XmlWebApplicationContext。                                                                                                                        |
| contextConfigLocation | 传给上下文实例（由contextClass指定）的字符串，用来指定上下文的位置。这个字符串可以被分成多个字符串（使用逗号作为分隔符） 来支持多个上下文（在多上下文的情况下，如果同一个bean被定义两次，后面一个优先）。|
 |namespace             |      默认为/WEB-INF/[server-name]-servlet.xml                            |


如下:

``` stylus
<servlet>
        <servlet-name>demo</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring-servlet-config.xml</param-value>
        </init-param>
</servlet>
<servlet-mapping>
        <servlet-name>demo</servlet-name>
        <url-pattern>/</url-pattern>
</servlet-mapping>
```
## Servlet上下文关系
DispatcherServlet的上下文是通过配置servlet的contextConfigLocation来加载的，默认实现是XmlWebApplicationContext。

值得注意的是DispatcherServlet的上下文仅仅是Spring MVC的上下文，而Spring加载的上下文是通过ContextLoaderListener来加载的。一般spring web项目中同时会使用这两种上下文，前者仅负责MVC相关bean的配置管理（如ViewResolver、Controller、MultipartResolver等），后者则负责整个spring相关bean的配置管理（如相关Service、DAO等）。

因此在/WEB-INF/[server-name]-servlet.xml中配置的Bean一般只针对Spring MVC有效，而在ContextLoaderListener配置文件下配置的bean则对整个spring有效。

上下文创建完后会放在ServletContext对象中，
其中ContextLoaderListener加载的上下文放在ServletContext的key为WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE属性中，
而DispatcherServlet加载的上下文在每次请求时会放一份在request对象的key为WEB_APPLICATION_CONTEXT_ATTRIBUTE属性中。
因而两者的获取方式也不一样，前者可以通过WebApplicationContextUtils.getRequiredWebApplicationContext(servletContext)或WebApplicationContextUtils.getWebApplicationContext(servletContext)或WebApplicationContextUtils.getWebApplicationContext(servletContext,attrname)方法来获取对应的applicationContext，
而后者则通过RequestContextUtils.getWebApplicationContext(request)或 WebApplicationContextUtils.getWebApplicationContext(servletContext,attrname)方法来获取对应的applicationContext。
注：对于ContextLoaderListener加载的上下文，attrname即上面提到的WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE；而对于DispatcherServlet中的上下文则为FrameworkServlet.class.getName() + ".CONTEXT." + getServletName()

## DispatcherServlet中使用的特殊的Bean
	DispatcherServlet默认使用WebApplicationContext作为上下文，因此我们来看一下该上下文中有哪些特殊的Bean：

1、Controller：处理器/页面控制器，做的是MVC中的C的事情，但控制逻辑转移到前端控制器了，用于对请求进行处理；

2、HandlerMapping：请求到处理器的映射，如果映射成功返回一个HandlerExecutionChain对象（包含一个Handler处理器（页面控制器）对象、多个HandlerInterceptor拦截器）对象；如BeanNameUrlHandlerMapping将URL与Bean名字映射，映射成功的Bean就是此处的处理器；

3、HandlerAdapter：HandlerAdapter将会把处理器包装为适配器，从而支持多种类型的处理器，即适配器设计模式的应用，从而很容易支持很多类型的处理器；如SimpleControllerHandlerAdapter将对实现了Controller接口的Bean进行适配，并且掉处理器的handleRequest方法进行功能处理；

4、ViewResolver：ViewResolver将把逻辑视图名解析为具体的View，通过这种策略模式，很容易更换其他视图技术；如InternalResourceViewResolver将逻辑视图名映射为jsp视图；

5、LocalResover：本地化解析，因为Spring支持国际化，因此LocalResover解析客户端的Locale信息从而方便进行国际化；

6、ThemeResovler：主题解析，通过它来实现一个页面多套风格，即常见的类似于软件皮肤效果；

7、MultipartResolver：文件上传解析，用于支持文件上传；

8、HandlerExceptionResolver：处理器异常解析，可以将异常映射到相应的统一错误界面，从而显示用户友好的界面（而不是给用户看到具体的错误信息）；

9、RequestToViewNameTranslator：当处理器没有返回逻辑视图名等相关信息时，自动将请求URL映射为逻辑视图名；

10、FlashMapManager：用于管理FlashMap的策略接口，FlashMap用于存储一个请求的输出，当进入另一个请求时作为该请求的输入，通常用于重定向场景
 

通过以上的bean可以看出，一般LocalResover、ViewResolver等需要配置在/WEB-INF/[server-name]-servlet.xml文件中。

## 最后
说了一堆，跟Spring MVC 不配置ContextLoaderListener有什么关系呢。。。

因为 ContextLoaderListener 本质上是创建了一个 WebApplicationContext ，所以你的项目里面，如果不使用 WebApplicationContext 就可以不配置该节点。

那么只要做这种配置也是可以的：

``` stylus
    <!-- Spring MVC -->  
        <servlet>  
            <servlet-name>teach</servlet-name>  
            <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>  
            <init-param>  
                <param-name>contextConfigLocation</param-name>  
                <param-value>/WEB-INF/teach-servlet.xml</param-value>  
            </init-param>  
            <load-on-startup>1</load-on-startup>  
        </servlet>  
        <servlet-mapping>  
            <servlet-name>teach</servlet-name>  
            <url-pattern>*.action</url-pattern>  
        </servlet-mapping>  
```
发现Spring MVC 所需的配置文件不使用context-param节点指定，直接在DispatcherServlet里面配置即可 
注意：这种情况下，你的应用程序是无法使用WebApplicationContext的

正常情况下，都会配置ContextLoaderListener，因为我们知道Spring IOC的两种实现

基础的就是BeanFactory，高级的就是ApplicationContext，除非在资源非常有限的情况下，才使用BeanFactory

否则都使用ApplicationContext，而WebApplicationContext就是其中的一种高级实现，它能提供很多有用的方法
那么在应用程序如何获取 WebApplicationContext 呢，有多种方式，最简单的就是

``` stylus
    WebApplicationContext wac = ContextLoader.getCurrentWebApplicationContext();  
```
这个很熟悉了吧，刚才提到了，当前应用的WebApplicationContext就保存在 ContextLoader的currentContextPerThread属性当中 

还有基于ServletContext上下文获取的方式

``` stylus
    ServletContext sc = request.getSession().getServletContext();  
    ApplicationContext ac1 = WebApplicationContextUtils.getRequiredWebApplicationContext(sc);  
    ApplicationContext ac2 = WebApplicationContextUtils.getWebApplicationContext(sc);  
    WebApplicationContext wac1 = (WebApplicationContext) sc.getAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE);  
```
还有一些更合适的，基于Spring提供的抽象类或者接口，在初始化Bean时注入ApplicationContext

继承自抽象类ApplicationObjectSupport
说明：抽象类ApplicationObjectSupport提供getApplicationContext()方法，可以方便的获取到ApplicationContext。
Spring初始化时，会通过该抽象类的setApplicationContext(ApplicationContext context)方法将ApplicationContext 对象注入。

继承自抽象类WebApplicationObjectSupport
说明：类似上面方法，调用getWebApplicationContext()获取WebApplicationContext
实现接口ApplicationContextAware
说明：实现该接口的setApplicationContext(ApplicationContext context)方法，并保存ApplicationContext 对象。
Spring初始化时，会通过该方法将ApplicationContext对象注入。