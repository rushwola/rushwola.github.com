# 下载软件
hadoop,eclipse插件：

hadoop-eclipse-plugin-2.6
http://download.csdn.net/download/tondayong1981/9437360


自己编译一个试试，因为插件跟操作系统和eclipse版本有关系。
这个是咱们网站的：
hadoop2.6-eclipse插件
链接：http://pan.baidu.com/s/1bn8Nm7H 密码：5t04

http://download.csdn.net/download/tondayong1981/9251587

下载完之后将它放在eclipse中的plugin目录下。

# 报错处理

DFS Locations 报错：
Error:Permission denied:user=Administrator,access=READ_EXECUTE,

2、将当前系统的帐号修改为hadoop

3、使用HDFS的命令行接口修改相应目录的权限，hadoop fs -chmod -R 777 /tmp,后面的/tmp是要上传文件的路径，不同的情况可能不一样，比如要上传的文件路径为hdfs://namenode/user/xxx.doc，则这样的修改可以，如果要上传的文件路径为hdfs://namenode/java/xxx.doc，则要修改的为hadoop fs -chmod 777 /java或者hadoop fs -chmod 777 /，java的那个需要先在HDFS里面建立Java目录，后面的这个是为根目录调整权限。

如果是window上开发hadoop程序，还需要下载window版本的dll文件并把他放到c:/windwos/system32里面。


http://blog.csdn.net/congcong68/article/details/42043093
