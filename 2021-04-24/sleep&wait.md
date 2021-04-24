# Java 中 sleep() 与 wait() 的区别 

1. sleep方法是属于Thread类，wait方法属于Object类
2. sleep方法没有释放锁，wait方法会让线程释放它持有的所有同步资源
3. sleep方法可以在任何地方使用，wait方法因为会对对象的“锁标志”进行操作，所以它们必须在synchronized函数或synchronized block中进行调用，否则运行时会报错
4. sleep是静态方法，只对是当前线程有效
5. 一个对象调用了wait方法，必须要采用notify()和notifyAll()方法唤醒该线程

如果线程A希望立即结束线程B，则可以对线程B对应的Thread实例调用interrupt方法。如果此刻线程B正在wait/sleep/join，则线程B会立刻抛出InterruptedException，在catch() {} 中直接return即可安全地结束线程。

##  yield() 和 join()

yield（）：暂停当前正在执行的线程对象。  

yield() 是停止当前线程，让同等优先权的线程或更高优先级的线程有执行的机会。如果没有的话，那么yield()方法将不会起作用，并且由可执行状态后马上又被执行。   

join（）是用于在某一个线程的执行过程中调用另一个线程执行，等到被调用的线程执行结束后，再继续执行当前线程。
如：t.join();//主要用于等待t线程运行结束，若无此句，main则会执行完毕，导致结果不可预测。