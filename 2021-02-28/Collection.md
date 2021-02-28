## 说说 List,Set,Map 三者的区别及如何选用？

* list:存储的元素是有序、可重复的。
* set:存储的元素是无序、不可重复的。
* Map:存储的元素是Key-value形式的，key无序且不可重复，value无序可重复

list:
* ArrayList：底层使用 Object[] 存储,线程不安全
* Vector：底层使用 Object[] 存储，线程安全
* LinkedList：底层使用 双向链表 存储，线程不安全

set:
* HashSet：底层是 HashMap，线程不安全的
* LinkedHashSet：HashSet 的子类，线程不安全的
* TreeSet：底层是红黑树，线程不安全的

Map:
* HashMap：底层是 数组+红黑树/链表 ，线程不安全
* LinkedHashMap：HashMap的子类，线程不安全
* HashTable 底层是 数组+链表 ，线程安全的
* TreeMap：线程不安全的


### 如何选用集合？
主要根据集合的特点来选用，比如我们需要根据键值获取到元素值时就选用 Map 接口下的集合，需要排序时选择 TreeMap,不需要排序时就选择 HashMap,需要保证线程安全就选用 ConcurrentHashMap。

当我们只需要存放元素值时，就选择实现Collection 接口的集合，需要保证元素唯一时选择实现 Set 接口的集合比如 TreeSet 或 HashSet，不需要就选择实现 List 接口的比如 ArrayList 或 LinkedList，然后再根据实现这些接口的集合的特点来选用。