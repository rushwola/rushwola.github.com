---
title: eclipse开发工具
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---
# maven 下载jar包不全的问题 
Eclipse中maven从远程仓库中下载jar包有时会很慢，有些甚至进度停止不动，这个时候我们可能会终止当前下载，但是终止jar包下载后会出现一个问题，再次打开Eclipse时，你会发现提示你项目中依赖的jar包找不到

此时我们可以通过如下方案解决

1.找到我们的本地maven仓库目录 我的是 H:\Java\maven\Repository

2.搜索出该目录下的*lastUpdated.properties文件并删除，如下图所示，可以通过模糊搜索匹配出这样的文件
3.搜索出该目录下的*.lastUpdated文件并删除



 

3.Maven 更新当前项目，maven就会继续下载缺失的依赖jar包，直至缺失jar包下载完成，上述问题就解决了。