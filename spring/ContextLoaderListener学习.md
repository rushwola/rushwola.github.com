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

## 