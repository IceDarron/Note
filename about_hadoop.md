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

目录结构
===
+ bin

|文件名称  |说明                                                                                 |
| -------- |------------------------------------------------------------------------------------:|
|hadoop	   |用于执行hadoop脚本命令，被hadoop-daemon.sh调用执行，也可以单独执行，一切命令的核心   |

+ sbin

|文件名称  |说明                                                                                                                               |
| -------- |----------------------------------------------------------------------------------------------------------------------------------:|
|hadoop-daemon.sh	   |通过执行hadoop命令来启动/停止一个守护进程(daemon)；该命令会被bin目录下面所有以start或stop开头的所有命令调用来执行命令，
                        hadoop-daemons.sh也是通过调用hadoop-daemon.sh来执行命令的，而hadoop-daemon.sh本身就是通过调用hadoop命令来执行任务。    |
|start-all.sh	       |全部启动，它会调用start-dfs.sh及start-mapred.sh                                                                        |
|start-dfs.sh	       |启动NameNode、DataNode以及SecondaryNameNode                                                                            |
|start-mapred.sh       |启动MapReduce                                                                                                          |
|stop-all.sh	       |全部停止，它会调用stop-dfs.sh及stop-mapred.sh                                                                          |
|stop-balancer.sh      |停止balancer                                                                                                           |
|stop-dfs.sh	       |停止NameNode、DataNode及SecondaryNameNode                                                                              |
|stop-mapred.sh	       |停止MapReduce                                                                                                          |

+ etc

|文件名称  |说明                                                                                                                               |
| -------- |----------------------------------------------------------------------------------------------------------------------------------:|
|core-site.xml	  |Hadoop核心全局配置文件，可以其他配置文件中引用该文件中定义的属性，如在hdfs-site.xml及mapred-site.xml中会引用该文件的属性；
                   该文件的模板文件存在于$HADOOP_HOME/src/core/core-default.xml，可将模板文件复制到conf目录，再进行修改。                      |
|hadoop-env.sh	  |Hadoop环境变量                                                                                                              |
|hdfs-site.xml    |HDFS配置文件，该模板的属性继承于core-site.xml；该文件的模板文件存于$HADOOP_HOME/src/hdfs/hdfs-default.xml，
                   可将模板文件复制到conf目录，再进行修改                                                                                      |
|mapred-site.xml  |MapReduce的配置文件，该模板的属性继承于core-site.xml；该文件的模板文件存于$HADOOP_HOME/src/mapred/mapredd-default.xml，
                   可将模板文件复制到conf目录，再进行修改                                                                                      |
|slaves	          |用于设置所有的slave的名称或IP，每行存放一个。如果是名称，那么设置的slave名称必须在/etc/hosts有IP映射配置                    |

+ lib
该目录下存放的是Hadoop运行时依赖的jar包，Hadoop在执行时会把lib目录下面的jar全部加到classpath中。

+ logs
该目录存放的是Hadoop运行的日志，查看日志对寻找Hadoop运行错误非常有帮助。

+ include
对外提供的编程库头文件（具体动态库和静态库在lib目录中），这些头文件均是用C++定义的，通常用于C++程序访问HDFS或者编写MapReduce程序。

+ libexec
各个服务对用的shell配置文件所在的目录，可用于配置日志输出、启动参数（比如JVM参数）等基本信息。

+ share
Hadoop各个模块编译后的jar包所在的目录。


常见问题
===
### Hadoop datanode无法启动的错误Incompatible namespaceIDs in /tmp/hadoop-ross/dfs/data解
启动Hadoop伪分布式部署的过程中，发现datanode没有正常启动，日志报错：
```xml
ERROR org.apache.hadoop.hdfs.server.datanode.DataNode: java.io.IOException: 
Incompatible namespaceIDs in /tmp/hadoop-root/dfs/data: namenode namespaceID = 1091972464;
 datanode namespaceID = 640175512
```

类似于：
```xml
Incompatible namespaceIDs in /tmp/hadoop-ross/dfs/data
```

原因：
```xml
Your Hadoop namespaceID became corrupted. Unfortunately the easiest thing to do reformat the HDFS.
```

解决方案：
```xml
You need to do something like this:

bin/stop-all.sh

rm -Rf /tmp/hadoop-your-username/*  -- 这句话是linux指令，相当于删除windows上的对应文件

bin/hadoop namenode -format
```

参考文献
===
http://blog.csdn.net/antgan/article/details/52067441

https://www.cnblogs.com/wuxun1997/p/6847950.html
