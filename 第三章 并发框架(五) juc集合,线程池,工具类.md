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

### 3.13 JUC线程池

### 3.13.1 FutureTask

#### 3.13.1.1 面试题

1. FutrueTask用来解决什么问题的?为什么会出现

   Future接口代表异步计算的结果，通过Future接口提供的方法可以查看异步计算是否执行完成，或者等待执行结果并获取执行结果，同时还可以取消执行。

   FutrueTask对线程做了封装,用来控制线程的状态

   //TODO

2. FutureTask类结构关系

   ![](resource\FutureTask.png)

3. FutureTask的线程安全是由什么保证的

   FutureTask中线程状态state 有volatile修饰符,使用cas改变其状态

4. FutureTask结果返回机制

   无论是通过runnable还是callable创建的FutureTask,参数都会被封装成为一个callable赋值到对象的callable变量

   ```java
   public FutureTask(Callable<V> callable) {
       if (callable == null)
           throw new NullPointerException();
       this.callable = callable;
       this.state = NEW;       // ensure visibility of callable
   }
   public FutureTask(Runnable runnable, V result) {
       this.callable = Executors.callable(runnable, result);
       this.state = NEW;       // ensure visibility of callable
   }
   //这是Executors.callable方法
   public static <T> Callable<T> callable(Runnable task, T result) {
       if (task == null)
          throw new NullPointerException();
       return new RunnableAdapter<T>(task, result);
   }
   static final class RunnableAdapter<T> implements Callable<T> {
       final Runnable task;
       final T result;
       RunnableAdapter(Runnable task, T result) {
           this.task = task;
           this.result = result;
       }
       public T call() {
           task.run();
           return result;
       }
   }
   ```

   后在run方法中执行callable.call得到结果赋值给outcome 变量

   ```java
   public void run() {
       //新建任务，CAS替换runner为当前线程
       if (state != NEW || !UNSAFE.compareAndSwapObject(this,runnerOffset,null,Thread.currentThread()))
           return;
       try {
           Callable<V> c = callable;
           if (c != null && state == NEW) {
               V result;
               boolean ran;
               try {
                   result = c.call();
                   ran = true;
               } catch (Throwable ex) {
                   result = null;
                   ran = false;
                   setException(ex);
               }
               if (ran)
                   set(result);//设置执行结果
           }
       } finally {
           // runner must be non-null until state is settled to
           // prevent concurrent calls to run()
           runner = null;
           // state must be re-read after nulling runner to prevent
           // leaked interrupts
           int s = state;
           if (s >= INTERRUPTING)
               handlePossibleCancellationInterrupt(s);//处理中断逻辑
       }
   }
   protected void set(V v) {
       if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {
           outcome = v;
           UNSAFE.putOrderedInt(this, stateOffset, NORMAL); // final state
           finishCompletion();//执行完毕，唤醒等待线程
       }
   }
   private void finishCompletion() {
       // assert state > COMPLETING;
       for (WaitNode q; (q = waiters) != null;) {
           if (UNSAFE.compareAndSwapObject(this, waitersOffset, q, null)) {//移除等待线程
               for (;;) {//自旋遍历等待线程
                   Thread t = q.thread;
                   if (t != null) {
                       q.thread = null;
                       LockSupport.unpark(t);//唤醒等待线程
                   }
                   WaitNode next = q.next;
                   if (next == null)
                       break;
                   q.next = null; // unlink to help gc
                   q = next;
               }
               break;
           }
       }
       //任务完成后调用函数，自定义扩展
       done();
   
       callable = null;        // to reduce footprint
   }
   private void handlePossibleCancellationInterrupt(int s) {
       //在中断者中断线程之前可能会延迟，所以我们只需要让出CPU时间片自旋等待
       if (s == INTERRUPTING)
           while (state == INTERRUPTING)
               Thread.yield(); // wait out pending interrupt
   }
   ```

   最后通过FutureTask.get方法获取执行结果(未执行完会阻塞)

   ```java
   //获取执行结果
   public V get() throws InterruptedException, ExecutionException {
       int s = state;
       if (s <= COMPLETING)
           s = awaitDone(false, 0L);
       return report(s);
   }
   //返回执行结果或抛出异常
   private V report(int s) throws ExecutionException {
       Object x = outcome;
       if (s == NORMAL)
           return (V)x;
       if (s >= CANCELLED)
           throw new CancellationException();
       throw new ExecutionException((Throwable)x);
   }
   //未执行完 等待
   private int awaitDone(boolean timed, long nanos)
       throws InterruptedException {
       final long deadline = timed ? System.nanoTime() + nanos : 0L;
       WaitNode q = null;
       boolean queued = false;
       for (;;) {//自旋
           if (Thread.interrupted()) {//获取并清除中断状态
               removeWaiter(q);//移除等待WaitNode
               throw new InterruptedException();
           }
   
           int s = state;
           //执行完成
           if (s > COMPLETING) {
               if (q != null)
                   q.thread = null;//置空等待节点的线程
               return s;
           }
           else if (s == COMPLETING) // cannot time out yet
               Thread.yield();
           else if (q == null)
               q = new WaitNode();
           else if (!queued)
               //CAS修改waiter
               queued = UNSAFE.compareAndSwapObject(this, waitersOffset,
                                                    q.next = waiters, q);
           else if (timed) {
               nanos = deadline - System.nanoTime();
               if (nanos <= 0L) {
                   removeWaiter(q);//超时，移除等待节点
                   return state;
               }
               LockSupport.parkNanos(this, nanos);//阻塞当前线程
           }
           else
               LockSupport.park(this);//阻塞当前线程
       }
   }
   
   ```

   

   

5. FutureTask内部运行状态的转变

   ```java
   //任务状态
   private volatile int state;
   private static final int NEW          = 0;
   private static final int COMPLETING   = 1;
   private static final int NORMAL       = 2;
   private static final int EXCEPTIONAL  = 3;
   private static final int CANCELLED    = 4;
   private static final int INTERRUPTING = 5;
   private static final int INTERRUPTED  = 6;
   ```

   + `NEW`:表示是个新的任务或者还没被执行完的任务。这是初始状态。
   + `COMPLETING`:任务已经执行完成或者执行任务的时候发生异常，但是任务执行结果或者异常原因还没有保存到outcome字段(outcome字段用来保存任务执行结果，如果发生异常，则用来保存异常原因)的时候，状态会从NEW变更到COMPLETING。但是这个状态会时间会比较短，属于中间状态。
   + `NORMAL`:任务已经执行完成并且任务执行结果已经保存到outcome字段，状态会从COMPLETING转换到NORMAL。这是一个最终态。
   + `EXCEPTIONAL`:任务执行发生异常并且异常原因已经保存到outcome字段中后，状态会从COMPLETING转换到EXCEPTIONAL。这是一个最终态。
   + `CANCELLED`:任务还没开始执行或者已经开始执行但是还没有执行完成的时候，用户调用了cancel(false)方法取消任务且不中断任务执行线程，这个时候状态会从NEW转化为CANCELLED状态。这是一个最终态。
   + `INTERRUPTING`: 任务还没开始执行或者已经执行但是还没有执行完成的时候，用户调用了cancel(true)方法取消任务并且要中断任务执行线程但是还没有中断任务执行线程之前，状态会从NEW转化为INTERRUPTING。这是一个中间状态。
   + `INTERRUPTED`:调用interrupt()中断任务执行线程之后状态会从INTERRUPTING转换到INTERRUPTED。这是一个最终态。 有一点需要注意的是，所有值大于COMPLETING的状态都表示任务已经执行完成(任务正常执行完成，任务执行异常或者任务被取消)。

   ![](resource\FutureTaskState.png)

6. FutureTask通常会怎么用

   ```java
   import java.util.concurrent.*;
    
   public class CallDemo {
    
       public static void main(String[] args) throws ExecutionException, InterruptedException {
    
           /**
            * 第一种方式:Future + ExecutorService
            * Task task = new Task();
            * ExecutorService service = Executors.newCachedThreadPool();
            * Future<Integer> future = service.submit(task1);
            * service.shutdown();
            */
    
    
           /**
            * 第二种方式: FutureTask + ExecutorService
            * ExecutorService executor = Executors.newCachedThreadPool();
            * Task task = new Task();
            * FutureTask<Integer> futureTask = new FutureTask<Integer>(task);
            * executor.submit(futureTask);
            * executor.shutdown();
            */
    
           /**
            * 第三种方式:FutureTask + Thread
            */
    
           // 2. 新建FutureTask,需要一个实现了Callable接口的类的实例作为构造函数参数
           FutureTask<Integer> futureTask = new FutureTask<Integer>(new Task());
           // 3. 新建Thread对象并启动
           Thread thread = new Thread(futureTask);
           thread.setName("Task thread");
           thread.start();
    
           try {
               Thread.sleep(1000);
           } catch (InterruptedException e) {
               e.printStackTrace();
           }
    
           System.out.println("Thread [" + Thread.currentThread().getName() + "] is running");
    
           // 4. 调用isDone()判断任务是否结束
           if(!futureTask.isDone()) {
               System.out.println("Task is not done");
               try {
                   Thread.sleep(2000);
               } catch (InterruptedException e) {
                   e.printStackTrace();
               }
           }
           int result = 0;
           try {
               // 5. 调用get()方法获取任务结果,如果任务没有执行完成则阻塞等待
               result = futureTask.get();
           } catch (Exception e) {
               e.printStackTrace();
           }
    
           System.out.println("result is " + result);
    
       }
    
       // 1. 继承Callable接口,实现call()方法,泛型参数为要返回的类型
       static class Task  implements Callable<Integer> {
    
           @Override
           public Integer call() throws Exception {
               System.out.println("Thread [" + Thread.currentThread().getName() + "] is running");
               int result = 0;
               for(int i = 0; i < 100;++i) {
                   result += i;
               }
    
               Thread.sleep(3000);
               return result;
           }
       }
   }
   ```


### 3.13.2 ThreadPoolExecutor

#### 3.13.2.1 ThreadPoolExecutor的状态

1. 关键属性

   ```java
   //这个属性是用来存放 当前运行的worker数量以及线程池状态的
   //int是32位的，这里把int的高3位拿来充当线程池状态的标志位,后29位拿来充当当前运行worker的数量
   private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
   //存放任务的阻塞队列
   private final BlockingQueue<Runnable> workQueue;
   //worker的集合,用set来存放
   private final HashSet<Worker> workers = new HashSet<Worker>();
   //历史达到的worker数最大值
   private int largestPoolSize;
   //当队列满了并且worker的数量达到maxSize的时候,执行具体的拒绝策略
   private volatile RejectedExecutionHandler handler;
   //超出coreSize的worker的生存时间
   private volatile long keepAliveTime;
   //常驻worker的数量
   private volatile int corePoolSize;
   //最大worker的数量,一般当workQueue满了才会用到这个参数
   private volatile int maximumPoolSize;
   ```

2. 内部状态

   ```java
   private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
   private static final int COUNT_BITS = Integer.SIZE - 3;
   private static final int CAPACITY   = (1 << COUNT_BITS) - 1;
   
   // runState is stored in the high-order bits
   private static final int RUNNING    = -1 << COUNT_BITS;
   private static final int SHUTDOWN   =  0 << COUNT_BITS;
   private static final int STOP       =  1 << COUNT_BITS;
   private static final int TIDYING    =  2 << COUNT_BITS;
   private static final int TERMINATED =  3 << COUNT_BITS;
   
   // Packing and unpacking ctl
   private static int runStateOf(int c)     { return c & ~CAPACITY; }
   private static int workerCountOf(int c)  { return c & CAPACITY; }
   private static int ctlOf(int rs, int wc) { return rs | wc; }
   ```

   其中AtomicInteger变量ctl的功能非常强大: 利用低29位表示线程池中线程数，通过高3位表示线程池的运行状态:

   - RUNNING: -1 << COUNT_BITS，即高3位为111，该状态的线程池会接收新任务，并处理阻塞队列中的任务；

   - SHUTDOWN:  0 << COUNT_BITS，即高3位为000，该状态的线程池不会接收新任务，但会处理阻塞队列中的任务；

   - STOP :  1 << COUNT_BITS，即高3位为001，该状态的线程不会接收新任务，也不会处理阻塞队列中的任务，而且会中断正在运行的任务；

   - TIDYING :  2 << COUNT_BITS，即高3位为010, 所有的任务都已经终止；

   - TERMINATED:  3 << COUNT_BITS，即高3位为011, terminated()方法已经执行完成

     ![](resource\ThreadPoolExecutorState.png)

#### 3.13.2.2 面试题

1. 为什么要有线程池

   用来统一分配调优监控线程:

   + 降低资源消耗(线程无限制创建,然后使用完毕后销毁)
   + 提高响应速度(无需创建线程)
   + 提高线程的可管理性

2. 举例说明java实现和管理线程池有哪些方式?

   从JDK 5开始，把工作单元与执行机制分离开来，工作单元包括Runnable和Callable，而执行机制由Executor框架提供。

3. 很多公司不推荐使用Executors创建线程池,为什么?推荐怎样使用

   线程池不允许使用Executors去创建，而是通过ThreadPoolExecutor的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。 说明：Executors各个方法的弊端：

   - newCachedThreadPool和newScheduledThreadPool:   主要问题是线程数最大数是Integer.MAX_VALUE，可能会创建数量非常多的线程，甚至OOM。
   - newFixedThreadPool和newSingleThreadExecutor:   主要问题是堆积的请求处理队列可能会耗费非常大的内存，甚至OOM。

   推荐方式1:引入commons-lang3包

   ```java
   ScheduledExecutorService executorService = new ScheduledThreadPoolExecutor(1,
           new BasicThreadFactory.Builder().namingPattern("example-schedule-pool-%d").daemon(true).build());
   ```

   推荐方式2:引入com.google.guava包

   ```java
   ThreadFactory namedThreadFactory = new ThreadFactoryBuilder().setNameFormat("demo-pool-%d").build();
   
   //Common Thread Pool
   ExecutorService pool = new ThreadPoolExecutor(5, 200, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>(1024), namedThreadFactory, new ThreadPoolExecutor.AbortPolicy());
   
   // excute
   pool.execute(()-> System.out.println(Thread.currentThread().getName()));
   
    //gracefully shutdown
   pool.shutdown();
   ```

   推荐方式3:

   spring配置线程池方式：自定义线程工厂bean需要实现ThreadFactory，可参考该接口的其它默认实现类，使用方式直接注入bean调用execute(Runnable task)方法即可

   ```java
       <bean id="userThreadPool" class="org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor">
           <property name="corePoolSize" value="10" />
           <property name="maxPoolSize" value="100" />
           <property name="queueCapacity" value="2000" />
   
       <property name="threadFactory" value= threadFactory />
           <property name="rejectedExecutionHandler">
               <ref local="rejectedExecutionHandler" />
           </property>
       </bean>
       
       //in code
       userThreadPool.execute(thread);
   ```

   

   

4. ThreadPoolExecutor有哪些核心参数

   ```java
   public ThreadPoolExecutor(int corePoolSize,
                                 int maximumPoolSize,
                                 long keepAliveTime,
                                 TimeUnit unit,
                                 BlockingQueue<Runnable> workQueue,
                                 RejectedExecutionHandler handler)
   ```

   1. `corePoolSize`线程池中的核心线程数，当提交一个任务时，线程池创建一个新线程执行任务，直到当前线程数等于corePoolSize, 即使有其他空闲线程能够执行新来的任务, 也会继续创建线程；如果当前线程数为corePoolSize，继续提交的任务被保存到阻塞队列中，等待被执行；如果执行了线程池的prestartAllCoreThreads()方法，线程池会提前创建并启动所有核心线程。

   2. `maximumPoolSize`线程池中允许的最大线程数。如果当前阻塞队列满了，且继续提交任务，则创建新的线程执行任务，前提是当前线程数小于maximumPoolSize；当阻塞队列是无界队列, 则maximumPoolSize则不起作用, 因为无法提交至核心线程池的线程会一直持续地放入workQueue.

   3. `keepAliveTime`线程空闲时的存活时间，即当线程没有任务执行时，该线程继续存活的时间；默认情况下，该参数只在线程数大于corePoolSize时才有用, 超过这个时间的空闲线程将被终止；

   4. `unit`keepAliveTime的单位

   5. `workQueue` 用来保存等待被执行的任务的阻塞队列.
      + `ArrayBlockingQueue`: 基于数组结构的有界阻塞队列，按FIFO排序任务；
      + `LinkedBlockingQuene`: 基于链表结构的阻塞队列，按FIFO排序任务，吞吐量通常要高于ArrayBlockingQuene；
      + `SynchronousQuene`: 一个不存储元素的阻塞队列，每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于LinkedBlockingQuene；
      + `PriorityBlockingQuene`: 具有优先级的无界阻塞队列；

   6. `LinkedBlockingQueue`比`ArrayBlockingQueue`在插入删除节点性能方面更优，但是二者在`put()`, `take()`任务的时均需要加锁，`SynchronousQueue`使用无锁算法，根据节点的状态判断执行，而不需要用到锁，其核心是`Transfer.transfer()`.

   7. `handler `线程池的饱和策略，当阻塞队列满了，且没有空闲的工作线程，如果继续提交任务，必须采取一种策略处理该任务，线程池提供了4种策略:

      + `AbortPolicy`: 直接抛出异常，默认策略；

      + `CallerRunsPolicy`: 用调用者所在的线程来执行任务；

      + `DiscardOldestPolicy`: 丢弃阻塞队列中靠最前的任务，并执行当前任务；

      + `DiscardPolicy`: 直接丢弃任务；

        当然也可以根据应用场景实现RejectedExecutionHandler接口，自定义饱和策略，如记录日志或持久化存储不能处理的任务。

   8. `threadFactory` 创建线程的工厂，通过自定义的线程工厂可以给每个新建的线程设置一个具有识别度的线程名。默认为DefaultThreadFactory

   

5. ThreadPoolExecutor可以创建哪三种线程池

   1. newFixedThreadPool

      ```java
      public static ExecutorService newFixedThreadPool(int nThreads) {
          return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
      }
      ```

      线程池的线程数量达corePoolSize后，即使线程池没有可执行任务时，也不会释放线程。

      FixedThreadPool的工作队列为无界队列LinkedBlockingQueue(队列容量为Integer.MAX_VALUE), 这会导致以下问题:

      - 线程池里的线程数量不超过corePoolSize,这导致了maximumPoolSize和keepAliveTime将会是个无用参数
      - 由于使用了无界队列, 所以FixedThreadPool永远不会拒绝, 即饱和策略失效

   2. newSingleThreadExecutor

      ```java
      public static ExecutorService newSingleThreadExecutor() {
          return new FinalizableDelegatedExecutorService
              (new ThreadPoolExecutor(1, 1,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>()));
      }
      ```

      初始化的线程池中只有一个线程，如果该线程异常结束，会重新创建一个新的线程继续执行任务，唯一的线程可以保证所提交任务的顺序执行.

      由于使用了无界队列, 所以SingleThreadPool永远不会拒绝, 即饱和策略失效

   3. newCachedThreadPool

      ```java
      public static ExecutorService newCachedThreadPool() {
          return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                          60L, TimeUnit.SECONDS,
                                          new SynchronousQueue<Runnable>());
      }
      ```

      线程池的线程数可达到Integer.MAX_VALUE，即2147483647，内部使用SynchronousQueue作为阻塞队列； 和newFixedThreadPool创建的线程池不同，newCachedThreadPool在没有任务执行时，当线程的空闲时间超过keepAliveTime，会自动释放线程资源，当提交新任务时，如果没有空闲线程，则创建新线程执行任务，会导致一定的系统开销； 执行过程与前两种稍微不同:

      - 主线程调用SynchronousQueue的offer()方法放入task, 倘若此时线程池中有空闲的线程尝试读取 SynchronousQueue的task, 即调用了SynchronousQueue的poll(), 那么主线程将该task交给空闲线程. 否则执行(2)
      - 当线程池为空或者没有空闲的线程, 则创建新的线程执行任务.
      - 执行完任务的线程倘若在60s内仍空闲, 则会被终止. 因此长时间空闲的CachedThreadPool不会持有任何线程资源.

6. 当队列满了并且达到maxSize的时候,会怎么样?

   会交给rejectHandler处理

7. 说说ThreadPoolExecutor有哪些RejectExecutionHandler策略?默认是什么

   + `AbortPolicy`: 直接抛出异常，默认策略；

   + `CallerRunsPolicy`: 用调用者所在的线程来执行任务；

   + `DiscardOldestPolicy`: 丢弃阻塞队列中靠最前的任务，并执行当前任务；

   + `DiscardPolicy`: 直接丢弃任务；

8. 简要说一下线程池的任务执行机制?execute->addWorker->runworker(getTask)

   线程池的工作线程通过Woker类实现，在ReentrantLock锁的保证下，把Woker实例插入到HashSet后，并启动Woker中的线程。 从Woker类的构造方法实现可以发现: 线程工厂在创建线程thread时，将Woker实例本身this作为参数传入，当执行start方法启动线程thread时，本质是执行了Worker的runWorker方法。 firstTask执行完成之后，通过getTask方法从阻塞队列中获取等待的任务，如果队列中没有任务，getTask方法会被阻塞并挂起，不会占用cpu资源；

9. 线程池中任务是如何提交的

   ![](resource\ExecutorSubmit.png)

   submit任务，等待线程池execute

   执行FutureTask类的get方法时，会把主线程封装成WaitNode节点并保存在waiters链表中， 并阻塞等待运行结果；

   FutureTask任务执行完成后，通过UNSAFE设置waiters相应的waitNode为null，并通过LockSupport类unpark方法唤醒主线程；

   ```java
   public class Test{
       public static void main(String[] args) {
   
           ExecutorService es = Executors.newCachedThreadPool();
           Future<String> future = es.submit(new Callable<String>() {
               @Override
               public String call() throws Exception {
                   try {
                       TimeUnit.SECONDS.sleep(2);
                   } catch (InterruptedException e) {
                       e.printStackTrace();
                   }
                   return "future result";
               }
           });
           try {
               String result = future.get();
               System.out.println(result);
           } catch (Exception e) {
               e.printStackTrace();
           }
       }
   }
   ```

   在实际业务场景中，Future和Callable基本是成对出现的，Callable负责产生结果，Future负责获取结果。

   1. Callable接口类似于Runnable，只是Runnable没有返回值。
   2. Callable任务除了返回正常结果之外，如果发生异常，该异常也会被返回，即Future可以拿到异步执行任务各种结果；
   3. Future.get方法会导致主线程阻塞，直到Callable任务执行完成；

10. 线程池中任务如何关闭

    ThreadPoolExecutor的状态参考 3.13.2.1

    shutdown方法会将线程池的状态设置为SHUTDOWN,线程池进入这个状态后,就拒绝再接受任务,然后会将剩余的任务全部执行完

    ```java
    public void shutdown() {
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            //检查是否可以关闭线程
            checkShutdownAccess();
            //设置线程池状态
            advanceRunState(SHUTDOWN);
            //尝试中断worker
            interruptIdleWorkers();
                //预留方法,留给子类实现
            onShutdown(); // hook for ScheduledThreadPoolExecutor
        } finally {
            mainLock.unlock();
        }
        tryTerminate();
    }
    
    private void interruptIdleWorkers() {
        interruptIdleWorkers(false);
    }
    
    private void interruptIdleWorkers(boolean onlyOne) {
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            //遍历所有的worker
            for (Worker w : workers) {
                Thread t = w.thread;
                //先尝试调用w.tryLock(),如果获取到锁,就说明worker是空闲的,就可以直接中断它
                //注意的是,worker自己本身实现了AQS同步框架,然后实现的类似锁的功能
                //它实现的锁是不可重入的,所以如果worker在执行任务的时候,会先进行加锁,这里tryLock()就会返回false
                if (!t.isInterrupted() && w.tryLock()) {
                    try {
                        t.interrupt();
                    } catch (SecurityException ignore) {
                    } finally {
                        w.unlock();
                    }
                }
                if (onlyOne)
                    break;
            }
        } finally {
            mainLock.unlock();
        }
    }
    ```

    shutdownNow做的比较绝，它先将线程池状态设置为STOP，然后拒绝所有提交的任务。最后中断左右正在运行中的worker,然后清空任务队列。

    ```java
    public List<Runnable> shutdownNow() {
        List<Runnable> tasks;
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            checkShutdownAccess();
            //检测权限
            advanceRunState(STOP);
            //中断所有的worker
            interruptWorkers();
            //清空任务队列
            tasks = drainQueue();
        } finally {
            mainLock.unlock();
        }
        tryTerminate();
        return tasks;
    }
    
    private void interruptWorkers() {
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            //遍历所有worker，然后调用中断方法
            for (Worker w : workers)
                w.interruptIfStarted();
        } finally {
            mainLock.unlock();
        }
    }
    ```

    

11. 配置线程池需要考虑的因素

    主要从 任务的优先级,任务的执行时间,任务的性质(i/o密集或cpu密集),任务的依赖关系考虑,尽可能使用有界的工作队列

    任务的性质可以使用不同规模的线程池:

    cpu密集:型使用尽量少的线程 Ncpu+1

    i/o密集:尽量多的线程(例如数据库连接池) Ncpu*2

    混合型:CPU密集型的任务与IO密集型任务的执行时间差别较小，拆分为两个线程池；否则没有必要拆分。

12. 如何监控线程池状态

    使用ThreadPoolExecutor的以下方法:

    + `getTaskCount()`:Returns the approximate total number of tasks that have ever been scheduled for execution.
    + `getCompletedTaskCount()` Returns the approximate total number of tasks that have completed execution. 返回结果少于getTaskCount()。
    + `getLargestPoolSize()` Returns the largest number of threads that have ever simultaneously been in the pool. 返回结果小于等于maximumPoolSize
    + `getPoolSize()` Returns the current number of threads in the pool.
    + `getActiveCount()` Returns the approximate number of threads that are actively executing tasks.

    

    ### 3.13.3 ScheduledThreadPoolExecutor

    #### 3.13.3.1 面试题
    
    1. ScheduledThreadPoolExecutor要解决什么样的ScheduledThreadPoolExecutor继承自 ThreadPoolExecutor，为任务提供延迟或周期执行，属于线程池的一种。
    
    2. ScheduledThreadPoolExecutor相比ThreadPoolExecutor有哪些特性?
    
       + 使用专门的任务类型—ScheduledFutureTask 来执行周期任务，也可以接收不需要时间调度的任务(这些任务通过 ExecutorService 来执行)。
    
       + 使用专门的存储队列—DelayedWorkQueue 来存储任务，DelayedWorkQueue 是无界延迟队列DelayQueue 的一种。相比ThreadPoolExecutor也简化了执行机制(delayedExecute方法，后面单独分析)。
    
       + 支持可选的run-after-shutdown参数，在池被关闭(shutdown)之后支持可选的逻辑来决定是否继续运行周期或延迟任务。并且当任务(重新)提交操作与 shutdown 操作重叠时，复查逻辑也不相同
    
    3. ScheduledThreadPoolExecutor有什么样的数据结构，核心内部类和抽象类?
    
       ![](resource\scheduledThreadPoolExecutor.png)
    
       ScheduledThreadPoolExecutor 内部构造了两个内部类 `ScheduledFutureTask` 和 `DelayedWorkQueue`:
    
       + `ScheduledFutureTask`: 继承了FutureTask，说明是一个异步运算任务；最上层分别实现了Runnable、Future、Delayed接口，说明它是一个可以延迟执行的异步运算任务。
    
       + `DelayedWorkQueue`: 这是 ScheduledThreadPoolExecutor 为存储周期或延迟任务专门定义的一个延迟队列，继承了 AbstractQueue，为了契合 ThreadPoolExecutor 也实现了 BlockingQueue 接口。它内部只允许存储 RunnableScheduledFuture 类型的任务。与 DelayQueue 的不同之处就是它只允许存放 RunnableScheduledFuture 对象，并且自己实现了二叉堆(DelayQueue 是利用了 PriorityQueue 的二叉堆结构)
    
    4. ScheduledThreadPoolExecutor有哪两个关闭策略? 区别是什么?
    
       ```java
       //关闭后继续执行已经存在的周期任务  
       private volatile boolean continueExistingPeriodicTasksAfterShutdown; 
       //关闭后继续执行已经存在的延时任务  
       private volatile boolean executeExistingDelayedTasksAfterShutdown = true;
       public void shutdown() {
           super.shutdown();
       }
       //取消并清除由于关闭策略不应该运行的所有任务
       @Override void onShutdown() {
           BlockingQueue<Runnable> q = super.getQueue();
           //获取run-after-shutdown参数
           boolean keepDelayed =
               getExecuteExistingDelayedTasksAfterShutdownPolicy();
           boolean keepPeriodic =
               getContinueExistingPeriodicTasksAfterShutdownPolicy();
           if (!keepDelayed && !keepPeriodic) {//池关闭后不保留任务
               //依次取消任务
               for (Object e : q.toArray())
                   if (e instanceof RunnableScheduledFuture<?>)
                       ((RunnableScheduledFuture<?>) e).cancel(false);
               q.clear();//清除等待队列
           }
           else {//池关闭后保留任务
               // Traverse snapshot to avoid iterator exceptions
               //遍历快照以避免迭代器异常
               for (Object e : q.toArray()) {
                   if (e instanceof RunnableScheduledFuture) {
                       RunnableScheduledFuture<?> t =
                           (RunnableScheduledFuture<?>)e;
                       if ((t.isPeriodic() ? !keepPeriodic : !keepDelayed) ||
                           t.isCancelled()) { // also remove if already cancelled
                           //如果任务已经取消，移除队列中的任务
                           if (q.remove(t))
                               t.cancel(false);
                       }
                   }
               }
           }
           tryTerminate(); //终止线程池
       }
       
       ```
    
       
    
       
    
    5. ScheduledThreadPoolExecutor中scheduleAtFixedRate 和 scheduleWithFixedDelay区别是什么?
    
       ```java
       /**
        * 创建一个周期执行的任务，第一次执行延期时间为initialDelay，
        * 之后每隔period执行一次，不等待第一次执行完成就开始计时
        */
       public ScheduledFuture<?> scheduleAtFixedRate(Runnable command,
                                                     long initialDelay,
                                                     long period,
                                                     TimeUnit unit) {
           if (command == null || unit == null)
               throw new NullPointerException();
           if (period <= 0)
               throw new IllegalArgumentException();
           //构建RunnableScheduledFuture任务类型
           ScheduledFutureTask<Void> sft =
               new ScheduledFutureTask<Void>(command,
                                             null,
                                             triggerTime(initialDelay, unit),//计算任务的延迟时间
                                             unit.toNanos(period));//计算任务的执行周期
           RunnableScheduledFuture<Void> t = decorateTask(command, sft);//执行用户自定义逻辑
           sft.outerTask = t;//赋值给outerTask，准备重新入队等待下一次执行
           delayedExecute(t);//执行任务
           return t;
       }
       
       /**
        * 创建一个周期执行的任务，第一次执行延期时间为initialDelay，
        * 在第一次执行完之后延迟delay后开始下一次执行
        */
       public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command,
                                                        long initialDelay,
                                                        long delay,
                                                        TimeUnit unit) {
           if (command == null || unit == null)
               throw new NullPointerException();
           if (delay <= 0)
               throw new IllegalArgumentException();
           //构建RunnableScheduledFuture任务类型
           ScheduledFutureTask<Void> sft =
               new ScheduledFutureTask<Void>(command,
                                             null,
                                             triggerTime(initialDelay, unit),//计算任务的延迟时间
                                             unit.toNanos(-delay));//计算任务的执行周期
           RunnableScheduledFuture<Void> t = decorateTask(command, sft);//执行用户自定义逻辑
           sft.outerTask = t;//赋值给outerTask，准备重新入队等待下一次执行
           delayedExecute(t);//执行任务
           return t;
       }
       ```
    
    6. 为什么ThreadPoolExecutor 的调整策略却不适用于 ScheduledThreadPoolExecutor?
    
       例如: 由于 ScheduledThreadPoolExecutor 是一个固定核心线程数大小的线程池，并且使用了一个无界队列，所以调整maximumPoolSize对其没有任何影响(所以 ScheduledThreadPoolExecutor 没有提供可以调整最大线程数的构造函数，默认最大线程数固定为Integer.MAX_VALUE)。此外，设置corePoolSize为0或者设置核心线程空闲后清除(allowCoreThreadTimeOut)同样也不是一个好的策略，因为一旦周期任务到达某一次运行周期时，可能导致线程池内没有线程去处理这些任务。
    
    7. Executors 提供了几种方法来构造 ScheduledThreadPoolExecutor?
    
       + `ScheduledFutureTask`: 继承了FutureTask，说明是一个异步运算任务；最上层分别实现了Runnable、Future、Delayed接口，说明它是一个可以延迟执行的异步运算任务。
       + `DelayedWorkQueue`: 这是 ScheduledThreadPoolExecutor 为存储周期或延迟任务专门定义的一个延迟队列，继承了 AbstractQueue，为了契合 ThreadPoolExecutor 也实现了 BlockingQueue 接口。它内部只允许存储 RunnableScheduledFuture 类型的任务。与 DelayQueue 的不同之处就是它只允许存放 RunnableScheduledFuture 对象，并且自己实现了二叉堆(DelayQueue 是利用了 PriorityQueue 的二叉堆结构)
    
    

