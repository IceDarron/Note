下载
===
hadoop官网：http://hadoop.apache.org/releases.html#Download

安装
===
### windows

#### 解压
解压下载的hadoop-x.y.z.tar。放置目录例如：E:\hadoop\2.6.5。

#### 配置环境变量
+ HADOOP_HOME，填写解压路径。
+ PATH，追加%HADOOP_HOME%\bin;。

#### 配置jdk
需要修改\etc\hadoop\hadoop-env.cmd中的JAVA_HOME。
例如：set JAVA_HOME=%JAVA_HOME%改为set JAVA_HOME=E:\hadoop\Java\jdk1.8.0_144。
如果使用系统的环境变量中的jdk，则可跳过次步。

#### hadoop配置文件
配置文件路径：\etc\hadoop\*

+ core-site.xml

+ mapred-site.xml

+ hdfs-site.xml

+ yarn-site.xml
