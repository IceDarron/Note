Q:Java并发包
===
> Java多线程相关类的实现都在Java的并发包concurrent，concurrent包主要包含3部分内容，第一个是atomic包，里面主要是一些原子类，比如AtomicInteger、AtomicIntegerArray等；第二个是locks包，里面主要是锁相关的类，比如ReentrantLock、Condition等；第三个就是属于concurrent包的内容，主要包括线程池相关类（Executors）、阻塞集合类（BlockingQueue）、并发Map类（ConcurrentHashMap）、线程相关类（Thread、Runnable、Callable）等。

![Image text](https://github.com/IceDarron/Note/blob/master/Image/jdk_concurrent.png)

### atomic包源码分析
> atomic包是专门为线程安全设计的Java包，包含多个原子操作类。其基本思想就是在多线程环境下，当有多个线程同时执行这些类的实例的方法时，具有排他性，一个线程进入方法执行指令时，不会被其他的线程打断，而别的线程就像自旋锁一样，一直等待该方法执行完成。

> 原子变量的底层使用了处理器提供的原子指令，但是不同的CPU架构可能提供的原子指令不一样，也有可能需要某种形式的内部锁，所以该方法不能绝对保证线程不被阻塞。

> atomic包一共有12个类，四种原子更新方式，分别是原子更新基本类型、原子更新数组、原子更新引用和原子更新字段。JDK1.5中引入了底层的支持，在int、long和对象的引用等类型上都公开了CAS的操作，并且JVM把它们编译为底层硬件提供的最有效的方法，在运行CAS的平台上，运行时把它们编译为相应的机器指令。在java.util.concurrent.atomic包下面的所有的原子变量类型中，比如AtomicInteger，都使用了这些底层的JVM支持为数字类型的引用类型提供一种高效的CAS操作。

> Unsafe中的操作一般都是基于CAS来实现的，CAS就是Compare and Swap的意思，比较并操作。很多的cpu直接支持CAS指令。CAS是一项乐观锁技术，当多个线程尝试使用CAS同时更新同一个变量时，只有其中一个线程能更新变量的值，而其它线程都失败，失败的线程并不会被挂起，而是被告知这次竞争中失败，并可以再次尝试。CAS有3个操作数，内存值V，旧的预期值A，要修改的新值B。当且仅当预期值A和内存值V相同时，将内存值V修改为B，否则什么都不做。

### lock包源码分析
> lock包里面主要是锁相关的类，比如ReentrantLock、Condition等。
Lock接口主要有lock、lockInterruptibly、tryLock、unlock、newCondition等方法

### LinkedBlockingQueue
> LinkedBlockingQueue是基于链表结构的阻塞队列，按照FIFO（先进先出）原则对元组进行排序，新元素是尾部插入，吞吐量通常高于ArrayBlockingQueue。该类中包含一个takeLock和基于takeLock的Condition对象notEmpty，一个putLock锁，和基于putLock的Condition对象notFull。在构造方法中会新new一个Node，last和head都指向该Node节点。
> 执行put操作时，首先获取putLock，如果链表节点数已经达到上限，则调用notFull.await等待；否则调用enqueue插入元素，插入成功后把count值原子加1，如果链表节点数未达到上限，则调用notFull.signal。然后获取takeLock，再调用notEmpty.signal通知。
> 执行take操作时，首先获取takeLock，如果链表为空，则调用notEmpty.await等待；否则调用dequeue取出元素，然后把count值原子减1，如果此时链表非空，则调用notEmpty.signal。然后获取putLock，再调用putLock.signal通知。

### ConcurrentHashMap
> ConcurrentHashMap是concurrent包中一个重要的类，其高效支并发操作，被广泛使用，Spring框架的底层数据结构就是使用ConcurrentHashMap实现的。同HashTable相比，它的锁粒度更细，而不是像HashTable一样为每个方法都添加了synchronized锁。Java8中的ConcurrentHashMap废弃了Segment（锁段）的概念，而是用CAS和synchronized方法来实现。利用CAS来获取table数组中的单个Node节点，获取成功进行更新操作时，再使用synchronized处理对应Node节点所对应链表（或红黑树）中的数据。

几个核心的内部类：
#### Node
> Node是最核心的内部类，它包装了key-value键值对，所有插入ConcurrentHashMap的数据都包装在这里面。它与HashMap中的定义很相似，但是但是有一些差别它对value和next属性设置了volatile同步锁，它不允许调用setValue方法直接改变Node的value域，它增加了find方法辅助map.get()方法。

#### TreeNode
> 树节点类，另外一个核心的数据结构。当链表长度过长的时候，会转换为TreeNode。但是与HashMap不相同的是，它并不是直接转换为红黑树，而是把这些结点包装成TreeNode放在TreeBin对象中，由TreeBin完成对红黑树的操作。而且TreeNode在ConcurrentHashMap继承自Node类，而并非HashMap中的继承自LinkedHashMap.Entry<K,V>类，也就是说TreeNode带有next指针，这样做的目的是方便基于TreeBin的访问。

#### TreeBin
> 这个类并不负责包装用户的key、value信息，而是包装的很多TreeNode节点。它代替了TreeNode的根节点，也就是说在实际的ConcurrentHashMap“数组”中，存放的是TreeBin对象，而不是TreeNode对象，这是与HashMap的区别。另外这个类还带有了读写锁。

#### put操作
> ConcurrentHashMap最常用的就是put和get两个方法。现在来介绍put方法，这个put方法依然沿用HashMap的put方法的思想，根据hash值计算这个新插入的点在table中的位置i，如果i位置是空的，直接放进去，否则进行判断，如果i位置是树节点，按照树的方式插入新的节点，否则把i插入到链表的末尾。ConcurrentHashMap中依然沿用这个思想，有一个最重要的不同点就是ConcurrentHashMap不允许key或value为null值。另外由于涉及到多线程，put方法就要复杂一点。在多线程中可能有

以下两个情况:
+ 如果一个或多个线程正在对ConcurrentHashMap进行扩容操作，当前线程也要进入扩容的操作中。这个扩容的操作之所以能被检测到，是因为transfer方法中在空结点上插入forward节点，如果检测到需要插入的位置被forward节点占有，就帮助进行扩容；
+ 如果检测到要插入的节点是非空且不是forward节点，就对这个节点加锁，这样就保证了线程安全。尽管这个有一些影响效率，但是还是会比hashTable的synchronized要好得多。

> 整体流程就是首先定义不允许key或value为null的情况放入  对于每一个放入的值，首先利用spread方法对key的hashcode进行一次hash计算，由此来确定这个值在table中的位置。如果这个位置是空的，那么直接放入，而且不需要加锁操作。
> 如果这个位置存在结点，说明发生了hash碰撞，首先进入sychnorized同步代码块，然后判断这个节点的类型。如果是链表节点（fh>0）,则得到的结点就是hash值相同的节点组成的链表的头节点。需要依次向后遍历确定这个新加入的值所在位置。如果遇到hash值与key值都与新加入节点是一致的情况，则只需要更新value值即可。否则依次向后遍历，直到链表尾插入这个结点。  如果加入这个节点以后链表长度大于8，就把这个链表转换成红黑树。如果这个节点的类型已经是树节点的话，直接调用树节点的插入方法进行插入新的值。

#### get方法
> get方法比较简单，给定一个key来确定value的时候，必须满足两个条件  key相同  hash值相同，对于节点可能在链表或树上的情况，需要分别去查找。
