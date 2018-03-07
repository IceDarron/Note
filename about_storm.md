下载
===

安装
===

启动
===

storm原理
===
Apache Storm是一个分布式实时大数据处理系统。

数据为流式的内存计算的一条一条处理，相对于hadoop（MapReduce）的批量离线硬盘上的处理完全不同。

Storm设计用于在容错和水平可扩展方法中处理大量数据。高性能，低延迟。

### Storm主要组成
#### 架构维度
+ Nimbus：主节点。Storm集群的Master节点，负责分发用户代码，指派给具体的Supervisor节点上的Worker节点，去运行Topology对应的组件（Spout/Bolt）的Task。
+ Supervisor： 从节点。通过Storm的配置文件中的supervisor.slots.ports配置项，可以指定在一个Supervisor上最大允许多少个Slot，每个Slot通过端口号来唯一标识，一个端口号对应一个Worker进程（如果该Worker进程被启动）。
+ Worker：Topology跨一个或多个Worker节点的进程执行。每个物理机会启动一个或多个worker。（进程）。Worker运行的任务类型只有两种，一种是Spout任务，一种是Bolt任务。
+ Executor：worker下启动的线程，每个worker进程里面会有多个Executor。每个Executor只会运行一个topology的一个component（spout或者bolt）task实例。（线程）。
+ Task：具体完成数据处理的实体单元。

#### 编程模型维度
+ Topology：一个实时应用的计算任务被打包作为Topology发布，这同Hadoop的MapReduce任务相似。 
+ Spout：Storm中的消息源，用于为Topology生产消息（数据），一般是从外部数据源（如Message Queue、RDBMS、NoSQL、Realtime Log）不间断地读取数据并发送给Topology消息（tuple元组）。 
+ Bolt：Storm中的消息处理者，用于为Topology进行消息的处理，Bolt可以执行过滤，聚合， 查询数据库等操作，而且可以一级一级的进行处理。 
+ Stream：产生的数据（tuple元组）。 Storm内部中数据传输的基本单元，里面封装了一个List对象，用来保存数据。
+ Stream grouping：在Bolt任务中定义的Stream进行区分。 

### worker executor task 三者之间关系
> 1个worker进程执行的是1个topology的子集（注：不会出现1个worker为多个topology服务）。
1个worker进程会启动1个或多个executor线程来执行1个topology的component(spout或bolt)。
因此，1个运行中的topology就是由集群中多台物理机上的多个worker进程组成的。

> executor是1个被worker进程启动的单独线程。
每个executor只会运行1个topology的1个component(spout或bolt)的task（注：task可以是1个或多个，
storm默认是1个component只生成1个task，executor线程里会在每次循环里顺序调用所有task实例）。

> task是最终运行spout或bolt中代码的单元（注：1个task即为spout或bolt的1个实例，
executor线程在执行期间会调用该task的nextTuple或execute方法）。
topology启动后，1个component(spout或bolt)的task数目是固定不变的，
但该component使用的executor线程数可以动态调整（例如：1个executor线程可以执行该component的1个或多个task实例）。
这意味着，对于1个component存在这样的条件：#threads<=#tasks（即：线程数小于等于task数目）。
默认情况下task的数目等于executor线程数目，即1个executor线程只运行1个task。

![storm集群架构](https://github.com/IceDarron/Note/blob/master/Image/storm_worker&executor&task.png)


### 集群架构
Storm集群采用主从架构方式，主节点是Nimbus，从节点是Supervisor，有关调度相关的信息存储到ZooKeeper集群中，架构如下图所示：

![storm集群架构](https://github.com/IceDarron/Note/blob/master/Image/storm_model.png)

Worker之间通过Netty传送数据。Storm与Zookeeper之间的交互过程。

### 编程模型

![storm集群架构](https://github.com/IceDarron/Note/blob/master/Image/storm_programming_model.png)

这块重点讲下storm streaming grouping，即数据在流向下一个点的时候，以什么的形式传播。目前支持一下几种类型：

+ Shuffle Grouping ：随机分组，尽量均匀分布到下游Bolt中将流分组定义为混排。这种混排分组意味着来自Spout的输入将混排，或随机分发给此Bolt中的任务。shuffle grouping对各个task的tuple分配的比较均匀。
+ Fields Grouping ：按字段分组，按数据中field值进行分组；相同field值的Tuple被发送到相同的Task这种grouping机制保证相同field值的tuple会去同一个task，这对于WordCount来说非常关键，如果同一个单词不去同一个task，那么统计出来的单词次数就不对了。“if the stream is grouped by the “user-id” field, tuples with the same “user-id” will alwaysGo to the same task”. —— 小示例
+ All grouping ：广播发送， 对于每一个tuple将会复制到每一个bolt中处理。
+ Global grouping ：全局分组，Tuple被分配到一个Bolt中的一个Task，实现事务性的Topology。Stream中的所有的tuple都会发送给同一个bolt任务处理，所有的tuple将会发送给拥有最小task_id的bolt任务处理。
+ None grouping ：不分组。不关注并行处理负载均衡策略时使用该方式，目前等同于shuffle grouping,另外storm将会把bolt任务和他的上游提供数据的任务安排在同一个线程下。
+ Direct grouping ：直接分组。指定分组。

常用的主要就是Fields Grouping。


离线计算与流式计算
===
### 离线计算

离线计算：批量获取数据、批量传输数据、周期性批量计算数据、数据展示

代表技术：Sqoop批量导入数据、HDFS批量存储数据、MapReduce批量计算数据、Hive批量计算数据、azkaban/oozie任务调度

### 流式计算

流式计算：数据实时产生、数据实时传输、数据实时计算、实时展示

代表技术：Flume实时获取数据、Kafka/metaq实时数据存储、Storm/JStorm实时数据计算、Redis实时结果缓存、持久化存储(mysql)。


示例代码
===
https://github.com/IceDarron/Storm-Kafka

参考文献
===
http://ifeve.com/storm-understanding-the-parallelism-of-a-storm-topology/

http://blog.csdn.net/evankaka/article/details/61190291

http://www.cnblogs.com/Jack47/p/storm_intro-1.html