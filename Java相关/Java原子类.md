Q：java源码（原子类java.util.concurrent.atomic）
===
>这个包提供了一系列原子类。这些类可以保证多线程环境下，当某个线程在执行atomic的方法时，不会被其他线程打断，而别的线程就像自旋锁一样，一直等到该方法执行完成，才由JVM从等待队列中选择一个线程执行。Atomic类在软件层面上是非阻塞的，它的原子性其实是在硬件层面上借助相关的指令来保证的。
#### Atomic包中的类可以分成4组：
+ AtomicBoolean，AtomicInteger，AtomicLong，AtomicReference
+ AtomicIntegerArray，AtomicLongArray
+ AtomicLongFieldUpdater，AtomicIntegerFieldUpdater，AtomicReferenceFieldUpdater
+ AtomicMarkableReference，AtomicStampedReference，AtomicReferenceArray
```java
public class test {
    public final int incrementAndGet() {  
        for (;;) {  
            int current = get();  
            int next = current + 1;  
            if (compareAndSet(current, next))  
                return next;  
        }  
    }  
}
```
>方法中采用了CAS操作，每次从内存中读取数据然后将此数据和+1后的结果进行CAS操作，如果成功就返回结果，否则重试直到成功为止。而compareAndSet利用JNI来完成CPU指令的操作。整体的过程就是这样子的，利用CPU的CAS指令，同时借助JNI来完成Java的非阻塞算法。

#### CAS的优缺点
>CAS由于是在硬件层面保证的原子性，不会锁住当前线程，它的效率是很高的。 
CAS虽然很高效的实现了原子操作，但是它依然存在三个问题。
+ ABA问题。CAS在操作值的时候检查值是否已经变化，没有变化的情况下才会进行更新。但是如果一个值原来是A，变成B，又变成A，那么CAS进行检查时会认为这个值没有变化，但是实际上却变化了。ABA问题的解决方法是使用版本号。在变量前面追加上版本号，每次变量更新的时候把版本号加一，那么A－B－A 就变成1A-2B－3A。从Java1.5开始JDK的atomic包里提供了一个类AtomicStampedReference来解决ABA问题。
+ 并发越高，失败的次数会越多，CAS如果长时间不成功，会极大的增加CPU的开销。因此CAS不适合竞争十分频繁的场景。
+ 只能保证一个共享变量的原子操作。当对多个共享变量操作时，CAS就无法保证操作的原子性，这时就可以用锁，或者把多个共享变量合并成一个共享变量来操作。比如有两个共享变量i＝2,j=a，合并一下ij=2a，然后用CAS来操作ij。从Java1.5开始JDK提供了AtomicReference类来保证引用对象的原子性，你可以把多个变量放在一个对象里来进行CAS操作。
