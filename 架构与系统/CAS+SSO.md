---
title: CAS+SSO
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---


网上资料:
http://blog.chinaunix.net/uid-22816738-id-3525939.html
http://www.imooc.com/article/3576
http://blog.csdn.net/u012891504/article/details/52472818
http://www.myexception.cn/h/1365771.html


jasig  cashttps
源码:
https://github.com/apereo/cas

将源码转化为eclipse工具,需要jdk1.8的支持.
并将以下代码注释掉:
/*
    def currentTime = java.time.ZonedDateTime.now()
    compileJava.doLast {
        buildDate = currentTime
        jar.manifest {
            attributes("Implementation-Date": project.buildDate)
        }
    }
	*/
http://blog.csdn.net/zhaoxusheng/article/details/70148806