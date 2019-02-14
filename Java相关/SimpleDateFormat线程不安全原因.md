Q:SimpleDateFormat线程不安全原因
===
https://www.cnblogs.com/lion88/p/6426019.html
+ 创建一个共享的SimpleDateFormat实例变量，但是在使用的时候，需要对这个变量进行同步
+ 使用ThreadLocal为每个线程都创建一个线程独享SimpleDateFormat变量
+ 需要的时候创建局部变量