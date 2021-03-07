## 简述 Synchronized，Volatile，可重入锁的不同使用场景及优缺点

1. Synchronized 和 ReentrantLock 
    * 都是可重入锁。
    * Synchronized是非公平锁，ReentrantLock支持非公平/公平锁。
    * Synchronized自动释放锁，ReentrantLock需要手动释放。
    * ReentrantLock支持等待可中断。

1. Synchronized 和 Volatile
    * Synchronized能保证原子性和可见性，Volatile只能保证可见性。
    * Volatile主要保证变量在多个线程的可见性，Synchronized保证了资源在多个线程的同步。