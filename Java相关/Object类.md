Q:Object类
===
Object类是所有Java类的祖先。每个类都使用 Object 作为超类。所有对象（包括数组）都实现这个类的方法。

+ Object() 默认构造方法
+ clone() 创建并返回此对象的一个副本。
+ equals(Object obj) 指示某个其他对象是否与此对象“相等”。
+ finalize() 当垃圾回收器确定不存在对该对象的更多引用时，由对象的垃圾回收器调用此方法。
+ getClass() 返回一个对象的运行时类。
+ hashCode() 返回该对象的哈希码值。
+ notify() 唤醒在此对象监视器上等待的单个线程。
+ notifyAll() 唤醒在此对象监视器上等待的所有线程。
+ toString() 返回该对象的字符串表示。
+ wait() 导致当前的线程等待，直到其他线程调用此对象的 notify() 方法或 notifyAll() 方法。
+ wait(long timeout) 导致当前的线程等待，直到其他线程调用此对象的 notify() 方法或 notifyAll() 方法，或者超过指定的时间量。
+ wait(long timeout, int nanos) 导致当前的线程等待，直到其他线程调用此对象的 notify() 方法或 notifyAll() 方法，或者其他某个线程中断当前线程，或者已超过某个实际时间量。

