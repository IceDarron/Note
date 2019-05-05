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
查询kafka偏移量：./kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list localhost:9092,localhost1:9092 --topic TEST_MONITOR -time -1
kafka生产者客户端命令：./kafka-console-producer.sh --broker-list localhost:9092 --topic testKJ1  
kafka消费者客户端命令：./kafka-console-consumer.sh -zookeeper localhost:2181 --from-beginning --topic testKJ1   
删除topic：./kafka-run-class.sh kafka.admin.DeleteTopicCommand --topic testKJ1 --zookeeper 127.0.0.1:2181 
           ./kafka-topics.sh --zookeeper localhost:2181 --delete --topic testKJ1  
停止broker：bin/kafka-server-stop.sh
```

ISR（in-sync replica已同步的副本）列表
===
生产者生产消息的时候，通过request.required.acks参数来设置数据的可靠性。

acks what happen
0
which means that the producer never waits for an acknowledgement from the broker.发过去就完事了，不关心broker是否处理成功，可能丢数据。

1
which means that the producer gets an acknowledgement after the leader replica has received the data. 当写Leader成功后就返回,其他的replica都是通过fetcher去同步的,所以kafka是异步写，主备切换可能丢数据。

-1
which means that the producer gets an acknowledgement after all in-sync replicas have received the data. 要等到isr里所有机器同步成功，才能返回成功，延时取决于最慢的机器。强一致，不会丢数据。


参考资料
===
http://blog.csdn.net/fengzheku/article/details/50585972  kafka删除topic

http://blog.csdn.net/code52/article/details/50935849  kafka常用命令

https://www.jianshu.com/p/d3e963ff8b70  kafka详解
