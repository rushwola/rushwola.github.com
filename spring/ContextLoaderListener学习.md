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
| contextConfigLocation | 传给上下文实例（由contextClass指定）的字符串，用来指定上下文的位置。这个字符串可以被分成多个字符串（使用逗号作为分隔符） 来支持多个上下文（在多上下文的情况下，如果同一个bean被定义两次，后面一个优先）。
默认为/WEB-INF/[server-name]-servlet.xml |
| namespace             |                                                                                                                                                                                                                                                    |


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

