Q:假如有Thread1、Thread2、Thread3、Thread4四条线程分别统计C、D、E、F四个盘的大小，所有线程都统计完毕交给Thread5线程去做汇总，应当如何实现？
===
```java
package com.darron;

import java.util.concurrent.Callable;
import java.util.concurrent.CountDownLatch;

public class TestMultiThread {

    /**
     * 功能实现：核心类java.util.concurrent.CountDownLatch
     * CountDownLatch : 一个线程(或者多个)， 等待另外N个线程完成某个事情之后才能执行。本质上来讲就是一个计数器
     * 其核心还是AQS，在此不多赘述。
     */
    public static void main(String[] args) {

        CountDownLatch countDownLatch = new CountDownLatch(4); // 设置线程数
        ExecutorService executorService = Executors.newCachedThreadPool();//线程池

        Future future0 = executorService.submit(new Worker("c", countDownLatch));
        Future future1 = executorService.submit(new Worker("d", countDownLatch));
        Future future2 = executorService.submit(new Worker("e", countDownLatch));
        Future future3 = executorService.submit(new Worker("f", countDownLatch));

        try {
            countDownLatch.await(); // 等待所有磁盘计算完毕
            executorService.shutdown(); // 提交所有任务
            int total = (int) future0.get() + (int) future1.get() + (int) future2.get() + (int) future3.get();
            System.out.println(total);
        } catch (Exception e) {
            e.printStackTrace();
        }

    }

    // 实现Callable主要是为了能获取返回值
    public static class Worker implements Callable {

        private String cdName;

        private int cdCapacity;

        private CountDownLatch countDownLatch;

        public Worker() {
        }

        public Worker(String cdName, CountDownLatch countDownLatch) {
            this.cdName = cdName;
            this.countDownLatch = countDownLatch;
        }

        @Override
        public Object call() throws Exception {
            System.out.println("start calculate " + cdName);
            cdCapacity = cdName.charAt(0); // 模拟计算磁盘空间大小
            System.out.println("end calculate " + cdName + "capacity " + cdCapacity);
            countDownLatch.countDown(); // 计数器减一
            return cdCapacity;
        }
    }
}
```