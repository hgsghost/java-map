# 第三章 并发框架(三) CAS,UNSAFEL,原子类

## 3.7  CAS,Unsafe,原子类

juc中多数类是使用cas实现的,cas是一种乐观锁(非互斥),synchronized和lock是悲观锁(互斥),cas的底层是通过unsafe实现的

### 3.7.1 面试题

1. 线程安全的实现方法有哪些
2. 什么是cas
3. cas使用示例,结合AtomicInteger
4. cas的问题和解决方案
5. AtomicInteger底层实现
6. 对unsafe的理解
7. 对原子类的理解?13个类分为4组
8. AtomicStampedReference是什么,它如何解决ABA问题
9. java中还有哪些类可以解决ABA问题

### 3.7.2 线程安全的实现方法有哪些

+ 互斥同步 synchronized/lock
+ 非阻塞 cas
+ 无同步 栈封闭,Threadlocal,可重入代码

### 3.7.3 什么是cas

compare-and-swap 比较交换,是cpu层面支持的原子指令,类比update a set id=3 where id=2

### 3.7.4 cas使用示例,结合AtomicInteger

```java
public class Test {
    private int i=0;
    public synchronized int add(){
        return i++;
    }
}
//------------------替换为
public class Test {
    private  AtomicInteger i = new AtomicInteger(0);
    public int add(){
        return i.addAndGet(1);
    }
}
```

### 3.7.5 cas的问题和解决方案

1. ABA问题

   解决思路是版本号 由a->b->a 变为 1a->2b->3a

2. 循环时间长开销大

   如果JVM能支持处理器提供的pause指令，那么效率会有一定的提升。pause指令有两个作用：第一，它可以延迟流水线执行命令(de-pipeline)，使CPU不会消耗过多的执行资源，延迟的时间取决于具体实现的版本，在一些处理器上延迟时间是零；第二，它可以避免在退出循环的时候因内存顺序冲突(Memory Order Violation)而引起CPU流水线被清空(CPU Pipeline Flush)，从而提高CPU的执行效率。

3. 只能保证一个变量的原子操作

   从Java 1.5开始，JDK提供了AtomicReference类来保证引用对象之间的原子性，就可以把多个变量放在一个对象里来进行CAS操作。

### 3.7.6 AtomicInteger底层实现

其实就是通过volatile+cas实现线程安全

+ volatile保证线程间可见
+ cas保证操作原子性

### 3.7.7 unsafe的理解

Unsafe是位于sun.misc包下的一个类，主要提供一些用于执行低级别、不安全操作的方法，如直接访问系统内存资源、自主管理内存资源等，这些方法在提升Java运行效率、增强Java语言底层资源操作能力方面起到了很大的作用。但由于Unsafe类使Java语言拥有了类似C语言指针一样操作内存空间的能力，这无疑也增加了程序发生相关指针问题的风险。在程序中过度、不正确使用Unsafe类会使得程序出错的概率变大，使得Java这种安全的语言变得不再“安全”，因此对Unsafe的使用一定要慎重。

![](.\resource\Unsafe.png)

### 3.7.8 对原子类的理解

1. 基本类型:AtomicBoolean,AtomicInteger,AtomicLong

   提供线程安全版本的基本类型

2. 数组:AtomicIntegerArray,AtomicLongArray,AtomicReferenceArray

   线程安全的修改数组中的对象

   ```java
   public class Demo5 {
       public static void main(String[] args) throws InterruptedException {
           AtomicIntegerArray array = new AtomicIntegerArray(new int[] { 0, 0 });
           System.out.println(array);
           System.out.println(array.getAndAdd(1, 2));
           System.out.println(array);
       }
   }
   ```

   

3. 引用类型:

   + AtomicReference:原子更新引用类型
   + AtomicStampedReference:原子更新引用类型, 内部使用Pair来存储元素值及其版本号。
   + AtomicMarkableReferce:原子更新带有标记位的引用类型。

   ```java
   public class AtomicReferenceTest {
       public static void main(String[] args){
           // 创建两个Person对象，它们的id分别是101和102。
           Person p1 = new Person(101);
           Person p2 = new Person(102);
           // 新建AtomicReference对象，初始化它的值为p1对象
           AtomicReference ar = new AtomicReference(p1);
           // 通过CAS设置ar。如果ar的值为p1的话，则将其设置为p2。
           ar.compareAndSet(p1, p2);
           Person p3 = (Person)ar.get();
           System.out.println("p3 is "+p3);
           System.out.println("p3.equals(p1)="+p3.equals(p1));
       }
   }
   class Person {
       volatile long id;
       public Person(long id) {
           this.id = id;
       }
       public String toString() {
           return "id:"+id;
       }
   }
   //输出
   p3 is id:102
   p3.equals(p1)=false
   ```

4. 原子更新字段

   + AtomicIntegerFieldUpdater: 原子更新整型的字段的更新器。

   + AtomicLongFieldUpdater: 原子更新长整型字段的更新器。

   + AtomicStampedFieldUpdater: 原子更新带有版本号的引用类型。

   + AtomicReferenceFieldUpdater: 原子更新引用类型

     ```java
     public class TestAtomicIntegerFieldUpdater {
         public static void main(String[] args){
             TestAtomicIntegerFieldUpdater tIA = new TestAtomicIntegerFieldUpdater();
             tIA.doIt();
         }
         public AtomicIntegerFieldUpdater<DataDemo> updater(String name){
             return AtomicIntegerFieldUpdater.newUpdater(DataDemo.class,name);
         }
         public void doIt(){
             DataDemo data = new DataDemo();
             System.out.println("publicVar = "+updater("publicVar").getAndAdd(data, 2));
             /*
                 * 由于在DataDemo类中属性value2/value3,在TestAtomicIntegerFieldUpdater中不能访问 IllegalAccessException
                 * */
             //System.out.println("protectedVar = "+updater("protectedVar").getAndAdd(data,2));
             //System.out.println("privateVar = "+updater("privateVar").getAndAdd(data,2));
     
             //System.out.println("staticVar = "+updater("staticVar").getAndIncrement(data));//报java.lang.IllegalArgumentException
             /*
                 * 下面报异常：must be integer
                 * */
             //System.out.println("integerVar = "+updater("integerVar").getAndIncrement(data));
             //System.out.println("longVar = "+updater("longVar").getAndIncrement(data));
         }
     }
     class DataDemo{
         public volatile int publicVar=3;
         protected volatile int protectedVar=4;
         private volatile  int privateVar=5;
         public volatile static int staticVar = 10;
         //public  final int finalVar = 11;
         public volatile Integer integerVar = 19;
         public volatile Long longVar = 18L;
     
     ```

     + 字段必须是volatile类型的，在线程之间共享变量时保证立即可见.eg:volatile int value = 3

     + 字段的描述类型(修饰符public/protected/default/private)是与调用者与操作对象字段的关系一致。也就是说调用者能够直接操作对象字段，那么就可以反射进行原子操作。但是对于父类的字段，子类是不能直接操作的，尽管子类可以访问父类的字段。

     + 只能是实例变量，不能是类变量，也就是说不能加static关键字。

     + 只能是可修改变量，不能使final变量，因为final的语义就是不可修改。实际上final的语义和volatile是有冲突的，这两个关键字不能同时存在。

     + 对于AtomicIntegerFieldUpdater和AtomicLongFieldUpdater只能修改int/long类型的字段，不能修改其包装类型(Integer/Long)。如果要修改包装类型就需要使用AtomicReferenceFieldUpdater。

### 3.7.8  AtomicStampedReference是什么,如何解决ABA问题

AtomicStampedReference主要维护包含一个对象引用以及一个可以自动更新的整数"stamp"的pair对象来解决ABA问题。

[详情可见](https://pdai.tech/md/java/thread/java-thread-x-juc-AtomicInteger.html#juc%e5%8e%9f%e5%ad%90%e7%b1%bb-cas-unsafe%e5%92%8c%e5%8e%9f%e5%ad%90%e7%b1%bb%e8%af%a6%e8%a7%a3)

### 3.7.9 java中可以解决ABA问题的其他类

AtomicMarkableReference，它不是维护一个版本号，而是维护一个boolean类型的标记，标记值有修改，了解一下。