# Lock
> 与synchronized相同，对共享资源提供了并发控制功能，但具备了synchronized一些没有的功能如:设置获取锁的超时时间，手动中断锁，无法知道锁是否被占用等等。
## 1.Lock接口
[源码分析](../Jdk_source_learn/util/locks/Lock.java) 
## 2. 内存可见性
> Lock的加解锁和synchronized有同样的内存语义，也就是说，下一个线程加锁后可以看到前一个线程解锁前发生的所有操作。
## 3. ReentrantLock
> 可重入锁，根据不同的构造函数可以设置公平与非公平。
## 4. ReentrantReadWriteLock
> 读写锁，既可以设置公平与非公平又可以使用读锁定与写锁定。设置公平时所有线程都会排队等待，非公平时根据加锁规则会排队或插队。读锁定时可以多个线程共享读锁，写锁定时只能有一个线程持有该锁。
## 4.1 加锁规则
* 公平锁
> 不允许插队
* 非公平锁
> 写锁可以随时插队
> 读锁仅在等待队列头结点不是想获取写锁的线程的时候可以插队