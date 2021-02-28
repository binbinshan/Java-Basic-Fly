## 集合类中的 List 和 Map 的线程安全版本是什么，如何保证线程安全的？

List线程安全的有：
* Vector：通过synchronized关键字给方法加锁，保证线程安全
* CopyOnWriteArrayList：通过给写操作添加互斥锁ReentrantLock，保证线程安全

Map线程安全的有：
* ConcurrentHashMap：jdk1.7通过使用分段锁，jdk1.8使用数组+链表+红黑树，CAS，保证线程安全
