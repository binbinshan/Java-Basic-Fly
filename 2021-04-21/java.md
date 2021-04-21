# Java 是如何实现线程安全的，哪些数据结构是线程安全的？

## 什么是线程安全？
不同的线程可以访问相同的资源，而不会 产生错误的行为或产生不可预知的结果。这样就是线程安全。

## 如何实现线程安全？
1. final
    * 当一个类实例的内部状态在构造之后不能被修改时，它的状态在构造后不能更改。 因此，它是线程安全的。

1. ThreadLocal
    * 创建每个线程私有的ThreadLocal，这样每个线程都有自己的字段。
    
1. 同步集合
    * 使用同步包装器来创建线程安全的集合

    ``` 
    Collection<Integer> syncCollection = Collections.synchronizedCollection(new ArrayList<>());
    ``` 
    
1. 并发集合
    * 使用并发集合来创建线程安全的集合，例如ConcurrentHashMap

1. 原子对象
    * 使用原子类对象，例如 AtomicInteger

1. synchronized
    * Synchronized 保证⽅法内部或代码块内部资源（数据）的互斥访问。即同⼀时间、由同⼀个Monitor监视的代码，最多只能有⼀个线程在访问。
    
1. Volatile
    * 保证被 Volatile关键字描述变量的操作具有可见性和有序性（禁止指令重排），但不能保证原子性

2. Lock
    * java.util.concurrent包下的一个接口，定义了一系列的锁操作方法。主要有ReentrantLock，ReentrantReadWriteLock.ReadLock，ReentrantReadWriteLock.WriteLock 实现类。


## 哪些数据结构是线程安全的？

1. HashTable
1. ConcurrentHashMap
1. CopyOnWriteArrayList
1. CopyOnWriteArraySet
1. ConcurrentLinkedQueue
1. Vector
1. StringBuffer

### HashTable
- 线程安全的实现方法：HashTable使用synchronized来修饰方法函数来保证线程安全
- 缺点：多线程运行环境下效率表现非常低

### ConcurrentHashMap
- 线程安全的实现方法：用分段锁（JDK1.7）或者是CAS+Synchronized（JDK1.8）来实现的。
- 优点：不仅保证了多线程运行环境下的数据访问安全性，而且性能上有长足的提升。

### CopyOnWriteArrayList
- 线程安全的实现方法：内部有volatile来保持数组，再加ReentrantLock互斥锁来实现的。
- 当add\update\delete操作的时候需要获取锁。当add元素的时候，先获取锁，再使用Arrays.copyOf()来拷贝新建新的数组，在副本上add元素，然后改变原引用指向新副本，最后再释放锁。
- 适用场景：读操作远远多于写操作的应用。

### ConcurrentLinkedQueue
- 基于链表的无界线程安全队列，按FIFO排序。
- 继承AbstractQueue,通过volatile实现多线程对竞争资源的互斥访问。

