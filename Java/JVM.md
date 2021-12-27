[TOC]

## 一、基本介绍

* JVM 是JAVA Virtual Machine（Java虚拟机）的缩写，JVM是一种用于计算设备的规范，他是一个虚拟出来的计算机，是通过在实际的计算机上仿真模拟各种计算功能来实现的。

* JVM是直接与操作系统进行交互的，与操作系统交互的结构图如下。

  ![image-20200823144002845](http://img.hurenjieee.com/uPic/image-20200823144002845.png)

## 二、一个Java文件的生命周期

![image-20200823144205787](http://img.hurenjieee.com/uPic/image-20200823144205787.png)

1. Java文件经过编译之后变成class字节码文件。
2. 字节码文件通过类加载器被搬运到JVM虚拟机当中来。
3. 虚拟机当中主要有五大块：
   <font color=purple>1. 方法区：线程共享区域，存在多线程安全问题，主要存放全局变量、常量，静态变量等。</font>
   <font color=purple>2. 堆：线程共享区域，存在多线程安全问题，主要存放对象实例和数组等。</font>
   <font color=green>3. 虚拟机栈：线程不共享区域，不存在多线程安全问题，主要存放局部变量等。</font>
   <font color=green>4. 本地方法栈：线程不共享区域，不存在多线程安全问题，主要区负责调用一些底层的C语言实现。</font>
   <font color=green>5. 程序计数器：指针，指向运行的地址</font>

### 2.1 程序运行示例

```java
// Animal.java
public class Animal {
    public String name;
    public Animal(String name) {
        this.name = name;
    }
    public void printName() {
        System.out.println("Animal ["+name+"]");
    }
}
// MainApp.java
public class MainApp {
    public static void main(String[] args) {
        Animal animal = new Animal("Puppy");
        animal.printName();
    }
}

```

1. 在编译好Java程序得到Main.class文件后，执行MainApp。
   * 系统就会启动一个JVM进程，JVM进程从classpath路径中找到一个MainApp.class的二进制文件，将MainApp的类信息加载到运行时数据区的方法区内，这个过程叫做MainApp类的加载。
2. JVM找到AppMain的主函数入口，开始执行main函数。
3. main函数执行的第一条命令是Animal animal = new Aninal("Puppy")。
   * 让JVM创建一个Animal对象，但是这个时候没有Animal类的信息。所以JVM马上加载Animal类，把Animal类的类型信息放到方法区中。
4. 加载完Animal类之后，Java虚拟机的第一件事情就是在堆区中为一个新的Animal实例分配内存，然后调用构造函数初始化Animal实例，这个Animal实例持有只想方法区的Animal类的类型信息（其中包含方法表，java动态绑定的底层实现）的 引用。
5. 当使用animal.printName()的时候，JVM根据animal引用找到Animal对象，然后更具Animal对象持有的引用定位到方法区中Animal类的类型信息的方法表，获取printName()函数的字节码的地址。
6. 开始运行printName()函数。

## 三、类加载器

![image-20200823153116097](http://img.hurenjieee.com/uPic/image-20200823153116097.png)

* 类加载器ClassLoader负责加载class文件，class文件在文件开头有特定的文件标示，将class文件字节码内容加载到内存中，并将这些内容转换成方法区中的运行时数据结构。ClassLoader只负责class文件的额加载，至于它是否可以运行，则有Execution Engine决定。

### 3.1 类加载的过程

* 从类被加载到虚拟机内存开始，到卸载出内存为止。他的整个生命周期分为7个阶段：
  1. 加载（Loading）
  2. 链接（可细分为以下三步）
     1. 验证（Verification）	
     2. 准备（Preparation）
     3. 解析（Resolution）
  3. 初始化（Initialization）
  4. 使用（Using）
  5. 卸载（Unloading）

![image-20200823155848238](http://img.hurenjieee.com/uPic/image-20200823155848238.png)

#### 3.1.1 加载

1. 将class文件加载到内存中。

2. 将静态数据结构（数据存在于class文件的结构）转化成方法区中运行时的数据结构。

   注意：方法区如果出现OOM，那么多半是因为加载的依赖太多

3. 在堆中生成一个代表这个类的java.lang.Class对象，作为数据访问的入口。

#### 3.1.2 链接

```java
private static int a = 3 ; 
a = 0 ; 
```

1. 验证：
   * 确保加载的类符合JVM规范和安全。保证被校验类的方法在运行时不会做出危害虚拟机安全的时间。
2. 准备：
   * 为static变量在方法区中申请空间，设置变量的初始值。例如，static int a = 3，在此阶段a会被初始化为0。
3. 解析：
   * 解析阶段是虚拟机将敞亮吃哪的符号引用替换为直接引用的过程。
     * 符号引用：简单的理解就是字符串，比如引用一个类，java.util.ArrayList这就是一个符号引用。
     * 直接引用：指针或者地址偏移量，引用对象一定在内存中（已经加载）。

#### 3.1.3 初始化

* 初始化时类加载的最后阶段，初始化阶段时执行类构造器<**clinit**\>()方法。在类构造器方法中，它将**由编译器自动收集类中的所有类变量的赋值动作（准备阶段的a正式赋值为3）和静态变量与静态语句块static{}合并初始化，为类的静态变量赋于正确的初始值。**

#### 3.1.4 使用

* 正常使用

#### 3.1.5 卸载

* GC把无用的对象从内存中卸载。

### 3.2 类加载的加载顺序

* 加载一个类进来之后，需要进行加载、链接、初始化、使用、卸载等一系列步骤。

* 那么同样的，加载一个class类的顺序，也是有优先级的。需要先加载我门rt.jar这个jar包下面的class类之后，我们磁能使用如下的代码：

  ```java
  HashMap<String,String> hashMap = new HashMap<String,String>;
  ```

* 加载顺序图：

  ![image-20200823163812571](http://img.hurenjieee.com/uPic/image-20200823163812571.png)

  1. **Bootstrap ClassLoader**
     * 负责加载$JAVA_HOME中jre/lib/rt.jar里面所有的class，由C++实现，不是ClassLoader的子类。
  2. **Extension ClassLoader**
     * 负责加载Java平台中扩展功能的一些jar包，包括$JAVA_HOME中jire/lib/*.jar 或者 -Djava.ext.dirs指定目录下的jar包。
  3. **App ClassLoader**
     * 负责加载classpath中指定的jar包及目录中的class
  4. **Custom ClassLoader**
     * 属于应用程序根据自身需要自定义的ClassLoader，如Tomcat、jboss都会根据J2EE规范自定义ClassLoader。

加载过程中会先检查类是否被已加载。

* 检查的顺序时自底向上的，从Custom ClassLoader到Bootstrap ClassLoader逐层检查，只要某个ClassLoader已加载就视为已记载此类，保证此类在所有ClassLoader中只加载一次。
* 加载的顺序时自顶向下的，也就时由上层来逐层加载此类。

![image-20200823170845122](http://img.hurenjieee.com/uPic/image-20200823170845122.png)

* 验证类加载机制

  ```java
  public class ClassLoaderTest {
      public static void main(String[] args) {
          ClassLoader loader = Thread.currentThread().getContextClassLoader();
          System.out.println(loader);
          System.out.println(loader.getParent());
          System.out.println(loader.getParent().getParent());
      }
  }
  // sun.misc.Launcher$AppClassLoader@18b4aac2
  // sun.misc.Launcher$ExtClassLoader@5451c3a8
  // null
  // 在获取ExtClassLoader的父loader的时候出现了null，这是因为Bootstrap Loader（引导类加载器）是用C++语言实现的，找不到一个确定的返回父Loader的方式，于是就返回null
  ```

### 3.3 双亲委派机制

* 原则：**层层向上找，先找到先使用，后面的一概不见。**

* 当一个类收到一个类加载请求时，它首先不会尝试自己去加载这个类，而是把这个请求委派给父类完成，每一个层次类加载器都是如此，因此所有的加载请求都应该传送到启动类加载其中，只有当父类加载器反馈自己无法完成这个请求的时候（在它的夹杂路径下没有找到所需加载的Class），子类加载起才会尝试自己去加载。

* 采用双亲委派的一个好处是必入加载位于rt.jar包中的类java.lang.Object，不管是哪个类加载器加载这个类，最终都是委托给顶层的启动类加载器进行加载，这样就保证了使用不同的类加载器最终得到的都是同一个Object对象。

* 验证双亲委派机制：

  ```java
  package java.lang;
  
  public class String {
      public static void main(String[] args) {
          System.out.println("hello world");
      }
  }
  // 运行报错
  // 错误: 在类 java.lang.String 中找不到 main 方法, 请将 main 方法定义为:
  // public static void main(String[] args)
  // 否则 JavaFX 应用程序类必须扩展javafx.application.Application
  ```
  
  为了保证我们开发的代码不会java当中自带的源代码，保证我们使用的class类都是最终的一个，所以才会有双亲委派模型这种机制，提供一种沙箱环境来保证我们加载的class类都是同一个。

## 四、运行时的数据区域

### 4.1 本地方法栈 Native Method Stack

![image-20200823234831595](http://img.hurenjieee.com/uPic/image-20200823234831595.png)

* 本地方法栈（native Method Stack）与虚拟机锁发挥的作用是非常相似的，其区别不过是虚拟机栈为虚拟机执行Java方法（也就时是字节码）服务，而本地方法栈则是为虚拟机使用到的Native方法服务（比如C语言写的程序和C++写的程序）

### 4.2 本地接口

* 本地接口的作用是融合不同的编程语言为Java所用，他的初衷是融合C/C++程序，Java诞生的时候是C/C++横行的时候，想要立足，必须有调用C/C++程序，于是就在内存中专门开辟了一块区域处理标记为native的代码，他的具体做法是Native Method Stack中登记native方法，在Execution Engine执行时加载native libraries

### 4.3 程序计数器/PC寄存器

![image-20200823234842864](http://img.hurenjieee.com/uPic/image-20200823234842864.png)

* PC寄存器就是一个指针，指向我们下一个需要运行的方法
* 程序计数器是一块非常小的内存空间，主要是用来对当前线程所执行的字节码的行号指示器；
* 而且程序计数器是内存区域中唯一一块不存在OOM的区域，每个线程都有一个程序计数器，是线程私有的，就是一个指针，指向方法区中的方法字节码（用来存储指向下一条指令的地址，也就是将要执行的指令代码），由执行引擎读取吓一跳指令，是一个非常小的内存空间，几乎可以忽略不记。
* 这块内存区域很小，它是当前线程所执行的字节码的行号指示器，字节码解释器通过改变这个计数器的值来选取下一条需要执行的字节码指令。
* 如果执行的是一个Native方法，那么这个计数器是空的。
* 用以完成完成分支、循环、跳转、异常处理、线程恢复等基础功能，不会发生内存溢出错误。

### 4.4 方法区

![image-20200823234854942](http://img.hurenjieee.com/uPic/image-20200823234854942.png)

* 方法去和Java堆一样，是线程共享等区域；它存储了每一个类的结构信息。
* 方法区的作用就是用来存储：已经被虚拟机加载的**类信息、常量、静态变量等**；
* 方法区也有另外的叫法：非堆，或者是永久代。

### 4.5 Java虚拟机栈

* **栈管运行，堆管存储。**

![image-20200824195252452](http://img.hurenjieee.com/uPic/image-20200824195252452.png)

#### 4.5.1 虚拟机栈的基本介绍

* 我们经常说的额“堆栈”，其中的栈就是虚拟机栈，更确切的说，大家谈的栈你是虚拟机中的局部变量表部分。
* 虚拟机栈描述的是：**Java方法执行的内存模型**；（说白了就是：虚拟机栈就是用来存储：**局部变量、栈操作、动态链表、方法出口**这些东西；这些东西有个特点：都是线程私有的，**所有的虚拟机栈是线程私有的**）
* 虚拟机可能出现异常有两种：
  * ==StackOverflowError==
    * 递归操作中，线程请求的栈深度大于虚拟机栈允许的最大深度，就会报错。
  * ==OutOfMermoryError==
    * java的虚拟机栈可以动态扩展，但随着扩展会不断的申请内存，当无法申请足够的内存时，就会报错

#### 4.5.2 虚拟机栈的生命周期

* 对于栈来说，不存在垃圾回收问题，只要程序执行结束，栈就over释放，生命周期和线程一致，是私有线程区域。
* **8种基本类型的变量+对象的应用变量+实例方法都是在函数的栈内存中分配。**

#### 4.5.3 虚拟机栈当中存放了什么数据

* ==局部变量==（Local Variables）：输入参数和输出参数以及方法内的变量。八种基本数据类型+引用类型（String，以及自己定义的class类）
* ==栈操作==（Operand Stack）：记录出栈、入栈的操作。
* ==栈帧数据==（Frame Data）：包含类文件、方法等等。

#### 4.5.4 虚拟机栈运行原理

* 栈中的数据都是以栈帧（Stack Frame）的格式存在，栈帧是一个内存区块，是一个数据集，是一个有关方法（Method）和运行期数据的数据集。

* 遵循“先进后出”/“后进先出”原则。

* 每个方法执行的同时都会创建一个栈帧，用于存储局部变量白哦、操作数栈、动态链表、方法出口等信息，每一个方法从调用直至执行完毕的过程，就对应着一个栈帧在虚拟机中入栈和出栈的过程。

* 栈的大小和具体JVM的实现有关，通常在256K-756K之间，约等于1Mb左右。

  ```java
  public class StackIn {
      //程序入口方法
      public static void main(String[] args) {
          StackIn stackIn = new StackIn();
          //调用A 方法，产生了栈帧 F1
          stackIn.A();
      }
      //最后弹出F1
      public void A(){
          System.out.println("A");
          // F1栈帧里面调用B方法，产生栈帧F2
          B();
      }
  
      //然后弹出F2
      public void B(){
          //F2栈帧里面调用C方法，产生栈帧F3
          System.out.println("B");
          C();
      }
  
      //栈帧F3  执行完成之后，先弹出F3
      public void C(){
          System.out.println("C");
      }
  }
  ```

* ![image-20200824210232216](http://img.hurenjieee.com/uPic/image-20200824210232216.png)

* 当一个方法A被调用时产生一个栈帧F1，并被压入到栈中。

  A方法又调用B方法，于是产生栈帧F2也被压入栈，

  B方法又调用C方法，于是产生栈帧F3也被压入栈，

  ……

  执行完毕后，先弹出F3栈帧，再弹出F2栈帧，再弹出F1栈帧……

#### 4.5.5 局部变量复用slot

* **局部变量表用于存放方法参数和方法内部定义的局部变量。**方法表的Code属性：max_locals数据项指明了该方法所需要分配的局部变量表的最大容量。

* 局部变量表的容量以Slot（Variable Slot：变量槽）为最小单位，其中64位长度的long和double类型的数据占用2个Slot，其余数据类型（boolen、byte、char、short、int、float、reference、retrunAddress）占用一个Slot（一个Slot可以存放32位以内的数据类型）。

* 虚拟机通过索引定位的方式使用局部变量表，索引值的范围是从0到局部变量表最大的Slot数量。32位数据类型的变量，索引n就是代表使用第n个Slot，如果是64位数量类型的变量，则说明会同时使用n和n+1两个Slot。

* 在方法执行时，虚拟机是使用局部变量表完成参数值到参数变量的传递过程。如果是实例方法（非static的方法），那么局部变量表中d第0位的Solt默认是用于传递方法所属实例对象的引用，在方法啊中可以通过this关键字来访问这个隐含的参数，其余参数则按照参数表顺序排列，占用从索引1开始的局部变量Slot。参数列表分配完毕后，再根据方法体内部定义的变量和作用与分配其余的Slot。

* 为了节省栈帧空间，局部变量表中的Slot是可以复用的，当方法执行位置（程序计数器在字节码的值）已经超过了某个变量，那么这个变量的Slot可以被其他变量复用。除了能节省栈帧空间，还伴随着可能会影响到系统垃圾收集的行为

  ```java
  public class StackGc {
      // 示例一  垃圾没有被回收
      public static void main(String[] args) {
          // 向内存填充64M的数据
          byte[] placeholder = new byte[64 * 1024 * 1024];
          System.gc();
      }
  
      // 示例二  垃圾没有被回收
     /*public static void main(String[] args) {
          // 向内存填充64M的数据
          {
              byte[] placeholder = new byte[64 * 1024 * 1024];
          }
          System.gc();
      }*/
  
      // 示例三  垃圾被回收
     /* public static void main(String[] args) {
          // 向内存填充64M的数据
          {
              byte[] placeholder = new byte[64 * 1024 * 1024];
          }
          int a = 0;
          System.gc();
      }*/
  }
  
  
  // 在IDEA当中执行以上代码，并加上参数 -verbose:gc  来查看我们的GC回收信息，我们会发现第一个方法没有GC回收，第二个方法没有垃圾回收，第三个方法GC回收了
  // 能否被回收的关键因素就是变量是否还有被引用，在第三个示例当中int a = 0;这个变量没有被其他引用，导致垃圾回收工作，回收垃圾
  ```

  #### 4.5.6 方法的参数对调用次数的影响

  * 如果一个方法有参数列表，一个方法没有参数列表。

    ```java
    public class MethodParameter {
        private static int count = 0;
    
        public static void recursion(int a, int b, int c) {
            System.out.println("我是有参数的方法，调用第" + count+"次");
            long l1 = 12;
            short sl = 1;
            byte b1 = 1;
            String s = "1";
            count++;
            recursion(1, 2, 3);
        }
        public static void recursion() {
            System.out.println("我是没有参数的方法，调用第" + count+"次");
            count++;
            recursion();
        }
    
        public static void main(String[] args) {
           // recursion();
            recursion(1, 2, 3);
        }
    }
    ```

    * 运行不同的重载方法，我门会明显的发现，没有参数的方法，调用次数会更多。因为我门的参数就是局部变量，局部变量的装载也需要一定的内存空间，我门可以通过参数 -Xss160K 来调整我们每个线程的堆栈大小，但是无论如何调整栈空间大小，我们依然会发现没有参数且没有局部变量的方法调用次数会远远比有参数有局部变量的方法调用的次数多得多。栈的大小和具体的JVM实现有关，通常在256K-756K之间，约等于1MB左右。

### 4.6 Java虚拟机堆

![image-20200824221412058](http://img.hurenjieee.com/uPic/image-20200824221412058.png)



#### 4.6.1 JVM 虚拟机堆的基本组成介绍

![image-20200824231138271](http://img.hurenjieee.com/uPic/image-20200824231138271.png)

1. JVM内存划分为堆内存和非堆内存，堆内存分为年轻代（Young Generation）、老年代（Old Generation），非堆内存就只有一个永久带（Permanent Generation）。
2. 年轻代又分为Eden和Suvivor区。Survivor区由FromSpace和ToSpace组成。Eden区栈大容量，Survivor两个区占小容量，默认比例是8:1:1。
3. 堆内存用途：存放的是对象，垃圾收集器就是收集这些对象，然后根据GC算法回收。
4. 非堆内存用途：永久代，也称为方法区，存储程序运行时长期存活的对象，比如类的元数据、方法、常量、属性等。

* 在JDK1.8版本中废弃了永久代，替代的是原空间（MetaSpace），元空间和永久代类似，都是方法区的实现，他们最大的区别是：元空间并不在JVM中，而是使用本地内存。元空间有两个参数：
  * MetaspaceSize：初始化元空间大小，控制发生GC阈值。
  * MaxMetaspaceSize：限制元空间大小上限，防止异常占用过多物理内存。

#### 4.6.2 JDK1.8为什么要移除永久代

* 在JDK1.8当中已经不存在永久代了，取而代之的是元数据区。元数据区与永久代的功能类似，都是用于存放一些不会被回收的对象，例如我们的数据库连接池这样的对象等，只不过在1.8中元数据最大的特性就是使用了堆外内存，也就是操作系统级别的物理内存，数据直接保存到了物理内存当中去了。
* 移除永久代的原因：为融合HotSpot JVM与JRockit JVM（新JVM技术）而做出的改变，因为JRockit没有永久代。
* **有了元空间之后就不会出现OOM了。**

#### 4.6.3 新生代介绍

![image-20200824235126218](http://img.hurenjieee.com/uPic/image-20200824235126218.png)

* 新生成的对象首先放到年轻代Eden区，当Eden区空间满了，触发Minor GC，存活下来的对象移动到Survivor0区。Suvivor0区满了后出发执行Minor GC，Suvivor0区存活对象移动到Suvivor1区，遮掩个保证了一段时间内总有一个Suvivor区为空。经过多次Minor GC仍然存活的对象移动到老年代。
* 老年代存储长期存活的对象，占满时会出发Major GC(Full GC)，GC期间会停止所有线程等待GC完成，进行老年代的内存清理。所以对响应要求高的应用尽量减少Major GC(Full GC)，避免响应超时。若养老去执行了Full GC之后发现依然无法进行对象的保存，就会产生OOM。如果出现java.lang.OutOfMemoryerror:Java head space异常，说明Java虚拟机的堆内存不够，原因有二：
  * Java虚拟机的堆内存设置不够，可以通过参数-Xms、-Xmx来调整。
  * 代码中创建了大量大对象，并且长时间不能被垃圾收集器收集（存在被引用）。
* **Minnor GC：清理年轻代**
* **Major GC：清理老年代**
* **Full GC：清理整个堆空间，包括年轻代和永久代。**
* **所有GC都会停止应用所有线程。**

#### 4.6.4 如何判断那些对象已经死亡可以被回收

![image-20200825201633322](http://img.hurenjieee.com/uPic/image-20200825201633322.png)

* 介绍了Java内存运行时数据区的各个部分，其中程序计数器、虚拟机栈、本地方法栈，3个区域随着线程的生存而生存的，内存的分配和回收都是确定的，随着线程结束，内存自然就被回收了，因此不需要考虑到垃圾回收的问题。
* 而Java堆和方法区则不一样，各线程共享，内存的分配和回收都是动态的。因此垃圾收集器所关注的都是堆和方法区这部分内存。

##### 4.6.4.1 引用计数法

* 给对象添加一个引用计数器，每当一有一个地方引用它时计数器+1，当引用失效时计数器-1，只要计数器等于0的对象就是不可能在被使用的。
* 此算法在大部分的情况下时一个不错的选择，也有一些著名的应用案例。但是在Java虚拟机中时没有使用的。
* 优点：实现简单，判断效率高
* 缺点：无法解决对象之间的循环引用。

##### 4.6.4.2 可达性分析计算方法

* 通过一系列的成为“GC Roots”的对象作为起始点，从这些节点开始向下搜索，搜索所走过的路径称为引用链，当一个对象到GC Roots没有使用任何引用链时，则说明该对象是不可用的。

* 主流的商用语言（Java、C#等）在主流实现中，都是通过可达性分析来判断对象是否存活的。

* 通过下图就可以看到GC Roots和对象之间的联系，Object1/2/3/4区域的对象是存活的，Object5/6/7均是可回收的对象。

  ![image-20200825203416958](http://img.hurenjieee.com/uPic/image-20200825203416958.png)

* 在Java语言中，可作为GC Roots的对象包含下面几种

  * **虚拟机栈（栈帧中的本地变量表）中引用的对象**
  * **方法区静态变量应用的对象**
  * **方法区中常量应用的对象**
  * **本地方法栈（即一般说的Native方法）中JNI引用的对象**

* 优点：更加精确和严谨，可以分析出循环数据结构的相互引用的情况。

* 缺点：实现更加复杂、需要分析大量数据，小号大量时间、分析过程需要GC停顿（引用关系不能发生变化），即停顿所有的Java执行线程（称为“Stop The World”，是垃圾回收重点关注的问题）。

#### 4.6.5 如何确切的宣告一个对象的死亡

宣告一个对象死亡，至少要经历两次标记

##### 4.6.5.1 第一次标记

* 如果一个对象进行可达性分析算法之后没发与GC Roots相连的引用链，那它将会第一次标记并且进行一次筛选。
* 筛选条件：判断此对象是否有必要执行finalize()方法。
* 筛选结果：当对象没有覆盖finalize()方法、或者finalize()方法已经被JVM执行过，则判定对象为可回收对象。如果对象没有必要执行finalize()方法，则被方法F-Queue队列中。稍后在JVM自动建立、低优先级的Finalizer线程（可能多个线程）中触发这个方法。

##### 4.6.5.2 第二次标记

* GC对F-Queue队列中的对象进行二次标记。
* 如果对象在finalize()方法中重新与引用链上的任何一个对象建立了关联，那么二次标记时会将它移出“即将回收”集合。如果此时对象还没有成功逃脱，那么只能被回收了。

##### 4.6.5.3 finalize()方法

* finalize()是Object类的一个方法，一个对象的finalize()方法只会被系统自动调用一次，经过finalize()方法逃脱死亡的对象，第二次不会再调用。
* 特别说明：并不提倡在程序中调用finalize()进行自救。因为它的执行时间不确定，甚至是否被执行也不确定（Java程序的不正常退出），而且运行代码高昂，无法保证哥哥对象的调用顺序（甚至有不同线程中的调用）。

#### 4.6.6 垃圾回收算法

* 垃圾收集算法主要涉及有以下算法：
  * 标记-删除算法
  * 复制算法
  * 标记-整理算法
  * 分代收集算法

##### 4.6.6.1 标记-清除算法

* 最基础的收集算法就是“标记-清除”（Mark-Sweep）算法，如同它的名字一样，算法分为“标记”和“清除”两个阶段：首先标记出所有需要回收的对象，在比较完成后统一回收被标记的对象。之所以说他是最基础的手机算法，是因为后续的收集算法都是基于这个思路并对其不足进行改进而得到。
* 不足点：
  * 标记和清除两个过程的效率都不高
  * 空间问题，标记清除之后会产生大量不连续的内存碎片，空间碎片太多可能导致以后在程序运行中需要分配较大对象时，无法找到足够的连续内存空间而不得不提前触发另一次垃圾手机动作。
* ![image-20200825210727653](http://img.hurenjieee.com/uPic/image-20200825210727653.png)

##### 4.6.6.2 复制算法

* 为了解决效率问题，一种称为“复制”的收集算法出现了，它将可用内存按容量划分为大小相等的两块，每次只使用其中的一块。当这一块的内存用完了，就将还存活的对象复制到另外一块上面，然后再把已使用的内存空间一次清理掉。这样是的每次都是对整个半区进行内存回收，内存分配时也就不用考虑内存随便的额复杂情况，只要移动堆顶指针，按顺序分配内存即可，实现简单，运行高效，只是这种算法的代价是将内存原来的一半，浪费较多
* ![image-20200825211308203](http://img.hurenjieee.com/uPic/image-20200825211308203.png)

* 现在的商业虚拟机都采用这种收集算法来回收新生代，IBM公司的专门研究表明，新生代中的对象98%是“朝生夕灭”的，所以并不需要按照 1:1 的比例来划分内存空间，而是将内存分为一块较大的Eden空间和两块较小的Survivor空间，每次使用Eden和其中一块Survivor。当回收时，将Eden和Survivor中还存活着的对象一次性地复制到另外一块Survivor空间上，最后清理掉Eden和刚才用过的Survivor空间。HotSpot虚拟机默认Eden和Survivor的大小比例是8:1，也就是每次新生代中可用内存空间为整个新生代容量的90%（80%+10%），只有10%的内存会被“浪费”。当然，98%的对象可回收只是一般场景下的数据，我们没有办法保证每次回收都只有不多于10%的对象存活，当Survivor空间不够用时，需要依赖其他内存（这里指老年代）进行分配担保（Handle Promotion）。

##### 4.6.6.3 标记-整理算法

* 复制手机算法在对象存活率较高时就要进行较多复制操作，效率将会变低。更关键的是，如果不想浪费50%的空间，就需要额外的空间进行分摊担保，以应对被使用的内存中所有对象都是100%存储哦的极端情况。所以在老年代一半不能直接使用这种算法。
* 根据老年代的特点，有人提出了另外一种“标记-整理”（Mark-Compact）算法，标记过程仍然与“标记-清除”算法一样，但是后续步骤不是直接对回收对象清醒清理，而是让所有存活的对象都想一端移动，然后直接清理掉边界以外的内存。
* ![image-20200825212101412](http://img.hurenjieee.com/uPic/image-20200825212101412.png)

##### 4.6.6.4 分代收集算法

* 当前商业虚拟机的垃圾收集都采用“分代收集”（Generational Collection）算法，这种算法并没有什么新的思想，只是根据对象存储周期不同将内存划分为几块。
* 一半是吧Java堆分为新生代和老年代，这样就可以根据各个年代特点进采用适当的收集算法。
* 在年轻代中，每次垃圾收集都发现大批对象死去，只有少量存货，那就选用复制算法，主要付出少量存货兑现过的复制成本就可以完成收集。
* 在老年代中，因为对象存活率高、没有额外空间对它进行分配担保，就必须使用“标记-清理”或者“标记-整理”算法来进行回收。

### 4.7 垃圾回收器介绍

垃圾回收器是垃圾回收算法的实现。

| 收集器            | 串行、并行 or 并发 | 新生代/老年代 | 算法               | 目标             | 适用场景                                |
| ----------------- | ------------------ | ------------- | ------------------ | ---------------- | --------------------------------------- |
| Serial            | 串行               | 新生代        | 复制算法           | 响应速度优先     | 单CPU坏境下的Client模式                 |
| Serial Old        | 并行               | 老年代        | 标记-整理          | 响应速度游戏爱你 | 单CPU坏境下的Client模式，CMS的后备预案  |
| ParNew            | 并行               | 新生代        | 复制算法           | 响应速度优先     | 多CPU环境时在Server模式下与CMS配合      |
| Parallel Scavenge | 并行               | 新生代        | 复制算法           | 吞吐量优先       | 在后台运算而不需要太多交互的任务        |
| Parallel Old      | 并行               | 老年代        | 标记-整理          | 吞吐量优先       | 在后台运算而不需要太多交互的任务        |
| CMS               | 并发               | 老年代        | 标记-清除          | 响应速度优先     | 在互联网站或者B/S系统服务端上的Java应用 |
| G1                | 并发               | both          | 标记-整理+复制算法 | 响应速度优先     | 面向服务端应用，将来替换CMS             |

* 到JDK8为止，默认的垃圾收集时Parallel Scanvente 和 Parallel Old
* 到JDK9开始，G1收集器成为默认的收集器
* 目前来看，G1回收器停顿时间最短而且没有明显确定，非常适合Web应用。在JDK8中测试Web应用，堆内存6G，新生代4.5G的情况下，Parallel Scavenge回收新生代停顿长达1.5秒。G1回收器回收同样大小的新生代只停顿了0.2秒。

### 4.8 JVM常用调整参数介绍

| 参数名称                    | 含义                                                       | 默认值               |                                                              |
| --------------------------- | ---------------------------------------------------------- | -------------------- | ------------------------------------------------------------ |
| -Xms                        | 初始堆大小                                                 | 物理内存的1/64(<1GB) | 默认(MinHeapFreeRatio参数可以调整)空余堆内存小于40%时，JVM就会增大堆直到-Xmx的最大限制。 |
| -Xmx                        | 最大堆大小                                                 | 物理内存的1/4(<1GB)  | 默认(MaxHeapFreeRatio参数可以调整)空余堆内存大于70%时，JVM会减少堆直到 -Xms的最小限制 |
| -Xmn                        | 年轻代大小(1.4or lator)                                    |                      | **注意**：此处的大小是（eden+ 2 survivor space).与jmap -heap中显示的New gen是不同的。 整个堆大小=年轻代大小 + 老年代大小 + 持久代（永久代）大小. 增大年轻代后,将会减小年老代大小.此值对系统性能影响较大,Sun官方推荐配置为整个堆的3/8 |
| -XX:NewSize                 | 设置年轻代大小(for 1.3/1.4)                                |                      |                                                              |
| -XX:MaxNewSize              | 年轻代最大值(for 1.3/1.4)                                  |                      |                                                              |
| -XX:PermSize                | 设置持久代(perm gen)初始值                                 | 物理内存的1/64       |                                                              |
| -XX:MaxPermSize             | 设置持久代最大值                                           | 物理内存的1/4        |                                                              |
| -Xss                        | 每个线程的堆栈大小                                         |                      | JDK5.0以后每个线程堆栈大小为1M,以前每个线程堆栈大小为256K.更具应用的线程所需内存大小进行 调整.在相同物理内存下,减小这个值能生成更多的线程.但是操作系统对一个进程内的线程数还是有限制的,不能无限生成,经验值在3000~5000左右 一般小的应用， 如果栈不是很深， 应该是128k够用的 大的应用建议使用256k。这个选项对性能影响比较大，需要严格的测试。（校长） 和threadstacksize选项解释很类似,官方文档似乎没有解释,在论坛中有这样一句话:"” -Xss is translated in a VM flag named ThreadStackSize” 一般设置这个值就可以了。 |
| -**XX:ThreadStackSize**     | Thread Stack Size                                          |                      | (0 means use default stack size) [Sparc: 512; Solaris x86: 320 (was 256 prior in 5.0 and earlier); Sparc 64 bit: 1024; Linux amd64: 1024 (was 0 in 5.0 and earlier); all others 0.] |
| -XX:NewRatio                | 年轻代(包括Eden和两个Survivor区)与年老代的比值(除去持久代) |                      | -XX:NewRatio=4表示年轻代与年老代所占比值为1:4,年轻代占整个堆栈的1/5 Xms=Xmx并且设置了Xmn的情况下，该参数不需要进行设置。 |
| -XX:SurvivorRatio           | Eden区与Survivor区的大小比值                               |                      | 设置为8,则两个Survivor区与一个Eden区的比值为2:8,一个Survivor区占整个年轻代的1/10 |
| -XX:LargePageSizeInBytes    | 内存页的大小不可设置过大， 会影响Perm的大小                |                      | =128m                                                        |
| -XX:+UseFastAccessorMethods | 原始类型的快速优化                                         |                      |                                                              |
| -XX:+DisableExplicitGC      | 关闭System.gc()                                            |                      | 这个参数需要严格的测试                                       |
| -XX:MaxTenuringThreshold    | 垃圾最大年龄                                               |                      | 如果设置为0的话,则年轻代对象不经过Survivor区,直接进入年老代. 对于年老代比较多的应用,可以提高效率.如果将此值设置为一个较大值,则年轻代对象会在Survivor区进行多次复制,这样可以增加对象再年轻代的存活 时间,增加在年轻代即被回收的概率 该参数只有在串行GC时才有效. |
| -XX:+AggressiveOpts         | 加快编译                                                   |                      |                                                              |
| -XX:+UseBiasedLocking       | 锁机制的性能改善                                           |                      |                                                              |
| -Xnoclassgc                 | 禁用垃圾回收                                               |                      |                                                              |
| -XX:SoftRefLRUPolicyMSPerMB | 每兆堆空闲空间中SoftReference的存活时间                    | 1s                   | softly reachable objects will remain alive for some amount of time after the last time they were referenced. The default value is one second of lifetime per free megabyte in the heap |
| -XX:PretenureSizeThreshold  | 对象超过多大是直接在旧生代分配                             | 0                    | 单位字节 新生代采用Parallel Scavenge GC时无效 另一种直接在旧生代分配的情况是大的数组对象,且数组中无外部引用对象. |
| -XX:TLABWasteTargetPercent  | TLAB占eden区的百分比                                       | 1%                   |                                                              |
| -XX:+**CollectGen0First**   | FullGC时是否先YGC                                          | false                |                                                              |

***\*并行收集器相关参数\****

| -XX:+UseParallelGC            | Full GC采用parallel MSC (此项待验证)              |      | 选择垃圾收集器为并行收集器.此配置仅对年轻代有效.即上述配置下,年轻代使用并发收集,而年老代仍旧使用串行收集.(此项待验证) |
| ----------------------------- | ------------------------------------------------- | ---- | ------------------------------------------------------------ |
| -XX:+UseParNewGC              | 设置年轻代为并行收集                              |      | 可与CMS收集同时使用 JDK5.0以上,JVM会根据系统配置自行设置,所以无需再设置此值 |
| -XX:ParallelGCThreads         | 并行收集器的线程数                                |      | 此值最好配置与处理器数目相等 同样适用于CMS                   |
| -XX:+UseParallelOldGC         | 年老代垃圾收集方式为并行收集(Parallel Compacting) |      | 这个是JAVA 6出现的参数选项                                   |
| -XX:MaxGCPauseMillis          | 每次年轻代垃圾回收的最长时间(最大暂停时间)        |      | 如果无法满足此时间,JVM会自动调整年轻代大小,以满足此值.       |
| -XX:+UseAdaptiveSizePolicy    | 自动选择年轻代区大小和相应的Survivor区比例        |      | 设置此选项后,并行收集器会自动选择年轻代区大小和相应的Survivor区比例,以达到目标系统规定的最低相应时间或者收集频率等,此值建议使用并行收集器时,一直打开. |
| -XX:GCTimeRatio               | 设置垃圾回收时间占程序运行时间的百分比            |      | 公式为1/(1+n)                                                |
| -XX:+**ScavengeBeforeFullGC** | Full GC前调用YGC                                  | true | Do young generation GC prior to a full GC. (Introduced in 1.4.1.) |

***\*CMS相关参数\****

| -XX:+UseConcMarkSweepGC                | 使用CMS内存收集                           |      | 测试中配置这个以后,-XX:NewRatio=4的配置失效了,原因不明.所以,此时年轻代大小最好用-Xmn设置.??? |
| -------------------------------------- | ----------------------------------------- | ---- | ------------------------------------------------------------ |
| -XX:+AggressiveHeap                    |                                           |      | 试图是使用大量的物理内存 长时间大内存使用的优化，能检查计算资源（内存， 处理器数量） 至少需要256MB内存 大量的CPU／内存， （在1.4.1在4CPU的机器上已经显示有提升） |
| -XX:CMSFullGCsBeforeCompaction         | 多少次后进行内存压缩                      |      | 由于并发收集器不对内存空间进行压缩,整理,所以运行一段时间以后会产生"碎片",使得运行效率降低.此值设置运行多少次GC以后对内存空间进行压缩,整理. |
| -XX:+CMSParallelRemarkEnabled          | 降低标记停顿                              |      |                                                              |
| -XX+UseCMSCompactAtFullCollection      | 在FULL GC的时候， 对年老代的压缩          |      | CMS是不会移动内存的， 因此， 这个非常容易产生碎片， 导致内存不够用， 因此， 内存的压缩这个时候就会被启用。 增加这个参数是个好习惯。 可能会影响性能,但是可以消除碎片 |
| -XX:+UseCMSInitiatingOccupancyOnly     | 使用手动定义初始化定义开始CMS收集         |      | 禁止hostspot自行触发CMS GC                                   |
| -XX:CMSInitiatingOccupancyFraction=70  | 使用cms作为垃圾回收 使用70％后开始CMS收集 | 92   | 为了保证不出现promotion failed(见下面介绍)错误,该值的设置需要满足以下公式[CMSInitiatingOccupancyFraction计算公式](#CMSInitiatingOccupancyFraction_value) |
| -XX:CMSInitiatingPermOccupancyFraction | 设置Perm Gen使用到达多少比率时触发        | 92   |                                                              |
| -XX:+CMSIncrementalMode                | 设置为增量模式                            |      | 用于单CPU情况                                                |
| -XX:+CMSClassUnloadingEnabled          |                                           |      |                                                              |

***\*辅助信息\****

| -XX:+PrintGC                          |                                                          | 输出形式:[GC 118250K->113543K(130112K), 0.0094143 secs] [Full GC 121376K->10414K(130112K), 0.0650971 secs] |
| ------------------------------------- | -------------------------------------------------------- | ------------------------------------------------------------ |
| -XX:+PrintGCDetails                   |                                                          | 输出形式:[GC [DefNew: 8614K->781K(9088K), 0.0123035 secs] 118250K->113543K(130112K), 0.0124633 secs] [GC [DefNew: 8614K->8614K(9088K), 0.0000665 secs][Tenured: 112761K->10414K(121024K), 0.0433488 secs] 121376K->10414K(130112K), 0.0436268 secs] |
| -XX:+PrintGCTimeStamps                |                                                          |                                                              |
| -XX:+PrintGC:PrintGCTimeStamps        |                                                          | 可与-XX:+PrintGC -XX:+PrintGCDetails混合使用 输出形式:11.851: [GC 98328K->93620K(130112K), 0.0082960 secs] |
| -XX:+PrintGCApplicationStoppedTime    | 打印垃圾回收期间程序暂停的时间.可与上面混合使用          | 输出形式:Total time for which application threads were stopped: 0.0468229 seconds |
| -XX:+PrintGCApplicationConcurrentTime | 打印每次垃圾回收前,程序未中断的执行时间.可与上面混合使用 | 输出形式:Application time: 0.5291524 seconds                 |
| -XX:+PrintHeapAtGC                    | 打印GC前后的详细堆栈信息                                 |                                                              |
| -Xloggc:filename                      | 把相关日志信息记录到文件以便分析. 与上面几个配合使用     | -Xloggc:c:/gc.log                                            |
| -XX:+PrintClassHistogram              | garbage collects before printing the histogram.          |                                                              |
| -XX:+PrintTLAB                        | 查看TLAB空间的使用情况                                   |                                                              |
| XX:+PrintTenuringDistribution         | 查看每次minor GC后新的存活周期的阈值                     | Desired survivor size 1048576 bytes, new threshold 7 (max 15) new threshold 7即标识新的存活周期的阈值为7。 |