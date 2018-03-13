下载
===
http://kafka.apache.org/downloads.html


安装
===
+ 依赖JDK1.8，zookeeper环境

+ 创建存放路径及解压
```text
# 存放安装包
mkdir /app/files 
cd /app/files
# 解压
tar -xzf kafka_2.11-0.9.0.0.tgz
# 移动解压后的文件到服务目录
mv /app/files/kafka_2.11-0.9.0.0 /app/kafka
```

+ 修改config/server.properties文件
```text
#指定broker.id，int类型，一般情况下请指定为ip，例如192.168.1.111,指定是111
broker.id=111
#broker的Hostname.
host.name=192.168.1.111
#- kafka log 文件目录 
log.dirs=/app/kafka/kafka-logs
#zk服务的连接地址
zookeeper.connect=192.168.1.111:2181,192.168.1.112:2181
#是否可以自动创建topic
auto.create.topics.enable=false
#是否可以删除topic
delete.topic.enable=true
```


启动关闭及常用命令
===
```text
# 注意命令中所有ip端口是否正确，部分指令不能使用localhost必须使用ip
启动：JMX_PORT=9997 nohup bin/kafka-server-start.sh config/server.properties &
停止：bin/kafka-server-stop.sh
查看日志： tail -f nohup.out –n200   
           tail -f logs/server.log –n200
查看topic的详细信息：./kafka-topics.sh -zookeeper 127.0.0.1:2181 -describe -topic testKJ1
创建topic：./kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic testKJ1
kafka生产者客户端命令：./kafka-console-producer.sh --broker-list localhost:9092 --topic testKJ1  
kafka消费者客户端命令：./kafka-console-consumer.sh -zookeeper localhost:2181 --from-beginning --topic testKJ1   
删除topic：./kafka-run-class.sh kafka.admin.DeleteTopicCommand --topic testKJ1 --zookeeper 127.0.0.1:2181 
           ./kafka-topics.sh --zookeeper localhost:2181 --delete --topic testKJ1  
停止broker：bin/kafka-server-stop.sh
```

参考资料
===
http://blog.csdn.net/fengzheku/article/details/50585972

http://blog.csdn.net/code52/article/details/50935849
