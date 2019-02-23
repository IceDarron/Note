Q:jvm运行时数据区。
===
![Image text](https://github.com/IceDarron/Note/blob/master/Image/jvm_data_zone.png)
### 程序计数器
>程序计数器是一块较小的内存空间，每个线程私有，作为当前线程所执行的字节码行号指示器。且如果执行java方法计数器记录，如果执行native方法则计数器值为空。
### java虚拟机栈
>用于描述java方法执行的内存模型，每个方法执行的时候创建一个栈帧，用于存储局部变量，操作数栈，动态链接，方法出口信息等。每一个方法调用至完成的过程，就对应着一个栈帧在虚拟机栈中入栈到出栈的过程。局部变量可以存储八大基本数据类型及对象引用类型。
### 本地方法栈
>相对于java虚拟机栈，本栈调用的方法为native方法服务。
### java堆
>java虚拟机管理的最大一块内存，所有线程共享，在虚拟机启动的时候创建，只用来存放对象实例。也是GC关注的主要区域。可以分为新生代（Eden，From survivor， To Survivor）和老年代。堆可以处于物理上不连续的内存空间中，只要逻辑上连续即可。

![Image text](https://github.com/IceDarron/Note/blob/master/Image/jvm_heap_model.png)

### 方法区
>所有线程共享，存放虚拟机加载的类信息，常量，静态变量，即时编译器编译后的代码等数据。
### 运行时常量池
>这部分内容在来加载后进入方法区。

### 直接内存
>是一种堆外内存，通过一个存储在java堆中的DirecByteBuffer对象作为这个内存的引用进行操作，是由native函数库直接分配的内存，这样可以减少java堆和native堆之间来回复制数据。

Q:Java内存模型
===
之前写过的堆栈程序计数器新生代老年代，属于java运行时数据区域，还不算是抽象的内存模型。

java虚拟机内存模型相当于将cpu高速缓存区和内存进行抽象。

主要分为工作内存和主内存。

八大操作：lock unlock read load use assign store write。

volatile保证了可见性和有序性，不保证原子性。主要原因个人理解为，volatile能保证（load、read）（store、write）动作相关联必须连续出现。但是同时执行use后，其中一个已经执行了store及write后，另一个线程后续在执行时会将小的数据覆盖会主内存。

happens-before先行发生原则。

Q:对象创建
===
![Image text](https://github.com/IceDarron/Note/blob/master/Image/jvm_create_object.png)
创建的关键字为new。当虚拟机遇到一个new指令时，会进行一下操作：
1. 检查指令参数是否在常量池中可以定位到一个类的符号引用，并检查所代表的类是否已经被加载、解析和初始化过，如果没有则先执行相应的类加载过程。
2. 为新生对象分配内存。有两种方式：指正碰撞（堆内存绝对规整，通过移动分界指针分配内存），空闲列表（堆内存不规整，维护一个列表，记录内存可用地方）。堆是否规整由所采用的垃圾收集器是否带有压缩整理功能决定。由于堆是线程共享，所以需要考虑分配内存是，多线程问题。一是通过CAS加失败重试的方法保证分配内存空间的操作的原子性。二是通过对每个线程预先分配堆内存的缓冲区。
3. 对分配完成后的内存空间进行初始化为零值。
4. 虚拟机对对象进行必要设置，主要设置对象内存头的信息。
5. 虚拟机已经产生一个新的对象，最后执行程序编写中的init方法。

Q:对象的内存布局
===
对象在内存中存储的布局分为三块区域：对象头，实例数据，对齐填充。
对象头：分为两部分，第一部分用于存储对象自身的运行时数据，如哈希吗，GC分代年龄，锁状态等。第二部分是类型指针，即对象指向它的类元数据的指针。
实例数据：对象真正存储的有效信息，即程序中定义的各种类型字段内容。无论父类继承还是子类定义，都记录。
对齐填充：自动内存管理系统要求对象起始地址必须是8字节的倍数。

Q:对象的访问定位
===
![Image text](https://github.com/IceDarron/Note/blob/master/Image/jvm_obj_visit.png)
栈中存放的引用类型，只规定了一个指向对象的引用，具体如何定位，由虚拟机实现，主要有两种方式：使用句柄，直接指针。（句柄可以理解为指向指针的指针，维护指向对象的指针的变化，对象的句柄本身不变化）

Q:对象是否可回收
===
1. 引用计数算法：通过计数器，引用+1，失效-1，为0时认为不可用。简单高效，但是很难解决对象之间相互循环引用的问题。
2. 可达性分析算法：通过GC Roots对象作为起点，向下搜索形成引用链，如果一个对象到GC Roots之间没有引用链，则认为不可用。GC 
3. Roots包括：虚拟机栈中引用的对象，方法去中类静态属性引用的对象，方法去中常量引用的对象，native方法引用的对象。
4. 可达性分析算法需要至少经历两次标记过程，如果不可达则进行标记并进行筛选（筛选条件为是否有必要执行finalize()方法，当没有重写或已经被虚拟机调用了则认为是没有必要执行），对于需要执行的情况下，则在finalize()方法中可以重新与引用链上的任意对象项链，将会被移除出即将回收的集合，从而避免回收。需要注意的是，任何一个对象的finalize方法只会被系统自动调用一次。不建议这么使用finalize方法，甚至不建议在程序中使用该方法。

Q:引用
===
+ 强引用：例如Object obj = new Object()，强引用存在，则永远不回收。
+ 软引用：通过jdk中SoftReference实现，在发生内存溢出异常之前，进行二次回收。
+ 弱引用：通过jdk中WeakReference实现，只生存到下次垃圾收集发生之前。无论内存是否足够，垃圾收集发生后都回收。
+ 虚引用：通过jdk中PhantomReference实现，无发通过该引用获取一个对象实例，唯一的目的是用来在回收时收到一条系统消息。

Q:垃圾收集算法
===
1. 标记-清除算法
先标记后清除，标记的时候会停顿所有线程。最基础的收集算法，但是效率较低，并且回收后的内存空间不连续，存在大连不连续内存碎片。影响分配大对象。
![Image text](https://github.com/IceDarron/Note/blob/master/Image/jvm_gc_mark_sweep.png)
2. 复制算法
内存分为两部分，运行时使用其中一部分，当发起回收时，将存活对象复制到空白内存部分，并清除原先使用的部分。简单高效，空间连续。但是损失内存空间。
经IBM研究新生代98%的对象朝生夕死，所以将new内存分为8:1:1（Eden，From Survivor，To Survivor），每次使用Eden和From Survivor，回收时将存活的对象放到To Survivor上（据说清理后会将To与From交换，保证每次GC后使用的都是Eden和From，回收复制时放置的地方总是To），并清理其他部分。如果Survivor空间不足则需要依赖old部分进行分配担保。
![Image text](https://github.com/IceDarron/Note/blob/master/Image/jvm_gc_copying.png)
3. 标记-整理算法
在标记-清除算法基础上，在标记阶段后不直接清除，而是让所有存活对象都向一端移动（整理），然后清除掉整理边界以外的内存。
![Image text](https://github.com/IceDarron/Note/blob/master/Image/jvm_gc_mark_compact.png)
4. 分代收集算法
是一种针对将内存分为不同区域，各个区域使用自己合适的收集算法，通常java堆分为new和old（新生代，老年代），new使用复制算法，old使用标记-xx算法。


Q:HotSpot的算法实现
===
1. 枚举根节点
为可达性分析做准备，必须停顿所有java执行线程，通过一组成为OopMap的数据结构得知哪些地方存放着对象引用。
2. 安全点
对于达到安全点的地方记录指令并存入OopMap中。通常有抢先式中断（基本已弃用）和主动式中断。抢先式中断：GC发生，首先发所有线程中断，发现中断的地方不在安全点则回复线程并跑到安全点。主动式中断，设置一个标志位，当需要中断时修改标志位，并让线程主动轮询标志位，在中断。标志位本身与安全点重合。
3. 安全区域
对于处于sleep和block的线程无法响应jvm中断请求，这些线程会进入安全区域，安全区域线程可以GC。当离开安全区域时，需要检查系统是否完成枚举根节点或GC，如果完成则线程继续执行，否则等待。


Q:垃圾收集器
===
HotSpot垃圾收集器关系图：
![Image text](https://github.com/IceDarron/Note/blob/master/Image/jvm_generation.png)
#### Serial：
>最基本，发展历史最悠久的单线程收集器，运行时必须暂停其他所有的工作线程，直到收集结束。但是简单高效，由于单线程没有线程交互开销，从而获得最高的单线程收集效率。
#### ParNew：
>与Serial基本相同，只是拥有了多线程收集能力。
#### Parallel Scavenge：
>设计出发点与其他收集器较为不同，它的目标是达到一个可控制的吞吐量，即CPU用于运行用户代码的时间与CPU总消耗时间的比值（其实就是从整体的视角来看，降低收集的停顿总时间，从而提高CPU使用率）。除此之外，它还拥有GC自适应的调节策略，动态调整参数以提供最适合的停顿时间或吞吐量。从多线程和回收算法来看与ParNew并无不同。
#### Serial Old：
>Serial的老年代版本，同样是单线程，用于搭配Paralle Scavenge和为CMS提供后备方案。但是它使用的标记-整理算法。
#### Parallel Old：
>Parallel Scavenge的老年代版本，多线程。主要是为了配合Parallel Scavenge，替代Serial Old。从而提供一个吞吐量优先的收集器组合。但是它使用的标记-整理算法。
#### CMS：
>是一种获取最短停顿时间的收集器，多用于互联网或者BS系统的服务端，重视响应速度，也是第一个并发收集器。基于标记-清除算法，分为四个阶段（初始标记，并发标记，重新标记，并发清除）。缺点在于对CPU资源敏感，无法处理浮动垃圾（即在并发清理阶段用户线程运行时出现的垃圾，并且存在预留空间不足而回收失败需要启用Serial Old回收垃圾），标记-清除算法的通病产生大量不连续空间碎片。
#### G1：
+ 并行与并发：多线程可以充分利用CPU性能，且GC时java程序仍旧可运行。
+ 分代收集：可以不需要配合其它收集器，自身可以采用不同的方式处理各个阶段的对象。
+ 空间整合：不会产生内存碎片。
+ 可预测的停顿：实时java（RTSJ）的一个特征，可以在指定M毫秒时间段内回收时间不超过N毫秒。
>java内存布局与其他收集器有很大区别，将整个java堆划分为多个大小相同的独立区域（region），新生代和老年代已经不是物理隔离，都是一部分可以不连续的region的集合。G1跟踪每个region的回收价值，在后台维护一个优先列表，回收价值最大的region。
>存在Remembered Set维护操作，用来保证回收时做可达性分析判断时不需要对整个java堆进行扫描。除去该操作G1共分为以下几步：初始标记，并发标记，最终标记，筛选回收。
#### GC日志：
>每个收集器的日志格式不一样，但虚拟机设计时尽量将各个收集器的日志维持了一定的共性。日志格式大致为：
+ 100.661 : [Full GC [Tenured : 0K->210(10240K), 0.149142 secs] 4603K->210K(19456K), [Perm : 2999K->2999K(21245K)], 0.0150007 secs] [Times:user=0.01 sys=0.00, real=0.02 secs]
+ 时间：[停顿类型 [GC发生区域 ： GC前内存已使用容量->GC后内存已使用容量(内存总容量)] GC前java堆已用容量->GC后java堆已用容量(堆总容量), 内存区域GC所占用时间][具体时间：用户态消耗cpu时间，内核态消耗cpu时间，操作从开始到结束所经过的墙钟时间] 
#### 其他部分
+ 停顿时间越短就越适合需要与用户交互的程序。而高吞吐量则高效利用CPU适合多后台运算少交互的任务。
+ 自动内存管理主要为对象分配内存和回收分配给对象的内存。对象的内存分配，大方向来看就是在堆上分配，且主要就是在Eden。这里介绍下：新生代GC(Minor GC)频分但回收速度快。老年代GC(Major GC/Full GC)速度较于Minor GC慢10倍以上。
+ 大对象：需要大量连续内存空间的java对象，最典型的大对象就是那种很长的字符串及数组。通常直接进入老年代。
+ 通过对象年龄计数器可以判断长期存活的对象是否将进入老年代，通常在Eden生成的对象在第一次Minor GC后仍然存活并被Survivor容纳的话将被移动到Survivor中并加一年龄。默认15岁进入老年代。当然也可以动态对象年龄判断，如果在Survior中相同年龄所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象直接进入老年代。
+ 空间分配担保：主要由于新生代采用复制算法，在某些情况下可能会空间不足，需要老年代担保才能进行Minor GC 。如果担保后仍然空间不足，那么需要进行一次Full GC 。通过前一次回收晋升老年代对象的容量的平均值作为经验，来判断是否担保成功。
![Image text](https://github.com/IceDarron/Note/blob/master/Image/jvm_serial&serial_old.png)
![Image text](https://github.com/IceDarron/Note/blob/master/Image/jvm_parnew&serial_old.png)
![Image text](https://github.com/IceDarron/Note/blob/master/Image/jvm_parallel_scavenge&parallel_old.png)
![Image text](https://github.com/IceDarron/Note/blob/master/Image/jvm_cms.png)
![Image text](https://github.com/IceDarron/Note/blob/master/Image/jvm_g1.png)


Q:JVM参数
===
### 堆设置 
-Xms:初始堆大小 
-Xmx:最大堆大小 
-XX:NewSize=n:设置年轻代大小 
-XX:NewRatio=n:设置年轻代和年老代的比值.如:为3,表示年轻代与年老代比值为1:3,年轻代占整个年轻代年老代和的1/4 
-XX:SurvivorRatio=n:年轻代中Eden区与两个Survivor区的比值.注意Survivor区有两个.如:3,表示Eden:Survivor=3:2,一个Survivor区占整个年轻代的1/5 
-XX:MaxPermSize=n:设置持久代大小 
### 收集器设置 
-XX:+UseSerialGC:设置串行收集器 
-XX:+UseParallelGC:设置并行收集器 
-XX:+UseParalledlOldGC:设置并行年老代收集器 
-XX:+UseConcMarkSweepGC:设置并发收集器 
### 垃圾回收统计信息 
-XX:+PrintGC 
-XX:+PrintGCDetails 
-XX:+PrintGCTimeStamps 
-Xloggc:filename 
### 并行收集器设置 
-XX:ParallelGCThreads=n:设置并行收集器收集时使用的CPU数.并行收集线程数. 
-XX:MaxGCPauseMillis=n:设置并行收集最大暂停时间 
-XX:GCTimeRatio=n:设置垃圾回收时间占程序运行时间的百分比.公式为1/(1+n) 
### 并发收集器设置 
-XX:+CMSIncrementalMode:设置为增量模式.适用于单CPU情况. 
-XX:ParallelGCThreads=n:设置并发收集器年轻代收集方式为并行收集时,使用的CPU数.并行收集线程数. 


Q:Java类加载机制
===
类的加载指的是将类的.class文件中的二进制数据读入到内存中，将其放在运行时数据区的方法区内，然后在堆区创建一个java.lang.Class对象，用来封装类在方法区内的数据结构。类的加载的最终产品是位于堆区中的Class对象，Class对象封装了类在方法区内的数据结构，并且向Java程序员提供了访问方法区内的数据结构的接口。

### 类的加载过程
![Image text](https://github.com/IceDarron/Note/blob/master/Image/jvm_classload_process.png)

#### 装载：查找并加载类的二进制数据（查找和导入Class文件）
加载是类加载过程的第一个阶段，在加载阶段，虚拟机需要完成以下三件事情：

+ 通过一个类的全限定名来获取其定义的二进制字节流。
+ 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。
+ 在Java堆中生成一个代表这个类的java.lang.Class对象，作为对方法区中这些数据的访问入口。

相对于类加载的其他阶段而言，加载阶段（准确地说，是加载阶段获取类的二进制字节流的动作）是可控性最强的阶段，因为开发人员既可以使用系统提供的类加载器来完成加载，也可以自定义自己的类加载器来完成加载。

加载阶段完成后，虚拟机外部的 二进制字节流就按照虚拟机所需的格式存储在方法区之中，而且在Java堆中也创建一个java.lang.Class类的对象，这样便可以通过该对象访问方法区中的这些数据。

#### 链接（分3个步骤）
##### 验证：确保被加载的类的正确性
验证是连接阶段的第一步，这一阶段的目的是为了确保Class文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全。验证阶段大致会完成4个阶段的检验动作：

+ 文件格式验证：验证字节流是否符合Class文件格式的规范；例如：是否以0xCAFEBABE开头、主次版本号是否在当前虚拟机的处理范围之内、常量池中的常量是否有不被支持的类型。
+ 元数据验证：对字节码描述的信息进行语义分析（注意：对比javac编译阶段的语义分析），以保证其描述的信息符合Java语言规范的要求；例如：这个类是否有父类，除了java.lang.Object之外。
+ 字节码验证：通过数据流和控制流分析，确定程序语义是合法的、符合逻辑的。
+ 符号引用验证：确保解析动作能正确执行。

验证阶段是非常重要的，但不是必须的，它对程序运行期没有影响，如果所引用的类经过反复验证，那么可以考虑采用-Xverifynone参数来关闭大部分的类验证措施，以缩短虚拟机类加载的时间。

##### 准备：为类的静态变量分配内存，并将其初始化为默认值
准备阶段是正式为类变量分配内存并设置类变量初始值的阶段，这些内存都将在方法区中分配。对于该阶段有以下几点需要注意：

+ 这时候进行内存分配的仅包括类变量（static），而不包括实例变量，实例变量会在对象实例化时随着对象一块分配在Java堆中。
+ 这里所设置的初始值通常情况下是数据类型默认的零值（如0、0L、null、false等），而不是被在Java代码中被显式地赋予的值。

假设一个类变量的定义为：public static int value = 3； 那么变量value在准备阶段过后的初始值为0，而不是3，因为这时候尚未开始执行任何Java方法，而把value赋值为3的putstatic指令是在程序编译后，存放于类构造器<clinit>（）方法之中的，所以把value赋值为3的动作将在初始化阶段才会执行。

##### 解析：把类中的符号引用转换为直接引用
解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程，解析动作主要针对类或接口、字段、类方法、接口方法、方法类型、方法句柄和调用限定符7类符号引用进行。符号引用就是一组符号来描述目标，可以是任何字面量。
直接引用就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄。

#### 初始化：对类的静态变量，静态代码块执行初始化操作
初始化，为类的静态变量赋予正确的初始值，JVM负责对类进行初始化，主要对类变量进行初始化。在Java中对类变量进行初始值设定有两种方式：

+ 声明类变量是指定初始值。
+ 使用静态代码块为类变量指定初始值。

### 加载器
JVM的类加载是通过ClassLoader及其子类来完成的，类的层次关系和加载顺序可以由下图来描述：

![Image text](https://github.com/IceDarron/Note/blob/master/Image/jvm_classloader.png)

+ Bootstrap ClassLoader 负责加载$JAVA_HOME中 jre/lib/rt.jar 里所有的class或Xbootclassoath选项指定的jar包。由C++实现，不是ClassLoader子类。
+ Extension ClassLoader 负责加载java平台中扩展功能的一些jar包，包括$JAVA_HOME中jre/lib/*.jar 或 -Djava.ext.dirs指定目录下的jar包。
+ App ClassLoader 负责加载classpath中指定的jar包及 Djava.class.path 所指定目录下的类和jar包。
+ Custom ClassLoader 通过java.lang.ClassLoader的子类自定义加载class，属于应用程序根据自身需要自定义的ClassLoader，如tomcat、jboss都会根据j2ee规范自行实现ClassLoader。

加载过程中会先检查类是否被已加载，检查顺序是自底向上，从Custom ClassLoader到BootStrap ClassLoader逐层检查，
只要某个classloader已加载，就视为已加载此类，保证此类只所有ClassLoader加载一次。而加载的顺序是自顶向下，
也就是由上层来逐层尝试加载此类。

类加载器虽然只用于实现类的加载动作，但是对于任意一个类，都需要由加载它的类加载器和这个类本身共同确立其在Java虚拟机中的唯一性。
通俗的说，JVM中两个类是否“相等”，首先就必须是同一个类加载器加载的，否则，
即使这两个类来源于同一个Class文件，被同一个虚拟机加载，只要类加载器不同，那么这两个类必定是不相等的。

### 双亲委派模型
从Java虚拟机的角度来说，只存在两种不同的类加载器：一种是启动类加载器（Bootstrap ClassLoader），
这个类加载器使用C++语言实现（HotSpot虚拟机中），是虚拟机自身的一部分；
另一种就是所有其他的类加载器，这些类加载器都有Java语言实现，独立于虚拟机外部，并且全部继承自java.lang.ClassLoader。

上述四种加载器之间的层次关系被称为类加载器的双亲委派模型。
该模型要求除了顶层的启动类加载器外，其余的类加载器都应该有自己的父类加载器，
而这种父子关系一般通过组合（Composition）关系来实现，而不是通过继承（Inheritance）。

#### 双亲委派模型过程
某个特定的类加载器在接到加载类的请求时，首先将加载任务委托给父类加载器，依次递归，如果父类加载器可以完成类加载任务，就成功返回；只有父类加载器无法完成此加载任务时，才自己去加载。

使用双亲委派模型的好处在于Java类随着它的类加载器一起具备了一种带有优先级的层次关系。
例如类java.lang.Object，它存在在rt.jar中，无论哪一个类加载器要加载这个类，
最终都是委派给处于模型最顶端的Bootstrap ClassLoader进行加载，
因此Object类在程序的各种类加载器环境中都是同一个类。
相反，如果没有双亲委派模型而是由各个类加载器自行加载的话，如果用户编写了一个java.lang.Object的同名类并放在ClassPath中，
那系统中将会出现多个不同的Object类，程序将混乱。
因此，如果开发者尝试编写一个与rt.jar类库中重名的Java类，可以正常编译，但是永远无法被加载运行。

#### 双亲委派模型的系统实现
```java
public class test {
    protected synchronized Class<?> loadClass(String name,boolean resolve)throws ClassNotFoundException{
        //check the class has been loaded or not
        Class c = findLoadedClass(name);
        if(c == null){
            try{
                if(parent != null){
                    c = parent.loadClass(name,false);
                }else{
                    c = findBootstrapClassOrNull(name);
                }
            }catch(ClassNotFoundException e){
                //if throws the exception ,the father can not complete the load
            }
            if(c == null){
                c = findClass(name);
            }
        }
        if(resolve){
            resolveClass(c);
        }
        return c;
    }
}
```

Q:Java内存模型与线程
===
+ Java内存模型的主要目标是定义程序中各个变量的访问规则，即在虚拟机中将变量存储到内存和从内存中取出变量这样的底层细节。变量包括了实例字段、静态字段和构成数组对象的元素，但不包括局部变量与方法参数，因为后者是线程私有的，不被共享。
+ 线程的工作内存中保存了被该线程使用到的变量的主内存副本拷贝（事实上，只拷贝在线程中有可能使用到的字段，不会把整个对象拷贝一次）。
+ 线程间变量的传递均需要通过主内存来完成。线程自己的变量均在自己的工作内存中进行，不能直接读写主内存变量，其中volatile修饰的变量，同样拥有工作内存的拷贝，但由于其特殊的操作顺序，可以保证可见性，所以看起来如同直接在主内存中读写访问。
+ 简单的说，主内存主要对应Java堆中的对象实例数据部分，工作内存则对应于虚拟机栈中的部分区域。
+ 内存模型中有八大操作，每个操作都是具有原子性：lock，unlock，read，load，use，assign，store，write。
+ volatile具有可见性和禁止指令重排序优化的特性，但不能保证并发下线程安全。
+ 并发过程中的三大特性：原子性，可见性，有序性。
+ 先行发生原则：判断数据是否存在竞争，线程是否安全的主要依据。
+ 线程的实现：内核线程，用户线程，用户线程及轻量级进程混合。
+ 线程调度：协同式线程调度，抢占式线程调度。
+ 线程状态：新建，运行，无限期等待和限期等待，阻塞，结束。


Q:JVM内存结构 VS Java内存模型 VS Java对象模型
===
https://www.hollischuang.com/archives/2509

Q:JVM内存模型操作
===
https://blog.csdn.net/l1394049664/article/details/81475380
