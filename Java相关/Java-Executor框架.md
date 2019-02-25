Q:Java并发编程(Executor框架)
===
Eexecutor作为灵活且强大的异步执行框架，其支持多种不同类型的任务执行策略，提供了一种标准的方法将任务的提交过程和执行过程解耦开发，基于生产者-消费者模式，其提交任务的线程相当于生产者，执行任务的线程相当于消费者，并用Runnable来表示任务，Executor的实现还提供了对生命周期的支持，以及统计信息收集，应用程序管理机制和性能监视等机制。

![Executor的UML图](https://github.com/IceDarron/Note/blob/master/Image/Executor_UML.png)

+ Executor：一个接口，其定义了一个接收Runnable对象的方法executor，其方法签名为executor(Runnable command),
+ ExecutorService：是一个比Executor使用更广泛的子类接口，其提供了生命周期管理的方法，以及可跟踪一个或多个异步任务执行状况返回Future的方法
+ AbstractExecutorService：ExecutorService执行方法的默认实现
+ ScheduledExecutorService：一个可定时调度任务的接口
+ ScheduledThreadPoolExecutor：ScheduledExecutorService的实现，一个可定时调度任务的线程池
+ ThreadPoolExecutor：线程池，可以通过调用Executors以静态工厂方式来创建线程池并返回一个ExecutorService对象

### Executor的生命周期
ExecutorService提供了管理Eecutor生命周期的方法，ExecutorService的生命周期包括了：运行  关闭和终止三种状态。

ExecutorService在初始化创建时处于运行状态。

shutdown方法等待提交的任务执行完成并不再接受新任务，在完成全部提交的任务后关闭

shutdownNow方法将强制终止所有运行中的任务并不再允许提交新任务

### ExecutorService 的submit（） 与execute（）区别 
+ 接收的参数不一样 submit（）可以接受runnable和callable  有返回值。execute（）接受runnable 无返回值
+ submit有返回值，而execute没有
+ submit方便Exception处理

### Executors
提供了一系列静态工厂方法用于创建各种线程池

### 通过Executors提供四种线程池
newCachedThreadPool创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。 

newFixedThreadPool 创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。 

newScheduledThreadPool 创建一个定长线程池，支持定时及周期性任务执行。 

newSingleThreadExecutor 创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。

### new Thread的弊端
+ 每次new Thread新建对象性能差。
+ 线程缺乏统一管理，可能无限制新建线程，相互之间竞争，及可能占用过多系统资源导致死机或oom。
+ 缺乏更多功能，如定时执行、定期执行、线程中断。
相比new Thread，Java提供的四种线程池的好处在于：
+ 重用存在的线程，减少对象创建、消亡的开销，性能佳。
+ 可有效控制最大并发线程数，提高系统资源的使用率，同时避免过多资源竞争，避免堵塞。
+ 提供定时执行、定期执行、单线程、并发数控制等功能。
