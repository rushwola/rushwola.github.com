
# scala  开发工具下载。
http://scala-ide.org/download/sdk.html

# scala  学习资料

http://blog.csdn.net/lovehuangjiaju/article/details/47746391

# 学习路线
首先，看twitter的scala课堂
https://link.zhihu.com/?target=http%3A//twitter.github.io/scala_school/zh_cn/
以及国人的一个教程（ http://zh.scala-tour.com/ ）。

看这两个入门教程的时候，遇到难点难以理解的时候需要祭出Martin Odersky写的权威教程scala编程:
https://link.zhihu.com/?target=http%3A//book.douban.com/subject/5377415/
两边参考下，很快就能对scala有基本的认识。

接下来就是实操项目，或者去读读源码，看到不明白的写法就翻翻权威教程。遇到完全出乎意料的写法时不要灰心 ，一开始看不懂scala代码是正常的。scala本身核心理念很简单，只是由于强大的设计，让scala具备了写出各种dsl的能力，而dsl的样子差异很大，所以让人觉得scala千变万化。抓住参数类型、返回值类型，留意这些类型实现了什么trait，是否有隐式转换。就能逐渐摸通代码的逻辑。

作者：Inside
链接：https://www.zhihu.com/question/26707124/answer/36476090
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
迷茫的时候，看看此blog或许会有收获。 http://hongjiang.info/scala/

#sbt 安装

1.由于官网下载不了，到csdn下载。
```
sbt-1.0.3.zip

```

2.修改配置参数
使用国内镜象
修改./conf/sbtconfig.txt
```
# Set the java args to high
-Xmx768M
-XX:MaxPermSize=384m

# Set the extra SBT options
# -Dsbt.log.format=true
# -Dsbt.ivy.home=D:/sbt/.ivy2
# -Dsbt.global.base=D:/sbt/.sbt
-Dsbt.repository.config=D:/sbt/conf/repo.properties

```
建立./conf/repo.properties文件
```
[repositories]
local
oschina: http://maven.oschina.net/content/groups/public/
typesafe: http://repo.typesafe.com/typesafe/ivy-releases/, [organization]/[module]/(scala_[scalaVersion]/)(sbt_[sbtVersion]/)[revision]/[type]s/[artifact](-[classifier]).[ext], bootOnly
sonatype-oss-releases
maven-central
sonatype-oss-snapshots

```
