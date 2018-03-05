下载
===

安装
===

启动
===

简介
===
Apache Storm是一个分布式实时大数据处理系统。

Storm设计用于在容错和水平可扩展方法中处理大量数据。

Storm框架主要由7部分组成：

+ Topology：一个实时应用的计算任务被打包作为Topology发布，这同Hadoop的MapReduce任务相似。 
+ Spout：Storm中的消息源，用于为Topology生产消息（数据），一般是从外部数据源（如Message Queue、RDBMS、NoSQL、Realtime Log）不间断地读取数据并发送给Topology消息（tuple元组）。 
+ Bolt：Storm中的消息处理者，用于为Topology进行消息的处理，Bolt可以执行过滤，聚合， 查询数据库等操作，而且可以一级一级的进行处理。 
+ Stream：产生的数据（tuple元组）。 
+ Stream grouping：在Bolt任务中定义的Stream进行区分。 
+ Task：每个Spout或者Bolt在集群执行许多任务。 
+ Worker：Topology跨一个或多个Worker节点的进程执行。
