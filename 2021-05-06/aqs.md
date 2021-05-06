# 简述 Java AQS 的原理以及使用场景
AQS全称Abstract Queue Synchronize抽象队列同步器，比如ReentrantLock，Semaphore，ReentrantReadWriteLock，SynchronousQueue，FutureTask 等底层的实现都是AQS。

AQS中有两个重要的属性分别是state、当前加锁线程 ；
另外还有一个等待队列。

比如现有两个线程获取锁，线程A首先通过cas获取到了锁，更新state为1，当前加锁线程=A。
此时线程B通过cas尝试几次获取锁失败，就会加入到等待队列中。
线程A执行完毕后，此时state =0  当前加锁线程= null ,会唤醒线程B继续通过cas获取锁。如果是非公平锁实现的话，此时来了一个线程C还是会有可能先于线程B执行。如果是公平锁，线程C发现线程B在等待队列中，会把自己加入到等待队列，让线程B先执行