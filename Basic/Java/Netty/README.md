

[TOC]

## 一、Netty介绍

### 1.1 基本介绍

1. Netty是有JBOSS提供的一个开源框架，现为Github上的独立项目。

2. Netty是一个异步的、基于事件驱动的网络应用框架，用以快速开发高性能、高可靠的网络IO程序。

   ![image-20201023110523257](http://img.hurenjieee.com/uPic/image-20201023110523257.png)

3. Netty主要针对在TCP协议下，面向Clients端的高并发应用，或者Peer-to-Peer场景下的大量数据持续传输的应用。

4. Netty本质是一个NIO框架，适用于服务器通讯相关的多种应用场景。

   ![image-20201023110725733](http://img.hurenjieee.com/uPic/image-20201023110725733.png)

### 1.2 应用场景

#### 1.2.1 互联网

* 互联网行业中，在各个分布式系统中，各个节点与服务之间需要高性能的RPC框架，Netty作为异步高性能的通信框架，通常被各种RPC框架作为基础通信组件使用。阿里分布式服务框架Dubbo就是一个典型的案例。

#### 1.2.2 游戏

* 游戏行业需要高性能、低延迟到网络通信，Netty提供了基础，开发者可以在此之上进行开发改造。

#### 1.2.3 大数据

* 经典的Hadoop的高性能通信和序列化组件Avro的 RPC 框架，默认采用Netty 进行跨界点通信。

#### 1.2.4 其他开源项目

使用到的开源项目：https://netty.io/wiki/related-projects.html

![image-20201023113130877](http://img.hurenjieee.com/uPic/image-20201023113130877.png)

## 二、Java BIO

### 2.1 I/O模型

#### 2.1.1 I/O模型基本说明

* I/O模型简单的理解：就是用什么样的通道进行数据的发送和接收，很大程度上决定够了程序通信的性能

* Java共支持3种网络编程模型I/O模式：BIO、NIO、AIO

  1. Java BIO：

     > 同步并阻塞，服务器实现模式为一个连接一个线程，即客户端有连接请求时服务端就要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销。
     >
     > ![image-20201023135715660](http://img.hurenjieee.com/uPic/image-20201023135715660.png)

  2. Java NIO：

     > 同步非阻塞，服务器实现模式为一个线程处理多个请求（连接），即客户端发送的链接都会注册到多路复用器上，多路复用器轮训到连接I/O请求就进行处理。
     >
     > ![image-20201023140139718](http://img.hurenjieee.com/uPic/image-20201023140139718.png)

  3. Java AIO：

     > 异步非阻塞，AIO引入异步通道的概念，采用了Practor模式，简化了程序编写，有效的请求才启动线程，它的特点是先由操作系统完成后才通知服务端程序启动线程去处理，一般适用于连接数较多且连接时间较长的应用。

### 2.2 BIO、NIO、AIO 适用场景分析

1. BIO方式适用于连接数据比较小且固定的架构，这种方式对服务器资源要求比较高，并发局限于应用中，JDK1.4以前的唯一选择，但是程序简单易理解。
2. NIO方式适用于连接数目多且连接比较短（轻操作）的架构，比如聊天服务器，弹幕系统，服务器间通讯等。编程比较复杂，JDK1.4开始支持。
3. AIO方式适用于连接数目多且连接比较长（重操作）的架构，比如相册服务器，充分调用OS参与并发操作，编程比较复杂，JDK1.7开始支持。

### 2.3 Java BIO 基本介绍

1. Java BIO就是传统的java io模型，其基本的类和接口在java.io
2. BIO(blocking I/O)：同步阻塞，服务器实现模式为一个连接一个线程，即客户端有连接请求时服务端酒需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销，可以通过线程池机制改善（实现多个客户端连接服务器）。
3. BIO方式适用于连接数目比较小且固定的架构，这种方式对服务器资源要求比较高，并发局限于应用中国呢，JDK1.4以前的唯一选择，程序简单理解。

### 2.4 Java BIO 工作机制

![image-20201023142152223](http://img.hurenjieee.com/uPic/image-20201023142152223.png)

编程流程：

1. 服务端启动一个ServerSocket
2. 客户端启动Socket对服务器进行通信，默认情况下服务器需要对每个客户简历一个线程与之通信
3. 客户端发出请求后，先咨询服务器是否有线程响应，如果没有则会等待，或者被拒绝。
4. 如果有响应，客户端线程会等等待请求结束后，再继续执行。

### 2.5 Java BIO 应用实例

通过BIO模型编写一个服务器端，监听6666端口，当有客户端连接时，就启动一个线程与之通讯。简易实用线程池机制改善，可以连接多个客户端。

```java

import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.concurrent.*;

public class BIOServer {

    public static void main(String[] args) throws IOException {

        ThreadFactory namedThreadFactory = new ThreadFactoryBuilder()
                .setNameFormat("demo-pool-%d").build();

        ExecutorService pool = new ThreadPoolExecutor(5, 200,
                0L, TimeUnit.MILLISECONDS,
                new LinkedBlockingQueue<Runnable>(1024), namedThreadFactory, new ThreadPoolExecutor.AbortPolicy());

        ServerSocket serverSocket = new ServerSocket(6666);

        System.out.println("server start");

        while(true){
            final Socket socket = serverSocket.accept();
            System.out.println("接收到请求");
            pool.execute(new Runnable() {
                @Override
                public void run() {
                    try {
                        InputStream inputStream = socket.getInputStream();
                        OutputStream outputStream = socket.getOutputStream();
                        byte[] bytes = new byte[1024];
                        while (true){
                            int read = inputStream.read(bytes);
                            if(read != -1){
                                System.out.println(Thread.currentThread().getName()+new String(bytes,0,read));
                                outputStream.write(bytes);
                            }else{
                                break;
                            }
                        }

                    } catch (IOException e) {
                        e.printStackTrace();
                    }}
            });
        }

    }
}
```

### 2.6 Java BIO 问题分析

1. 每个请求都需要创建的独立的线程，与对应的客户端进行数据Read、业务处理、数据Write。
2. 当数据量较大时，需要创建大量线程来处理连接，系统资源占用较大。
3. 连接建立后，如果当前线程暂时没有数据刻度，则线程会阻塞在Read操作上，造成线程资源浪费。



## 三、Java NIO编程

### 3.1 Java NIO基本介绍

1. Java NIO全称java non-blocking IO，是指JDK提供的新API。从JDk1.4开始，Java提供了一系列改进输入/输出的新特性，是同步非阻塞的。

2. NIO的相关类都放在java.nio包及子包下，并且对原java.io包重的很多类进行改写。

3. NIO有三大核心部分：**Channel（通道）**，**Buffer（缓冲区）**，**Selector（选择器）**。

4. **NIO是面向缓冲区的，或者面向块编程的**。数据读取到一个它稍后处理的缓冲区，需要时可在缓冲区中前后移动，这就增加了处理过程中的灵活性，使用它可以提供非阻塞时的高伸缩好性网络

5. Java的NIO时非阻塞模式，使一个线程从某通道发送请求或者读取数据，但是它仅能得到目前可用的数据，如果目前没有数据可用时，就什么都不会获取，而不是保持线程阻塞，所以直至数据变的可以读取之前，该线程可以继续做其他的全事情。非阻塞写也是如此，一个线程请求写入一些数据到某通道，但是不需要等待它完全写褥，这个线程同时可以去做别的事情。

6. 通俗理解：NIO时可以做到用一个线程来处理多个操作的。假设有10000个请求过来，根据实际情况，可以分配50或者100个线程来处理。不像之前的额阻塞IO那样，非的分配10000个。

7. HTTP2.0使用了多路复用的技术，做到同一个连接并发处理多个请求，而且并发请求的数量比HTTP1.1打了多几个数量级。

8. ```java
   // Buffer是可读可写的
   import java.nio.IntBuffer;
   
   public class BasicBuffer {
       public static void main(String[] args) {
           // 创建一个大小为5的buffer
           IntBuffer intBuffer = IntBuffer.allocate(5);
           intBuffer.put(1);
           intBuffer.put(2);
           intBuffer.put(3);
           intBuffer.put(4);
           intBuffer.put(5);
   
           //flip是将当前的位置position赋值给limit，所以适用于读写当前位置之前的数据。
           intBuffer.flip();
   
           while (intBuffer.hasRemaining()){
               System.out.println(intBuffer.get());
           }
   
       }
   }
   ```

### 3.2 NIO 和 BIO的比较

1. BIO以流的方式处理数据，而NIO以快的方式处理数据，块I/O的效率比流I/O高很多
2. BIO是阻塞的，NIO是非阻塞的
3. BIO基于字节流和字符流进行操作，而NIO基于channel和Buffer进行操作，数据总是从通道读取到缓冲去，或者从缓冲区写入到通道中，Selector用于监听多个通道的事件（比如：连接请求，数据到达等），因此使用单个线程就可以监听多个客户端通道。

### 3.3 NIO三大核心原理示意图

NIO三大核心：Selector、Channel、Buffer：

![image-20201023163248800](http://img.hurenjieee.com/uPic/image-20201023163248800.png)

特点：

1. 每个Channel都会对应一个Buffer
2. Selector对应一个线程，一个线程对应多个Channel
3. 上图反应了有三个Channel注册到该Selector程序
4. 程序切换到哪个channel是由事件决定的，Event就一个重要的概念
5. Selecor会根据不同的时间，在各个通道上切换
6. Buffer就是一个内存块，底层是有一个数组
7. 数据的读取写入是通过Buffer，NIO的buffer是可以赌也可以写的，需要通过flip方法去换。channel是双向的，可以返回底层操作系统的情况，比如Linux，底层的操作系统就是双向的。

### 3.4 缓冲区（Buffer）

#### 3.4.1 基本介绍

缓冲区本质上是一个可以读写数据的内存块，可以理解成是一个容器对象（含数组），该对象提供了一组方法，可以更轻松地使用内存块，缓冲区对象内置了一些机制，能够跟踪和记录缓冲区的状态变化情况。Channel提供从文件、网络读取数据的渠道，但是读取或写入的数据必须经由Buffer。

![image-20201023165848838](http://img.hurenjieee.com/uPic/image-20201023165848838.png)

#### 3.4.2 Buffer类及其子类

在NIO中，Buffer是一个顶层父类，它是一个抽象类类

* 不同的Buffer子类定义了不同的数据存储类型

  ![image-20201023171445290](http://img.hurenjieee.com/uPic/image-20201023171445290.png)

* Buffer类定义了所有的缓冲区都具有的四个属性来提供关于其所包含的原属的信息：

  ```java
  
      // Invariants: mark <= position <= limit <= capacity
      // 标记
  		private int mark = -1;
      // 位置(position)：下一个要读取或写入的数据的索引，每次读写缓冲区数据时都会改变值，为下次读写操作做准备
      private int position = 0;
      // 限制(limit)：缓冲区的当前终点，超过当前终点不应该读取或写入的数据的索引。即位于limit后的数据不可读写。
      private int limit;
  		// 容量(capacity):表示Buffer最大数据容量，缓冲区容量不能为负，并且创建后不能更改。
      private int capacity;
  
  ```

* Buffer相关方法总览

  ![image-20201023172341165](http://img.hurenjieee.com/uPic/image-20201023172341165.png)

#### 3.4.3 ByteBuffer

从前面可以看出对于Java中的基本数据类型（Boolean除外），都有一个Buffer类型与之相对应，最常用的就是ByteBuffer类（二进制数据）。

![image-20201023173435923](http://img.hurenjieee.com/uPic/image-20201023173435923.png)

### 3.5 通道（Channel）

#### 3.5.1 基本介绍

* NIO的通道类似于流，但有些区别如下：

  * 通道可以同时进行读写，而流只能读或者只能写
  * 通道可以实现异步读写数据
  * 通道可以从缓冲区读数据，也可以写数据到缓冲。

* Channel在NIO中是一个接口

  ```java
  public interface Channel extends Closeable {
  
      public boolean isOpen();
  
      public void close() throws IOException;
  
  }
  ```

* 常用的Channel类有：

  * FileChannel：文件的数据读写
  * DatagramChannel：UDP的数据读写
  * ServerSocketChannel：TCP的数据读写
  * SocketChannel：TCP的数据读写

  ![image-20201023174359830](http://img.hurenjieee.com/uPic/image-20201023174359830.png)

#### 3.5.2 FileChannel

FileChannel主要用来对本地文件进行IP操作，常见的方法有：

* public int read(ByteBuffer dst)，从通道读取数据并放到缓冲区中
* public int write(ByteBuffer src)，把缓冲区的数据写到通道中
* public long transferFrom(ReadableByteChannel src, long position, long count)，从目标通道中复制数据到当前通道
* public long transferTo(long position, long count, WritableByteChannel target)，把数据从当前通道复制给目标通道

##### 3.5.2.1 本地文件写数据

```java

import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;

public class NIOWriteFile {
    public static void main(String[] args) throws IOException {
        String str = "hello world";
        // 如果文件不存在，创建文件
        File file = new File("./test01.txt");
        if (!file.exists()) {
            file.createNewFile();
        }
        // 获取文件输出流
        FileOutputStream fileOutputStream = new FileOutputStream("./test01.txt");
        // 数据流获取Channel
        FileChannel channel = fileOutputStream.getChannel();
        // 创建缓冲区Buffer
        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
        // 把数据放入Buffer
        byteBuffer.put(str.getBytes());
        // flip
        byteBuffer.flip();
        // 通道写入buffer
        channel.write(byteBuffer);
        fileOutputStream.close();
    }
}

```

##### 3.5.2.2 本地文件读数据

```java

import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;

public class NIOReadFile {
    public static void main(String[] args) throws IOException {i
        // 获取文件输入流
        FileInputStream fileInputStream = new FileInputStream("./test01.txt");
        // 数据流获取Channel
        FileChannel channel = fileInputStream.getChannel();
        // 创建缓冲区Buffer
        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
        // 通道写入buffer
        int read = channel.read(byteBuffer);
        System.out.println(new String(byteBuffer.array(), 0, read));
        fileInputStream.close();
    }
}
```

##### 3.5.2.3 使用一个Buffer完成文件读取、写入

![image-20201023180127005](http://img.hurenjieee.com/uPic/image-20201023180127005.png)

```java

import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;

public class NIOCopyFile {
    public static void main(String[] args) throws IOException {
        String str = "hello world";
        // 如果文件不存在，创建文件
        File file = new File("./test01.txt");
        // 获取文件输入流
        FileInputStream fileInputStream = new FileInputStream("./test01.txt");
        // 获取文件输出流
        FileOutputStream fileOutputStream = new FileOutputStream("./test02.txt");
        // 数据流获取Channel
        FileChannel inputChannel = fileInputStream.getChannel();
        // 数据流获取Channel
        FileChannel outputChannel = fileOutputStream.getChannel();
        // 创建缓冲区Buffer
        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);

        while(true){
            // 清空缓冲区
            byteBuffer.clear();
            // 读取数据
            int read = inputChannel.read(byteBuffer);
            if(read == -1){
                break;
            }
            // 切换flip
            byteBuffer.flip();
            // 写入数据
            outputChannel.write(byteBuffer);

        }

        fileInputStream.close();
        fileOutputStream.close();
    }
}

```

##### 3.5.2.4 文件拷贝

```java

import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.nio.channels.FileChannel;

public class NIOTransferFromFile {
    public static void main(String[] args) throws IOException {
        // 获取文件输入流
        FileInputStream fileInputStream = new FileInputStream("./test01.txt");
        // 获取文件输出流
        FileOutputStream fileOutputStream = new FileOutputStream("./test03.txt");
        // 数据流获取Channel
        FileChannel inputChannel = fileInputStream.getChannel();
        // 数据流获取Channel
        FileChannel outputChannel = fileOutputStream.getChannel();

        outputChannel.transferFrom(inputChannel, 0, inputChannel.size());

        fileInputStream.close();
        fileOutputStream.close();
    }
}
```

##### 3.5.2.5 关于Buffer和Channel的注意事项和细节

1. ByteBuffer支持类型化的put和get，put放入的是什么数据类型，get就应该使用相应的数据类型来取出，否则可能有BufferUnderflowException异常。

   > ```java
   > 
   > import java.nio.ByteBuffer;
   > 
   > public class NIOByteBufferGetPut {
   > 
   >     public static void main(String[] args) {
   >         ByteBuffer byteBuffer = ByteBuffer.allocate(65);
   >         //类型化方式放入数据
   >         byteBuffer.putInt(100);
   >         byteBuffer.putLong(9);
   >         byteBuffer.putChar('天');
   >         byteBuffer.putShort((short) 4);
   > 
   >         // 切换到取出
   >         byteBuffer.flip();
   > 
   >         System.out.println(byteBuffer.getInt());
   >         System.out.println(byteBuffer.getLong());
   > //        Exception in thread "main" java.nio.BufferUnderflowException
   > //        System.out.println(byteBuffer.getLong());
   >         System.out.println(byteBuffer.getChar());
   >         System.out.println(byteBuffer.getShort());
   >     }
   > }
   > ```

2. 可以把一个普通的eBuffer转换为只读Buffer

   > ```java
   > public class ReadOnlyBuffer {
   > 
   >     public static void main(String[] args) {
   >         ByteBuffer byteBuffer = ByteBuffer.allocate(64);
   >         for (int i = 0; i < 64; i++) {
   >             byteBuffer.put((byte)i);
   >         }
   > 
   >         byteBuffer.flip();
   > 
   >         ByteBuffer readOnlyBuffer = byteBuffer.asReadOnlyBuffer();
   >         System.out.println(readOnlyBuffer.getClass());
   >         //class java.nio.HeapByteBufferR
   > 
   >         //遍历输出
   >         while (readOnlyBuffer.hasRemaining()){
   >             System.out.println(readOnlyBuffer.get());
   >         }
   > 
   >         readOnlyBuffer.put((byte)70);
   >         // Exception in thread "main" java.nio.ReadOnlyBufferException
   >     }
   > }
   > ```

3. NIO还提供了MappedByteBuffer，可以让文件直接在内存（堆外的内存）中进行修改，而如何同步到文件中由NIO来完成。

   > ```java
   > import java.io.RandomAccessFile;
   > import java.nio.MappedByteBuffer;
   > import java.nio.channels.FileChannel;
   > 
   > /**
   >  * NIO还提供了MappedByteBuffer，可以让文件直接在内存（堆外的内存）中进行修改，而如何同步到文件中由NIO来完成。
   >  */
   > public class MappedByteBufferTest {
   > 
   >     public static void main(String[] args) throws Exception {
   >         RandomAccessFile randomAccessFile = new RandomAccessFile("1.txt", "rw");//获取对应的通道
   >         FileChannel channel = randomAccessFile.getChannel();
   >         /**
   >          *参数1: FileChannel.MapMode.READ_WRITE 使用的读写模式
   >          *参数2:0:可以直接修改的起始位置
   >          *参数3:5:是映射到内存的大小(不是索引位置),即将1.txt的多少个字节映射到内存*可以直接修改的范围就是0-5
   >          *实际类型 DirectByteBuffer
   >          */
   >         MappedByteBuffer mappedByteBuffer = channel.map(FileChannel.MapMode.READ_WRITE, 0, 5);
   >         mappedByteBuffer.put(0, (byte) 'H');
   >         mappedByteBuffer.put(3, (byte) '9');
   >         mappedByteBuffer.put(5, (byte) 'Y');//IndexOutOfBoundsException
   >         randomAccessFile.close();
   >         System.out.println("修改成功~");
   >     }
   > }
   > ```

4. NIO还支持通过多个Buffer（即Buffer数组）完成读写操作，即Scattering和Gathering。

   > ```java
   > 
   > import java.io.IOException;
   > import java.net.InetSocketAddress;
   > import java.nio.ByteBuffer;
   > import java.nio.channels.ServerSocketChannel;
   > import java.nio.channels.SocketChannel;
   > import java.util.Arrays;
   > 
   > /**
   >  * NIO还支持通过多个Buffer（即Buffer数组）完成读写操作，即Scattering和Gathering。
   >  *
   >  * @author jack
   >  */
   > public class ScatteringAndGatheringTest {
   >     public static void main(String[] args) throws IOException {
   >         // 使用ServerSocketChannel 和 Socket Channel 网络
   >         ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
   >         InetSocketAddress inetSocketAddress = new InetSocketAddress(7000);
   >         // 绑定到端口
   >         serverSocketChannel.bind(inetSocketAddress);
   >         // 创建buffer数组
   >         ByteBuffer[] byteBuffers = new ByteBuffer[2];
   >         byteBuffers[0] = ByteBuffer.allocate(5);
   >         byteBuffers[1] = ByteBuffer.allocate(3);
   > 
   >         SocketChannel socketChannel = serverSocketChannel.accept();
   >         int messageLength = 8;
   >         while (true) {
   >             int byteRead = 0;
   >             while (byteRead < messageLength) {
   >                 long l = socketChannel.read(byteBuffers);
   >                 byteRead += l;
   >                 System.out.println("byteRead=" + byteRead);
   >                 //使用流打印,看看当前的这个buffer的postion和limit
   >                 Arrays.asList(byteBuffers).stream().map(buffer -> {
   >                     return "postion=" + buffer.position() +
   >                             ", limit=" + buffer.limit();
   >                 }).forEach(System.out::println);
   >             }
   > 
   > 
   >             // buffer 遍历flip
   >             Arrays.asList(byteBuffers).forEach(buffer -> {
   >                 buffer.flip();
   >             });
   >             // 输出
   >             socketChannel.write(byteBuffers);
   >             // 本地清理
   >             Arrays.asList(byteBuffers).forEach(buffer -> {
   > 
   >                 //遍历输出
   >                 while (buffer.hasRemaining()) {
   >                     System.out.println(buffer.get());
   >                 }
   >                 buffer.clear();
   >             });
   > 
   > 
   >             Arrays.asList(byteBuffers).stream().map(buffer -> {
   >                 return "postion=" + buffer.position() +
   >                         ", limit=" + buffer.limit();
   >             }).forEach(System.out::println);
   > 
   >             System.out.println("end now===============");
   >         }
   >     }
   > }
   > ```

### 3.6 选择器（Selector）

#### 3.6.1 基本介绍

* Java的NIO，用于非阻塞的IO方式。可以用一个西芹成，处理多个客户端连接，就会用到Selector。

* Selector能够检测多个注册在通道上是否有事件发生（注意：多个Channel以事件的方式可以注册到同一个Selector），如果有时间发生，便获取时间然后针对每个额事件进行相应的处理。这样就可以用一个单线程管理多个通道，也就是管理多个连接和请求。

  ![image-20201026151128000](http://img.hurenjieee.com/uPic/image-20201026151128000.png)

* 只有在 连接/通道 真正有读写事件发生时，才会进行读写，就大大地减少了系统开销，并且不必为每个连接都创建一个线程，不用去维护多个线程。避免了多线程之间的上下文切换导致的开销。

* 示意图：

  ![image-20201026141554006](http://img.hurenjieee.com/uPic/image-20201026141554006.png)

  > 1. Netty的IO线程NioEventLoop聚合了Selector，并且同时并发处理成百上千个客户端连接
  > 2. 当线程从某客户端Socket通道进行读写数据时，若没有数据可用时，该线程可以进行其他任务。
  > 3. 线程通常讲非阻塞IO的空闲时间用于其他通道上执行IO操作，所以单独的线程可以管理多个输入和输出通道。
  > 4. 由于读写操作都是非阻塞的，这就可以充分提升IO线程的运行效率，避免由于频繁IO阻塞导致线程的挂起。
  > 5. 一个IO线程可以并发处理N个客户端连接和读写操作，这从根本上解决了传统同步阻塞IO一连接一线程模型，架构的性能、弹性伸缩能力和可靠性都得到了极大的提升。

#### 3.6.2 Selector类相关方法

```java


public abstract class Selector implements Closeable {

    protected Selector() { }

    /**
     * 得到一个选择器对象
     */
    public static Selector open() throws IOException {
        return SelectorProvider.provider().openSelector();
    }

    public abstract boolean isOpen();

    public abstract SelectorProvider provider();

    /**
     * 从内部结婚中得到所有的SelectionKey
     */
    public abstract Set<SelectionKey> keys();

    /**
     * 
     */
    public abstract Set<SelectionKey> selectedKeys();

    /**
     * 不阻塞，立即返回
     */
    public abstract int selectNow() throws IOException;

    /**
     * 监控所有注册的通道，当其中有IO操作可以进行时，讲对应的SelectionKey加入到哪步集合中并进行返回，参数timeout(ms)用来设置超时时间
     */
    public abstract int select(long timeout)
        throws IOException;

    /**
     * 阻塞
     */
    public abstract int select() throws IOException;

    /**
     * 唤醒Selector
     */
    public abstract Selector wakeup();

    /**
    * 关闭
    */
    public abstract void close() throws IOException;

}

```

### 3.7 NIO 非阻塞网络编程原理分析图

NIO非阻塞网络编程相关关系图：

* Selector
* SelectionKey
* ServerSocketChannel
* SocketChannel

![image-20201026152413862](http://img.hurenjieee.com/uPic/image-20201026152413862.png)

> 1. 当客户端连接时，会通过ServerSocketChannel得到SocketChannel
> 2. 将SocketChannel注册到Selector上，register(Selector sel,int ops)，一个selector上可以注册多个SocketChannel
> 3. Selector进行监听select方法，返回有事件发生的通道的个数
> 4. 注册后返回一个SelectionKey，会与该Selector关联（集合存储）
> 5. 进一步得到各个有事件发生的SelectionKey
> 6. 再通过SelectionKey反向获取SocketChannel，方法channel()
> 7. 可以痛殴得到的channel，完成业务处理。

### 3.8 NIO 非阻塞网络编程

* 服务端

  ```java
  
  import java.io.IOException;
  import java.net.InetSocketAddress;
  import java.nio.ByteBuffer;
  import java.nio.channels.*;
  import java.util.Iterator;
  import java.util.Set;
  
  /**
   * NIO Server端
   */
  public class NIOServer {
      public static void main(String[] args) throws IOException {
          // 创建ServerSocket -> ServerSocket
          ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
  
          // 得到Selector对象
          Selector selector = Selector.open();
  
          // 绑定监听端口
          serverSocketChannel.socket().bind(new InetSocketAddress(6666));
  
          // 设置为非组设
          serverSocketChannel.configureBlocking(false);
  
          // 把 serverSocketChannel注册到Selector关心事件为 OP_ACCEPT
          serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
  
          // 循环等待客户端连接
          while (true) {
              // 等待1s，如果没有事件发生，循环
              if (selector.select(1000) == 0) {
                  System.out.println("connect 1s,no connection");
                  continue;
              }
  
              // 如果select大于0，可以通过selector.selectedKeys获取时间即可
              Set<SelectionKey> selectionKeys = selector.selectedKeys();
              // 获取selectionKey
              Iterator<SelectionKey> keyIterator = selectionKeys.iterator();
  
              while (keyIterator.hasNext()) {
                  SelectionKey key = keyIterator.next();
                  // 不同类型的事件处理
                  if (key.isAcceptable()) {
                      // OP_ACCEPT，有新客户端连接
                      // 生成SocketChannel
                      SocketChannel socketChannel = serverSocketChannel.accept();
                      System.out.println("connected:" + socketChannel);
                      // 设置为非阻塞
                      socketChannel.configureBlocking(false);
                      // 将SocketChannel注册到selector，关注事件为OP_READ，同时SocketChannel关联一个buffer
                      socketChannel.register(selector, SelectionKey.OP_READ, ByteBuffer.allocate(1024));
                  }
                  if (key.isReadable()) {
                      // OP_READ，读事件
                      // 通过Key反向获取channel
                      SocketChannel channel = (SocketChannel) key.channel();
                      // 获取channel关联的buffer
                      ByteBuffer byteBuffer = (ByteBuffer) key.attachment();
                      int result = channel.read(byteBuffer);
                      if(result == -1){
                          channel.close();
                          key.cancel();
                      }else {
                          System.out.println("get client's msg:" + new String(byteBuffer.array(), 0, byteBuffer.limit()));
                      }
                  }
                  // 因为已经处理过事件了，移除，避免后面重复处理
                  keyIterator.remove();
              }
  
          }
      }
  }
  ```

* 客户端

  ```java
  import java.io.IOException;
  import java.net.InetSocketAddress;
  import java.nio.ByteBuffer;
  import java.nio.channels.SocketChannel;
  
  /**
   * NIO 客户端
   */
  public class NIOClient {
  
      public static void main(String[] args) throws IOException {
          SocketChannel socketChannel = SocketChannel.open();
          socketChannel.configureBlocking(false);
          InetSocketAddress inetSocketAddress = new InetSocketAddress("127.0.0.1",6666);
          if(!socketChannel.connect(inetSocketAddress)){
              while (!socketChannel.finishConnect()){
                  System.out.println("因为连接需要时间，客户端不会阻塞，可以做其它工作.");
              }
          }
  
          String str = "hello world";
          ByteBuffer byteBuffer = ByteBuffer.wrap(str.getBytes());
          socketChannel.write(byteBuffer);
          System.in.read();
      }
  }
  ```

### 3.9 SelectionKey

* SelectionKey表示Selector和网路通道的注册管理

  ```java
  		// 读操作
  		public static final int OP_READ = 1 << 0;
  		// 写操作
      public static final int OP_WRITE = 1 << 2;
  		// 连接已经建立
      public static final int OP_CONNECT = 1 << 3;
  		// 有新的网络连接可以accept
      public static final int OP_ACCEPT = 1 << 4;
  ```

* SelectionKey常用的方法

  ![image-20201027114033252](http://img.hurenjieee.com/uPic/image-20201027114033252.png)

### 3.10 ServerSocketChannel

* SeverSocketChannel在服务器监听新的客户端Socket连接

* ServerSocketChannel常用的方法

  ![image-20201027115026640](http://img.hurenjieee.com/uPic/image-20201027115026640.png)

###  3.11 SocketChannel

* Socketchannel，网络IO通道，具体负责读写操作。NIO把缓冲区的数据写入通道，或者把通道里的数据读到缓冲区。

* SocketChannel常用的方法

  ![image-20201027115228441](http://img.hurenjieee.com/uPic/image-20201027115228441.png)

### 3.12 群聊系统

Server

```java

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.*;
import java.util.Iterator;

public class GroupChatServer {

    private Selector selector;
    private ServerSocketChannel serverSocketChannel;
    private static final int PORT = 6667;


    public GroupChatServer() {
        try {
            // 获取选择器
            selector = Selector.open();
            // ServerSocketChannel
            serverSocketChannel = ServerSocketChannel.open();
            // 绑定端口
            serverSocketChannel.socket().bind(new InetSocketAddress(PORT));
            // 设置为非堵塞模式
            serverSocketChannel.configureBlocking(false);
            // 注册到selector上
            serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }


    //监听
    public void listen() {
        try {
            // 循环处理
            while (true) {
                int count = selector.select(2000);
                if (count > 0) {
                    // 获取集合
                    Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
                    while (iterator.hasNext()) {
                        // 获取SelectionKey
                        SelectionKey key = iterator.next();
                        if (key.isAcceptable()) {
                            SocketChannel sc = serverSocketChannel.accept();
                            sc.configureBlocking(false);
                            sc.register(selector, SelectionKey.OP_READ);
                            System.out.println(sc.getRemoteAddress() + "上线");
                            sendOtherClient(sc.getRemoteAddress() + "上线", sc);
                        }
                        if (key.isReadable()) {
                            readData(key);
                        }
                        // 别忘了删除事件
                        iterator.remove();
                    }
                } else {
                    System.out.println("等待中。。。。");
                }


            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {

        }
    }

    // 读取客户端消息
    private void readData(SelectionKey key) {
        SocketChannel channel = null;
        try {
            channel = (SocketChannel) key.channel();
            ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
            int read = channel.read(byteBuffer);
            if (read > 0) {
                String string = new String(byteBuffer.array());
                System.out.println("from 客户端" + string.trim());
                // 向其他客户端发送消息
                sendOtherClient(string, channel);
            }
        } catch (Exception e) {
            e.printStackTrace();
            try {
                System.out.println(channel.getRemoteAddress()+"离线了");
                // 取消注册
                key.cancel();
                channel.close();
            } catch (IOException ex) {
                ex.printStackTrace();
            }
        }

    }

    // 转发消息到其他客户端
    private void sendOtherClient(String msg, SocketChannel self) throws IOException {
        // 获取到所有的连接
        for (SelectionKey key : selector.keys()) {
            SelectableChannel channel = key.channel();
            // 排除自己
            if (channel instanceof SocketChannel &&
                    channel != self) {
                SocketChannel socketChannel = (SocketChannel) channel;
                ByteBuffer wrap = ByteBuffer.wrap(msg.getBytes());
                socketChannel.write(wrap);

            }
        }
    }

    public static void main(String[] args) {
        GroupChatServer groupChatServer = new GroupChatServer();
        groupChatServer.listen();

    }
}
```

Client

```java


import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.*;
import java.util.Iterator;
import java.util.Scanner;
import java.util.Set;

public class GroupChatClient {

    private static final String HOST = "127.0.0.1";
    private static final int PORT = 6667;
    private Selector selector;
    private SocketChannel socketChannel;
    private String username;

    public GroupChatClient() {
        try {
            // 获取选择器
            selector = Selector.open();
            // ServerSocketChannel
            socketChannel = SocketChannel.open(new InetSocketAddress(HOST, PORT));
            // 设置为非堵塞模式
            socketChannel.configureBlocking(false);
            // 注册到selector上
            this.socketChannel.register(selector, SelectionKey.OP_READ);
            username = socketChannel.getLocalAddress().toString().substring(1);
            // 客户端连接
            System.out.println(socketChannel.getLocalAddress()+"客户端连接成功");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public void sendInfo(String info){
        try {
            info = username + "说："+ info;
            socketChannel.write(ByteBuffer.wrap(info.getBytes()));
        }catch (Exception e){
            e.printStackTrace();
        }
    }

    public void readInfo(){
        try{
            int select = selector.select();
            if(select > 0){
                Set<SelectionKey> selectionKeys = selector.selectedKeys();
                Iterator<SelectionKey> iterator = selectionKeys.iterator();
                while (iterator.hasNext()){
                    SelectionKey key = iterator.next();
                    if(key.isAcceptable()){

                    }
                    if(key.isReadable()){
                        SocketChannel channel = (SocketChannel) key.channel();
                        ByteBuffer allocate = ByteBuffer.allocate(1024);
                        channel.read(allocate);
                        String string = new String(allocate.array());
                        System.out.println(string.trim());
                    }
                    iterator.remove();
                }
            }
        }catch (Exception e){
            e.printStackTrace();
        }
    }


    public static void main(String[] args) {
        GroupChatClient groupChatClient = new GroupChatClient();
        new Thread(){
            @Override
            public void run() {
                while (true) {
                    groupChatClient.readInfo();
                    try {
                        sleep(3000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }.start();;

        Scanner scanner = new Scanner(System.in);

        while (scanner.hasNextLine()){
            String s  = scanner.nextLine();
            groupChatClient.sendInfo(s);
        }
    }
}
```

## 四 、AIO

### 4.1 Java AIO 基本介绍

1. JDK 7引入了 Asynchronous IO，即AIO。在进行IO编程中，常用到的两种模式：Reactor和PRoactor。Java 的 NIO就是Reactor，当有事件触发时，服务器端得到通知，进行相应的处理。
2. AIO即NIO2.0，叫异步不阻塞的IO。AIO引入异步通道的概念，采用了Proactor模式，简化了程序编写，有效的请求才启动线程，它的特点是先由操作系统完成之后才通知服务端启动线程去处理，一般适用于连接数较多且连接事件较长的应用。
3. AIO介绍：http://www.52im.net/thread-306-1-1.html

### 4.2 BIO、NIO、AIO对比

|          | BIO      | NIO                    | AIO        |
| -------- | -------- | ---------------------- | ---------- |
| IO模型   | 同步阻塞 | 同步非阻塞（多路复用） | 异步非阻塞 |
| 编程难度 | 简单     | 复杂                   | 复杂       |
| 可靠性   | 差       | 好                     | 好         |
| 吞吐量   | 低       | 高                     | 高         |

* **同步阻塞**：到理发店理发，就一直等理发师，知道轮到自己理发。
* **同步非阻塞**：到理发店理发，发现前面有其他人理发，给理发师说下，先干其他事情，一会过来再看看是否轮到自己。
* **异步非阻塞**：给理发师打电话，让理发师上门服务，自己干其他事情，理发师自己来家给你理发。