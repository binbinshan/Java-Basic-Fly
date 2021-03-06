## synchronized 关键字底层是如何实现的？它与 ReentrantLock 相比优缺点分别是什么？

### 实现原理
synchronized修饰的同步语句块，实现是使用monitorenter和monitorexit指令，其中monitorenter指向同步代码块开始位置，monitorexit指向同步代码块结束的位置
synchronized修饰的方法，则是用acc_synchronized 来标记，该标识指明了该方法是一个同步方法，两者的本质都是对对象监视器 monitor 的获取。

### Synchronized执行流程：
判断Mark word里是否是当前线程，且偏向锁状态为true，如果是，则获得偏向锁。如果不是，那么执行CAS将Mark work修改为当前线程，如果成功，则获得偏向锁，并设置偏向锁为true。否则发生竞争，撤销偏向锁，升级为轻量级锁。
当前线程执行CAS，将mark word替换为锁记录指针。如果成功获得轻量级锁。如果失败，当前线程尝试自旋获取锁，如果获取到，处于轻量级锁。如果一定次数后没获取到锁，锁升级为重量级锁。
重量级锁是独占锁。通过monitor对象实现。monitor enter 进入数+1，当前线程获得锁，如果其他线程已经获得锁，那么该线程阻塞，直到锁计数为0。monitor exit表示锁计数-1，直到进入数为0时，当前线程释放锁。

### 与 ReentrantLock 相比优缺点分别是什么？
实现：synchronized 是 JVM 实现的，是一个关键字 ，ReentrantLock 是基于JDK实现的，实现的是Lock接口；
性能：JDK1.6后对synchronized优化后，两者性能差距不大。
锁：synchronized是非公平锁，而ReentrantLock既可以是公平也可以是非公平。
绑定锁的条件：synchronized只支持一个条件，ReentrantLock则可以支出多个条件。

除非要使用ReentrantLock的高级功能外，基于都是用synchronized，因为synchronized会自动释放锁，不会导致死锁，并且是jvm原生支持的。
