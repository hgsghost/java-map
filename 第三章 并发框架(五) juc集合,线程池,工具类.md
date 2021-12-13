# 第三章 并发框架(五) juc集合,线程池,工具类

## 3.12 JUC集合

### 3.12.1 ConcurrentHashMap

#### 3.12.1.1 面试题

1. 为什么HashTable慢? 它的并发度是什么? 那么ConcurrentHashMap并发度是什么?

   HashTable是synchronized锁整个类,ConcurrentHashMap是通过锁segment(槽)来实现高并发,默认有16个槽(每个槽类似一个HashTable),HashTable并发度是1,ConcurrentHashMap在jdk1.7是16

2. ConcurrentHashMap在JDK1.7和JDK1.8中实现有什么差别? JDK1.8解決了JDK1.7中什么问题

   1.7中使用分段锁,1.8使用数组+链表+红黑树+Cas

   解决了ConcurrentHashMap并发度的限制

3. ConcurrentHashMap JDK1.7实现的原理是什么?

    分段锁机制

4. ConcurrentHashMap JDK1.8实现的原理是什么? 

   数组+链表+红黑树，CAS

5. ConcurrentHashMap JDK1.7中Segment数(concurrencyLevel)默认值是多少? 为何一旦初始化就不可再扩容?

   默认16,不可扩容原因待补充//TODO

6. ConcurrentHashMap JDK1.7说说其put的机制

   //TODO

7. ConcurrentHashMap JDK1.7是如何扩容的?

   rehash(注：segment 数组不能扩容，扩容是 segment 数组某个位置内部的数组 HashEntry<K,V>[] 进行扩容)

   详情//TODO

8. ConcurrentHashMap JDK1.8是如何扩容的? 

   tryPresize

   详情//TODO

9. ConcurrentHashMap JDK1.8链表转红黑树的时机是什么? 临界值为什么是8?

   //TODO

10. ConcurrentHashMap JDK1.8是如何进行数据迁移的? 

    transfer

    //TODO

### 3.12.2  CopyOnWriteArrayList



