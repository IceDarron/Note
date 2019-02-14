下载
===
https://www.apache.org/dyn/closer.cgi/zookeeper/

安装（集群）
===
+ 依赖JDK1.8

+ 创建存放路径及解压
```text
# 存放安装包
mkdir /app/files 
cd /app/files
# 解压
tar -xzf zookeeper-3.4.6.tar.gz
# 移动解压后的文件到服务目录
mv /app/files/zookeeper-3.4.6 /app/zookeeper
```

+ 修改日志文件

config/log4j.properties

+ 修改配置文件
```text
tickTime=2000
# 必须配置data
dataDir=/app/zookeeper/data
clientPort=2182
maxClientCnxns=60
initLimit=5
syncLimit=2
# 2881端口为zk内部通讯端口 3881为zk选举端口
server.1=192.168.1.111:2881:3881
server.2=192.168.1.112:2881:3881
server.3=192.168.1.113:2881:3881
```

+ 创建data文件
```text
mkdir /app/zookeeper/data
touch /app/zookeeper/data/myid

# 修改data/myid文件，每台服务器根据自己的zoo.cfg文件修改myid内容。建议zoo.cfg中对应ip的server.x配置中，x=ip的最后一段。myid内容=x。
# 例如192.168.1.111服务器上zoo.cfg内容为server.111=192.168.1.111:2881:3881
#                          myid内容就应该是111
```

启动关闭及常用命令
===
```text
启动 ：./zkServer.sh start
停止 ：./zkServer.sh stop
查看状态：./zkServer.sh status
```


参考资料
===
http://blog.csdn.net/lihao21/article/details/51778255