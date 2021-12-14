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

#### 3.12.2.2 代码详细

//TODO

### 3.12.2  CopyOnWriteArrayList

#### 3.12.2.1 面试题

1. 非并发集合的Fail-fast机制

   在非并发集合中(例如ArrayList,HashMap),说的的是集合在**迭代器**迭代的过程中,集合的结构发生改变,会发生Fail-fast 抛出ConcurrentModificationException异常

2. 为什么说ArrayList查询快而增删慢?

   ArrayList基于数组,数组在内存中连续排列,所以可以根据索引下标直接找到目标的内存地址,增删会牵涉到数组中元素的移动,所以查询快增删慢

3. 对比ArrayList说说CopyOnWriteArrayList的增删改查实现原理? 

   ArrayList增删是数组内元素的移动,CopyOnWriteArrayList是基于拷贝,将整个数组拷贝到副本中后将副本设置为对象的arr

4. 弱一致性的迭代器原理是怎么样的

   弱一致性迭代器的核心原理是:

   因为cow是基于复制来增删的,所以cow的当前数组可以保证一直不会改变,迭代器中有个快照变量指向当前数组,当cow进行增删操作,先前获取的迭代器中的快照变量依然指向当时的数组(此时cow中的数组指向拷贝后的数组),进而保证迭代器不会抛出ConcurrentModificationException错误

5. CopyOnWriteArrayList为什么并发安全且性能比Vector好

   性能好是因为:

   Vector在增删改**查**方法中都加了锁

   cow在增删改中加了锁,cow读效率高的多

   安全性好是因为:

   cow使用拷贝的方式(读写分离)解决了迭代器的线程安全问题(Vector可能发生fail-fast)

6. CopyOnWriteArrayList有何缺陷，说说其应用场景

   + 写操作需要拷贝数组,如果数组很大,消耗内存且可能触发gc,影响性能
   + 不能用于实时读场景,写操作需要时间拷贝数组(尤其是数组很大的时候)

#### 3.12.2.2 代码详细

//TODO

### 3.12.3 ConcurrentLinkedQueue

ConcurerntLinkedQueue一个基于链接节点的无界线程安全队列。此队列按照 FIFO(先进先出)原则对元素进行排序。队列的头部是队列中时间最长的元素。队列的尾部 是队列中时间最短的元素。新的元素插入到队列的尾部，队列获取操作从队列头部获得元素。当多个线程共享访问一个公共 collection 时，ConcurrentLinkedQueue是一个恰当的选择。此队列不允许使用null元素。

#### 3.12.3.1 面试题

1. 要想用线程安全的队列有哪些选择?

   Vector

   Collections.synchronizedList(List<T> list)

   ConcurrentLinkedQueue

2. ConcurrentLinkedQueue实现的数据结构

   和LinkedQueue一样使用了单链表结构

3. ConcurrentLinkedQueue底层原理? 全程无锁(CAS)

   节点之间的操作(增删改)都使用了cas

4. ConcurrentLinkedQueue的核心方法有哪些? offer()，poll()，peek()，isEmpty()等队列常用方法

   //TODO

5. ConcurrentLinkedQueue的HOPS(延迟更新的策略)的设计?

   如果让tail永远作为队列的队尾节点，实现的代码量会更少，而且逻辑更易懂。但是，这样做有一个缺点，如果大量的入队操作，每次都要执行CAS进行tail的更新，汇总起来对性能也会是大大的损耗。如果能减少CAS更新的操作，无疑可以大大提升入队的操作效率，所以doug lea大师每间隔1次(tail和队尾节点的距离为1)进行才利用CAS更新tail。对head的更新也是同样的道理，虽然，这样设计会多出在循环中定位队尾节点，但总体来说读的操作效率要远远高于写的性能，因此，多出来的在循环中定位尾节点的操作的性能损耗相对而言是很小的。

6. ConcurrentLinkedQueue适合什么样的使用场景

   ConcurrentLinkedQueue通过无锁来做到了更高的并发量，是个高性能的队列，但是使用场景相对不如阻塞队列常见，毕竟取数据也要不停的去循环，不如阻塞的逻辑好设计，但是在并发量特别大的情况下，是个不错的选择，性能上好很多，而且这个队列的设计也是特别费力，尤其的使用的改良算法和对哨兵的处理。整体的思路都是比较严谨的，这个也是使用了无锁造成的，我们自己使用无锁的条件的话，这个队列是个不错的参考。

#### 3.12.3.2 代码详细

//TODO

### 3.12.4 BlockingQueue

![](resource\BlockingQueue.png)

一个线程将会持续生产新对象并将其插入到队列之中，直到队列达到它所能容纳的临界点。也就是说，它是有限的。如果该阻塞队列到达了其临界点，负责生产的线程将会在往里边插入新对象时发生阻塞。它会一直处于阻塞之中，直到负责消费的线程从队列中拿走一个对象。 负责消费的线程将会一直从该阻塞队列中拿出对象。如果消费线程尝试去从一个空的队列中提取对象的话，这个消费线程将会处于阻塞之中，直到一个生产线程把一个对象丢进队列。

#### 3.12.4.1  面试题

1. 什么是BlockingDeque

   java.util.concurrent 包里的 BlockingDeque 接口表示一个线程安全放入和提取实例的双端队列。

   BlockingDeque 类是一个双端队列，在不能够插入元素时，它将阻塞住试图插入元素的线程；在不能够抽取元素时，它将阻塞住试图抽取的线程。 deque(双端队列) 是 "Double Ended Queue" 的缩写。因此，双端队列是一个你可以从任意一端插入或者抽取元素的队列。

   在线程既是一个队列的生产者又是这个队列的消费者的时候可以使用到 BlockingDeque。如果生产者线程需要在队列的两端都可以插入数据，消费者线程需要在队列的两端都可以移除数据，这个时候也可以使用 BlockingDeque。BlockingDeque 图解:

   ![](resource\BlockingDeque.png)

2. BlockingQueue大家族有哪些? 

   + ArrayBlockingQueue

     ArrayBlockingQueue 是一个有界的阻塞队列，其内部实现是将对象放到一个数组里。有界也就意味着，它不能够存储无限多数量的元素。它有一个同一时间能够存储元素数量的上限。你可以在对其初始化的时候设定这个上限，但之后就无法对这个上限进行修改了(译者注: 因为它是基于数组实现的，也就具有数组的特性: 一旦初始化，大小就无法修改)。 ArrayBlockingQueue 内部以 FIFO(先进先出)的顺序对元素进行存储。队列中的头元素在所有元素之中是放入时间最久的那个，而尾元素则是最短的那个。

   + DelayQueue

     DelayQueue 对元素进行持有直到一个特定的延迟到期。注入其中的元素必须实现 java.util.concurrent.Delayed 接口，该接口定义:

     ```java
     public interface Delayed extends Comparable<Delayed< {
         public long getDelay(TimeUnit timeUnit);
     }
     ```

     DelayQueue 将会在每个元素的 getDelay() 方法返回的值的时间段之后才释放掉该元素。如果返回的是 0 或者负值，延迟将被认为过期，该元素将会在 DelayQueue 的下一次 take  被调用的时候被释放掉。

     Delayed 接口也继承了 java.lang.Comparable 接口，这也就意味着 Delayed 对象之间可以进行对比。这个可能在对 DelayQueue 队列中的元素进行排序时有用，因此它们可以根据过期时间进行有序释放。

   + LinkedBlockingQueue

     LinkedBlockingQueue 内部以一个链式结构(链接节点)对其元素进行存储。如果需要的话，这一链式结构可以选择一个上限。如果没有定义上限，将使用 Integer.MAX_VALUE 作为上限。

     LinkedBlockingQueue 内部以 FIFO(先进先出)的顺序对元素进行存储。队列中的头元素在所有元素之中是放入时间最久的那个，而尾元素则是最短的那个。

   + PriorityBlockingQueue

     PriorityBlockingQueue 是一个无界的并发队列。它使用了和类 java.util.PriorityQueue 一样的排序规则。你无法向这个队列中插入 null 值。 所有插入到 PriorityBlockingQueue 的元素必须实现 java.lang.Comparable 接口。因此该队列中元素的排序就取决于你自己的 Comparable 实现。 注意 PriorityBlockingQueue 对于具有相等优先级(compare() == 0)的元素并不强制任何特定行为。

     同时注意，如果你从一个 PriorityBlockingQueue 获得一个 Iterator 的话，该 Iterator 并不能保证它对元素的遍历是以优先级为序的

   + SynchronousQueue

     SynchronousQueue 是一个特殊的队列，它的内部同时只能够容纳单个元素。如果该队列已有一元素的话，试图向队列中插入一个新元素的线程将会阻塞，直到另一个线程将该元素从队列中抽走。同样，如果该队列为空，试图向队列中抽取一个元素的线程将会阻塞，直到另一个线程向队列中插入了一条新的元素。 据此，把这个类称作一个队列显然是夸大其词了。它更多像是一个汇合点。

3. BlockingQueue适合用在什么样的场景

   BlockingQueue 通常用于一个线程生产对象，而另外一个线程消费这些对象的场景。

4. BlockingQueue常用的方法

   |      | 抛异常         | 特定值        | 阻塞         | 超时                             |
   | ---- | -------------- | ------------- | ------------ | -------------------------------- |
   | 插入 | addFirst(o)    | offerFirst(o) | putFirst(o)  | offerFirst(o, timeout, timeunit) |
   | 移除 | removeFirst(o) | pollFirst(o)  | takeFirst(o) | pollFirst(timeout, timeunit)     |
   | 检查 | getFirst(o)    | peekFirst(o)  |              |                                  |

   |      | 抛异常        | 特定值       | 阻塞        | 超时                            |
   | ---- | ------------- | ------------ | ----------- | ------------------------------- |
   | 插入 | addLast(o)    | offerLast(o) | putLast(o)  | offerLast(o, timeout, timeunit) |
   | 移除 | removeLast(o) | pollLast(o)  | takeLast(o) | pollLast(timeout, timeunit)     |
   | 检查 | getLast(o)    | peekLast(o)  |             |                                 |

   四组不同的行为方式解释:

   - 抛异常: 如果试图的操作无法立即执行，抛一个异常。
   - 特定值: 如果试图的操作无法立即执行，返回一个特定的值(常常是 true / false)。
   - 阻塞: 如果试图的操作无法立即执行，该方法调用将会发生阻塞，直到能够执行。
   - 超时: 如果试图的操作无法立即执行，该方法调用将会发生阻塞，直到能够执行，但等待时间不会超过给定值。返回一个特定值以告知该操作是否成功(典型的是 true / false)。

5. BlockingDeque 与BlockingQueue有何关系，请对比下它们的方法?

   BlockingDeque 接口继承自 BlockingQueue 接口。这就意味着你可以像使用一个 BlockingQueue 那样使用 BlockingDeque。如果你这么干的话，各种插入方法将会把新元素添加到双端队列的尾端，而移除方法将会把双端队列的首端的元素移除。正如 BlockingQueue 接口的插入和移除方法一样。

   方法对比

   | BlockingQueue | BlockingDeque   |
   | ------------- | --------------- |
   | add()         | addLast()       |
   | offer() x 2   | offerLast() x 2 |
   | put()         | putLast()       |
   | remove()      | removeFirst()   |
   | poll() x 2    | pollFirst()     |
   | take()        | takeFirst()     |
   | element()     | getFirst()      |
   | peek()        | peekFirst()     |

6. BlockingDeque适合用在什么样的场景?

   在线程既是一个队列的生产者又是这个队列的消费者的时候可以使用到 BlockingDeque。如果生产者线程需要在队列的两端都可以插入数据，消费者线程需要在队列的两端都可以移除数据，这个时候也可以使用 BlockingDeque。

7. BlockingDeque大家族有哪些?

   + LinkedBlockingDeque

     LinkedBlockingDeque 是一个双端队列，在它为空的时候，一个试图从中抽取数据的线程将会阻塞，无论该线程是试图从哪一端抽取数据。

   

8. BlockingDeque 与BlockingQueue实现例子

   ```java
   //dequeue
   BlockingDeque<String> deque = new LinkedBlockingDeque<String>();
   deque.addFirst("1");
   deque.addLast("2");
   String two = deque.takeLast();
   String one = deque.takeFirst();
   ```

   ```java
   //queue
   public class BlockingQueueExample {
       public static void main(String[] args) throws Exception {
           BlockingQueue queue = new ArrayBlockingQueue(1024);
           Producer producer = new Producer(queue);
           Consumer consumer = new Consumer(queue);
           new Thread(producer).start();
           new Thread(consumer).start();
           Thread.sleep(4000);
       }
   }
   class Producer implements Runnable{
       protected BlockingQueue queue = null;
       public Producer(BlockingQueue queue) {
           this.queue = queue;
       }
       public void run() {
           try {
               queue.put("1");
               Thread.sleep(1000);
               queue.put("2");
               Thread.sleep(1000);
               queue.put("3");
           } catch (InterruptedException e) {
               e.printStackTrace();
           }
       }
   }
   class Consumer implements Runnable{
       protected BlockingQueue queue = null;
       public Consumer(BlockingQueue queue) {
           this.queue = queue;
       }
       public void run() {
           try {
               System.out.println(queue.take());
               System.out.println(queue.take());
               System.out.println(queue.take());
           } catch (InterruptedException e) {
               e.printStackTrace();
           }
       }
   }
   ```

   
