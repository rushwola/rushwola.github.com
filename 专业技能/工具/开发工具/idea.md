# 软件下载

从http://www.jetbrains.com/下载社区版本即可。

#  IDEA community社区版集成tomcat

虽然IntelliJ IDEA社区版没有自带tomcat。
但是我们可以装一个插件——smart tomcat。

安装　smart tomcat 插件，然后 重启IDEA。

然后 Run >> Edit Configurations >> 填写如下几项：

Name：配置的名字，随便起。一般就用项目名
Tomcat Server：tomcat的路径
Deployment：webapp所在的路径
Contex Path：上下文路径。会自己识别出来，一般我们不改这个。
Server Port：默认是8080，可以改成其它
VM options: 可选的。没有参数就不填

但是社区版本热部署web项目找不到解决方案。

# 专业版本

专业版的破解方法:
https://www.cnblogs.com/jpfss/p/8872358.html
