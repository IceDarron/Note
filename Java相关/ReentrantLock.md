Q：ReentrantLock
===
![Image text](https://github.com/IceDarron/Note/blob/master/Image/reentrantLock_uml.png)
>ReentrantLock类在java.util.concurrent.locks包中，它的上一级的包java.util.concurrent主要是常用的并发控制类。
java.util.concurrent.lock 中的 Lock 框架是锁定的一个抽象，它允许把锁定的实现作为 Java 类，而不是作为语言的特性来实现。这就为 Lock 的多种实现留下了空间，各种实现可能有不同的调度算法、性能特性或者锁定语义。 ReentrantLock 类实现了 Lock ，它拥有与 synchronized 相同的并发性和内存语义，但是添加了类似轮询锁、定时锁等候和可中断锁等候的一些特性。此外，它还提供了在激烈争用情况下更佳的性能。（换句话说，当许多线程都想访问共享资源时，JVM 可以花更少的时候来调度线程，把更多时间用在执行线程上。）
ReentrantLock支持两种锁模式，公平锁和非公平锁。默认的实现是非公平的。

