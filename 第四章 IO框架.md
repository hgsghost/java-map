# 第四章 I/O框架

## 1. 基础io

### 1.1 io类型

#### 1.1.1 从传输方式上分为字节流和字符流

![](resource\IoByteStream.png)

![](resource\IoCharStream.png)

**字节流和字符流的区别**

字节流读取单个字节，字符流读取单个字符(一个字符根据编码的不同，对应的字节也不同，如 UTF-8 编码是 3 个字节，中文编码是 2 个字节。)

字节流用来处理二进制文件(图片、MP3、视频文件)，字符流用来处理文本文件(可以看做是特殊的二进制文件，使用了某种编码，人可以阅读)。

#### 1.1.2 字符转字节

Input/OutputStreamReader/Writer

编码就是把字符转换为字节，而解码是把字节重新组合成字符。

如果编码和解码过程使用不同的编码方式那么就出现了乱码。

- GBK 编码中，中文字符占 2 个字节，英文字符占 1 个字节；
- UTF-8 编码中，中文字符占 3 个字节，英文字符占 1 个字节；
- UTF-16be 编码中，中文字符和英文字符都占 2 个字节。

![](resource\IoDecode.png)

#### 1.1.3 从数据操作上分

![](resource\IoDataOperate.png)

## 1.2 io的设计模式

装饰者(Decorator)和具体组件(ConcreteComponent)都继承自组件(Component)，具体组件的方法实现不需要依赖于其它对象，而装饰者组合了一个组件，这样它可以装饰其它装饰者或者具体组件。所谓装饰，就是把这个装饰者套在被装饰者之上，从而动态扩展被装饰者的功能。装饰者的方法有一部分是自己的，这属于它的功能，然后调用被装饰者的方法实现，从而也保留了被装饰者的功能。可以看到，具体组件应当是装饰层次的最低层，因为只有具体组件的方法实现不需要依赖于其它对象。

![](resource\ioDecorate.png)

以 InputStream 为例，

- InputStream 是抽象组件；
- FileInputStream 是 InputStream 的子类，属于具体组件，提供了字节流的输入操作；
- FilterInputStream 属于抽象装饰者，装饰者用于装饰组件，为组件提供额外的功能。例如 BufferedInputStream 为 FileInputStream 提供缓存的功能。

![](resource\IoDecoratorInputStream.png)

```java
//实例化一个具有缓存功能的字节流对象时，只需要在 FileInputStream 对象上再套一层 BufferedInputStream 对象即可。
FileInputStream fileInputStream = new FileInputStream(filePath);
BufferedInputStream bufferedInputStream = new BufferedInputStream(fileInputStream);
```

## 1.3 常用io类

有以下几类

- 磁盘操作: File
- 字节操作: InputStream 和 OutputStream
- 字符操作: Reader 和 Writer
- 对象操作: Serializable
- 网络操作: Socket

相关代码如下

```java
//递归遍历目录文件
public static void listAllFiles(File dir) {
    if (dir == null || !dir.exists()) {
        return;
    }
    if (dir.isFile()) {
        System.out.println(dir.getName());
        return;
    }
    for (File file : dir.listFiles()) {
        listAllFiles(file);
    }
}
//字节操作
public static void copyFile(String src, String dist) throws IOException {
    FileInputStream in = new FileInputStream(src);
    FileOutputStream out = new FileOutputStream(dist);
    byte[] buffer = new byte[20 * 1024];
    // read() 最多读取 buffer.length 个字节
    // 返回的是实际读取的个数
    // 返回 -1 的时候表示读到 eof，即文件尾
    while (in.read(buffer, 0, buffer.length) != -1) {
        out.write(buffer);
    }
    in.close();
    out.close();
}
//字符操作
public static void readFileContent(String filePath) throws IOException {
    FileReader fileReader = new FileReader(filePath);
    BufferedReader bufferedReader = new BufferedReader(fileReader);
    String line;
    while ((line = bufferedReader.readLine()) != null) {
        System.out.println(line);
    }
    // 装饰者模式使得 BufferedReader 组合了一个 Reader 对象
    // 在调用 BufferedReader 的 close() 方法时会去调用 Reader 的 close() 方法
    // 因此只要一个 close() 调用即可
    bufferedReader.close();
}
//对象的序列化操作
/*
序列化就是将一个对象转换成字节序列，方便存储和传输。 序列化: ObjectOutputStream.writeObject() 反序列化: ObjectInputStream.readObject() 不会对静态变量进行序列化，因为序列化只是保存对象的状态，静态变量属于类的状态。
*/
//序列化的类需要实现 Serializable 接口，它只是一个标准，没有任何方法需要实现，但是如果不去实现它的话而进行序列化，会抛出异常。
//transient 关键字可以使一些属性不会被序列化。
public static void main(String[] args) throws IOException, ClassNotFoundException {
    A a1 = new A(123, "abc");
    String objectFile = "file/a1";
    ObjectOutputStream objectOutputStream = new ObjectOutputStream(new FileOutputStream(objectFile));
    objectOutputStream.writeObject(a1);
    objectOutputStream.close();
    
    ObjectInputStream objectInputStream = new ObjectInputStream(new FileInputStream(objectFile));
    A a2 = (A) objectInputStream.readObject();
    objectInputStream.close();
    System.out.println(a2);
}
private static class A implements Serializable {
    private int x;
    private String y;
    A(int x, String y) {
        this.x = x;
        this.y = y;
    }
    @Override
    public String toString() {
        return "x = " + x + "  " + "y = " + y;
    }
}
//网络操作
InetAddress.getByName(String host);
InetAddress.getByAddress(byte[] address);

public static void main(String[] args) throws IOException {
    URL url = new URL("http://www.baidu.com");
    /* 字节流 */
    InputStream is = url.openStream();
    /* 字符流 */
    InputStreamReader isr = new InputStreamReader(is, "utf-8");
    /* 提供缓存功能 */
    BufferedReader br = new BufferedReader(isr);
    String line;
    while ((line = br.readLine()) != null) {
        System.out.println(line);
    }
    br.close();
}

```

## 1.4 unix io模型

一个输入操作通常包括两个阶段:

- 等待数据准备好
- 从内核向进程复制数据

对于一个套接字上的输入操作，第一步通常涉及等待数据从网络中到达。当所等待分组到达时，它被复制到内核中的某个缓冲区。第二步就是把数据从内核缓冲区复制到应用进程缓冲区。

Unix 下有五种 I/O 模型:

- 阻塞式 I/O
- 非阻塞式 I/O
- I/O 复用(select 和 poll)
- 信号驱动式 I/O(SIGIO)
- 异步 I/O(AIO)

1. 阻塞式i/o

   应用进程被阻塞，直到数据复制到应用进程缓冲区中才返回。

   应该注意到，在阻塞的过程中，其它程序还可以执行，因此阻塞不意味着整个操作系统都被阻塞。因为其他程序还可以执行，因此不消耗 CPU 时间，这种模型的执行效率会比较高。

   ![](resource\Uninxbio.png)

   ![](resource\unixbiocn.png)

   

2. 非阻塞式 I/O

   应用进程执行系统调用之后，内核返回一个错误码。应用进程可以继续执行，但是需要不断的执行系统调用来获知 I/O 是否完成，这种方式称为轮询(polling)。

   由于 CPU 要处理更多的系统调用，因此这种模型是比较低效的。

   ![](resource\unixnio.png)

   ![](resource\unixniocn.png)

3. I/O 复用(select 和 poll)

   使用 select 或者 poll 等待数据，并且可以等待多个套接字中的任何一个变为可读，这一过程会被阻塞，当某一个套接字可读时返回。之后再使用 recvfrom 把数据从内核复制到进程中。

   它可以让单个进程具有处理多个 I/O 事件的能力。又被称为 Event Driven I/O，即事件驱动 I/O。

   如果一个 Web 服务器没有 I/O 复用，那么每一个 Socket 连接都需要创建一个线程去处理。如果同时有几万个连接，那么就需要创建相同数量的线程。并且相比于多进程和多线程技术，I/O 复用不需要进程线程创建和切换的开销，系统开销更小。

   ![](resource\unixEventDrivenIo.png)

   ![](resource\unixEventDrivenIoCN.JPG)

   

4. 信号驱动IO

   应用进程使用 sigaction 系统调用，内核立即返回，应用进程可以继续执行，也就是说等待数据阶段应用进程是非阻塞的。内核在数据到达时向应用进程发送 SIGIO 信号，应用进程收到之后在信号处理程序中调用 recvfrom 将数据从内核复制到应用进程中。

   相比于非阻塞式 I/O 的轮询方式，信号驱动 I/O 的 CPU 利用率更高。

   ![](resource\unixSigactionIO.png)

   ![](resource\unixSigactionIOCN.png)

   

5. 异步IO

   进行 aio_read 系统调用会立即返回，应用进程继续执行，不会被阻塞，内核会在所有操作完成之后向应用进程发送信号。

   异步 I/O 与信号驱动 I/O 的区别在于，异步 I/O 的信号是通知应用进程 I/O 完成，而信号驱动 I/O 的信号是通知应用进程可以开始 I/O。

   ![](resource\unixAsynIo.png)

   

![](resource\unixAsynIoCn.png)

**io模型比较**

- 同步 I/O: 应用进程在调用 recvfrom 操作时会阻塞。
- 异步 I/O: 不会阻塞。

阻塞式 I/O、非阻塞式 I/O、I/O 复用和信号驱动 I/O 都是同步 I/O，虽然非阻塞式 I/O 和信号驱动 I/O 在等待数据阶段不会阻塞，但是在之后的将数据从内核复制到应用进程这个操作会阻塞。

![](resource\unixIoCompare.png)

**注 IO多路复用是重点**

## 1.5 传统BIO详解

BIO就是: blocking IO。最容易理解、最容易实现的IO工作方式，应用程序向操作系统请求网络IO操作，这时应用程序会一直等待；另一方面，操作系统收到请求后，也会等待，直到网络上有数据传到监听端口；操作系统在收集数据后，会把数据发送给应用程序；最后应用程序受到数据，并解除等待状态。

### 1.5.1 重要概念

**阻塞IO**和**非阻塞IO**

这两个概念是`程序级别`的。主要描述的是程序请求操作系统IO操作后，如果IO资源没有准备好，那么程序该如何处理的问题: 前者等待；后者继续执行(并且使用线程一直轮询，直到有IO资源准备好了)

**同步IO**和**非同步IO**

这两个概念是`操作系统级别`的。主要描述的是操作系统在收到程序请求IO操作后，如果IO资源没有准备好，该如何响应程序的问题: 前者不响应，直到IO资源准备好以后；后者返回一个标记(好让程序和自己知道以后的数据往哪里通知)，当IO资源准备好以后，再用事件机制返回给程序。

### 1.5.2 传统BIO通信方式

以前大多数网络通信方式都是阻塞模式的，即:

- 客户端向服务器端发出请求后，客户端会一直等待(不会再做其他事情)，直到服务器端返回结果或者网络出现问题。
- 服务器端同样的，当在处理某个客户端A发来的请求时，另一个客户端B发来的请求会等待，直到服务器端的这个处理线程完成上一个处理。

![](resource\BioTraditional.png)

问题:

+ 同一时间，服务器只能接受来自于客户端A的请求信息；虽然客户端A和客户端B的请求是同时进行的，但客户端B发送的请求信息只能等到服务器接受完A的请求数据后，才能被接受。

+ 由于服务器一次只能处理一个客户端请求，当处理完成并返回后(或者异常时)，才能进行第二次请求的处理。很显然，这样的处理方式在高并发的情况下，是不能采用的

### 1.5.3 多线程方式BIO

上面说的情况是服务器只有一个线程的情况，那么读者会直接提出我们可以使用多线程技术来解决这个问题:

- 当服务器收到客户端X的请求后，(读取到所有请求数据后)将这个请求送入一个独立线程进行处理，然后主线程继续接受客户端Y的请求。
- 客户端一侧，也可以使用一个子线程和服务器端进行通信。这样客户端主线程的其他工作就不受影响了，当服务器端有响应信息的时候再由这个子线程通过 监听模式/观察模式(等其他设计模式)通知主线程。

![](resource\BioTraditionalMulityThread.png)

局限性:

+ 虽然在服务器端，请求的处理交给了一个独立线程进行，但是操作系统通知accept()的方式还是单个的。也就是，实际上是服务器接收到数据报文后的“业务处理过程”可以多线程，但是数据报文的接受还是需要一个一个的来(下文的示例代码和debug过程我们可以明确看到这一点)

+ 在linux系统中，可以创建的线程是有限的。我们可以通过cat /proc/sys/kernel/threads-max 命令查看可以创建的最大线程数。当然这个值是可以更改的，但是线程越多，CPU切换所需的时间也就越长，用来处理真正业务的需求也就越少。

+ 创建一个线程是有较大的资源消耗的。JVM创建一个线程的时候，即使这个线程不做任何的工作，JVM都会分配一个堆栈空间。这个空间的大小默认为128K，您可以通过-Xss参数进行调整。当然您还可以使用ThreadPoolExecutor线程池来缓解线程的创建问题，但是又会造成BlockingQueue积压任务的持续增加，同样消耗了大量资源。

+ 另外，如果您的应用程序大量使用长连接的话，线程是不会关闭的。这样系统资源的消耗更容易失控。 那么，如果你真想单纯使用线程解决阻塞的问题，那么您自己都可以算出来您一个服务器节点可以一次接受多大的并发了。看来，单纯使用线程解决这个问题不是最好的办法。

### 1.5.4 BIO通信方式问题

BIO的问题关键不在于是否使用了多线程(包括线程池)处理这次请求，而在于accept()、read()的操作点都是被阻塞。造成io过程变为串行操作效率低下

那么重点的问题并不是“是否使用了多线程”，而是为什么accept()、read()方法会被阻塞。即: 异步IO模式 就是为了解决这样的并发性存在的。但是为了说清楚异步IO模式，在介绍IO模式的时候，我们就要首先了解清楚，什么是 阻塞式同步、非阻塞式同步、多路复用同步模式。

API文档中对于 serverSocket.accept() 方法的使用描述:

> Listens for a connection to be made to this socket and accepts it. The method blocks until a connection is made.

阻塞式同步IO的工作原理:

- 服务器线程发起一个accept动作，询问操作系统 是否有新的socket套接字信息从端口X发送过来。

+ 注意，是询问操作系统。也就是说socket套接字的IO模式支持是基于操作系统的，那么自然同步IO/异步IO的支持就是需要操作系统级别的了。

如果操作系统没有发现有套接字从指定的端口X来，那么操作系统就会等待。这样serverSocket.accept()方法就会一直等待。这就是为什么accept()方法为什么会阻塞: 它内部的实现是使用的操作系统级别的同步IO。

## 1.6 NIO基础

Standard IO是对字节流的读写，在进行IO之前，首先创建一个流对象，流对象进行读写操作都是按字节 ，一个字节一个字节的来读或写。而NIO把IO抽象成块，类似磁盘的读写，每次IO操作的单位都是一个块，块被读入内存之后就是一个byte[]，NIO一次可以读或写多个字节。

+ **流与块**

  I/O 与 NIO 最重要的区别是数据打包和传输的方式，I/O 以流的方式处理数据，而 NIO 以块的方式处理数据。

  面向流的 I/O 一次处理一个字节数据: 一个输入流产生一个字节数据，一个输出流消费一个字节数据。为流式数据创建过滤器非常容易，链接几个过滤器，以便每个过滤器只负责复杂处理机制的一部分。不利的一面是，面向流的 I/O 通常相当慢。

  面向块的 I/O 一次处理一个数据块，按块处理数据比按流处理数据要快得多。但是面向块的 I/O 缺少一些面向流的 I/O 所具有的优雅性和简单性。

  I/O 包和 NIO 已经很好地集成了，java.io.* 已经以 NIO 为基础重新实现了，所以现在它可以利用 NIO 的一些特性。例如，java.io.* 包中的一些类包含以块的形式读写数据的方法，这使得即使在面向流的系统中，处理速度也会更快

+ **通道和缓冲区**

  1. 通道

     通道 Channel 是对原 I/O 包中的流的模拟，可以通过它读取和写入数据。

     通道与流的不同之处在于，流只能在一个方向上移动(一个流必须是 InputStream 或者 OutputStream 的子类)，而通道是双向的，可以用于读、写或者同时用于读写。

     通道包括以下类型:

     + FileChannel: 从文件中读写数据；
     + DatagramChannel: 通过 UDP 读写网络中数据；
     + SocketChannel: 通过 TCP 读写网络中数据；
     + ServerSocketChannel: 可以监听新进来的 TCP 连接，对每一个新进来的连接都会创建一个 SocketChannel。

  2. 缓冲区

     发送给一个通道的所有数据都必须首先放到缓冲区中，同样地，从通道中读取的任何数据都要先读到缓冲区中。也就是说，不会直接对通道进行读写数据，而是要先经过缓冲区。

     缓冲区实质上是一个数组，但它不仅仅是一个数组。缓冲区提供了对数据的结构化访问，而且还可以跟踪系统的读/写进程。

     缓冲区包括以下类型:

     - ByteBuffer
     - CharBuffer
     - ShortBuffer
     - IntBuffer
     - LongBuffer
     - FloatBuffer
     - DoubleBuffer

     **缓冲区状态变量**

     - capacity: 最大容量；
     - position: 当前已经读写的字节数；
     - limit: 还可以读写的字节数。

     状态变量的改变过程举例:

     ① 新建一个大小为 8 个字节的缓冲区，此时 position 为 0，而 limit = capacity = 8。capacity 变量不会改变，下面的讨论会忽略它。

     ![](resource\NioCacheStatus1.png)

     ② 从输入通道中读取 5 个字节数据写入缓冲区中，此时 position 移动设置为 5，limit 保持不变

     ![](resource\NioCacheStatus2.png)

     ③ 在将缓冲区的数据写到输出通道之前，需要先调用 flip() 方法，这个方法将 limit 设置为当前 position，并将 position 设置为 0。

     ![](resource\NioCacheStatus3.png)

     ④ 从缓冲区中取 4 个字节到输出缓冲中，此时 position 设为 4。

     ![](resource\NioCacheStatus4.png)

     ⑤ 最后需要调用 clear() 方法来清空缓冲区，此时 position 和 limit 都被设置为最初位置。

     ![](resource\NioCacheStatus5.png)

     **NIO复制文件示例**

     ```java
     public static void fastCopy(String src, String dist) throws IOException {
     
         /* 获得源文件的输入字节流 */
         FileInputStream fin = new FileInputStream(src);
     
         /* 获取输入字节流的文件通道 */
         FileChannel fcin = fin.getChannel();
     
         /* 获取目标文件的输出字节流 */
         FileOutputStream fout = new FileOutputStream(dist);
     
         /* 获取输出字节流的通道 */
         FileChannel fcout = fout.getChannel();
     
         /* 为缓冲区分配 1024 个字节 */
         ByteBuffer buffer = ByteBuffer.allocateDirect(1024);
     
         while (true) {
     
             /* 从输入通道中读取数据到缓冲区中 */
             int r = fcin.read(buffer);
     
             /* read() 返回 -1 表示 EOF */
             if (r == -1) {
                 break;
             }
     
             /* 切换读写 */
             buffer.flip();
     
             /* 把缓冲区的内容写入输出文件中 */
             fcout.write(buffer);
             
             /* 清空缓冲区 */
             buffer.clear();
         }
     }
     ```

     

+ **选择器**

  NIO 常常被叫做非阻塞 IO，主要是因为 NIO 在网络通信中的非阻塞特性被广泛使用。

  NIO 实现了 IO 多路复用中的 Reactor 模型，一个线程 Thread 使用一个选择器 Selector 通过轮询的方式去监听多个通道 Channel 上的事件，从而让一个线程就可以处理多个事件。

  通过配置监听的通道 Channel 为非阻塞，那么当 Channel 上的 IO 事件还未到达时，就不会进入阻塞状态一直等待，而是继续轮询其它 Channel，找到 IO 事件已经到达的 Channel 执行。

  因为创建和切换线程的开销很大，因此使用一个线程来处理多个事件而不是一个线程处理一个事件具有更好的性能。

  应该注意的是，只有套接字 Channel 才能配置为非阻塞，而 FileChannel 不能，为 FileChannel 配置非阻塞也没有意义。

  ![](resource\NioSelector.png)

  ```java
  //创建选择器
  Selector selector = Selector.open();
  //通道注册选择器
  ServerSocketChannel ssChannel = ServerSocketChannel.open();
  ssChannel.configureBlocking(false);
  ssChannel.register(selector, SelectionKey.OP_ACCEPT);
  //监听事件
  //使用 select() 来监听到达的事件，它会一直阻塞直到有至少一个事件到达。
  int num = selector.select();
  //获取到达的事件
  Set<SelectionKey> keys = selector.selectedKeys();
  Iterator<SelectionKey> keyIterator = keys.iterator();
  while (keyIterator.hasNext()) {
      SelectionKey key = keyIterator.next();
      if (key.isAcceptable()) {
          // ...
      } else if (key.isReadable()) {
          // ...
      }
      keyIterator.remove();
  }
  //事件循环
  //因为一次 select() 调用不能处理完所有的事件，并且服务器端有可能需要一直监听事件，因此服务器端处理事件的代码一般会放在一个死循环内。
  while (true) {
      int num = selector.select();
      Set<SelectionKey> keys = selector.selectedKeys();
      Iterator<SelectionKey> keyIterator = keys.iterator();
      while (keyIterator.hasNext()) {
          SelectionKey key = keyIterator.next();
          if (key.isAcceptable()) {
              // ...
          } else if (key.isReadable()) {
              // ...
          }
          keyIterator.remove();
      }
  }
  
  ```

  通道必须配置为非阻塞模式，否则使用选择器就没有任何意义了，因为如果通道在某个事件上被阻塞，那么服务器就不能响应其它事件，必须等待这个事件处理完毕才能去处理其它事件，显然这和选择器的作用背道而驰。

  在将通道注册到选择器上时，还需要指定要注册的具体事件，主要有以下几类:

  - SelectionKey.OP_CONNECT
  - SelectionKey.OP_ACCEPT
  - SelectionKey.OP_READ
  - SelectionKey.OP_WRITE

  **套接字NIO示例**

  ```java
  public class NIOServer {
  
      public static void main(String[] args) throws IOException {
          Selector selector = Selector.open();
          ServerSocketChannel ssChannel = ServerSocketChannel.open();
          ssChannel.configureBlocking(false);
          ssChannel.register(selector, SelectionKey.OP_ACCEPT);
          ServerSocket serverSocket = ssChannel.socket();
          InetSocketAddress address = new InetSocketAddress("127.0.0.1", 8888);
          serverSocket.bind(address);
          while (true) {
              selector.select();
              Set<SelectionKey> keys = selector.selectedKeys();
              Iterator<SelectionKey> keyIterator = keys.iterator();
  
              while (keyIterator.hasNext()) {
  
                  SelectionKey key = keyIterator.next();
  
                  if (key.isAcceptable()) {
  
                      ServerSocketChannel ssChannel1 = (ServerSocketChannel) key.channel();
  
                      // 服务器会为每个新连接创建一个 SocketChannel
                      SocketChannel sChannel = ssChannel1.accept();
                      sChannel.configureBlocking(false);
  
                      // 这个新连接主要用于从客户端读取数据
                      sChannel.register(selector, SelectionKey.OP_READ);
  
                  } else if (key.isReadable()) {
  
                      SocketChannel sChannel = (SocketChannel) key.channel();
                      System.out.println(readDataFromSocketChannel(sChannel));
                      sChannel.close();
                  }
  
                  keyIterator.remove();
              }
          }
      }
  
      private static String readDataFromSocketChannel(SocketChannel sChannel) throws IOException {
  
          ByteBuffer buffer = ByteBuffer.allocate(1024);
          StringBuilder data = new StringBuilder();
  
          while (true) {
  
              buffer.clear();
              int n = sChannel.read(buffer);
              if (n == -1) {
                  break;
              }
              buffer.flip();
              int limit = buffer.limit();
              char[] dst = new char[limit];
              for (int i = 0; i < limit; i++) {
                  dst[i] = (char) buffer.get(i);
              }
              data.append(dst);
              buffer.clear();
          }
          return data.toString();
      }
  }
  public class NIOClient {
      public static void main(String[] args) throws IOException {
          Socket socket = new Socket("127.0.0.1", 8888);
          OutputStream out = socket.getOutputStream();
          String s = "hello world";
          out.write(s.getBytes());
          out.close();
      }
  }
  
  ```

  **NIO和BIO对比**

  - NIO 是非阻塞的
  - NIO 面向块，I/O 面向流

  个人感悟:

  BIO是 

  应用调用系统->系统内核无数据阻塞->系统内核数据准备好->复制数据到用户空间->复制完成通知应用->应用调用系统->... 的串行操作

  NIO通过多路复用实现了

  应用调用系统监控多个通道->轮询多个通道系统内核无数据阻塞->轮询发现某几个通道系统内核数据准备好->串行/并行(多线程)复制数据到用户空间->复制完成通知应用->继续轮询

  本质上将BIO的串行IO,变成并行的等待内核中数据准备后,轮询发现准备完成的通道,串行/并行的将内核中的数据拷贝到用户空间并执行业务逻辑