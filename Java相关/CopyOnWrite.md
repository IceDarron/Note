Q:CopyOnWrite
===
### CopyOnWrite简介
Copy-On-Write简称COW，是一种用于程序设计中的优化策略。
CopyOnWrite容器即写时复制的容器。
通俗的理解是当我们往一个容器添加元素的时候，不直接往当前容器添加，
而是先将当前容器进行Copy，复制出一个新的容器，然后新的容器里添加元素，添加完元素之后，
再将原容器的引用指向新的容器。
这样做的好处是我们可以对CopyOnWrite容器进行并发的读，而不需要加锁，因为当前容器不会添加任何元素。
所以CopyOnWrite容器也是一种读写分离的思想，读和写不同的容器。

从JDK1.5开始Java并发包里提供了两个使用CopyOnWrite机制实现的并发容器,
它们是CopyOnWriteArrayList和CopyOnWriteArraySet。

### CopyOnWriteArrayList的实现原理
向ArrayList里添加元素，可以发现在添加的时候是需要加锁的，否则多线程写的时候会Copy出N个副本出来。
```java
package com.darron.cow;

import java.util.Map;

public class CopyOnWriteArrayList {

    public boolean add(T e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {

            Object[] elements = getArray();

            int len = elements.length;
            // 复制出新数组

            Object[] newElements = Arrays.copyOf(elements, len + 1);
            // 把新元素添加到新数组里

            newElements[len] = e;
            // 把原数组引用指向新数组

            setArray(newElements);

            return true;

        } finally {

            lock.unlock();

        }

    }

    final void setArray(Object[] a) {
        array = a;
    }
}
```

读的时候不需要加锁，如果读的时候有多个线程正在向ArrayList添加数据，读还是会读到旧的数据，
因为写的时候不会锁住旧的ArrayList。
```java
package com.darron.cow;

import java.util.Map;

public class CopyOnWriteArrayList {
    public E get(int index) {
        return get(getArray(), index);
    }
}
```

### CopyOnWrite的应用场景
CopyOnWrite并发容器用于读多写少的并发场景。比如白名单，黑名单，商品类目的访问和更新场景，
假如我们有一个搜索网站，用户在这个网站的搜索框中，输入关键字搜索内容，但是某些关键字不允许被搜索。
这些不能被搜索的关键字会被放在一个黑名单当中，黑名单每天晚上更新一次。
当用户搜索时，会检查当前关键字在不在黑名单当中，如果在，则提示不能搜索。
```java
package com.darron.cow;

import java.util.Map;

import com.ifeve.book.forkjoin.CopyOnWriteMap;

/**
 * 黑名单服务
 *
 * @author fangtengfei
 *
 */
public class BlackListServiceImpl {

    private static CopyOnWriteMap<String, Boolean> blackListMap = new CopyOnWriteMap<String, Boolean>(
            1000);

    public static boolean isBlackList(String id) {
        return blackListMap.get(id) == null ? false : true;
    }

    public static void addBlackList(String id) {
        blackListMap.put(id, Boolean.TRUE);
    }

    /**
     * 批量添加黑名单
     *
     * @param ids
     */
    public static void addBlackList(Map<String,Boolean> ids) {
        blackListMap.putAll(ids);
    }

}
```

### 影响性能的两点
+ 减少扩容开销。根据实际需要，初始化CopyOnWriteMap的大小，避免写时CopyOnWriteMap扩容的开销。
+ 使用批量添加。因为每次添加，容器每次都会进行复制，所以减少添加次数，可以减少容器的复制次数。

### 缺点
#### 内存占用问题
因为CopyOnWrite的写时复制机制，所以在进行写操作的时候，内存里会同时驻扎两个对象的内存，
旧的对象和新写入的对象（注意:在复制的时候只是复制容器里的引用，只是在写的时候会创建新对象添加到新容器里，而旧容器的对象还在使用，所以有两份对象内存）。
如果这些对象占用的内存比较大，比如说200M左右，那么再写入100M数据进去，内存就会占用300M，那么这个时候很有可能造成频繁的Yong GC和Full GC。
之前我们系统中使用了一个服务由于每晚使用CopyOnWrite机制更新大对象，造成了每晚15秒的Full GC，应用响应时间也随之变长。

针对内存占用问题，可以通过压缩容器中的元素的方法来减少大对象的内存消耗，
比如，如果元素全是10进制的数字，可以考虑把它压缩成36进制或64进制。
或者不使用CopyOnWrite容器，而使用其他的并发容器，如ConcurrentHashMap。

#### 数据一致性问题
CopyOnWrite容器只能保证数据的最终一致性，不能保证数据的实时一致性。
所以如果你希望写入的的数据，马上能读到，请不要使用CopyOnWrite容器。