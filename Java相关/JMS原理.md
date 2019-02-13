Q:jms原理
===
JMS（JAVA Message Service,java消息服务）API是一个消息服务的标准或者说是规范，允许应用程序组件基于JavaEE平台创建、发送、接收和读取消息。它使分布式通信耦合度更低，消息服务更加可靠以及异步性。

### 基本概念
JMS是java的消息服务，JMS的客户端之间可以通过JMS服务进行异步的消息传输。

### 消息模型
+ Point-to-Point(P2P)
+ Publish/Subscribe(Pub/Sub)