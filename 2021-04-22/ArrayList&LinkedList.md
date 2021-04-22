# 简述 ArrayList 与 LinkedList 的底层实现以及常见操作的时间复杂度 

ArrayList底层实现是数组，支持随机访问。其主要方法时间复杂度：
* add（）： 需要花费O(1)的时间
* add（index，element） ：平均以 O(n)时间运行
* get（） ：始终是固定时间的O(1)操作
* remove（）：以线性O（n）时间运行。必须迭代整个数组以找到符合删除条件的元素

LinkedList底层实现是链表，其主要时间复杂度：
* add（）：任何位置支持O(1)的时间插入
* get（）：搜索元素需要O(n)的时间
* remove（）：删除元素支持O(1)的时间

## ArrayList和LinkedList的区别？
1. ArrayList底层实现是动态数组，LinkedList底层实现是链表
2. ArrayList对于获取元素操作get比LinkedList更快。
3. 对于新增和删除操作add和remove，LinedList比较占优势，因为ArrayList要移动数据