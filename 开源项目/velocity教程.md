---
title: velocity教程 
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---
Velocity是一个基于Java的模板引擎，通过特定的语法，Velocity可以获取在java语言中定义的对象，从而实现界面和java代码的真正分离，这意味着可以使用velocity替代jsp的开发模式了(实际上笔者所在的公司已经这么做了)。这使得前端开发人员可以和 Java 程序开发人员同步开发一个遵循 MVC 架构的 web 站点，在实际应用中，velocity还可以应用于很多其他的场景.

1. Velocity的介绍

Velocity是一个基于Java的模板引擎，其提供了一个Context容器，在java代码里面我们可以往容器中存值，然后在vm文件中使用特定的语法获取，这是velocity基本的用法，其与jsp、freemarker并称为三大视图展现技术，相对于jsp而言，velocity对前后端的分离更加彻底：在vm文件中不允许出现java代码，而jsp文件中却可以.

作为一个模块引擎，除了作为前后端分离的MVC展现层，Velocity还有一些其他用途，比如源代码生成、自动email和转换xml等，具体的用法可以参考下面的描述。

# Velocity 模板引擎介绍

在现今的软件开发过程中，软件开发人员将更多的精力投入在了重复的相似劳动中。特别是在如今特别流行的 MVC 架构模式中，软件各个层次的功能更加独立，同时代码的相似度也更加高。所以我们需要寻找一种来减少软件开发人员重复劳动的方法，让程序员将更多的精力放在业务逻辑以及其他更加具有创造力的工作上。Velocity 这个模板引擎就可以在一定程度上解决这个问题。

Velocity 是一个基于 Java 的模板引擎框架，提供的模板语言可以使用在 Java 中定义的对象和变量上。Velocity 是 Apache 基金会的项目，开发的目标是分离 MVC 模式中的持久化层和业务层。但是在实际应用过程中，Velocity 不仅仅被用在了 MVC 的架构中，还可以被用在以下一些场景中。

1.Web 应用：开发者在不使用 JSP 的情况下，可以用 Velocity 让 HTML 具有动态内容的特性。
2. 源代码生成：Velocity 可以被用来生成 Java 代码、SQL 或者 PostScript。有很多开源和商业开发的软件是使用 Velocity 来开发的。
3. 自动 Email：很多软件的用户注册、密码提醒或者报表都是使用 Velocity 来自动生成的。使用 Velocity 可以在文本文件里面生成邮件内容，而不是在 Java 代码中拼接字符串。
4. 转换 xml：Velocity 提供一个叫 Anakia 的 ant 任务，可以读取 XML 文件并让它能够被 Velocity 模板读取。一个比较普遍的应用是将 xdoc 文档转换成带样式的 HTML 文件。
