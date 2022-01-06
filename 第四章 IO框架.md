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

