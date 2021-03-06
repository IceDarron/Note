Q:MyBatis原理。
===
![Image text](https://github.com/IceDarron/Note/blob/master/Image/mybatis_structure.png)
![Image text](https://github.com/IceDarron/Note/blob/master/Image/mybatis_main.png)
##### 缓存,动态SQL,java动态代理
>MyBatis 是支持定制化 SQL、存储过程以及高级映射的优秀的持久层框架。MyBatis 避免了几乎所有的 JDBC 代码和手工设置参数以及抽取结果集。MyBatis 使用简单的 XML 或注解来配置和映射基本体，将接口和 Java 的 POJOs(Plain Old Java Objects,普通的 Java对象)映射成数据库中的记录。

+ 1、Mybatis实现了接口绑定，使用更加方便。 
+ 2、对象关系映射的改进，效率更高。
+ 3、MyBatis采用功能强大的基于OGNL的表达式来消除其他元素。

>原理详解：MyBatis应用程序根据XML配置文件创建SqlSessionFactory，SqlSessionFactory在根据配置，配置来源于两个地方，一处是配置文件，一处是Java代码的注解，获取一个SqlSession。SqlSession包含了执行sql所需要的所有方法，可以通过SqlSession实例直接运行映射的sql语句，完成对数据的增删改查和事务提交等，用完之后关闭SqlSession。

>MyBatis的主要成员：
+ Configuration        MyBatis所有的配置信息都保存在Configuration对象之中，配置文件中的大部分配置都会存储到该类中
+ SqlSession            作为MyBatis工作的主要顶层API，表示和数据库交互时的会话，完成必要数据库增删改查功能
+ Executor               MyBatis执行器，是MyBatis 调度的核心，负责SQL语句的生成和查询缓存的维护
+ StatementHandler 封装了JDBC Statement操作，负责对JDBC statement 的操作，如设置参数等
+ ParameterHandler  负责对用户传递的参数转换成JDBC Statement 所对应的数据类型
+ ResultSetHandler   负责将JDBC返回的ResultSet结果集对象转换成List类型的集合
+ TypeHandler          负责java数据类型和jdbc数据类型(也可以说是数据表列类型)之间的映射和转换
+ MappedStatement  MappedStatement维护一条<select|update|delete|insert>节点的封装
+ SqlSource              负责根据用户传递的parameterObject，动态地生成SQL语句，将信息封装到BoundSql对象中，并返回
+ BoundSql              表示动态生成的SQL语句以及相应的参数信息

>接口绑定是通过java动态代理实现的，通过实现动态代理生成以Xml文件中的命名空间相同的接口进行绑定。

>动态SQL主要是解决串联sql的问题，主要是利用基于OGNL的表达式来实现。

>缓存。默认情况下是没有开启缓存的,除了局部的 session 缓存,可以增强变现而且处理循环 依赖也是必须的。通过开启二级缓存可以实现一下效果：
+ 映射语句文件中的所有 select 语句将会被缓存。
+ 映射语句文件中的所有 insert,update 和 delete 语句会刷新缓存。
+ 缓存会使用 Least Recently Used(LRU,最近最少使用的)算法来收回。
+ 根据时间表(比如 no Flush Interval,没有刷新间隔), 缓存不会以任何时间顺序 来刷新。
+ 缓存会存储列表集合或对象(无论查询方法返回什么)的 1024 个引用。
+ 缓存会被视为是 read/write(可读/可写)的缓存,意味着对象检索不是共享的,而 且可以安全地被调用者修改,而不干扰其他调用者或线程所做的潜在修改。
映射文件修改<cache eviction="FIFO" flushInterval="60000" size="512" readOnly="true"/>
提供的回收策略：
+ LRU – 最近最少使用的:移除最长时间不被使用的对象。
+ FIFO – 先进先出:按对象进入缓存的顺序来移除它们。
+ SOFT – 软引用:移除基于垃圾回收器状态和软引用规则的对象。
+ WEAK – 弱引用:更积极地移除基于垃圾收集器状态和弱引用规则的对象。

https://www.cnblogs.com/luoxn28/p/6417892.html


Q:MyBatis如何管理session和cache？
===
管理session：
>SqlSessionFactoryBuilder
这个类可以被实例化、使用和丢弃，一旦创建了 SqlSessionFactory，就不再需要它了。因此 SqlSessionFactoryBuilder 实例的最佳作用域是方法作用域（也就是局部方法变量）。你可以重用 SqlSessionFactoryBuilder 来创建多个 SqlSessionFactory 实例，但是最好还是不要让其一直存在以保证所有的 XML 解析资源开放给更重要的事情。

>SqlSessionFactory
一旦被创建就应该在应用的运行期间一直存在，没有任何理由对它进行清除或重建。使用 SqlSessionFactory 的最佳实践是在应用运行期间不要重复创建多次，多次重建 SqlSessionFactory 被视为一种代码“坏味道（bad smell）”。因此 SqlSessionFactory 的最佳作用域是应用作用域。有很多方法可以做到，最简单的就是使用单例模式或者静态单例模式。

>SqlSession
每个线程都应该有它自己的 SqlSession 实例。SqlSession 的实例不是线程安全的，因此是不能被共享的，所以它的最佳的作用域是请求或方法作用域。绝对不能将 SqlSession 实例的引用放在一个类的静态域，甚至一个类的实例变量也不行。也绝不能将 SqlSession 实例的引用放在任何类型的管理作用域中，比如 Servlet 架构中的 HttpSession。如果你现在正在使用一种 Web 框架，要考虑 SqlSession 放在一个和 HTTP 请求对象相似的作用域中。换句话说，每次收到的 HTTP 请求，就可以打开一个 SqlSession，返回一个响应，就关闭它。这个关闭操作是很重要的，你应该把这个关闭操作放到 finally 块中以确保每次都能执行关闭。

管理cache：
>一级缓存是SqlSession级别的缓存，每个SqlSession对象都有一个哈希表用于缓存数据，不同SqlSession对象之间缓存不共享。同一个SqlSession对象对象执行2遍相同的SQL查询，在第一次查询执行完毕后将结果缓存起来，这样第二遍查询就不用向数据库查询了，直接返回缓存结果即可。MyBatis默认是开启一级缓存的。
BaseExecutor: 基础执行器抽象类。
二级缓存是mapper(Namespace)级别的缓存，二级缓存是跨SqlSession的，多个SqlSession对象可以共享同一个二级缓存。不同的SqlSession对象执行两次相同的SQL语句，第一次会将查询结果进行缓存，第二次查询直接返回二级缓存中的结果即可。
CachingExecutor: 二级缓存执行器。

>delegate机制在缓存中的使用，主要是当在二级缓存中查询时如果未命中，则将查询委托给一级缓存查询。

>当SQL语句进行更新操作(删除/添加/更新)时，会清空对应的缓存，保证缓存中存储的都是最新的数据。MyBatis的二级缓存对细粒度的数据级别的缓存实现不友好，比如如下需求：对商品信息进行缓存，由于商品信息查询访问量大，但是要求用户每次都能查询最新的商品信息，此时如果使用mybatis的二级缓存就无法实现当一个商品变化时只刷新该商品的缓存信息而不刷新其它商品的信息，因为mybaits的二级缓存区域以mapper为单位划分，当一个商品信息变化会将所有商品信息的缓存数据全部清空。

![Image text](https://github.com/IceDarron/Note/blob/master/Image/mybatis_executor.png)
![Image text](https://github.com/IceDarron/Note/blob/master/Image/mybatis_cache_delegate.png)


   
Q:mysql常用的引擎及引擎的特点。
===
#### ISAM,MyISAM,HEAP,InnoDB
>ISAM：ISAM是一个定义明确且历经时间考验的数据表格管理方法，它在设计之时就考虑到 数据库被查询的次数要远大于更新的次数。
因此，ISAM执行读取操作的速度很快，而且不占用大量的内存和存储资源。
ISAM的两个主要不足之处在于，它不支持事务处理，也不能够容错：如果你的硬盘崩溃了，那么数据文件就无法恢复了。
如果你正在把ISAM用在关键任务应用程序里，那就必须经常备份你所有的实时数据，通过其复制特性，MYSQL能够支持这样的备份应用程序。

>MyISAM：MyISAM是MySQL的ISAM扩展格式和缺省的数据库引擎。
除了提供ISAM里所没有的索引和字段管理的大量功能，MyISAM还使用一种表格锁定（这里可以产生一个知识点：就是常见的锁表）的机制，
来优化多个并发的读写操作，其代价是你需要经常运行OPTIMIZE TABLE命令，来恢复被更新机制所浪费的空间。MyISAM还有一些有用的扩展，
例如用来修复数据库文件的MyISAMCHK工具和用来恢复浪费空间的 MyISAMPACK工具。
MYISAM强调了快速读取操作，这可能就是为什么MySQL受到了WEB开发如此青睐的主要原因：在WEB开发中你所进行的大量数据操作都是读取操作。
所以，大多数虚拟主机提供商和INTERNET平台提供商只允许使用MYISAM格式。MyISAM格式的一个重要缺陷就是不能在表损坏后恢复数据。

>HEAP：HEAP允许只驻留在内存里的临时表格。
驻留在内存里让HEAP要比ISAM和MYISAM都快，但是它所管理的数据是不稳定的，而且如果在关机之前没有进行保存，那么所有的数据都会丢失。
在数据行被删除的时候，HEAP也不会浪费大量的空间。HEAP表格在你需要使用SELECT表达式来选择和操控数据的时候非常有用。
要记住，在用完表格之后就删除表格。

>InnoDB：InnoDB数据库引擎都是造就MySQL灵活性的技术的直接产品，这项技术就是MYSQL++ API。
在使用MYSQL的时候，你所面对的每一个挑战几乎都源于ISAM和MyISAM数据库引擎不支持事务处理（transaction process）也不支持外来键。
尽管要比ISAM和 MyISAM引擎慢很多，但是InnoDB包括了对事务处理和外来键的支持，这两点都是前两个引擎所没有的。
如前所述，如果你的设计需要这些特性中的一者 或者两者，那你就要被迫使用后两个引擎中的一个了。 

>MySQL 官方对InnoDB是这样解释的：InnoDB给MySQL提供了具有提交、回滚和崩溃恢复能力的事务安全（ACID兼容）存储引擎。
InnoDB锁定在行级并且也在SELECT语句提供一个Oracle风格一致的非锁定读，这些特色增加了多用户部署和性能。
没有在InnoDB中扩大锁定的需要，因为在InnoDB中行级锁定适合非常小的空间。InnoDB也支持FOREIGN KEY强制。
在SQL查询中，你可以自由地将InnoDB类型的表与其它MySQL的表的类型混合起来，甚至在同一个查询中也可以混合。

>InnoDB是为处理巨大数据量时的最大性能设计，它的CPU效率可能是任何其它基于磁盘的关系数据库引擎所不能匹敌的。

>InnoDB存储引擎被完全与MySQL服务器整合，InnoDB存储引擎为在主内存中缓存数据和索引而维持它自己的缓冲池。
InnoDB存储它的表＆索引在一个表空间中，表空间可以包含数个文件（或原始磁盘分区）。这与MyISAM表不同，比如在MyISAM表中每个表被存在分离的文件中。InnoDB 表可以是任何尺寸，即使在文件尺寸被限制为2GB的操作系统上。

>InnoDB默认地被包含在MySQL二进制分发中。Windows Essentials installer使InnoDB成为Windows上MySQL的默认表。

>InnoDB被用来在众多需要高性能的大型数据库站点上产生。著名的Internet新闻站点Slashdot.org运行在InnoDB上。

>一般来说：

MyISAM适合：

+ 做很多count 的计算；
+ 插入不频繁，查询非常频繁；
+ 没有事务。

InnoDB适合：

+ 可靠性要求比较高，或者要求事务；
+ 表更新和查询都相当的频繁，并且表锁定的机会比较大的情况。

>一般情况下，MySQL会默认提供多种存储引擎，可以通过下面的查看:

+ 看你的MySQL现在已提供什么存储引擎: mysql> show engines;
+ 看你的MySQL当前默认的存储引擎: mysql> show variables like '%storage_engine%';
+ 你要看某个表用了什么引擎(在显示结果里参数engine后面的就表示该表当前用的存储引擎): mysql> show create table 表名;