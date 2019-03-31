概念
===
### 角色
+ Leader   负责投票的发起和决议，更新系统状态。
+ Follower 用于接受客户请求并向客户端返回结果，参与选主过程中的投票。
+ Observer 可以接受客户端连接，并将请求转发到Leader，提高读取效率，为了扩展系统。不参与投票，只同步Leader状态。
+ Client   请求发起方 

### 特性
+ 最终一致性  client不论连接到哪个Server，展示给它都是同一个视图，这是zookeeper最重要的性能。
+ 可靠性      具有简单、健壮、良好的性能，如果消息m被到一台服务器接受，那么它将被所有的服务器接受。
+ 实时性      Zookeeper保证客户端将在一个时间间隔范围内获得服务器的更新信息，或者服务器失效的信息。但由于网络延时等原因，Zookeeper不能保证两个客户端能同时得到刚更新的数据，如果需要最新数据，应该在读数据之前调用sync()接口。
+ 等待无关    wait-free，慢的或者失效的client不得干预快速的client的请求，使得每个client都能有效的等待。
+ 原子性      更新只能成功或者失败，没有中间状态。
+ 顺序性      包括全局有序和偏序两种：全局有序是指如果在一台服务器上消息a在消息b前发布，则在所有Server上消息a都将在消息b前被发布；偏序是指如果一个消息b在消息a后被同一个发送者发布，a必将排在b前面。

### 原子广播-Zab协议
Zab协议包含两种模式：恢复模式（选主）和广播模式（同步）。
#### server状态
+ LOOKING：当前Server不知道leader是谁，正在搜寻，或者是服务启动的时候。
+ LEADING：当前Server即为选举出来的leader。
+ FOLLOWING：leader已经选举出来，当前Server与之同步。

### zxid
znode节点的状态信息中包含czxid。
ZooKeeper状态的每一次改变, 都对应着一个递增的Transaction id, 该id称为zxid. 由于zxid的递增性质, 如果zxid1小于zxid2, 那么zxid1肯定先于zxid2发生.创建任意节点, 或者更新任意节点的数据, 或者删除任意节点, 都会导致Zookeeper状态发生改变, 从而导致zxid的值增加。

### zookeeper提供了什么功能
zookeeper功能上分为两部分，文件系统和通知机制。
#### 文件系统
zookeeper是维护了一个像文件系统的数据结构。每个子目录都被称为znode，使用者可以对这个znode进行CRUD。
znode又可以分为临时和永久，顺序和无序。

znode有四种类型：
+ PERSISTENT-持久化目录节点    客户端与zookeeper断开连接后，该节点依旧存在。
+ PERSISTENT_SEQUENTIAL-持久化顺序编号目录节点    客户端与zookeeper断开连接后，该节点依旧存在，只是Zookeeper给该节点名称进行顺序编号。
+ EPHEMERAL-临时目录节点    客户端与zookeeper断开连接后，该节点被删除。
+ EPHEMERAL_SEQUENTIAL-临时顺序编号目录节点    客户端与zookeeper断开连接后，该节点被删除，只是Zookeeper给该节点名称进行顺序编号。
#### 通知机制watcher
客户端可以注册znode的监听，当znode发生相应变化使，通知客户端。

事件类型（跟Znode节点相关的）：
+ EventType.NodeCreated
+ EventType.NodeDataChanged
+ EventType.NodeChildrenChanged
+ EventType.NodeDeleted
+ EventType.NONE  ---连接上zk后触发此事件类型

状态类型（跟客户端实例相关的）：
+ KeeperState.Disconnected
+ KeeperState.SyncConnected
+ KeeperState.AuthFailed
+ KeeperState.Expired

ZooKeeper 的 Watcher 机制主要包括客户端线程、客户端 WatchManager 和 ZooKeeper 服务器三部分。 
总的来说可以分为三个过程：客户端注册 Watcher、服务器处理 Watcher 和客户端回调 Watcher。
+ ZooKeeper ：部署在远程主机上的 ZooKeeper 集群，当然，也可能是单机的。
+ Client ：分布在各处的 ZooKeeper 的 jar 包程序，被引用在各个独立应用程序中。
+ WatchManager ：一个接口，用于管理各个监听器，只有一个方法 materialize()，返回一个 Watcher 的 set。

特性：
+ 一次性  watcher注册后只能使用一次，再次使用需要重新注册，这样可以减轻服务端压力。
+ 客户端串行执行  客户端watcher回调是一个串行同步的过程，保证了顺序。
+ 轻量  watcherevent是watcher最小的通知单元，只包含了三部分：通知状态，事件类型，节点路径。

### 利用zookeeper实现的功能
+ 命名服务  通过命名服务实现了服务注册及发现。
+ 配置管理  可以统一修改集群配置文件。
+ 集群管理  主要包含机器退出及加入，选举master。
+ 分布式锁  由于zookeeper的一致性文件系统，实现分布式锁。




参考资料
===
https://blog.csdn.net/shengqianfeng/article/details/79508376  Java验证zookeeper的watcher机制
https://blog.csdn.net/hohoo1990/article/details/78617336  wathcer机制理解

