# ReentrantLock和Synchronized的对比?

* 使用方式
    1. Synchronized可以修饰实例方法，静态方法，代码块。自动释放锁。
    2. ReentrantLock一般需要try catch finally语句，在try中获取锁，在finally释放锁。需要手动释放锁。

* 公平和非公平
    1. Synchronized只有非公平锁。
    2. ReentrantLock提供公平和非公平两种锁，默认是非公平的。公平锁通过构造函数传递true表示。

* 可重入锁
    1. Synchronized和ReentrantLock都是可重入的

* 可中断的
    1. Synchronized是不可中断的。
    2. ReentrantLock提供可中断和不可中断两种方式。

* 绑定锁的条件
    1. synchronized只支持一个条件
    2. ReentrantLock则可以支出多个条件。

