---
title: NIO浅谈
date: 2020-01-02 23:08:24
description: Java NIO（New IO-Non Blocking IO 非阻塞IO）是从Java 1.4版本开始引入的一个新的IO API，可以替代标准的Java IO API。NIO与原来的IO有同样的作用和目的，但是使用的方式完全不同，NIO支持面向缓冲区的、基于通道的IO操作。NIO将以更加高效的方式进行文件的读写操作,而且NIO是非阻塞的。

image: 
categories:
 - Java
tags:
 - NIO
draft: true
---

## 1.1. 简介
> Java NIO（New IO-Non Blocking IO 非阻塞IO）是从Java 1.4版本开始引入的一个新的IO API，可以替代标准的Java IO API。NIO与原来的IO有同样的作用和目的，但是使用的方式完全不同，NIO支持面向缓冲区的、基于通道的IO操作。NIO将以更加高效的方式进行文件的读写操作,而且NIO是非阻塞的。

## 1.2. Buffer
> 一个用于特定基本数据类型的容器。由 java.nio 包定义的，所有缓冲区都是 Buffer 抽象类的子类。Java NIO 中的 Buffer 主要用于与 NIO 通道进行交互，数据是从通道读入缓冲区，从缓冲区写入通道中的。

> Buffer 就像一个数组，可以保存多个相同类型的数据。根据数据类型不同(boolean 除外) ，有以下 Buffer 常用子类：

* ByteBuffer
* CharBuffer
* ShortBuffer
* IntBuffer
* LongBuffer
* FloatBuffer
* DoubleBuffer

> 上述 Buffer 类他们都采用相似的方法进行管理数据，只是各自管理的数据类型不同而已。都是通过如下方法获取一个 Buffer对象:

```java
// 创建一个容量为 capacity 的 XxxBuffer 对象
static XxxBuffer allocate(int capacity) 
```

### 1.2.1. Buffer 中的重要概念
> **容量 (capacity) ：** 表示 Buffer 最大数据容量，缓冲区容量不能为负，并且创建后不能更改。  
>**边界(limit)：** 第一个不应该读取或写入的数据的索引，即位于 limit 后的数据不可读写。缓冲区的限制不能为负，并且不能大于其容量。  
>**位置 (position)：** 下一个要读取或写入的数据的索引。缓冲区的位置不能为负，并且不能大于其边界，默认值为0  
>**标记 (mark)与重置 (reset)：** 标记是一个索引，通过 Buffer 中的 mark() 方法指定 Buffer 中一个特定的 position，之后可以通过调用   reset() 方法恢复到这个 position，当调用mark方法后，当前position操作位置会被标记，标记的position可以通过reset来操作。，mark默认值为-1

**标记、位置、限制、容量不能违反该规则： 0 <= mark（默认值为-1） <= position <= limit <= capacity**

![Buffer中的属性关系](https://gitee.com/uRick/oss/raw/master/blog/Buffer中的属性关系.png)


### 1.2.2. 常用方法

|          方法          | 说明                                                         |
| :--------------------: | :----------------------------------------------------------- |
|    Buffer clear(）     | 清空缓冲区并返回对缓冲区的引用，也就是重置position=0，limit = capacity，mark = -1 |
|     Buffer flip(）     | 将缓冲区的界限设置为当前位置，并将当前位置重置为 0，limit = position，position = 0，mark = -1 |
|    int capacity(）     | 返回缓冲区的容量大小                                         |
|      int limit()       | 返回缓冲区中的界限位置                                       |
|  Buffer limit(int n）  | 将设置缓冲区界限为 n, 并返回一个具有新 limit 的缓冲区对象    |
|     Buffer mark(）     | 对缓冲区设置标记                                             |
|     int position()     | 返回缓冲区的当前位置 position                                |
| Buffer position(int n) | 将设置缓冲区的当前位置为 n , 并返回修改后的 Buffer 对象      |
|    int remaining()     | 返回 position 和 limit 之间的元素个数，limit - position，缓冲区中还有多少数据 |
|     Buffer reset()     | 将位置 position 转到以前设置的 mark 所在的位置，被标记的位置position = m |
|    Buffer rewind()     | 将位置设为为 0， 取消设置的 mark，position = 0;mark = -1     |
|   boolean isDirect()   | 判断当前缓冲区是不是直接缓冲区                               |


**方法实例：**

```java
 String str = "Rick";
        ByteBuffer allocate = ByteBuffer.allocate(10);
        allocate.put(str.getBytes());

        System.out.println(allocate.limit());
        System.out.println(allocate.position());
        System.out.println(allocate.mark());
        System.out.println(allocate.capacity());

        System.out.println("------------切换读模式-----------------");
        allocate.flip();

        System.out.println(allocate.limit());
        System.out.println(allocate.position());
        System.out.println(allocate.mark());
        System.out.println(allocate.capacity());

        System.out.println("--------------------->>>");
        byte[] bytes = "Lee".getBytes();
        ByteBuffer put = allocate.put(2, bytes[0]);
        System.out.println(allocate.limit());
        System.out.println(allocate.position());
        System.out.println(allocate.mark());
        System.out.println(allocate.capacity());

        byte[] b = new byte[4];
        put.get(b);
        System.out.println(new String(b));

        System.out.println("------------切换重复读模式-----------------");
        allocate.rewind();
        System.out.println(allocate.limit());
        System.out.println(allocate.position());
        System.out.println(allocate.mark());
        System.out.println(allocate.capacity());

        System.out.println("------------清空缓冲区-----------------");
        allocate.clear();// 清空缓冲区，但是缓冲区的数据依然存在，

        System.out.println(allocate.limit());
        System.out.println(allocate.position());
        System.out.println(allocate.mark());
        System.out.println(allocate.capacity());

        if (allocate.hasRemaining()) {
            System.out.println("remaining：" + allocate.remaining());
        }
```

**注意：**
> Buffer 所有子类提供了两个用于数据操作的方法：get()与 put() 方法,用于获取buffer中的数据和想buffer中放入缓冲区数据

```java
get() //读取单个字节，缓冲区当前位置position 所处的位置
get(byte[] dst)//批量读取多个字节到 dst 中
get(int index)//读取指定索引位置的字节(不会移动 position)


put(byte b)//将给定单个字节写入缓冲区的当前位置
put(byte[] src)//将 src 中的字节写入缓冲区的当前位置
put(int index, byte b)//将指定字节写入缓冲区的索引位置(不会移动 position),也就是 **替换原位置上的数据**
```

### 1.2.3. 科普一下
> 在NIO中是分为 **直接缓冲区** 和 **非直接缓冲区**；直接缓冲区是在物理内存开辟操作空间，非直接是在JVM的基础之上进行操作的；

> 字节缓冲区要么是直接的，要么是非直接的。如果为直接字节缓冲区，则 Java 虚拟机会尽最大努力直接在此缓冲区上执行本机 I/O 操作。也就是说，在每次调用基础操作系统的一个本机 I/O 操作之前（或之后），虚拟机都会尽量避免将缓冲区的内容复制到中间缓冲区中（或从中间缓冲区中复制内容）  

> 直接字节缓冲区可以通过调用此类的 **allocateDirect()** 工厂方法来创建。此方法返回的缓冲区进行分配和 **取消分配所需成本通常高于非直接缓冲区** 。直接缓冲区的内容可以 **驻留在常规的垃圾回收堆之外**，因此，它们对应用程序的内存需求量造成的影响可能并不明显。所以，建议将直接缓冲区主要分配给那些易受基础系统的本机 I/O 操作影响的大型、持久的缓冲区。一般情况下，最好仅在直接缓冲区能在程序性能方面带来明显好处时分配它们 

>直接字节缓冲区还可以通过 **FileChannel 的 map() 方法** 将文件区域 **直接映射到内存** 中来创建。该方法返回MappedByteBuffer 。Java 平台的实现有助于通过 JNI 从本机代码创建直接字节缓冲区。如果以上这些缓冲区中的某个缓冲区实例指的是不可访问的内存区域，则试图访问该区域不会更改该缓冲区的内容，并且将会在访问期间或稍后的某个时间导致抛出不确定的异常

> 字节缓冲区是直接缓冲区还是非直接缓冲区可通过调用其 isDirect() 方法来确定。提供此方法是为了能够在性能关键型代码中执行显式缓冲区管理。

![非直接缓冲区](https://gitee.com/uRick/oss/raw/master/blog/NIO非直接缓冲区.png)

![直接缓冲区](https://gitee.com/uRick/oss/raw/master/blog/NIO直接缓冲区.png)


## 1.3. Channel
> 通道（Channel）：由 java.nio.channels 包定义的。Channel 表示 IO 源与目标打开的连接。Channel 类似于传统的“流”。只不过 Channel本身不能直接访问数据，Channel 只能与Buffer 进行交互. 如：Channel 是轨道，Buffer是火车
![内存地址空间与用户地址空间交互关系](https://gitee.com/uRick/oss/raw/master/blog/内存地址空间与用户地址空间交互关系.png)

**Java 为 Channel 接口提供的最主要实现类如下:**

* FileChannel：用于读取、写入、映射和操作文件的通道。
* DatagramChannel：通过 UDP 读写网络中的数据通道。
* SocketChannel：通过 TCP 读写网络中的数据。
* ServerSocketChannel：可以监听新进来的 TCP 连接，对每一个新进来的连接都会创建一个 SocketChannel

**常见获取通道的几种方式：**

* FileInputStream
* FileOutputStream 
* RandomAccessFile 
* DatagramSocket 
* Socket 
* ServerSock

**对于以上几个类都支持通道，可以使用getChannel获取Channeld对象**

```java
 FileInputStream fileInputStream = new FileInputStream("src\\nio\\channel\\1.txt");
        FileOutputStream fileOutputStream = new FileOutputStream("src\\nio\\channel\\2.txt");
        FileChannel channel = fileInputStream.getChannel();
        FileChannel channel1 = fileOutputStream.getChannel();
        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);

        while (channel.read(byteBuffer) != -1) {
            byteBuffer.flip();
            channel1.write(byteBuffer);
            byteBuffer.clear();
        }

        fileInputStream.close();
        channel.close();
        fileOutputStream.close();
        channel1.close();
```

```java
  // 不知道为什么没有数据
        FileChannel fileChannel = FileChannel.open(Paths.get("src\\nio\\channel\\2.txt"), StandardOpenOption.READ);
        FileChannel fileChannel2 = FileChannel.open(Paths.get("src\\nio\\channel\\5.txt"), StandardOpenOption.READ, StandardOpenOption.WRITE, StandardOpenOption.CREATE);

        MappedByteBuffer map = fileChannel.map(FileChannel.MapMode.READ_ONLY, 0, fileChannel.size());
        MappedByteBuffer map1 = fileChannel2.map(FileChannel.MapMode.READ_WRITE, 0, fileChannel.size());

        map.flip();
        byte[] b = new byte[map.limit()];
        System.out.println(map.limit() + " -> " + map.position() + " -> " + map.mark());
        map.get(b);
        map1.put(b);

        fileChannel.close();
        fileChannel2.close();

```

还有其他常见方式：

1. Files.newByteChannel(……)
2. 通过通道的静态方法 open() 打开并返回指定通道如：

```
  SocketChannel channel = SocketChannel.open(new InetSocketAddress("127.0.0.1", 8088));
```

### 1.3.1. 通道的分散读(Scatter)聚集写(Gather)

1. 分散读：分散读取（Scattering Reads）是指从 Channel 中读取的数据“分散”到多个 Buffer 中。

![分散读](https://gitee.com/uRick/oss/raw/master/blog/分散读.png)

```java
// 按照缓冲区的顺序，从 Channel 中读取的数据依次将 Buffer 填满
 RandomAccessFile raf1 = new RandomAccessFile("src\\nio\\channel\\斗破苍穹.txt", "rw");
//1. 获取通道
FileChannel channel1 = raf1.getChannel();

//2. 分配指定大小的缓冲区
ByteBuffer buf1 = ByteBuffer.allocate(100);
ByteBuffer buf2 = ByteBuffer.allocate(1024);

//3. 分散读取
ByteBuffer[] bufs = {buf1, buf2};
channel1.read(bufs);

for (ByteBuffer byteBuffer : bufs) {
    byteBuffer.flip();
}

System.out.println(new String(bufs[0].array(), 0, bufs[0].limit()));
System.out.println("-----------------》》》》》");
System.out.println(new String(bufs[1].array(), 0, bufs[1].limit()));
```


2. 聚集写：聚集写入（Gathering Writes）是指将多个 Buffer 中的数据“聚集”到 Channel。

![聚集写：聚集写入（Gathering](https://gitee.com/uRick/oss/raw/master/blog/聚集写.png)


```java
// 按照缓冲区的顺序，写入 position 和 limit 之间的数据到 Channel 
ByteBuffer buf1 = ByteBuffer.allocate(100);
ByteBuffer buf2 = ByteBuffer.allocate(1024);
ByteBuffer[] bufs = {buf1, buf2};
//4. 聚集写入
RandomAccessFile raf2 = new RandomAccessFile("1.txt", "rw");
FileChannel channel2 = raf2.getChannel();

channel2.write(bufs);

for (ByteBuffer byteBuffer : bufs) {
    byteBuffer.flip();
}

System.out.println(new String(bufs[0].array(), 0, bufs[0].limit()));
System.out.println("-----------------》》》》》");
System.out.println(new String(bufs[1].array(), 0, bufs[1].limit()));
```

### 1.3.2. FileChannel常用方法

| 方法                          | 说明                                         |
| ----------------------------- | -------------------------------------------- |
| int read(ByteBuffer dst)      | 从 Channel 中读取数据到 ByteBuffer           |
| long read(ByteBuffer[] dsts)  | 将 Channel 中的数据“分散”到 ByteBuffer[]     |
| int write(ByteBuffer src)     | 将 ByteBuffer 中的数据写入到 Channel         |
| long write(ByteBuffer[] srcs) | 将 ByteBuffer[] 中的数据“聚集”到 Channel     |
| long position()               | 返回此通道的文件位置                         |
| FileChannel position(long p)  | 设置此通道的文件位置                         |
| long size()                   | 返回此通道的文件的当前大小                   |
| FileChannel truncate(long s)  | 将此通道的文件截取为给定大小                 |
| void force(boolean metaData)  | 强制将所有对此通道的文件更新写入到存储设备中 |


### 1.3.3. 科普一下
> 什么是阻塞与非阻塞，顾名思义，阻塞就是需要排队等待，非阻塞就是不需要排队，合理利用可用的资源，并将资源利用最大化。

> **传统的 IO 流都是阻塞式的** 。也就是说，当一个线程调用 read() 或 write()时，该线程被阻塞，直到有一些数据被读取或写入，该线程在此期间不能执行其他任务。因此，在完成网络通信进行 IO 操作时，由于线程会阻塞，所以**服务器端必须为每个客户端都提供一个独立的线程进行处理，当服务器端需要处理大量客户端时，性能急剧下降**。

> **Java NIO 是非阻塞模式的** 。当线程从某通道进行读写数据时，若没有数据可用时，该线程可以进行其他任务。线程通常将非阻塞 IO 的空闲时间用于在其他通道上执行 IO 操作，所以单独的线程可以管理多个输入和输出通道。因此，**NIO 可以让服务器端使用一个或有限几个线程来同时处理连接到服务器端的所有客户端**。但是这种方式是怎么切换的呢？怎知道哪一个线程是空闲的呢？没有数据可用呢？当然这个肯定需要一个监控器，这个监控器就是Selector

![阻塞实例](https://gitee.com/uRick/oss/raw/master/blog/NO_阻塞实例.png)

![非阻塞实例](https://gitee.com/uRick/oss/raw/master/blog/NIO_非阻塞实例.png)





### 1.3.4. 选择器（Selector）
> 选择器是JavaNIO 中不可缺少的组件（实现非阻塞时）

> 选择器（Selector） 是 SelectableChannle 对象的多路复用器，Selector 可以同时监控多个 SelectableChannel 的 IO 状况，也就是说，利用 Selector 可使一个单独的线程管理多个 Channel。Selector 是非阻塞IO的核心。

![Selector类关系图](https://gitee.com/uRick/oss/raw/master/blog/NIO_Selector类关系图.png)

#### 1.3.4.1. 获取选择器

```java
// 获取选择器
Selector selector = Selector.open();
// 向渠道中注入一个选择器，指定选择器监听的事件
// 当调用 register(Selector sel, int ops) 将通道注册选择器时，选择器对通道的监听事件，需要通过第二个参数 ops 指定
serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
```

#### 1.3.4.2. SelectionKey
> **表示 SelectableChannel 和 Selector 之间的注册关系**。每次向选择器注册通道时就会 **选择一个事件(选择键)**。选择键包含两个表示为整数值的操作集。操作集的每一位都表示该键的通道所支持的一类可选择操作

**四个常量事件：**

1. 读 : SelectionKey.OP_READ 
2. 写 : SelectionKey.OP_WRITE 
3. 连接 : SelectionKey.OP_CONNECT 
4. 接收 : SelectionKey.OP_ACCEPT 

若注册时不止监听一个事件，则可以使用“位或”操作符连接

```java
int key = SelectionKey.OP_ACCEPT | SelectionKey.OP_WRITE | SelectionKey.OP_READ;
```

### 1.3.5. SelectionKey 常用方法
| 方法                        | 说明                             |
| --------------------------- | -------------------------------- |
| int interestOps()           | 获取感兴趣事件集合               |
| int readyOps()              | 获取通道已经准备就绪的操作的集合 |
| SelectableChannel channel() | 获取注册通道                     |
| Selector selector()         | 返回选择器                       |
| boolean isReadable()        | 检测 Channal 中读事件是否就绪    |
| boolean isWritable(）       | 检测 Channal 中写事件是否就绪    |
| boolean isConnectable(）    | 检测 Channal 中链接事件是否就绪  |
| boolean isAcceptable()      | 检测 Channal 中接收事件是否就绪  |

### 1.3.6. Selector常用方法
| 方法                     | 说明                                                         |
| ------------------------ | ------------------------------------------------------------ |
| Set\<SelectionKey\> keys() | 所有的 SelectionKey 集合。代表注册在该Selector上的Channel    |
| selectedKeys(）          | 被选择的 SelectionKey 集合。返回此Selector的已选择键集       |
| int select(）            | 监控所有注册的Channel，当它们中间有需要处理的 IO 操作时，该方法返回，并将对应得的 SelectionKey 加入被选择的SelectionKey 集合中，该方法返回这些 Channel 的数量。 |
| int select(long timeout) | 可以设置超时时长的 select() 操作                             |
| int selectNow(）         | 执行一个立即返回的 select() 操作，该方法不会阻塞线程         |
| Selector wakeup(）       | 使一个还未返回的 select() 方法立即返回                       |
| void close(）            | void close() 关                                              |


### 1.3.7. SocketChannel/DatagramChannel实例

1.  **SocketChannel客户端**

```java
 SocketChannel socketChannel = SocketChannel.open(new InetSocketAddress("127.0.0.1", 9998));

        // 切换通道为非阻塞式
        socketChannel.configureBlocking(false);

        Scanner scanner = new Scanner(System.in);

        ByteBuffer b = ByteBuffer.allocate(1024);
        while (scanner.hasNext()) {
            System.out.println("sdsds");
            String next = scanner.next();
            b.put(next.getBytes());
            b.flip();
            socketChannel.write(b);
            b.clear();
        }

        socketChannel.close();
    }
```

2. **SocketChannel服务端**

```java
 ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        // 开启非阻塞支持
        serverSocketChannel.configureBlocking(false);

        serverSocketChannel.bind(new InetSocketAddress(9998));

        // 获取选择器
        Selector selector = Selector.open();
        // 向渠道中注入一个选择器，指定选择器监听的事件
        int key = SelectionKey.OP_ACCEPT | SelectionKey.OP_WRITE | SelectionKey.OP_READ;
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

        // 轮询获取选择器中所有注册的“已经就绪的事件”
        while (selector.select() > 0) {
            Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
            while (iterator.hasNext()) {
                // 拿到选择的就绪事件
                SelectionKey sk = iterator.next();
                // 判断什么事件就绪
                if (sk.isAcceptable()) {
                    // 若接收就绪则接收客户端连接
                    SocketChannel accept = serverSocketChannel.accept();
                    // 开启非阻塞支持
                    accept.configureBlocking(false);

                    accept.register(selector, SelectionKey.OP_READ);
                } else if (sk.isReadable()) {
                    SocketChannel channel = (SocketChannel) sk.channel();
                    ByteBuffer bb = ByteBuffer.allocate(1024);
                    int len;
                    while ((len = channel.read(bb)) != -1) {
                        bb.flip();
                        System.out.println(new String(bb.array(), 0, len));
                        bb.clear();
                    }
                }
                // 取消选择键
                iterator.remove();
            }
```

2. **DatagramChannel发送端**

```java
DatagramChannel dc = DatagramChannel.open();
		
		dc.configureBlocking(false);
		
		ByteBuffer buf = ByteBuffer.allocate(1024);
		
		Scanner scan = new Scanner(System.in);
		
		while(scan.hasNext()){
			String str = scan.next();
			buf.put((new Date().toString() + ":\n" + str).getBytes());
			buf.flip();
			dc.send(buf, new InetSocketAddress("127.0.0.1", 9898));
			buf.clear();
		}
		
		dc.close();
```

3. **DatagramChannel接收端**

```java
DatagramChannel dc = DatagramChannel.open();
		
		dc.configureBlocking(false);
		
		dc.bind(new InetSocketAddress(9898));
		
		Selector selector = Selector.open();
		
		dc.register(selector, SelectionKey.OP_READ);
		
		while(selector.select() > 0){
			Iterator<SelectionKey> it = selector.selectedKeys().iterator();
			
			while(it.hasNext()){
				SelectionKey sk = it.next();
				
				if(sk.isReadable()){
					ByteBuffer buf = ByteBuffer.allocate(1024);
					
					dc.receive(buf);
					buf.flip();
					System.out.println(new String(buf.array(), 0, buf.limit()));
					buf.clear();
				}
			}
			
			it.remove();
		}
```

## 1.4. 管道（Pipe）
> 管道是2个线程之间的单向数据连接。Pipe有一个source通道和一个sink通道。数据会被写到sink通道，从source通道读取。

```java
		//1. 获取管道
		Pipe pipe = Pipe.open();
		
		//2. 将缓冲区中的数据写入管道
		ByteBuffer buf = ByteBuffer.allocate(1024);
		
		Pipe.SinkChannel sinkChannel = pipe.sink();
		buf.put("通过单向管道发送数据".getBytes());
		buf.flip();
		sinkChannel.write(buf);
		
		//3. 读取缓冲区中的数据
		Pipe.SourceChannel sourceChannel = pipe.source();
		buf.flip();
		int len = sourceChannel.read(buf);
		System.out.println(new String(buf.array(), 0, len));
		
		sourceChannel.close();
		sinkChannel.close();
	}
```

## 1.5. Channels工具类

| 方法                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ReadableByteChannel newChannel(final InputStream in)         | 返回一个将从给定的输入流读取数据的通道                       |
| WritableByteChannel newChannel(final OutputStream out)       | 返回一个将向给定的输出流写入数据的通道。                     |
| InputStream newInputStream(ReadableByteChannel ch)           | 返回一个将从给定的通道读取字节的流                           |
| OutputStream newOutputStream(final WritableByteChannel ch)   | 返回一个将向给定的通道写入字节的流。                         |
| InputStream newInputStream(final AsynchronousByteChannel ch) | 构造一个输入流，该流对多个并发线程访问时安全的，通过字节异步同步通道 |
| OutputStream newOutputStream(final AsynchronousByteChannel ch) | 构造一个输出流，该流对多个并发线程访问时安全的，通过字节异步同步通道 |
| Reader newReader(ReadableByteChannel ch,                                CharsetDecoder dec,                                int minBufferCap) | 返回一个 reader，它将从给定的通道读取字节并依据提供的CharsetDecoder 对读取到的字节进行解码。 |
| Reader newReader(ReadableByteChannel ch,                                String csName) | 返回一个 reader，它将从给定的通道读取字节并依据提供的字符集名称将读取到的字节解码成字符 |
| Writer newWriter(final WritableByteChannel ch,                                final CharsetEncoder enc,                                final int minBufferCap) | 返回一个 writer，它将使用提供的 CharsetEncoder 对象对字符编码并写到给定的通道中 |
| Writer newWriter(WritableByteChannel ch,                                String csName) | 返回一个 writer，它将依据提供的字符集名称对字符编码并写到
给定的通道中。 |

## 1.6. Path类
> 表示一个目录序列，其后跟着文件名，对应Path类提供一个工具类Paths用户构造Path路径对象
> 它是路径的一个抽象名字序列，不需要指定实际存在的文件路径
> 同时对老板的API做了很好的兼容，java.nio.file.Path#toFile /java.io.File#toPath

### 1.6.1. 主要方法

```java
// 
Path resolve(Path other);// 若other是绝对路径，则返回other(Path中几个路径拼接的方法都是这种处理机制)，否则是this链接other的路径
Path resolve(String other);// 同上
Path resolveSibling(Path other);// 若other是绝对路径，则返回other(Path中几个路径拼接的方法都是这种处理机制),否则返回this的父路径链接other路径，即获取other的兄弟路径
Path resolveSibling(String other);// 同上
Path relativize(Path other);// 返回this相对于other的路径 this:/other/others other:/other/others/cat 最终路径：..\..\others\cat
Path normalize();//去除冗余的.和.. 的路径元素
Path getRoot();//获取当前路径的根路径
Path getFileName();// 返回当前路径的最后一个路径（或者文件名）如：/file/other 这个路径返回other
Path getParent() 返回当前路径的父路径，若没有父路径则返回null
```

## 1.7. Paths
> 对应Path的一个工具类，只有两个方法

```java
// 返回链接的路径
public static Path get(String first, String... more) {
        return FileSystems.getDefault().getPath(first, more);
    }

// 这个URI路径
public static Path get(URI uri){……}
```

## 1.8. Files
> 大大提高文件访问效率，同时代码量减少了需要，Files工具类提供常见的文件操作，当然这些快捷操作方式只适用一个中等长度的文件，大文件或者二进制文件，可以使用传统的api 方式（IO）操作.

### 1.8.1. 主要方法说明

```java
// IO流操作的转换
public static InputStream newInputStream(Path path, OpenOption... options)
public static OutputStream newOutputStream(Path path, OpenOption... options)
public static BufferedReader newBufferedReader(Path path, Charset cs)
public static BufferedReader newBufferedReader(Path path) 
public static BufferedWriter newBufferedWriter(Path path, Charset cs,OpenOption... options)
public static BufferedWriter newBufferedWriter(Path path, OpenOption... options)

// 其他文件操作方法可查看API
```

### 1.8.2. 文件读取案例
> 相对于IO阻塞流操作这个太便捷了，相当牛X

```java
     Path path = Paths.get("src\\nio\\hello.txt");
    // 获取文件内容转换为List集合
    List<String> files = Files.readAllLines(path, Charset.forName("utf8"));
    for (String file : files) {
        System.out.println(file);
    }

```

### 1.8.3. 搜索文件案例
> 非常强大支持通配搜索文件，使用方法 public static DirectoryStream<Path> newDirectoryStream(Path dir, String glob)

```java
try (DirectoryStream<Path> entries = Files.newDirectoryStream(Paths.get("src\\nio"), "*.java")) {
            for (Path entry : entries) {
                System.out.println(entry.toString());
            }
        }
```

**Glob通配表达式：**
*注意：*  若在windows中使用glob 语法，需要进行两次转义：Files.newDirectoryStream(dir,"C:\\\\")

![](https://gitee.com/uRick/oss/raw/master/blog/Glob模式.png)


### 1.8.4. 访问文件树案例
>使用方法  public static Path walkFileTree(Path start, FileVisitor<? super Path> visitor)
>使用该方法可以很好的处理多个文件，并且提供FileVisitor来实现文件操作过程进行相关处理

```java
Path path = Files.walkFileTree(Paths.get("src\\nio"), new FileVisitor<Path>() {
            /*
             * FileVisitResult 返回对象，表示执行对应的方法后续的处理动作
             * java.nio.file.FileVisitResult.CONTINUE 继续访问下一个文件
             * java.nio.file.FileVisitResult.TERMINATE 终止访问
             * java.nio.file.FileVisitResult.SKIP_SUBTREE 继续访问，但不在范文这个目录的任何项了
             * java.nio.file.FileVisitResult.SKIP_SIBLINGS 继续访问，但是不在访问这个文件的兄弟文件了（同该文件的同一个目录的的文件）
             *
             *
             */
            // 一个目录被操作前
            @Override
            public FileVisitResult preVisitDirectory(Path dir, BasicFileAttributes attrs) throws IOException {
                System.out.println("目录：" + dir.toString() + "被操作了");

                return FileVisitResult.CONTINUE;
            }

            // 这里表示在遇到一个文件或者目录是怎么操作

            @Override
            public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
                System.out.println("文件：" + file.toString() + "被操作了");
                return FileVisitResult.CONTINUE;
            }

            @Override
            public FileVisitResult visitFileFailed(Path file, IOException exc) throws IOException {
                System.out.println("文件：" + file.toString() + "操作失败了");

                return FileVisitResult.CONTINUE;
            }

            // 目录被操作后
            @Override
            public FileVisitResult postVisitDirectory(Path dir, IOException exc) throws IOException {
                System.out.println("目录：" + dir.toString() + "被操作了");
                return FileVisitResult.CONTINUE;
            }
        });
```


