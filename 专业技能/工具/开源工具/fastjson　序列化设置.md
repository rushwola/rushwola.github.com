---
title: fastjson　序列化设置
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---


  <!-- 输出key时是否使用双引号 -->
           <value>QuoteFieldNames</value>
           <!-- 是否输出值为null的字段 -->
           <!-- <value>WriteMapNullValue</value> -->
           <!-- 数值字段如果为null,输出为0,而非null -->
           <value>WriteNullNumberAsZero</value>
           <!-- List字段如果为null,输出为[],而非null -->
           <value>WriteNullListAsEmpty</value>
           <!-- 字符类型字段如果为null,输出为"",而非null -->
           <value>WriteNullStringAsEmpty</value>
           <!-- Boolean字段如果为null,输出为false,而非null -->
           <value>WriteNullBooleanAsFalse</value>
           <!-- null String不输出  -->
           <value>WriteNullStringAsEmpty</value>
           <!-- null String也要输出  -->
           <!-- <value>WriteMapNullValue</value> -->

           <!-- Date的日期转换器 -->
           <value>WriteDateUseDateFormat</value>