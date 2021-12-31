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
    



#### 3.13.4 Fork/Join框架

#### 3.13.4.1 面试题

1. Fork/Join主要用来解决什么样的问题?

   Fork/Join框架是Java并发工具包中的一种可以将一个大任务拆分为很多小任务来异步执行的工具，自JDK1.7引入。

2. Fork/Join框架主要包含哪三个模块? 模块之间的关系是怎么样的?

   任务对象`ForkJoinTask`

   执行Fork/Join的线程 `ForkJoinWorkerThread`

   线程池`ForkJoinPool`

   这三者的关系是: ForkJoinPool可以通过池中的ForkJoinWorkerThread来处理ForkJoinTask任务。

3. ForkJoinPool类继承关系?

   ![](resource\ForkJoinPool.png)

   ForkJoinWorkerThreadFactory: 内部线程工厂接口，用于创建工作线程ForkJoinWorkerThread

   DefaultForkJoinWorkerThreadFactory: ForkJoinWorkerThreadFactory 的默认实现类

   InnocuousForkJoinWorkerThreadFactory: 实现了 ForkJoinWorkerThreadFactory，无许可线程工厂，当系统变量中有系统安全管理相关属性时，默认使用这个工厂创建工作线程。

   EmptyTask: 内部占位类，用于替换队列中 join 的任务。

   ManagedBlocker: 为 ForkJoinPool 中的任务提供扩展管理并行数的接口，一般用在可能会阻塞的任务(如在 Phaser 中用于等待 phase 到下一个generation)。

   WorkQueue: ForkJoinPool 的核心数据结构，本质上是work-stealing 模式的双端任务队列，内部存放 ForkJoinTask 对象任务，使用 @Contented 注解修饰防止伪共享。

   - 工作线程在运行中产生新的任务(通常是因为调用了 fork())时，此时可以把 WorkQueue 的数据结构视为一个栈，新的任务会放入栈顶(top 位)；工作线程在处理自己工作队列的任务时，按照 LIFO 的顺序。
   - 工作线程在处理自己的工作队列同时，会尝试窃取一个任务(可能是来自于刚刚提交到 pool 的任务，或是来自于其他工作线程的队列任务)，此时可以把 WorkQueue 的数据结构视为一个 FIFO 的队列，窃取的任务位于其他线程的工作队列的队首(base位)。

   伪共享状态: 缓存系统中是以缓存行(cache line)为单位存储的。缓存行是2的整数幂个连续字节，一般为32-256个字节。最常见的缓存行大小是64个字节。当多线程修改互相独立的变量时，如果这些变量共享同一个缓存行，就会无意中影响彼此的性能，这就是伪共享。

4. ForkJoinTask抽象类继承关系? 

   ![](resource\ForkJoinTask.png)

   在实际运用中，我们一般都会继承 RecursiveTask 、RecursiveAction 或 CountedCompleter 来实现我们的业务需求，而不会直接继承 ForkJoinTask 类。

   ForkJoinTask 实现了 Future 接口，说明它也是一个可取消的异步运算任务，实际上ForkJoinTask 是 Future 的轻量级实现，主要用在纯粹是计算的函数式任务或者操作完全独立的对象计算任务。fork 是主运行方法，用于异步执行；而 join 方法在任务结果计算完毕之后才会运行，用来合并或返回计算结果。 其内部类都比较简单，ExceptionNode 是用于存储任务执行期间的异常信息的单向链表；其余四个类是为 Runnable/Callable 任务提供的适配器类，用于把 Runnable/Callable 转化为 ForkJoinTask 类型的任务(因为 ForkJoinPool 只可以运行 ForkJoinTask 类型的任务)

5. 整个Fork/Join 框架的执行流程/运行机制是怎么样的?

   ![](resource\ForkJoinFlow.png)

   1. ForkJoinPool.submit/invoke/execute接受ForkJoinTask参数,属于外部任务,会存放在workQueues的偶数槽位
   2. ForkJoinTask中的fork方法会分割子任务存放到WorkQueues的奇数槽位
   3. 放入WorkQueues后会调用signalWork方法唤醒Work/新建Work线程
   4. Work线程的run方法中会调用scan方法获取Task执行

   

6. 具体阐述Fork/Join的分治思想和work-stealing 实现方式?

   1. 分治思想就是递归

   ![](resource\DivideToRule.png)

   2. 工作窃取

      work-stealing(工作窃取)算法: 线程池内的所有工作线程都尝试找到并执行已经提交的任务，或者是被其他活动任务创建的子任务(如果不存在就阻塞等待)。这种特性使得 ForkJoinPool 在运行多个可以产生子任务的任务，或者是提交的许多小任务时效率更高。尤其是构建异步模型的 ForkJoinPool 时，对不需要合并(join)的事件类型任务也非常适用。

      在 ForkJoinPool 中，线程池中每个工作线程(ForkJoinWorkerThread)都对应一个任务队列(WorkQueue)，工作线程优先处理来自自身队列的任务(LIFO或FIFO顺序，参数 mode 决定)，然后以FIFO的顺序随机窃取其他队列中的任务。

      具体思路如下:

      - 每个线程都有自己的一个WorkQueue，该工作队列是一个双端队列。

      - 队列支持三个功能push、pop、poll

      - push/pop只能被队列的所有者线程调用，而poll可以被其他线程调用。

      - 划分的子任务调用fork时，都会被push到自己的队列中。

      - 默认情况下，工作线程从自己的双端队列获出任务并执行。

      - 当自己的队列为空时，线程随机从另一个线程的队列末尾调用poll方法窃取任务。

        ![](resource\WorkStealing.png)

7. 有哪些JDK源码中使用了Fork/Join思想?

   我们常用的数组工具类 Arrays 在JDK 8之后新增的并行排序方法(parallelSort)就运用了 ForkJoinPool 的特性，还有 ConcurrentHashMap 在JDK 8之后添加的函数式方法(如forEach等)也有运用。

8. 如何使用Executors工具类创建ForkJoinPool?

   Java8在Executors工具类中新增了两个工厂方法:

   ```JAVA
   // parallelism定义并行级别
   public static ExecutorService newWorkStealingPool(int parallelism);
   // 默认并行级别为JVM可用的处理器个数
   // Runtime.getRuntime().availableProcessors()
   public static ExecutorService newWorkStealingPool();
   ```

9. 写一个例子: 用ForkJoin方式实现1+2+3+...+100000?

   ```JAVA
   public class Test {
   	static final class SumTask extends RecursiveTask<Integer> {
   		private static final long serialVersionUID = 1L;
   		
   		final int start; //开始计算的数
   		final int end; //最后计算的数
   		
   		SumTask(int start, int end) {
   			this.start = start;
   			this.end = end;
   		}
   
   		@Override
   		protected Integer compute() {
   			//如果计算量小于1000，那么分配一个线程执行if中的代码块，并返回执行结果
   			if(end - start < 1000) {
   				System.out.println(Thread.currentThread().getName() + " 开始执行: " + start + "-" + end);
   				int sum = 0;
   				for(int i = start; i <= end; i++)
   					sum += i;
   				return sum;
   			}
   			//如果计算量大于1000，那么拆分为两个任务
   			SumTask task1 = new SumTask(start, (start + end) / 2);
   			SumTask task2 = new SumTask((start + end) / 2 + 1, end);
   			//执行任务
   			task1.fork();
   			task2.fork();
   			//获取任务执行的结果
   			return task2.join() + task1.join();
   		}
   	}
   	
   	public static void main(String[] args) throws InterruptedException, ExecutionException {
   		ForkJoinPool pool = new ForkJoinPool();
   		ForkJoinTask<Integer> task = new SumTask(1, 10000);
   		pool.submit(task);
   		System.out.println(task.get());
   	}
   }
   //结果
   ForkJoinPool-1-worker-1 开始执行: 1-625
   ForkJoinPool-1-worker-7 开始执行: 6251-6875
   ForkJoinPool-1-worker-6 开始执行: 5626-6250
   ForkJoinPool-1-worker-10 开始执行: 3751-4375
   ForkJoinPool-1-worker-13 开始执行: 2501-3125
   ForkJoinPool-1-worker-8 开始执行: 626-1250
   ForkJoinPool-1-worker-11 开始执行: 5001-5625
   ForkJoinPool-1-worker-3 开始执行: 7501-8125
   ForkJoinPool-1-worker-14 开始执行: 1251-1875
   ForkJoinPool-1-worker-4 开始执行: 9376-10000
   ForkJoinPool-1-worker-8 开始执行: 8126-8750
   ForkJoinPool-1-worker-0 开始执行: 1876-2500
   ForkJoinPool-1-worker-12 开始执行: 4376-5000
   ForkJoinPool-1-worker-5 开始执行: 8751-9375
   ForkJoinPool-1-worker-7 开始执行: 6876-7500
   ForkJoinPool-1-worker-1 开始执行: 3126-3750
   50005000
   
   ```

   

10. Fork/Join在使用时有哪些注意事项? 结合JDK中的斐波那契数列实例具体说明。

    1. 避免不必要的fork()

       表面上看上去两个子任务都fork()，然后join()两次似乎更自然。但事实证明，直接调用compute()效率更高。因为直接调用子任务的compute()方法实际上就是在当前的工作线程进行了计算(线程重用)，这比“将子任务提交到工作队列，线程又从工作队列中拿任务”快得多。

       > 当一个大任务被划分成两个以上的子任务时，尽可能使用前面说到的三个衍生的invokeAll方法，因为使用它们能避免不必要的fork()。

    2. 注意fork()、compute()、join()的顺序

       类型1

       ```java
       right.fork(); // 计算右边的任务
       long leftAns = left.compute(); // 计算左边的任务(同时右边任务也在计算)
       long rightAns = right.join(); // 等待右边的结果
       return leftAns + rightAns;
       ```

       类型2

       ```java
       left.fork(); // 计算完左边的任务
       long leftAns = left.join(); // 等待左边的计算结果
       long rightAns = right.compute(); // 再计算右边的任务
       return leftAns + rightAns;
       ```

       类型3

       ```java
       long rightAns = right.compute(); // 计算完右边的任务
       left.fork(); // 再计算左边的任务
       long leftAns = left.join(); // 等待左边的计算结果
       return leftAns + rightAns;
       ```

       后两种不是并行

    3. 选择合适的子任务粒度

       选择划分子任务的粒度(顺序执行的阈值)很重要，因为使用Fork/Join框架并不一定比顺序执行任务的效率高: 如果任务太大，则无法提高并行的吞吐量；如果任务太小，子任务的调度开销可能会大于并行计算的性能提升，我们还要考虑创建子任务、fork()子任务、线程调度以及合并子任务处理结果的耗时以及相应的内存消耗。

       官方文档给出的粗略经验是: 任务应该执行`100~10000`个基本的计算步骤。决定子任务的粒度的最好办法是实践，通过实际测试结果来确定这个阈值才是“上上策”。

       > 和其他Java代码一样，Fork/Join框架测试时需要“预热”或者说执行几遍才会被JIT(Just-in-time)编译器优化，所以测试性能之前跑几遍程序很重要。

    4. 避免重量级任务划分与结果合并

       Fork/Join的很多使用场景都用到数组或者List等数据结构，子任务在某个分区中运行，最典型的例子如并行排序和并行查找。拆分子任务以及合并处理结果的时候，应该尽量避免System.arraycopy这样耗时耗空间的操作，从而最小化任务的处理开销。

    5. 斐波那契数列示例(大量重复计算效率贼低)

       ```java
       public static void main(String[] args) {
           ForkJoinPool forkJoinPool = new ForkJoinPool(4); // 最大并发数4
           Fibonacci fibonacci = new Fibonacci(20);
           long startTime = System.currentTimeMillis();
           Integer result = forkJoinPool.invoke(fibonacci);
           long endTime = System.currentTimeMillis();
           System.out.println("Fork/join sum: " + result + " in " + (endTime - startTime) + " ms.");
       }
       //以下为官方API文档示例
       static  class Fibonacci extends RecursiveTask<Integer> {
           final int n;
           Fibonacci(int n) {
               this.n = n;
           }
           @Override
           protected Integer compute() {
               if (n <= 1) {
                   return n;
               }
               {//第一种写法
                   Fibonacci f1 = new Fibonacci(n - 1);
                   f1.fork(); 
                   Fibonacci f2 = new Fibonacci(n - 2);
                   return f2.compute() + f1.join(); 
               }
               {//第二种写法
                   //invokeAll会把传入的任务的第一个交给当前线程来执行，其他的任务都fork加入工作队列，这样等于利用当前线程也执行任务了。
                   Fibonacci f1 = new Fibonacci(n - 1);
                   Fibonacci f2 = new Fibonacci(n - 2);
                   invokeAll(f1,f2);
                   return f2.join() + f1.join();
               }
               
           }
       }
       ```




### 3.14.1 CountDownLatch详解

#### 3.14.1.1 面试题

1. 什么是CountDownLatch?

   countDownLatch是一个juc工具类,它可以支持执行完多个线程之后(count变成0)执行另外的多个线程

   从源码可知，其底层是由AQS提供支持，所以其数据结构可以参考AQS的数据结构，而AQS的数据结构核心就是两个虚拟队列: 同步队列sync queue 和条件队列condition queue，不同的条件会有不同的条件队列。CountDownLatch典型的用法是将一个程序分为n个互相独立的可解决任务，并创建值为n的CountDownLatch。当每一个任务完成时，都会在这个锁存器上调用countDown，等待问题被解决的任务调用这个锁存器的await，将他们自己拦住，直至锁存器计数结束。

2. CountDownLatch底层原理

   结合demo分析CountDownLatch原理

   ```java
   import java.util.concurrent.CountDownLatch;
   
   class MyThread extends Thread {
       private CountDownLatch countDownLatch;
       
       public MyThread(String name, CountDownLatch countDownLatch) {
           super(name);
           this.countDownLatch = countDownLatch;
       }
       
       public void run() {
           System.out.println(Thread.currentThread().getName() + " doing something");
           try {
               Thread.sleep(1000);
           } catch (InterruptedException e) {
               e.printStackTrace();
           }
           System.out.println(Thread.currentThread().getName() + " finish");
           countDownLatch.countDown();
       }
   }
   
   public class CountDownLatchDemo {
       public static void main(String[] args) {
           CountDownLatch countDownLatch = new CountDownLatch(2);
           MyThread t1 = new MyThread("t1", countDownLatch);
           MyThread t2 = new MyThread("t2", countDownLatch);
           t1.start();
           t2.start();
           System.out.println("Waiting for t1 thread and t2 thread to finish");
           try {
               countDownLatch.await();
           } catch (InterruptedException e) {
               e.printStackTrace();
           }            
           System.out.println(Thread.currentThread().getName() + " continue");        
       }
   }
   //某次的结果
   //Waiting for t1 thread and t2 thread to finish
   t1 doing something
   t2 doing something
   t1 finish
   t2 finish
   main continue
   
   ```

   时序图

   ![](resource\CountDownLatchTimeSeris.png)

   说明: 首先main线程会调用await操作，此时main线程会被阻塞，等待被唤醒，之后t1线程执行了countDown操作，最后，t2线程执行了countDown操作，此时main线程就被唤醒了，可以继续运行。下面，进行详细分析。

   1. countDownLatch.await调用如下,调用之后主线程被park

   ![](resource\CountDonwLatchAwait.png)

   2. t1线程执行countDownLatch.countDown操作，主要调用的函数如下

      ![](resource\CountDownLatchCountDown.png)

   3. t2线程执行countDownLatch.countDown操作，主要调用的函数如下

      ![](resource\CountDownLatchCountDown2.png)

   4. main线程获取cpu资源，继续运行，由于main线程是在parkAndCheckInterrupt函数中被禁止的，所以此时，继续在parkAndCheckInterrupt函数运行

      ![](resource\CountDownLatchMainUnpark.png)

      说明: main线程恢复，继续在parkAndCheckInterrupt函数中运行，之后又会回到最终达到的状态为AQS的state为0，并且head与tail指向同一个结点，该节点的额nextWaiter域还是指向SHARED结点。

3. CountDownLatch一次可以唤醒几个任务?

   因为await的时候使用的是tryAcquireShared(共享锁),所以唤醒的是多个任务

4. CountDownLatch有哪些主要方法?

   await(),countDown() 详见第二条面试题

   await

   ```java
   public void await() throws InterruptedException {
       // 转发到sync对象上
       sync.acquireSharedInterruptibly(1);
   }
   public final void acquireSharedInterruptibly(int arg)
           throws InterruptedException {
       if (Thread.interrupted())
           throw new InterruptedException();
       if (tryAcquireShared(arg) < 0)
           doAcquireSharedInterruptibly(arg);
   }
   protected int tryAcquireShared(int acquires) {
       return (getState() == 0) ? 1 : -1;
   }
   private void doAcquireSharedInterruptibly(int arg) throws InterruptedException {
       // 添加节点至等待队列
       final Node node = addWaiter(Node.SHARED);
       boolean failed = true;
       try {
           for (;;) { // 无限循环
               // 获取node的前驱节点
               final Node p = node.predecessor();
               if (p == head) { // 前驱节点为头节点
                   // 试图在共享模式下获取对象状态
                   int r = tryAcquireShared(arg);
                   if (r >= 0) { // 获取成功
                       // 设置头节点并进行繁殖
                       setHeadAndPropagate(node, r);
                       // 设置节点next域
                       p.next = null; // help GC
                       failed = false;
                       return;
                   }
               }
               if (shouldParkAfterFailedAcquire(p, node) &&
                   parkAndCheckInterrupt()) // 在获取失败后是否需要禁止线程并且进行中断检查
                   // 抛出异常
                   throw new InterruptedException();
           }
       } finally {
           if (failed)
               cancelAcquire(node);
       }
   }
   private void setHeadAndPropagate(Node node, int propagate) {
       // 获取头节点
       Node h = head; // Record old head for check below
       // 设置头节点
       setHead(node);
       /*
           * Try to signal next queued node if:
           *   Propagation was indicated by caller,
           *     or was recorded (as h.waitStatus either before
           *     or after setHead) by a previous operation
           *     (note: this uses sign-check of waitStatus because
           *      PROPAGATE status may transition to SIGNAL.)
           * and
           *   The next node is waiting in shared mode,
           *     or we don't know, because it appears null
           *
           * The conservatism in both of these checks may cause
           * unnecessary wake-ups, but only when there are multiple
           * racing acquires/releases, so most need signals now or soon
           * anyway.
           */
       // 进行判断
       if (propagate > 0 || h == null || h.waitStatus < 0 ||
           (h = head) == null || h.waitStatus < 0) {
           // 获取节点的后继
           Node s = node.next;
           if (s == null || s.isShared()) // 后继为空或者为共享模式
               // 以共享模式进行释放
               doReleaseShared();
       }
   }
   private void doReleaseShared() {
       /*
           * Ensure that a release propagates, even if there are other
           * in-progress acquires/releases.  This proceeds in the usual
           * way of trying to unparkSuccessor of head if it needs
           * signal. But if it does not, status is set to PROPAGATE to
           * ensure that upon release, propagation continues.
           * Additionally, we must loop in case a new node is added
           * while we are doing this. Also, unlike other uses of
           * unparkSuccessor, we need to know if CAS to reset status
           * fails, if so rechecking.
           */
       // 无限循环
       for (;;) {
           // 保存头节点
           Node h = head;
           if (h != null && h != tail) { // 头节点不为空并且头节点不为尾结点
               // 获取头节点的等待状态
               int ws = h.waitStatus; 
               if (ws == Node.SIGNAL) { // 状态为SIGNAL
                   if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0)) // 不成功就继续
                       continue;            // loop to recheck cases
                   // 释放后继结点
                   unparkSuccessor(h);
               }
               else if (ws == 0 &&
                           !compareAndSetWaitStatus(h, 0, Node.PROPAGATE)) // 状态为0并且不成功，继续
                   continue;                // loop on failed CAS
           }
           if (h == head) // 若头节点改变，继续循环  
               break;
       }
   }
   
   ```

   ![](resource\CountDownLatchAwaitDetail.png)

   countDown

   ```java
   public void countDown() {
       sync.releaseShared(1);
   }
   public final boolean releaseShared(int arg) {
       if (tryReleaseShared(arg)) {
           doReleaseShared();
           return true;
       }
       return false;
   }
   protected boolean tryReleaseShared(int releases) {
       // Decrement count; signal when transition to zero
       // 无限循环
       for (;;) {
           // 获取状态
           int c = getState();
           if (c == 0) // 没有被线程占有
               return false;
           // 下一个状态
           int nextc = c-1;
           if (compareAndSetState(c, nextc)) // 比较并且设置成功
               return nextc == 0;
       }
   }
   private void doReleaseShared() {
       /*
           * Ensure that a release propagates, even if there are other
           * in-progress acquires/releases.  This proceeds in the usual
           * way of trying to unparkSuccessor of head if it needs
           * signal. But if it does not, status is set to PROPAGATE to
           * ensure that upon release, propagation continues.
           * Additionally, we must loop in case a new node is added
           * while we are doing this. Also, unlike other uses of
           * unparkSuccessor, we need to know if CAS to reset status
           * fails, if so rechecking.
           */
       // 无限循环
       for (;;) {
           // 保存头节点
           Node h = head;
           if (h != null && h != tail) { // 头节点不为空并且头节点不为尾结点
               // 获取头节点的等待状态
               int ws = h.waitStatus; 
               if (ws == Node.SIGNAL) { // 状态为SIGNAL
                   if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0)) // 不成功就继续
                       continue;            // loop to recheck cases
                   // 释放后继结点
                   unparkSuccessor(h);
               }
               else if (ws == 0 &&
                           !compareAndSetWaitStatus(h, 0, Node.PROPAGATE)) // 状态为0并且不成功，继续
                   continue;                // loop on failed CAS
           }
           if (h == head) // 若头节点改变，继续循环  
               break;
       }
   }
   
   ```

   ![](resource\CountDownLatchCountDownDetail.png)

5. CountDownLatch适用于什么场景?

   两组任务之间有依赖关系

6. 实现一个容器，提供两个方法，add，size 写两个线程，线程1添加10个元素到容器中，线程2实现监控元素的个数，当个数到5个时，线程2给出提示并结束?

   以下实现只是为了演示CountDownLatch,实际开发不会这么用

   ```java
   public class CountDownLanchTest {
       //实现一个容器，提供两个方法，add，size 写两个线程，线程1添加10个元素到容器中，线程2实现监控元素的个数，当个数到5个时，线程2给出提示并结束
       volatile List<Object> list=new ArrayList<>();
       public CountDownLatch countDownLatch=new CountDownLatch(1);
       void add(Object o){
           list.add(o);
       }
       int getSize(){
           return list.size();
       }
       public static void main(String[] args) throws InterruptedException {
           CountDownLanchTest countDownLanchTest=new CountDownLanchTest();
           A a=new A(countDownLanchTest);
           B b=new B(countDownLanchTest);
           new Thread(b).start();
           Thread.sleep(1000);
           new Thread(a).start();
   
       }
       static class A implements Runnable{
           CountDownLanchTest countDownLanchTest;
           public A(CountDownLanchTest countDownLanchTest) {
               this.countDownLanchTest=countDownLanchTest;
           }
           @Override
           public void run() {
               for(int i=0;i<10;i++){
                   countDownLanchTest.add(new Object());
                   System.out.println("线程"+Thread.currentThread().getName()+"添加第"+countDownLanchTest.getSize());
                   if(i==4){
                       countDownLanchTest.countDownLatch.countDown();
                   }
                   try {
                       Thread.sleep(50);
                   } catch (InterruptedException e) {
                       e.printStackTrace();
                   }
   
               }
           }
       }
       static class B implements Runnable{
           CountDownLanchTest countDownLanchTest;
           public B(CountDownLanchTest countDownLanchTest) {
               this.countDownLanchTest=countDownLanchTest;
           }
           @Override
           public void run() {
   
               int size=countDownLanchTest.getSize();
               if(size!=5){
                   try {
                       countDownLanchTest.countDownLatch.await();
                   } catch (InterruptedException e) {
                       e.printStackTrace();
                   }
               }
               System.out.println("线程"+Thread.currentThread().getName()+"查看到集合数量"+countDownLanchTest.getSize()+"退出啦");
           }
       }
   }
   ```



### 3.14.2  CyclicBarrier

#### 3.14.2.1 面试题

1. 什么是CyclicBarrier

   对于CountDownLatch，其他线程为游戏玩家，比如英雄联盟，主线程为控制游戏开始的线程。在所有的玩家都准备好之前，主线程是处于等待状态的，也就是游戏不能开始。当所有的玩家准备好之后，下一步的动作实施者为主线程，即开始游戏。

   对于CyclicBarrier，假设有一家公司要全体员工进行团建活动，活动内容为翻越三个障碍物，每一个人翻越障碍物所用的时间是不一样的。但是公司要求所有人在翻越当前障碍物之后再开始翻越下一个障碍物，也就是所有人翻越第一个障碍物之后，才开始翻越第二个，以此类推。类比地，每一个员工都是一个“其他线程”。当所有人都翻越的所有的障碍物之后，程序才结束。而主线程可能早就结束了，这里我们不用管主线程。

2. CyclicBarrier底层实现原理

   通过以下示例来讲解

   ```java
   import java.util.concurrent.BrokenBarrierException;
   import java.util.concurrent.CyclicBarrier;
   
   class MyThread extends Thread {
       private CyclicBarrier cb;
       public MyThread(String name, CyclicBarrier cb) {
           super(name);
           this.cb = cb;
       }
       
       public void run() {
           System.out.println(Thread.currentThread().getName() + " going to await");
           try {
               cb.await();
               System.out.println(Thread.currentThread().getName() + " continue");
           } catch (Exception e) {
               e.printStackTrace();
           }
       }
   }
   public class CyclicBarrierDemo {
       public static void main(String[] args) throws InterruptedException, BrokenBarrierException {
           CyclicBarrier cb = new CyclicBarrier(3, new Thread("barrierAction") {
               public void run() {
                   System.out.println(Thread.currentThread().getName() + " barrier action");
                   
               }
           });
           MyThread t1 = new MyThread("t1", cb);
           MyThread t2 = new MyThread("t2", cb);
           t1.start();
           t2.start();
           System.out.println(Thread.currentThread().getName() + " going to await");
           cb.await();
           System.out.println(Thread.currentThread().getName() + " continue");
   
       }
   }
   //某次可能的结果
   t1 going to await
   main going to await
   t2 going to await
   t2 barrier action
   t2 continue
   t1 continue
   main continue
   
   ```

   时序图

   ![](resource\CyclicBarrierDemoTimeSeries.png)

   1. 主线程执行await

      ![](resource\CyclicBarrierDemoMainAwait.png)

      说明: 由于ReentrantLock的默认采用非公平策略，所以在dowait函数中调用的是ReentrantLock.NonfairSync的lock函数，由于此时AQS的状态是0，表示还没有被任何线程占用，故main线程可以占用，之后在dowait中会调用trip.await函数，最终的结果是条件队列中存放了一个包含main线程的结点，并且被禁止运行了，同时，main线程所拥有的资源也被释放了，可以供其他线程获取。

   2. t1线程执行await

      ![](C:\Users\index-dev\Desktop\javamap\java-map\resource\CyclicBarrierDemoT1Await.png)

      说明: 可以看到，之后condition queue(条件队列)里面有两个节点，包含t1线程的结点插入在队列的尾部，并且t1线程也被禁止了，因为执行了park操作，此时两个线程都被禁止了

   3. t2线程执行await

      ![](resource\CyclicBarrierDemoT2Await.png)

      说明: 由上图可知，在t2线程执行await操作后，会直接执行command.run方法，不是重新开启一个线程，而是最后进入屏障的线程执行。同时，会将Condition queue中的所有节点都转移到Sync queue中，并且最后main线程会被unpark，可以继续运行。main线程获取cpu资源，继续运行。

   4. main线程获取cpu资源，继续运行

      ![](resource\CyclicBarrierDemoMainRun.png)

      其中，由于main线程是在AQS.CO的wait中被park的，所以恢复时，会继续在该方法中运行。运行过后，t1线程被unpark，它获得cpu资源可以继续运行。

   5. t1线程获取cpu资源，继续运行

      ![](resource\CyclicBarrierDemoT1Run.png)

      其中，由于t1线程是在AQS.CO的wait方法中被park，所以恢复时，会继续在该方法中运行。运行过后，Sync queue中保持着一个空节点。头节点与尾节点均指向它。

      注意: 在线程await过程中中断线程会抛出异常，所有进入屏障的线程都将被释放。至于CyclicBarrier的其他用法，读者可以自行查阅API，不再累赘。

   

3. CountDownLatch和CyclicBarrier对比

   1. countDownLatch是减计数器后执行主线程,cyclicBarrier是加计数器后将线程放到codition队列中(减计数器)并完成减等待进入屏障的线程数量(count)

   2. countDownLatch减完之后不可重复,cyclicBarrier破坏屏障之后会createNewGeneration进入下一个循环

      注:有资料说countDownLatch中下一步是主线程并不准确,下一步也可以是其他线程

4. CyclicBarrier的核心函数有哪些

   doAwait(),nextGeneration(),breakBarrier()

   doAwait

   ```java
   private int dowait(boolean timed, long nanos)
       throws InterruptedException, BrokenBarrierException,
               TimeoutException {
       // 保存当前锁
       final ReentrantLock lock = this.lock;
       // 锁定
       lock.lock();
       try {
           // 保存当前代
           final Generation g = generation;
           
           if (g.broken) // 屏障被破坏，抛出异常
               throw new BrokenBarrierException();
   
           if (Thread.interrupted()) { // 线程被中断
               // 损坏当前屏障，并且唤醒所有的线程，只有拥有锁的时候才会调用
               breakBarrier();
               // 抛出异常
               throw new InterruptedException();
           }
           
           // 减少正在等待进入屏障的线程数量
           int index = --count;
           if (index == 0) {  // 正在等待进入屏障的线程数量为0，所有线程都已经进入
               // 运行的动作标识
               boolean ranAction = false;
               try {
                   // 保存运行动作
                   final Runnable command = barrierCommand;
                   if (command != null) // 动作不为空
                       // 运行
                       command.run();
                   // 设置ranAction状态
                   ranAction = true;
                   // 进入下一代
                   nextGeneration();
                   return 0;
               } finally {
                   if (!ranAction) // 没有运行的动作
                       // 损坏当前屏障
                       breakBarrier();
               }
           }
   
           // loop until tripped, broken, interrupted, or timed out
           // 无限循环
           for (;;) {
               try {
                   if (!timed) // 没有设置等待时间
                       // 等待
                       trip.await(); 
                   else if (nanos > 0L) // 设置了等待时间，并且等待时间大于0
                       // 等待指定时长
                       nanos = trip.awaitNanos(nanos);
               } catch (InterruptedException ie) { 
                   if (g == generation && ! g.broken) { // 等于当前代并且屏障没有被损坏
                       // 损坏当前屏障
                       breakBarrier();
                       // 抛出异常
                       throw ie;
                   } else { // 不等于当前带后者是屏障被损坏
                       // We're about to finish waiting even if we had not
                       // been interrupted, so this interrupt is deemed to
                       // "belong" to subsequent execution.
                       // 中断当前线程
                       Thread.currentThread().interrupt();
                   }
               }
   
               if (g.broken) // 屏障被损坏，抛出异常
                   throw new BrokenBarrierException();
   
               if (g != generation) // 不等于当前代
                   // 返回索引
                   return index;
   
               if (timed && nanos <= 0L) { // 设置了等待时间，并且等待时间小于0
                   // 损坏屏障
                   breakBarrier();
                   // 抛出异常
                   throw new TimeoutException();
               }
           }
       } finally {
           // 释放锁
           lock.unlock();
       }
   }
   ```

   ![](resource\CyclicBarrierDoAwait.png)

### nextGeneration

```java
private void nextGeneration() {
    // signal completion of last generation
    // 唤醒所有线程
    trip.signalAll();
    // set up next generation
    // 恢复正在等待进入屏障的线程数量
    count = parties;
    // 新生一代
    generation = new Generation();
}
public final void signalAll() {
    if (!isHeldExclusively()) // 不被当前线程独占，抛出异常
        throw new IllegalMonitorStateException();
    // 保存condition队列头节点
    Node first = firstWaiter;
    if (first != null) // 头节点不为空
        // 唤醒所有等待线程
        doSignalAll(first);
}
private void doSignalAll(Node first) {
    // condition队列的头节点尾结点都设置为空
    lastWaiter = firstWaiter = null;
    // 循环
    do {
        // 获取first结点的nextWaiter域结点
        Node next = first.nextWaiter;
        // 设置first结点的nextWaiter域为空
        first.nextWaiter = null;
        // 将first结点从condition队列转移到sync队列
        transferForSignal(first);
        // 重新设置first
        first = next;
    } while (first != null);
}
final boolean transferForSignal(Node node) {
    /*
        * If cannot change waitStatus, the node has been cancelled.
        */
    if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
        return false;

    /*
        * Splice onto queue and try to set waitStatus of predecessor to
        * indicate that thread is (probably) waiting. If cancelled or
        * attempt to set waitStatus fails, wake up to resync (in which
        * case the waitStatus can be transiently and harmlessly wrong).
        */
    Node p = enq(node);
    int ws = p.waitStatus;
    if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
        LockSupport.unpark(node.thread);
    return true;
}
private Node enq(final Node node) {
    for (;;) { // 无限循环，确保结点能够成功入队列
        // 保存尾结点
        Node t = tail;
        if (t == null) { // 尾结点为空，即还没被初始化
            if (compareAndSetHead(new Node())) // 头节点为空，并设置头节点为新生成的结点
                tail = head; // 头节点与尾结点都指向同一个新生结点
        } else { // 尾结点不为空，即已经被初始化过
            // 将node结点的prev域连接到尾结点
            node.prev = t; 
            if (compareAndSetTail(t, node)) { // 比较结点t是否为尾结点，若是则将尾结点设置为node
                // 设置尾结点的next域为node
                t.next = node; 
                return t; // 返回尾结点
            }
        }
    }
}
```

![](resource\CyclicBarrierNewGeneration.png)

breakBarrier

```java
private void breakBarrier() {
    // 设置状态
    generation.broken = true;
    // 恢复正在等待进入屏障的线程数量
    count = parties;
    // 唤醒所有线程
    trip.signalAll();
}
```





### 3.14.3 semaphore

#### 3.14.3.1 面试题

1. 什么是Semaphore

   Semaphore底层是基于AbstractQueuedSynchronizer来实现的。Semaphore称为计数信号量，它允许n个任务同时访问某个资源，可以将信号量看做是在向外分发使用资源的许可证，只有成功获取许可证，才能使用资源。

2. Semaphore内部原理

   通过demo来讲解原理

   ```java
   import java.util.concurrent.Semaphore;
   
   class MyThread extends Thread {
       private Semaphore semaphore;
       public MyThread(String name, Semaphore semaphore) {
           super(name);
           this.semaphore = semaphore;
       }
       public void run() {        
           int count = 3;
           System.out.println(Thread.currentThread().getName() + " trying to acquire");
           try {
               semaphore.acquire(count);
               System.out.println(Thread.currentThread().getName() + " acquire successfully");
               Thread.sleep(1000);
           } catch (InterruptedException e) {
               e.printStackTrace();
           } finally {
               semaphore.release(count);
               System.out.println(Thread.currentThread().getName() + " release successfully");
           }
       }
   }
   
   public class SemaphoreDemo {
       public final static int SEM_SIZE = 10;
       
       public static void main(String[] args) {
           Semaphore semaphore = new Semaphore(SEM_SIZE);
           MyThread t1 = new MyThread("t1", semaphore);
           MyThread t2 = new MyThread("t2", semaphore);
           t1.start();
           t2.start();
           int permits = 5;
           System.out.println(Thread.currentThread().getName() + " trying to acquire");
           try {
               semaphore.acquire(permits);
               System.out.println(Thread.currentThread().getName() + " acquire successfully");
               Thread.sleep(1000);
           } catch (InterruptedException e) {
               e.printStackTrace();
           } finally {
               semaphore.release();
               System.out.println(Thread.currentThread().getName() + " release successfully");
           }      
       }
   }
   //可能的结果
   main trying to acquire
   main acquire successfully
   t1 trying to acquire
   t1 acquire successfully
   t2 trying to acquire
   t1 release successfully
   main release successfully
   t2 acquire successfully
   t2 release successfully
   ```

   时序图

   ![](resource\SemaphoreTimeSeries.png)

   1. main线程执行acquire

      ![](resource\SemaphoreMainAcquire.png)

      此时，可以看到只是AQS的state变为了5，main线程并没有被阻塞，可以继续运行

   2. t1线程执行acquire

   ![](resource\SemaphoreT1Acquire.png)

   3. t2执行acquire

      ![](resource\SemaphoreT2Acquire.png)

      此时，t2线程获取许可不会成功，之后会导致其被禁止运行，值得注意的是，AQS的state还是为2。

   4. t1执行semaphore.release操作

      ![](resource\SemaphoreT1Release.png)

      此时，t2线程将会被unpark，并且AQS的state为5，t2获取cpu资源后可以继续运行。

   5. main线程执行semaphore.release操作。

      ![](resource\SemaphoreMainRelease.png)

      此时，t2线程还会被unpark，但是不会产生影响，此时，只要t2线程获得CPU资源就可以运行了。此时，AQS的state为10

   6. t2获取CPU资源，继续运行，此时t2需要恢复现场，回到parkAndCheckInterrupt函数中，也是在should继续运行

      ![](resource\SemaphoreT2Run.png)

      此时，可以看到，Sync queue中只有一个结点，头节点与尾节点都指向该结点，在setHeadAndPropagate的函数中会设置头节点并且会unpark队列中的其他结点。

   7. t2线程执行semaphore.release操作。

      ![](resource\SemaphoreT2Release.png)

       t2线程经过release后，此时信号量的许可又变为10个了，此时Sync queue中的结点还是没有变化。

3. Semaphore常用方法有哪些? 如何实现线程同步和互斥的

   acquire(),release()

   1. acquire

      ```java
      public void acquire() throws InterruptedException {
          sync.acquireSharedInterruptibly(1);
      }
      ```

      ![](resource\SemaphoreAcquire.png)

      注意:在第5步中会调用addWaiter方法将当前线程放到同步队列中,在发现获取不到执行权限后自己阻塞,被唤醒后死循环执行第6-7步尝试再次获取执行权限

   2. release

      ```java
      public void release() {
          sync.releaseShared(1);
      }
      ```

      ![](resource\SemaphoreRelease.png)

4. Semaphore适合用在什么场景

   适用多个线程同时访问共享资源的情况下限制同时访问的数量

5. 单独使用Semaphore是不会使用到AQS的条件队列

   不同于CyclicBarrier和ReentrantLock，单独使用Semaphore是不会使用到AQS的条件队列的，其实，只有进行await操作才会进入条件队列，其他的都是在同步队列中，只是当前线程会被park

6. 关于Semaphore的几个问题

   + Semaphore初始化有10个令牌，11个线程同时各调用1次acquire方法，会发生什么?

     一个线程无法获取令牌阻塞

   + Semaphore初始化有10个令牌，一个线程重复调用11次acquire方法，会发生什么?

     第11次阻塞,没有类似锁的可重入概念,也没有其他线程会唤醒他

   + Semaphore初始化有1个令牌，1个线程调用一次acquire方法，然后调用两次release方法，之后另外一个线程调用acquire(2)方法，此线程能够获取到足够的令牌并继续运行吗?

     能,release方法增加令牌,而且令牌数量可以大于默认值

   + Semaphore初始化有2个令牌，一个线程调用1次release方法，然后一次性获取3个令牌，会获取到吗?

     能,同上,而且semaphore没有对acquire和release方法的顺序限制,只要调用release就会增加令牌数量



### 3.14.4 Phaser

#### 3.14.4.1 面试题

1. Phaser主要解决什么问题?与CyclicBarrier和CountDownLatch有什么区别

   Phaser又称“阶段器”，用来解决控制多个线程分阶段共同完成任务的情景问题。它与CountDownLatch和CyclicBarrier类似，都是等待一组线程完成工作后再执行下一步，协调线程的工作。但在CountDownLatch和CyclicBarrier中我们都不可以动态的配置parties，而Phaser可以动态注册需要协调的线程，相比CountDownLatch和CyclicBarrier就会变得更加灵活。

2. 如果使用CountDownLatch实现Phaser的功能如何实现

   //TODO

3. Phaser的运行机制

   ![](resource\ParserRun.png)

   + **Registration(注册)**

     跟其他barrier不同，在phaser上注册的parties会随着时间的变化而变化。任务可以随时注册(使用方法register,bulkRegister注册，或者由构造器确定初始parties)，并且在任何抵达点可以随意地撤销注册(方法arriveAndDeregister)。就像大多数基本的同步结构一样，注册和撤销只影响内部count；不会创建更深的内部记录，所以任务不能查询他们是否已经注册。(不过，可以通过继承来实现类似的记录)

   + **Synchronization(同步机制)**

     和CyclicBarrier一样，Phaser也可以重复await。方法arriveAndAwaitAdvance的效果类似CyclicBarrier.await。phaser的每一代都有一个相关的phase number，初始值为0，当所有注册的任务都到达phaser时phase+1，到达最大值(Integer.MAX_VALUE)之后清零。使用phase number可以独立控制 到达phaser 和 等待其他线程 的动作，通过下面两种类型的方法:

     > - **Arrival(到达机制)** arrive和arriveAndDeregister方法记录到达状态。这些方法不会阻塞，但是会返回一个相关的arrival phase number；也就是说，phase number用来确定到达状态。当所有任务都到达给定phase时，可以执行一个可选的函数，这个函数通过重写onAdvance方法实现，通常可以用来控制终止状态。重写此方法类似于为CyclicBarrier提供一个barrierAction，但比它更灵活。
     > - **Waiting(等待机制)** awaitAdvance方法需要一个表示arrival phase number的参数，并且在phaser前进到与给定phase不同的phase时返回。和CyclicBarrier不同，即使等待线程已经被中断，awaitAdvance方法也会一直等待。中断状态和超时时间同样可用，但是当任务等待中断或超时后未改变phaser的状态时会遭遇异常。如果有必要，在方法forceTermination之后可以执行这些异常的相关的handler进行恢复操作，Phaser也可能被ForkJoinPool中的任务使用，这样在其他任务阻塞等待一个phase时可以保证足够的并行度来执行任务。

   + **Termination(终止机制)**

     可以用isTerminated方法检查phaser的终止状态。在终止时，所有同步方法立刻返回一个负值。在终止时尝试注册也没有效果。当调用onAdvance返回true时Termination被触发。当deregistration操作使已注册的parties变为0时，onAdvance的默认实现就会返回true。也可以重写onAdvance方法来定义终止动作。forceTermination方法也可以释放等待线程并且允许它们终止。

   + **Tiering(分层结构)**

     Phaser支持分层结构(树状构造)来减少竞争。注册了大量parties的Phaser可能会因为同步竞争消耗很高的成本， 因此可以设置一些子Phaser来共享一个通用的parent。这样的话即使每个操作消耗了更多的开销，但是会提高整体吞吐量。 在一个分层结构的phaser里，子节点phaser的注册和取消注册都通过父节点管理。子节点phaser通过构造或方法register、bulkRegister进行首次注册时，在其父节点上注册。子节点phaser通过调用arriveAndDeregister进行最后一次取消注册时，也在其父节点上取消注册。

   + **Monitoring(状态监控)** 

     由于同步方法可能只被已注册的parties调用，所以phaser的当前状态也可能被任何调用者监控。在任何时候，可以通过getRegisteredParties获取parties数，其中getArrivedParties方法返回已经到达当前phase的parties数。当剩余的parties(通过方法getUnarrivedParties获取)到达时，phase进入下一代。这些方法返回的值可能只表示短暂的状态，所以一般来说在同步结构里并没有啥卵用。

     

4. Phaser的使用实例

   

   
