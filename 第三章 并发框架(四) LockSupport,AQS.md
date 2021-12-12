# 第三章 并发框架(四) LockSupport,AQS

## 3.8 LockSupport详解

### 3.8.1 面试题

1. 为什么LockSupport也是基础核心类?Aqs框架主要借助两个类 unsafe(cas操作)和LockSupport
2. 分别通过wati/notify和lockSupport的park/unpark实现同步
3. LockSupport.park()会释放资源吗?condition.await()呢
4. Thread.sleep()/Object.wait()/Condition.await()/LockSupport.park() 对比 **重点**
5. 如果wait()之前执行notify()会怎么样,park()之前执行unpark()会怎么样

### 3.8.2 为什么LockSupport是基础核心类?

LockSupport用来创建锁和其他同步类的基本线程阻塞原语,当调用LockSupport.park()时表示当前线程会等待,直到获得许可,调用LockSupport.unpark()时,需要目标线程对象作为参数

AQS的需要ucsafe来执行cas操作,需要LockSupport来阻塞线程(LockSupport底层还是unsafe)

### 3.8.3  分别通过wati/notify和lockSupport的park/unpark实现同步

```java
class MyThread extends Thread {
    
    public void run() {
        synchronized (this) {
            System.out.println("before notify");            
            notify();
            System.out.println("after notify");    
        }
    }
}
public class WaitAndNotifyDemo {
    public static void main(String[] args) throws InterruptedException {
        MyThread myThread = new MyThread();            
        synchronized (myThread) {
            try {        
                myThread.start();
                // 主线程睡眠3s
                Thread.sleep(3000);
                System.out.println("before wait");
                // 阻塞主线程
                myThread.wait();
                System.out.println("after wait");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }            
        }        
    }
}
//结果
before wait
before notify
after notify
after wait

```

```java
import java.util.concurrent.locks.LockSupport;

class MyThread extends Thread {
    private Object object;
    public MyThread(Object object) {
        this.object = object;
    }
    public void run() {
        System.out.println("before unpark");
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        // 获取blocker
        System.out.println("Blocker info " + LockSupport.getBlocker((Thread) object));
        // 释放许可
        LockSupport.unpark((Thread) object);
        // 休眠500ms，保证先执行park中的setBlocker(t, null);
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        // 再次获取blocker
        System.out.println("Blocker info " + LockSupport.getBlocker((Thread) object));
        System.out.println("after unpark");
    }
}
public class test {
    public static void main(String[] args) {
        MyThread myThread = new MyThread(Thread.currentThread());
        myThread.start();
        System.out.println("before park");
        // 获取许可
        LockSupport.park("ParkAndUnparkDemo");
        System.out.println("after park");
    }
}
//结果
before park
before unpark
Blocker info ParkAndUnparkDemo
after park
Blocker info null
after unpark
```

特别注意:先执行notify再执行wait是无效的,会一直阻塞,但是先执行unpark,再执行park可以有效解除线程阻塞

```java
import java.util.concurrent.locks.LockSupport;

class MyThread extends Thread {
    private Object object;

    public MyThread(Object object) {
        this.object = object;
    }

    public void run() {
        System.out.println("before unpark");        
        // 释放许可
        LockSupport.unpark((Thread) object);
        System.out.println("after unpark");
    }
}

public class ParkAndUnparkDemo {
    public static void main(String[] args) {
        MyThread myThread = new MyThread(Thread.currentThread());
        myThread.start();
        try {
            // 主线程睡眠3s
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("before park");
        // 获取许可
        LockSupport.park("ParkAndUnparkDemo");
        System.out.println("after park");
    }
}
//结果
before unpark
after unpark
before park
after park
```

```java
import java.util.concurrent.locks.LockSupport;

class MyThread extends Thread {
    private Object object;

    public MyThread(Object object) {
        this.object = object;
    }

    public void run() {
        System.out.println("before interrupt");        
        try {
            // 休眠3s
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }    
        Thread thread = (Thread) object;
        // 中断线程
        thread.interrupt();
        System.out.println("after interrupt");
    }
}
public class InterruptDemo {
    public static void main(String[] args) {
        MyThread myThread = new MyThread(Thread.currentThread());
        myThread.start();
        System.out.println("before park");
        // 获取许可
        LockSupport.park("ParkAndUnparkDemo");
        System.out.println("after park");
    }
}
//结果
before park
before interrupt
after interrupt
after park

```

通过以上可以发现interrupt线程的情况下可以释放阻塞中的线程

### 3.8.4  LockSupport.park()会释放资源吗?condition.await()呢
不会,LockSupport.park()只负责阻塞线程不会释放资源,condition.wait()的方法中有释放资源的代码和LockSupport.park(),所以会释放资源

### 3.8.5  Thread.sleep()/Object.wait()/Condition.await()/LockSupport.park() 对比 **重点**

1. Thread.sleep()
   + 不会释放资源
   + 需要设置时间
   + 会自动唤醒
2. Object.wait()
   + 释放资源
   + 可以设置/不设置时间
   + 设置时间自动唤醒/不设置时间一直阻塞
   + 在wait()之前执行notify()无效
3. Condition.await()
   + 和Object.wait()基本一致,不同点是 
     1. Condition.await()会将线程添加到条件队列中
     2. Condition.await()会完全释放锁,之后调用park()
4. LockSupport.park()
   + 不释放资源
   + 可以设置/不设置时间
   + 设置时间自动唤醒/不设置时间一直阻塞
   + 在park()之前执行unpark(),park()会失效,直接继续执行

### 3.8.6  如果wait()之前执行notify()会怎么样,park()之前执行unpark()会怎么样

wait()之前执行notify()无效,park()之前执行unpark()会使park()无效

##  3.9 AQS

### 3.9.1 面试题

1. 什么是AQS,为什么它是核心
2. AQS核心思想是什么,如何实现
3. AQS定义了什么样的资源获取方式?
4. AQS使用什么设计模式
5. AQS底层数据结构
6. AQS源码分析
7. AQS的核心方法
8. AQS使用实例

### 3.9.2 什么是AQS,为什么它是核心

AQS(AbstractQueuedSynchronizer)是构建锁和同步器的框架,使用AQS能够简单高效的构建应用广泛的大量同步器,例如 ReentrantLock,Semaphore等等,也可以根据AQS自定义同步器

### 3.9.3  AQS核心思想是什么,如何实现

AQS核心思想是,如果被请求的共享资源空闲,则将请求资源的线程设置为有效的工作线程,并将共享资源设置为占用状态

如果被请求的资源被占用,就需要一套线程阻塞等待以及被唤醒时锁分配机制,这个机制是AQS通过clh队列实现,即将暂时获取不到锁的线程放入队列中

CLH(Craig,Landin,and Hagersten)队列是是一个虚拟的双向队列(虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系)。AQS是将每条请求共享资源的线程封装成一个CLH锁队列的一个结点(Node)来实现锁的分配。

AQS使用int 成员变量来表示锁的同步状态,通过FIFO队列来获取资源线程的排队工作.AQS通过cas实现对变量的原子操作

```java
private volatile int state;//共享变量，使用volatile修饰保证线程可见性
//操作state
//返回同步状态的当前值
protected final int getState() {  
        return state;
}
 // 设置同步状态的值
protected final void setState(int newState) { 
        state = newState;
}
//原子地(CAS操作)将同步状态值设置为给定值update如果当前同步状态的值等于expect(期望值)
protected final boolean compareAndSetState(int expect, int update) {
        return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```

### 3.9.4  **AQS对资源的共享方式分为两种**

1. exclusive:独占模式
   + 公平锁:按排队顺序来
   + 非公平锁(ReentrantLock默认):谁抢到是谁的
2. share:共享模式,多线程同时访问共享资源例如ReentrantReadWriteLock里的读锁

不同的自定义同步器的争用共享资源的方式不同,自定义同步器只需要实现共享资源state的获取和释放即可,至于具体线程等待队列的维护,AQS已经在上层做好了

### 3.9.5  AQS使用什么设计模式

AQS使用了模板方法模式自定义同步器需要重写以下方法:

```java
isHeldExclusively()//该线程是否正在独占资源。只有用到condition才需要去实现它。
tryAcquire(int)//独占方式。尝试获取资源，成功则返回true，失败则返回false。
tryRelease(int)//独占方式。尝试释放资源，成功则返回true，失败则返回false。
tryAcquireShared(int)//共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
tryReleaseShared(int)//共享方式。尝试释放资源，成功则返回true，失败则返回false。
```

默认情况下，每个方法都抛出 UnsupportedOperationException。 这些方法的实现必须是内部线程安全的，并且通常应该简短而不是阻塞。AQS类中的其他方法都是final ，所以无法被其他类使用，只有这几个方法可以被其他类使用。

以ReentrantLock为例，state初始化为0，表示未锁定状态。A线程lock()时，会调用tryAcquire()独占该锁并将state+1。此后，其他线程再tryAcquire()时就会失败，直到A线程unlock()到state=0(即释放锁)为止，其它线程才有机会获取该锁。当然，释放锁之前，A线程自己是可以重复获取此锁的(state会累加)，这就是可重入的概念。但要注意，获取多少次就要释放多么次，这样才能保证state是能回到零态的。

### 3.9.6  AQS的底层数据结构

AQS底层使用CLH队列是一种虚拟的双向队列(拟的双向队列即不存在队列实例，仅存在结点之间的关联关系),每个线程被封装成一个节点

ConditionQueue不是必须的,是一个单向链表,只有当使用Condition时，才会存在此单向链表。并且可能会有多个Condition queue。

![](.\resource\AQSclh.png)

### 3.9.7 源码分析

#### 3.9.7.1 类的继承关系

AbstractQueuedSynchronizer继承自AbstractOwnableSynchronizer抽象类，并且实现了Serializable接口，可以进行序列化。

```java
public abstract class AbstractQueuedSynchronizer extends AbstractOwnableSynchronizer implements java.io.Serializable
```

AbstractOwnableSynchronizer源码如下

```java
public abstract class AbstractOwnableSynchronizer implements java.io.Serializable {
    
    // 版本序列号
    private static final long serialVersionUID = 3737899427754241961L;
    // 构造方法
    protected AbstractOwnableSynchronizer() { }
    // 独占模式下的线程
    private transient Thread exclusiveOwnerThread;
    // 设置独占线程 
    protected final void setExclusiveOwnerThread(Thread thread) {
        exclusiveOwnerThread = thread;
    }
    // 获取独占线程 
    protected final Thread getExclusiveOwnerThread() {
        return exclusiveOwnerThread;
    }
}
```

AbstractOwnableSynchronizer抽象类中，可以设置独占资源线程和获取独占资源线程。分别为setExclusiveOwnerThread与getExclusiveOwnerThread方法，这两个方法会被子类调用。

#### 3.9.7.2 内部类-Node

```java
static final class Node {
    // 模式，分为共享与独占
    // 共享模式
    static final Node SHARED = new Node();
    // 独占模式
    static final Node EXCLUSIVE = null;        
    // 结点状态
    // CANCELLED，值为1，表示当前的线程被取消
    // SIGNAL，值为-1，表示当前节点的后继节点包含的线程需要运行，也就是unpark
    // CONDITION，值为-2，表示当前节点在等待condition，也就是在condition队列中
    // PROPAGATE，值为-3，表示当前场景下后续的acquireShared能够得以执行
    // 值为0，表示当前节点在sync队列中，等待着获取锁
    static final int CANCELLED =  1;
    static final int SIGNAL    = -1;
    static final int CONDITION = -2;
    static final int PROPAGATE = -3;        

    // 结点状态
    volatile int waitStatus;        
    // 前驱结点
    volatile Node prev;    
    // 后继结点
    volatile Node next;        
    // 结点所对应的线程
    volatile Thread thread;        
    // 下一个等待者
    Node nextWaiter;
    
    // 结点是否在共享模式下等待
    final boolean isShared() {
        return nextWaiter == SHARED;
    }
    // 获取前驱结点，若前驱结点为空，抛出异常
    final Node predecessor() throws NullPointerException {
        // 保存前驱结点
        Node p = prev; 
        if (p == null) // 前驱结点为空，抛出异常
            throw new NullPointerException();
        else // 前驱结点不为空，返回
            return p;
    }
    // 无参构造方法
    Node() {    // Used to establish initial head or SHARED marker
    }
    // 构造方法
        Node(Thread thread, Node mode) {    // Used by addWaiter
        this.nextWaiter = mode;
        this.thread = thread;
    }
    // 构造方法
    Node(Thread thread, int waitStatus) { // Used by Condition
        this.waitStatus = waitStatus;
        this.thread = thread;
    }
}
```

#### 3.9.7.3 内部类-ConditionObject类

```java
// 内部类
public class ConditionObject implements Condition, java.io.Serializable {
    // 版本号
    private static final long serialVersionUID = 1173984872572414699L;
    /** First node of condition queue. */
    // condition队列的头结点
    private transient Node firstWaiter;
    /** Last node of condition queue. */
    // condition队列的尾结点
    private transient Node lastWaiter;

    /**
        * Creates a new {@code ConditionObject} instance.
        */
    // 构造方法
    public ConditionObject() { }

    // Internal methods

    /**
        * Adds a new waiter to wait queue.
        * @return its new wait node
        */
    // 添加新的waiter到wait队列
    private Node addConditionWaiter() {
        // 保存尾结点
        Node t = lastWaiter;
        // If lastWaiter is cancelled, clean out.
        if (t != null && t.waitStatus != Node.CONDITION) { // 尾结点不为空，并且尾结点的状态不为CONDITION
            // 清除状态为CONDITION的结点
            unlinkCancelledWaiters(); 
            // 将最后一个结点重新赋值给t
            t = lastWaiter;
        }
        // 新建一个结点
        Node node = new Node(Thread.currentThread(), Node.CONDITION);
        if (t == null) // 尾结点为空
            // 设置condition队列的头结点
            firstWaiter = node;
        else // 尾结点不为空
            // 设置为节点的nextWaiter域为node结点
            t.nextWaiter = node;
        // 更新condition队列的尾结点
        lastWaiter = node;
        return node;
    }

    /**
        * Removes and transfers nodes until hit non-cancelled one or
        * null. Split out from signal in part to encourage compilers
        * to inline the case of no waiters.
        * @param first (non-null) the first node on condition queue
        */
    private void doSignal(Node first) {
        // 循环
        do {
            if ( (firstWaiter = first.nextWaiter) == null) // 该节点的nextWaiter为空
                // 设置尾结点为空
                lastWaiter = null;
            // 设置first结点的nextWaiter域
            first.nextWaiter = null;
        } while (!transferForSignal(first) &&
                    (first = firstWaiter) != null); // 将结点从condition队列转移到sync队列失败并且condition队列中的头结点不为空，一直循环
    }

    /**
        * Removes and transfers all nodes.
        * @param first (non-null) the first node on condition queue
        */
    private void doSignalAll(Node first) {
        // condition队列的头结点尾结点都设置为空
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

    /**
        * Unlinks cancelled waiter nodes from condition queue.
        * Called only while holding lock. This is called when
        * cancellation occurred during condition wait, and upon
        * insertion of a new waiter when lastWaiter is seen to have
        * been cancelled. This method is needed to avoid garbage
        * retention in the absence of signals. So even though it may
        * require a full traversal, it comes into play only when
        * timeouts or cancellations occur in the absence of
        * signals. It traverses all nodes rather than stopping at a
        * particular target to unlink all pointers to garbage nodes
        * without requiring many re-traversals during cancellation
        * storms.
        */
    // 从condition队列中清除状态为CANCEL的结点
    private void unlinkCancelledWaiters() {
        // 保存condition队列头结点
        Node t = firstWaiter;
        Node trail = null;
        while (t != null) { // t不为空
            // 下一个结点
            Node next = t.nextWaiter;
            if (t.waitStatus != Node.CONDITION) { // t结点的状态不为CONDTION状态
                // 设置t节点的额nextWaiter域为空
                t.nextWaiter = null;
                if (trail == null) // trail为空
                    // 重新设置condition队列的头结点
                    firstWaiter = next;
                else // trail不为空
                    // 设置trail结点的nextWaiter域为next结点
                    trail.nextWaiter = next;
                if (next == null) // next结点为空
                    // 设置condition队列的尾结点
                    lastWaiter = trail;
            }
            else // t结点的状态为CONDTION状态
                // 设置trail结点
                trail = t;
            // 设置t结点
            t = next;
        }
    }

    // public methods

    /**
        * Moves the longest-waiting thread, if one exists, from the
        * wait queue for this condition to the wait queue for the
        * owning lock.
        *
        * @throws IllegalMonitorStateException if {@link #isHeldExclusively}
        *         returns {@code false}
        */
    // 唤醒一个等待线程。如果所有的线程都在等待此条件，则选择其中的一个唤醒。在从 await 返回之前，该线程必须重新获取锁。
    public final void signal() {
        if (!isHeldExclusively()) // 不被当前线程独占，抛出异常
            throw new IllegalMonitorStateException();
        // 保存condition队列头结点
        Node first = firstWaiter;
        if (first != null) // 头结点不为空
            // 唤醒一个等待线程
            doSignal(first);
    }

    /**
        * Moves all threads from the wait queue for this condition to
        * the wait queue for the owning lock.
        *
        * @throws IllegalMonitorStateException if {@link #isHeldExclusively}
        *         returns {@code false}
        */
    // 唤醒所有等待线程。如果所有的线程都在等待此条件，则唤醒所有线程。在从 await 返回之前，每个线程都必须重新获取锁。
    public final void signalAll() {
        if (!isHeldExclusively()) // 不被当前线程独占，抛出异常
            throw new IllegalMonitorStateException();
        // 保存condition队列头结点
        Node first = firstWaiter;
        if (first != null) // 头结点不为空
            // 唤醒所有等待线程
            doSignalAll(first);
    }

    /**
        * Implements uninterruptible condition wait.
        * <ol>
        * <li> Save lock state returned by {@link #getState}.
        * <li> Invoke {@link #release} with saved state as argument,
        *      throwing IllegalMonitorStateException if it fails.
        * <li> Block until signalled.
        * <li> Reacquire by invoking specialized version of
        *      {@link #acquire} with saved state as argument.
        * </ol>
        */
    // 等待，当前线程在接到信号之前一直处于等待状态，不响应中断
    public final void awaitUninterruptibly() {
        // 添加一个结点到等待队列
        Node node = addConditionWaiter();
        // 获取释放的状态
        int savedState = fullyRelease(node);
        boolean interrupted = false;
        while (!isOnSyncQueue(node)) { // 
            // 阻塞当前线程
            LockSupport.park(this);
            if (Thread.interrupted()) // 当前线程被中断
                // 设置interrupted状态
                interrupted = true; 
        }
        if (acquireQueued(node, savedState) || interrupted) // 
            selfInterrupt();
    }

    /*
        * For interruptible waits, we need to track whether to throw
        * InterruptedException, if interrupted while blocked on
        * condition, versus reinterrupt current thread, if
        * interrupted while blocked waiting to re-acquire.
        */

    /** Mode meaning to reinterrupt on exit from wait */
    private static final int REINTERRUPT =  1;
    /** Mode meaning to throw InterruptedException on exit from wait */
    private static final int THROW_IE    = -1;

    /**
        * Checks for interrupt, returning THROW_IE if interrupted
        * before signalled, REINTERRUPT if after signalled, or
        * 0 if not interrupted.
        */
    private int checkInterruptWhileWaiting(Node node) {
        return Thread.interrupted() ?
            (transferAfterCancelledWait(node) ? THROW_IE : REINTERRUPT) :
            0; 
    }

    /**
        * Throws InterruptedException, reinterrupts current thread, or
        * does nothing, depending on mode.
        */
    private void reportInterruptAfterWait(int interruptMode)
        throws InterruptedException {
        if (interruptMode == THROW_IE)
            throw new InterruptedException();
        else if (interruptMode == REINTERRUPT)
            selfInterrupt();
    }

    /**
        * Implements interruptible condition wait.
        * <ol>
        * <li> If current thread is interrupted, throw InterruptedException.
        * <li> Save lock state returned by {@link #getState}.
        * <li> Invoke {@link #release} with saved state as argument,
        *      throwing IllegalMonitorStateException if it fails.
        * <li> Block until signalled or interrupted.
        * <li> Reacquire by invoking specialized version of
        *      {@link #acquire} with saved state as argument.
        * <li> If interrupted while blocked in step 4, throw InterruptedException.
        * </ol>
        */
    // // 等待，当前线程在接到信号或被中断之前一直处于等待状态
    public final void await() throws InterruptedException {
        if (Thread.interrupted()) // 当前线程被中断，抛出异常
            throw new InterruptedException();
        // 在wait队列上添加一个结点
        Node node = addConditionWaiter();
        // 
        int savedState = fullyRelease(node);
        int interruptMode = 0;
        while (!isOnSyncQueue(node)) {
            // 阻塞当前线程
            LockSupport.park(this);
            if ((interruptMode = checkInterruptWhileWaiting(node)) != 0) // 检查结点等待时的中断类型
                break;
        }
        if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
            interruptMode = REINTERRUPT;
        if (node.nextWaiter != null) // clean up if cancelled
            unlinkCancelledWaiters();
        if (interruptMode != 0)
            reportInterruptAfterWait(interruptMode);
    }

    /**
        * Implements timed condition wait.
        * <ol>
        * <li> If current thread is interrupted, throw InterruptedException.
        * <li> Save lock state returned by {@link #getState}.
        * <li> Invoke {@link #release} with saved state as argument,
        *      throwing IllegalMonitorStateException if it fails.
        * <li> Block until signalled, interrupted, or timed out.
        * <li> Reacquire by invoking specialized version of
        *      {@link #acquire} with saved state as argument.
        * <li> If interrupted while blocked in step 4, throw InterruptedException.
        * </ol>
        */
    // 等待，当前线程在接到信号、被中断或到达指定等待时间之前一直处于等待状态 
    public final long awaitNanos(long nanosTimeout)
            throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        Node node = addConditionWaiter();
        int savedState = fullyRelease(node);
        final long deadline = System.nanoTime() + nanosTimeout;
        int interruptMode = 0;
        while (!isOnSyncQueue(node)) {
            if (nanosTimeout <= 0L) {
                transferAfterCancelledWait(node);
                break;
            }
            if (nanosTimeout >= spinForTimeoutThreshold)
                LockSupport.parkNanos(this, nanosTimeout);
            if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
                break;
            nanosTimeout = deadline - System.nanoTime();
        }
        if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
            interruptMode = REINTERRUPT;
        if (node.nextWaiter != null)
            unlinkCancelledWaiters();
        if (interruptMode != 0)
            reportInterruptAfterWait(interruptMode);
        return deadline - System.nanoTime();
    }

    /**
        * Implements absolute timed condition wait.
        * <ol>
        * <li> If current thread is interrupted, throw InterruptedException.
        * <li> Save lock state returned by {@link #getState}.
        * <li> Invoke {@link #release} with saved state as argument,
        *      throwing IllegalMonitorStateException if it fails.
        * <li> Block until signalled, interrupted, or timed out.
        * <li> Reacquire by invoking specialized version of
        *      {@link #acquire} with saved state as argument.
        * <li> If interrupted while blocked in step 4, throw InterruptedException.
        * <li> If timed out while blocked in step 4, return false, else true.
        * </ol>
        */
    // 等待，当前线程在接到信号、被中断或到达指定最后期限之前一直处于等待状态
    public final boolean awaitUntil(Date deadline)
            throws InterruptedException {
        long abstime = deadline.getTime();
        if (Thread.interrupted())
            throw new InterruptedException();
        Node node = addConditionWaiter();
        int savedState = fullyRelease(node);
        boolean timedout = false;
        int interruptMode = 0;
        while (!isOnSyncQueue(node)) {
            if (System.currentTimeMillis() > abstime) {
                timedout = transferAfterCancelledWait(node);
                break;
            }
            LockSupport.parkUntil(this, abstime);
            if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
                break;
        }
        if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
            interruptMode = REINTERRUPT;
        if (node.nextWaiter != null)
            unlinkCancelledWaiters();
        if (interruptMode != 0)
            reportInterruptAfterWait(interruptMode);
        return !timedout;
    }

    /**
        * Implements timed condition wait.
        * <ol>
        * <li> If current thread is interrupted, throw InterruptedException.
        * <li> Save lock state returned by {@link #getState}.
        * <li> Invoke {@link #release} with saved state as argument,
        *      throwing IllegalMonitorStateException if it fails.
        * <li> Block until signalled, interrupted, or timed out.
        * <li> Reacquire by invoking specialized version of
        *      {@link #acquire} with saved state as argument.
        * <li> If interrupted while blocked in step 4, throw InterruptedException.
        * <li> If timed out while blocked in step 4, return false, else true.
        * </ol>
        */
    // 等待，当前线程在接到信号、被中断或到达指定等待时间之前一直处于等待状态。此方法在行为上等效于: awaitNanos(unit.toNanos(time)) > 0
    public final boolean await(long time, TimeUnit unit)
            throws InterruptedException {
        long nanosTimeout = unit.toNanos(time);
        if (Thread.interrupted())
            throw new InterruptedException();
        Node node = addConditionWaiter();
        int savedState = fullyRelease(node);
        final long deadline = System.nanoTime() + nanosTimeout;
        boolean timedout = false;
        int interruptMode = 0;
        while (!isOnSyncQueue(node)) {
            if (nanosTimeout <= 0L) {
                timedout = transferAfterCancelledWait(node);
                break;
            }
            if (nanosTimeout >= spinForTimeoutThreshold)
                LockSupport.parkNanos(this, nanosTimeout);
            if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
                break;
            nanosTimeout = deadline - System.nanoTime();
        }
        if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
            interruptMode = REINTERRUPT;
        if (node.nextWaiter != null)
            unlinkCancelledWaiters();
        if (interruptMode != 0)
            reportInterruptAfterWait(interruptMode);
        return !timedout;
    }

    //  support for instrumentation

    /**
        * Returns true if this condition was created by the given
        * synchronization object.
        *
        * @return {@code true} if owned
        */
    final boolean isOwnedBy(AbstractQueuedSynchronizer sync) {
        return sync == AbstractQueuedSynchronizer.this;
    }

    /**
        * Queries whether any threads are waiting on this condition.
        * Implements {@link AbstractQueuedSynchronizer#hasWaiters(ConditionObject)}.
        *
        * @return {@code true} if there are any waiting threads
        * @throws IllegalMonitorStateException if {@link #isHeldExclusively}
        *         returns {@code false}
        */
    //  查询是否有正在等待此条件的任何线程
    protected final boolean hasWaiters() {
        if (!isHeldExclusively())
            throw new IllegalMonitorStateException();
        for (Node w = firstWaiter; w != null; w = w.nextWaiter) {
            if (w.waitStatus == Node.CONDITION)
                return true;
        }
        return false;
    }

    /**
        * Returns an estimate of the number of threads waiting on
        * this condition.
        * Implements {@link AbstractQueuedSynchronizer#getWaitQueueLength(ConditionObject)}.
        *
        * @return the estimated number of waiting threads
        * @throws IllegalMonitorStateException if {@link #isHeldExclusively}
        *         returns {@code false}
        */
    // 返回正在等待此条件的线程数估计值
    protected final int getWaitQueueLength() {
        if (!isHeldExclusively())
            throw new IllegalMonitorStateException();
        int n = 0;
        for (Node w = firstWaiter; w != null; w = w.nextWaiter) {
            if (w.waitStatus == Node.CONDITION)
                ++n;
        }
        return n;
    }

    /**
        * Returns a collection containing those threads that may be
        * waiting on this Condition.
        * Implements {@link AbstractQueuedSynchronizer#getWaitingThreads(ConditionObject)}.
        *
        * @return the collection of threads
        * @throws IllegalMonitorStateException if {@link #isHeldExclusively}
        *         returns {@code false}
        */
    // 返回包含那些可能正在等待此条件的线程集合
    protected final Collection<Thread> getWaitingThreads() {
        if (!isHeldExclusively())
            throw new IllegalMonitorStateException();
        ArrayList<Thread> list = new ArrayList<Thread>();
        for (Node w = firstWaiter; w != null; w = w.nextWaiter) {
            if (w.waitStatus == Node.CONDITION) {
                Thread t = w.thread;
                if (t != null)
                    list.add(t);
            }
        }
        return list;
    }
}
```

此类实现了了Condition接口，Condition接口定义了条件操作规范，具体如下

```java
public interface Condition {

    // 等待，当前线程在接到信号或被中断之前一直处于等待状态
    void await() throws InterruptedException;
    
    // 等待，当前线程在接到信号之前一直处于等待状态，不响应中断
    void awaitUninterruptibly();
    
    //等待，当前线程在接到信号、被中断或到达指定等待时间之前一直处于等待状态 
    long awaitNanos(long nanosTimeout) throws InterruptedException;
    
    // 等待，当前线程在接到信号、被中断或到达指定等待时间之前一直处于等待状态。此方法在行为上等效于: awaitNanos(unit.toNanos(time)) > 0
    boolean await(long time, TimeUnit unit) throws InterruptedException;
    
    // 等待，当前线程在接到信号、被中断或到达指定最后期限之前一直处于等待状态
    boolean awaitUntil(Date deadline) throws InterruptedException;
    
    // 唤醒一个等待线程。如果所有的线程都在等待此条件，则选择其中的一个唤醒。在从 await 返回之前，该线程必须重新获取锁。
    void signal();
    
    // 唤醒所有等待线程。如果所有的线程都在等待此条件，则唤醒所有线程。在从 await 返回之前，每个线程都必须重新获取锁。
    void signalAll();
}
```

Condition接口中定义了await、signal方法，用来等待条件、释放条件。之后会详细分析CondtionObject的源码。

#### 3.7.9.4 AQS类的属性

属性中包含了头结点head，尾结点tail，状态state、自旋时间spinForTimeoutThreshold，还有AbstractQueuedSynchronizer抽象的属性在内存中的偏移地址，通过该偏移地址，可以获取和设置该属性的值，同时还包括一个静态初始化块，用于加载内存偏移地址。

```java
public abstract class AbstractQueuedSynchronizer extends AbstractOwnableSynchronizer
    implements java.io.Serializable {    
    // 版本号
    private static final long serialVersionUID = 7373984972572414691L;    
    // 头结点
    private transient volatile Node head;    
    // 尾结点
    private transient volatile Node tail;    
    // 状态
    private volatile int state;    
    // 自旋时间
    static final long spinForTimeoutThreshold = 1000L;
    
    // Unsafe类实例
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    // state内存偏移地址
    private static final long stateOffset;
    // head内存偏移地址
    private static final long headOffset;
    // state内存偏移地址
    private static final long tailOffset;
    // tail内存偏移地址
    private static final long waitStatusOffset;
    // next内存偏移地址
    private static final long nextOffset;
    // 静态初始化块
    static {
        try {
            stateOffset = unsafe.objectFieldOffset
                (AbstractQueuedSynchronizer.class.getDeclaredField("state"));
            headOffset = unsafe.objectFieldOffset
                (AbstractQueuedSynchronizer.class.getDeclaredField("head"));
            tailOffset = unsafe.objectFieldOffset
                (AbstractQueuedSynchronizer.class.getDeclaredField("tail"));
            waitStatusOffset = unsafe.objectFieldOffset
                (Node.class.getDeclaredField("waitStatus"));
            nextOffset = unsafe.objectFieldOffset
                (Node.class.getDeclaredField("next"));

        } catch (Exception ex) { throw new Error(ex); }
    }
}
```

### 3.7.9.5 类的构造方法

```java
protected AbstractQueuedSynchronizer() { }    
```

#### 3.7.9.6 类的核心方法

##### 3.7.9.5.1 acquire方法

该方法以独占模式获取(资源)，忽略中断，即线程在aquire过程中，中断此线程是无效的。源码如下:

```java 
public final void acquire(int arg) {
    if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

![](.\resource\AQSacquire.png)

+ 首先调用tryAcquire方法,此线程会试图在独占模式下获取对象状态,返回true则获取执行权限,AQS中此方法默认抛出异常,需要在子类中重写此方法
+ 如果tryAcquire失败,则调用addWaiter方法,addWaiter方法会将线程封装成一个节点放入sync queue
+ 调用acquireQueued方法,此方法会通过parkAndCheckInterrupt方法将未获取锁的线程park

**addWaiter方法**

```java
// 添加等待者
private Node addWaiter(Node mode) {
    // 新生成一个结点，默认为独占模式
    Node node = new Node(Thread.currentThread(), mode);
    // Try the fast path of enq; backup to full enq on failure
    // 保存尾结点
    Node pred = tail;
    if (pred != null) { // 尾结点不为空，即已经被初始化
        // 将node结点的prev域连接到尾结点
        node.prev = pred; 
        if (compareAndSetTail(pred, node)) { // 比较pred是否为尾结点，是则将尾结点设置为node 
            // 设置尾结点的next域为node
            pred.next = node;
            return node; // 返回新生成的结点
        }
    }
    enq(node); // 尾结点为空(即还没有被初始化过)，或者是compareAndSetTail操作失败，则入队列
    return node;
}
```

addWaiter方法使用快速添加的方式往sync queue尾部添加结点，如果sync queue队列还没有初始化，则会使用enq插入队列中，enq方法源码如下

````java
private Node enq(final Node node) {
    for (;;) { // 无限循环，确保结点能够成功入队列
        // 保存尾结点
        Node t = tail;
        if (t == null) { // 尾结点为空，即还没被初始化
            if (compareAndSetHead(new Node())) // 头结点为空，并设置头结点为新生成的结点
                tail = head; // 头结点与尾结点都指向同一个新生结点
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
````

enq方法会使用无限循环来确保节点的成功插入。

**acquireQueue方法**

```java
// sync队列中的结点在独占且忽略中断的模式下获取(资源)
final boolean acquireQueued(final Node node, int arg) {
    // 标志
    boolean failed = true;
    try {
        // 中断标志
        boolean interrupted = false;
        for (;;) { // 无限循环
            // 获取node节点的前驱结点
            final Node p = node.predecessor(); 
            if (p == head && tryAcquire(arg)) { // 前驱为头结点并且成功获得锁
                setHead(node); // 设置头结点
                p.next = null; // help GC
                failed = false; // 设置标志
                return interrupted; 
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

首先找到当前线程的前节点,如果前节点是头节点并且当前线程可以获取到锁,将当前节点设置为头结点并且占有锁,执行业务代码,否则调用shouldParkAfterFailedAcquire和parkAndCheckInterrupt方法

**shouldParkAfterFailedAcquire方法**

```java
// 当获取(资源)失败后，检查并且更新结点状态
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    // 获取前驱结点的状态
    int ws = pred.waitStatus;
    if (ws == Node.SIGNAL) // 状态为SIGNAL，为-1
        /*
            * This node has already set status asking a release
            * to signal it, so it can safely park.
            */
        // 可以进行park操作
        return true; 
    if (ws > 0) { // 表示状态为CANCELLED，为1
        /*
            * Predecessor was cancelled. Skip over predecessors and
            * indicate retry.
            */
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0); // 找到pred结点前面最近的一个状态不为CANCELLED的结点
        // 赋值pred结点的next域
        pred.next = node; 
    } else { // 为PROPAGATE -3 或者是0 表示无状态,(为CONDITION -2时，表示此节点在condition queue中) 
        /*
            * waitStatus must be 0 or PROPAGATE.  Indicate that we
            * need a signal, but don't park yet.  Caller will need to
            * retry to make sure it cannot acquire before parking.
            */
        // 比较并设置前驱结点的状态为SIGNAL
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL); 
    }
    // 不能进行park操作
    return false;
}
```

**parkAndCheckInterrupt**

```java
// 进行park操作并且返回该线程是否被中断
private final boolean parkAndCheckInterrupt() {
    // 在许可可用之前禁用当前线程，并且设置了blocker
    LockSupport.park(this);
    return Thread.interrupted();  // 当前线程是否已被中断，并清除中断标记位
}
```

如果在acquireQueued方法中出现了异常(tryAcquire可以自定义实现),会调用cancelAcquire方法取消获取资源并且将当前节点设置为CANCELLED 代码如下:

```java
// 取消继续获取(资源)
private void cancelAcquire(Node node) {
    // Ignore if node doesn't exist
    // node为空，返回
    if (node == null)
        return;
    // 设置node结点的thread为空
    node.thread = null;

    // Skip cancelled predecessors
    // 保存node的前驱结点
    Node pred = node.prev;
    while (pred.waitStatus > 0) // 找到node前驱结点中第一个状态小于0的结点，即不为CANCELLED状态的结点
        node.prev = pred = pred.prev;

    // predNext is the apparent node to unsplice. CASes below will
    // fail if not, in which case, we lost race vs another cancel
    // or signal, so no further action is necessary.
    // 获取pred结点的下一个结点
    Node predNext = pred.next;

    // Can use unconditional write instead of CAS here.
    // After this atomic step, other Nodes can skip past us.
    // Before, we are free of interference from other threads.
    // 设置node结点的状态为CANCELLED
    node.waitStatus = Node.CANCELLED;

    // If we are the tail, remove ourselves.
    if (node == tail && compareAndSetTail(node, pred)) { // node结点为尾结点，则设置尾结点为pred结点
        // 比较并设置pred结点的next节点为null
        compareAndSetNext(pred, predNext, null); 
    } else { // node结点不为尾结点，或者比较设置不成功
        // If successor needs signal, try to set pred's next-link
        // so it will get one. Otherwise wake it up to propagate.
        int ws;
        if (pred != head &&
            ((ws = pred.waitStatus) == Node.SIGNAL ||
                (ws <= 0 && compareAndSetWaitStatus(pred, ws, Node.SIGNAL))) &&
            pred.thread != null) { // (pred结点不为头结点，并且pred结点的状态为SIGNAL)或者 
                                // pred结点状态小于等于0，并且比较并设置等待状态为SIGNAL成功，并且pred结点所封装的线程不为空
            // 保存结点的后继
            Node next = node.next;
            if (next != null && next.waitStatus <= 0) // 后继不为空并且后继的状态小于等于0
                compareAndSetNext(pred, predNext, next); // 比较并设置pred.next = next;
        } else {
            unparkSuccessor(node); // 释放node的前一个结点
        }

        node.next = node; // help GC
    }
}

```

**unparkSuccessor方法**

```java
// 释放后继结点
private void unparkSuccessor(Node node) {
    /*
        * If status is negative (i.e., possibly needing signal) try
        * to clear in anticipation of signalling.  It is OK if this
        * fails or if status is changed by waiting thread.
        */
    // 获取node结点的等待状态
    int ws = node.waitStatus;
    if (ws < 0) // 状态值小于0，为SIGNAL -1 或 CONDITION -2 或 PROPAGATE -3
        // 比较并且设置结点等待状态，设置为0
        compareAndSetWaitStatus(node, ws, 0);

    /*
        * Thread to unpark is held in successor, which is normally
        * just the next node.  But if cancelled or apparently null,
        * traverse backwards from tail to find the actual
        * non-cancelled successor.
        */
    // 获取node节点的下一个结点
    Node s = node.next;
    if (s == null || s.waitStatus > 0) { // 下一个结点为空或者下一个节点的等待状态大于0，即为CANCELLED
        // s赋值为空
        s = null; 
        // 从尾结点开始从后往前开始遍历
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0) // 找到等待状态小于等于0的结点，找到最前的状态小于等于0的结点
                // 保存结点
                s = t;
    }
    if (s != null) // 该结点不为为空，释放许可
        LockSupport.unpark(s.thread);
}
```

该方法会释放(unpark)当前节点的后继节点

![](.\resource\AQSUnparkSuccessor.png)

**acquireQueue的整体逻辑**

1. 首先判断当前节点的前节点是否是头节点,如果是尝试获取锁
2. 如果1步骤满足,则将当前节点设置为头节点,方法返回执行业务代码
3. 如果1步骤不满足,调用shouldParkAfterFailedAcquire将前节点waitStatus改为signal ,并且将当前线程阻塞(如果前节点本身不是signal,第一次循环该为signal,第二次循环当前线程阻塞)
4. 当前线程阻塞之后,需求前面的节点对当前线程unpark(前面的节点调用unparkSuccessor方法)才能继续执行

#### 3.7.9.5.2 release方法

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) { // 释放成功
        // 保存头结点
        Node h = head; 
        if (h != null && h.waitStatus != 0) // 头结点不为空并且头结点状态不为0
            unparkSuccessor(h); //释放头结点的后继结点
        return true;
    }
    return false;
}
```

tryRelease方法会将state减去相应的值来释放锁,如果释放之后state==0则返回true,

unparkSuccessor会释放参数节点后面的节点

#### 3.7.9.6  AbstractQueuedSynchronizer示例详解一

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

class MyThread extends Thread {
    private Lock lock;
    public MyThread(String name, Lock lock) {
        super(name);
        this.lock = lock;
    }
    
    public void run () {
        lock.lock();
        try {
            System.out.println(Thread.currentThread() + " running");
        } finally {
            lock.unlock();
        }
    }
}
public class AbstractQueuedSynchonizerDemo {
    public static void main(String[] args) {
        Lock lock = new ReentrantLock();
        
        MyThread t1 = new MyThread("t1", lock);
        MyThread t2 = new MyThread("t2", lock);
        t1.start();
        t2.start();    
    }
}
//结果
Thread[t1,5,main] running
Thread[t2,5,main] running
```

t1线程lock调用如下

![](.\resource\t1ThreadCallMethod.png)

t2线程lock调用如下

![](.\resource\t2ThreadMethodCall.png)

t1线程unlock如下

![](.\resource\t1ThreadUnlockCallMethod.png)

t2线程在t1线程unlock后获取锁

![](.\resource\t2ThreadGetlockCallMethod.png)

t2线程unlock如下

![](.\resource\t2ThreadUnlockCallMethod.png)

#### 3.7.9.7  AbstractQueuedSynchronizer示例详解二

```java
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Depot {
    private int size;
    private int capacity;
    private Lock lock;
    private Condition fullCondition;
    private Condition emptyCondition;
    
    public Depot(int capacity) {
        this.capacity = capacity;    
        lock = new ReentrantLock();
        fullCondition = lock.newCondition();
        emptyCondition = lock.newCondition();
    }
    
    public void produce(int no) {
        lock.lock();
        int left = no;
        try {
            while (left > 0) {
                while (size >= capacity)  {
                    System.out.println(Thread.currentThread() + " before await");
                    fullCondition.await();
                    System.out.println(Thread.currentThread() + " after await");
                }
                int inc = (left + size) > capacity ? (capacity - size) : left;
                left -= inc;
                size += inc;
                System.out.println("produce = " + inc + ", size = " + size);
                emptyCondition.signal();
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
    
    public void consume(int no) {
        lock.lock();
        int left = no;
        try {            
            while (left > 0) {
                while (size <= 0) {
                    System.out.println(Thread.currentThread() + " before await");
                    emptyCondition.await();
                    System.out.println(Thread.currentThread() + " after await");
                }
                int dec = (size - left) > 0 ? left : size;
                left -= dec;
                size -= dec;
                System.out.println("consume = " + dec + ", size = " + size);
                fullCondition.signal();
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```

```java
class Consumer {
    private Depot depot;
    public Consumer(Depot depot) {
        this.depot = depot;
    }
    
    public void consume(int no) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                depot.consume(no);
            }
        }, no + " consume thread").start();
    }
}

class Producer {
    private Depot depot;
    public Producer(Depot depot) {
        this.depot = depot;
    }
    
    public void produce(int no) {
        new Thread(new Runnable() {
            
            @Override
            public void run() {
                depot.produce(no);
            }
        }, no + " produce thread").start();
    }
}

public class ReentrantLockDemo {
    public static void main(String[] args) throws InterruptedException {
        Depot depot = new Depot(500);
        new Producer(depot).produce(500);
        new Producer(depot).produce(200);
        new Consumer(depot).consume(500);
        new Consumer(depot).consume(200);
    }
}
```

```java
produce = 500, size = 500
Thread[200 produce thread,5,main] before await
consume = 500, size = 0
Thread[200 consume thread,5,main] before await
Thread[200 produce thread,5,main] after await
produce = 200, size = 200
Thread[200 consume thread,5,main] after await
consume = 200, size = 0
```

一种可能的情况![](\resource\AQSProductConsumer.png)

p1调用lock.lock后直接执行代码,p1执行single无效(condition中没有节点)

p2调用lock.lock后阻塞

![](.\resource\p2lock.png)

类推得到c1,c2如下

![](.\resource\c2lock.png)

p1执行unlock之后

![](.\resource\p1unlock.png)

p1执行wait方法

![](.\resource\p1wait.png)

说明: 最终到达的状态是新生成了一个结点，包含了p2线程，此结点在condition queue中；并且sync queue中p2线程被禁止了，因为在执行了LockSupport.park操作。从方法一些调用可知，在await操作中线程会释放锁资源，供其他线程获取。同时，head结点后继结点的包含的线程的许可被释放了，故其可以继续运行。由于此时，只有c1线程可以运行，故运行c1。

c1线程acquireQueue中tryAcquire成功获取执行权限

![](.\resource\c1run.png)

c1线程执行signal方法

![](.\resource\c1signal.png)

说明: signal方法达到的最终结果是将包含p2线程的结点从condition queue中转移到sync queue中，之后condition queue为null，之前的尾结点的状态变为SIGNAL。

c1执行unlock,c2获取执行权限

![](.\resource\c1PreUnlock.png)

c2await

![](.\resource\c2preAwait.png)

c2 await时会唤醒后面的线程p2

![](.\resource\c2await.png)

p2 signal

![](resource\p2signal.png)

p2 unlock

![](resource\p2unlock.png)

c2继续执行 signal无效后执行unlock 结束整个流程

#### 3.7.9.8 AQS总结

每一个结点都是由前一个结点唤醒

当结点发现前驱结点是head并且尝试获取成功，则会轮到该线程运行。

condition queue中的结点向sync queue中转移是通过signal操作完成的。

当结点的状态为SIGNAL时，表示后面的结点需要运行。

