# 多线程

## 一、线程的创建方式

1. 继承Thread
2. 实现Runnable接口
3. 通过Callable和Future创建线程
4. 通过线程池创建线程

## 二、Thread和Runnable的区别

* 一个类继承Thread，则不适合资源共享。但是如果实现了Runable接口的话，则很容易的实现资源共享。
* 实现Runnable接口比继承更有优势：
  1. 适合多个相同的程序代码线程去共享同一个资源。
  2. 可以避免java中的单继承的局限性
  3. 增加代码的健壮性，实现解耦操作，代码可以被多个线程共享，代码和线程独立。
  4. 线程池只能放入实现Runnable或者Callable类线程，不能放入继承Thread的类。

## 三、Runable和Callable有什么区别

* Runnable接口中的run()方法的返回值是void
* Callable接口的call()方法是有返回值的，是一个泛型。和Future、FutureTask配合可以用来获取异步执行的结果。

## 四、线程有那些状态

* 初始状态
* 就绪状态
* 运行装态
* 阻塞状态
* 死亡状态

![](http://img.hurenjieee.com/%E7%BA%BF%E7%A8%8B%E7%8A%B6%E6%80%81%E8%BD%AC%E6%8D%A2.jpg)

## 五、sleep()和wait()有什么区别

1. sleep()是Thread类的静态方法，是线程用来控制自身流程的。而wait()方法是Object类的方法，用于线程之间通信。
2. 调用wait()的时候，方法会释放当前持有的锁，而sleep方法不会释放锁。
3. 实现：
   1. sleep()方法导致程序暂停执行指定的时间，让出CPU给其他线程，但是他的监控状态依然保持着，当指定时间到了又会自动恢复运行状态。
   2. 调用wait()方法的时候，线程会放弃对象锁，进入等待次对象的等待锁定池，只有针对此对象调用notify()方法后本线程才进入对象锁定池准备。

## 六、创建线程池的几种方式

1. **newFixedThreadPool(int nThreads)**

   > 创建一个固定长度的线程池，每当提交一个任务就会创建一个线程，直到线程池的最大数量，这时线程规模将不再会变化，当下昵称发生未预期的错误而结束时，线程池会补充一个新的线程。
   >
   > 允许请求队列长度为Integer.MAX_VALUE，可能会创建大量的请求，从而导致OOM。

2. **newCachedThreadPool()**

   > 创建一个可缓存的线程池，如果线程池的规模超过了处理需求，将自动回收空闲线程，而当需求增加时，则可以自动添加新线程，线程池的规模不存在任何限制。
   >
   > 允许创建线程数量为Integer.MAX_VALUE，可能会创建大量的线程，从而导致OOM。

3. **newSingleThreadExecutor()**

   > 这是一个单线程的Executor，它创建单个工作线程来执行任务，如果这个线程异常结束，会创建一个新的来替代它；它的特点时能确保依照任务在队列顺序来串行执行。
   >
   > 允许请求队列长度为Integer.MAX_VALUE，可能会创建大量的请求，从而导致OOM。

4. **newScheduledThreadPool(int corePoolSize)**

   > 创建了一个固定长度的线程池，而且以延迟或者定时的方式来执行任务，类似于Timer。

推荐自己创建多线程，规避资源耗尽的风险。

==**线程池7大参数**==

1. **corePoolSize**

   > 线程池核心线程大小
   >
   > 线程池中会维护一个最小的线程数量，即使这些线程处于空闲状态，他们也不会被销毁，除非设置了allowCoreThreadTimeOut。

2. **maximunPoolSize**

   > 线程池最大线程数量
   >
   > 一个任务被提交到线程池后，首先会缓存到工作队列中，如果工作队列满了，则会创建一个新线程，然后从工作队列中取出一个任务交给新线程来处理，而将刚提交的任务放入工作队列。线程池不会无限制的去创建新线程，最大限制数量为maximumPoolSize。

3. **keepAliveTime**

   > 控先线程存活时间
   >
   > 一个线程如果处于空闲状态，并且当前的线程数量大于corePoolSize，那么在指定时间后，这个空闲线程会被销毁。

4. **unit**

   > 空闲线程存货时间单位

5. **workQueue**

   > 新任务被提交之后，会先进入此工作队列中，任务调度时再从队列中取出任务。
   >
   > JDK提供了四种工作队列：
   >
   > 1. ArrayBlockingQueue
   >
   >    基于数组的有界阻塞队列，按FIFO排序。新任务进来后，会放到该队列的队尾，有界的数组可以防止资源耗尽问题。当线程池数量达到corePoolSize后，再有新任务进来，则会将任务放入该队列的队尾，等待被调度。如果队列已经是慢的，则创建一个新线程。如果线程数量已经达到了maxPoolSize，则会执行拒绝策略。
   >
   > 2. LinkedBlockingQueue
   >
   >    基于链表的无界阻塞队列（最大容量为Integer.MAX_VALUE），按照FIFO排序。由于该队列的近似无界性，当线程池中线程数量达到corePoolSize后，再有新的任务进来，会一致存在该队列，而不会去创建新线程直到maxPoolSize，因此使用该工作队列时，参数maxPoolSize其实时不起作用的。
   >
   > 3. SynchronousQueue
   >
   >    一个不缓存任务的阻塞队列，生产者放入一个任务必须等到消费者取出这个任务。也就是说新任务进来时，不会缓存，而是被调度执行该任务，如果没有可用线程，则创建线程，如果线程数量达到maxPoolSize，则执行执行拒绝策略。
   >
   > 1. PriorityBlockingQueue
   >
   >    据有优先级的无界阻塞队列，优先级通过参数Comparator实现。

6. **threadFactory**

   > 线程工厂
   >
   > 创建一个线程时使用的工厂，可以用来设定线程名，是否为daemon线程等等。

7. **handler**

   > 拒绝策略
   >
   > 当工作队列中的任务已到达最大限制，并且线程池中的线程数量也达到最大限制，这时候就会执行拒绝策略。
   >
   > JDK提供了4种拒绝策略：
   >
   > 1. CallerRunPolicy
   >
   >    在调用这线程中直接执行被拒绝任务的run方法，除非线程池已经shutdown，则直接拒绝任务
   >
   > 2. AbortPolicy
   >
   >    直接丢弃任务，并抛RejectedExecutionException异常
   >
   > 3. DiscardPolicy
   >
   >    直接丢弃策略，什么都不做
   >
   > 4. DiscardOldestPolicy
   >
   >    抛弃进入队列最早的那个任务，然后尝试把这个拒绝的任务放入队列

![](http://img.hurenjieee.com/1649861-20200317152016551-1682542588.png)

## 七、如何保证多线程的运行安全

* **并发编程三要素**

  * 原子性

  > 提供互斥访问，同一个时刻只能有一个线程对数据进行操作，（atomic，synchronized）
  * 可见性

  > 一个线程对主内存的修改可以及时地被其他线程看到，（synchronized，colatile）。
  * 有序性

  >一个线程观察其他线程的指令执行顺序，由于指令重排，该观察结构一般杂乱无序，（happens-before原则）

## 八、死锁是什么？如何防止死锁

* 死锁是指两个或者两个以上的进程在执行过程中，因争夺资源而造成一种互相等待的现象，若无外力作用，他们都将无法推动下去。

* 死锁的四个必要条件：

  > * 互斥条件：一个资源每次只能被一个进程使用
  > * 请求和保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放
  > * 不可剥夺条件：进程已获得的资源，在未使用完之前，不能强行剥夺
  > * 环路等待条件：若干进程之间形成一种头尾相接的循环等待资源关系

## 九、ThreadLocal是什么？以及其使用场景？

* 线程局部变量是局限于线程内部的变量，属于线程自身所有，不在多个线程间共享。
* Java提供ThreadLocal类来支持线程局部变量，是一种线程安全的方式。
* 原理：

> 每个运行的线程都会有一个类型为TheadLocal.ThreadLocal的map，这个map就是用来存储这个线程绑定的变量，map的key就是ThreadLocal对象，value就是线程正在执行的任务重的某个变量的包装类Entry。
>
> 然而ThreadLocal是弱引用，即使线程正在执行中，只要Thread对象引用被置为null，Entry的key就会自动在下一次YGC时被垃圾回收。
>
> 而在TheadLocal使用set()和get()时，又会自动地将那个key==null的value置为null，使value能够被垃圾回收，避免内存泄漏。

* **ThreadLocal注意事项**：

  * **脏数据**：
  > 线程的复用会产生脏数据。由于线程池会重用Thread对象，那么与Thread绑定的类与静态属性ThreadLocal变量也会被重用。如果在实现的线程run方法体内不显式调用remove方法清理与线程相关的ThreadLoad信息，那么倘若下一个进程不调用set()设置初始值，就可能get()到重用的线程信息，包括ThreadLocal所关联的线程对象的value的值。

  * **内存泄漏**：

  > 通常我们会使用static关键字来修饰ThreadLocal（这也是在源码注释中所推荐的）。在此场景下，其生命周期就不回随着线程结束而结束，寄希望于ThreadLocal对象失去引用后，触发弱引用机智来回收Entry的Value就不现实了。如果不进行remove操作，那么这个线程执行完成后，通过ThreadLocal对象持有的对象是不回释放。

  * **父子线程共享线程变量**：

  > 很多场景下通过ThreadLocal来传递全局上下文，会发现子线程的value和主线程不一致。比如用ThreadLocal来存储监控系统的某个标记位，暂且命名为traceId。某次请求下所有的traceId都是一致的，以获取可以统计解析的日志文件。但在实际的开发过程中过，发现子线程里的traceId为null，根猪线程的并不一致。这就需要使用inheritableThreadLocal来解决父子进程之间共享线程变量的问题，是整个连接过程中的traceId一致，

## 十、synchronized和Lock有什么区别

| 区别 | synchronized                     | Lock                                                     |
| ---- | -------------------------------- | -------------------------------------------------------- |
| 类型 | java内置关键字                   | java接口                                                 |
| 状态 | 无法获取锁状态                   | 可以获取锁状态                                           |
| 释放 | 自动释放锁                       | 手动释放                                                 |
| 等待 | 线程A获取锁之后，线程B会一直等待 | 不会一直等待，如尝试获取到锁，线程可以不用一直等待就结束 |
| 特性 | 可重入、不可中断、非公平         | 可重入、可中断、可公平                                   |
| 场景 | 适合代码少量的同步问题           | 适合代码大量的同步问题                                   |

## 十一、synchronized和ReentrantLock区别

1. synchronized是关键字，ReentrantLock是类。

2. ReentranLock是类，那么它就提供了比synchronized更有灵活的特性，可以继承、可以有方法、可以有各种各样的类变量。
   * ReentrantLock扩展性体现：

    >* ReentrantLock可以对获取锁的等待时间进行设置，这样就避免了死锁
    >* ReentranLock可以获取各种锁的信息
    >* ReentranLock可以灵活地实现多路通知

* 二者的锁机制其实也是不一样的：ReentrantLock底层调用的是Unsafe的park方法加锁，synchronized操作的阿应该是对象头中的mark word。

## 十二、sychronized（对象锁）和static synchronized（类锁）

* **sychronized**：

> 对类的==**当前实例（当前对象）==**进行枷锁，防止其他线程同时访问该类的该实例的==**所有**==synchronized块。注意这里是“类的当前实例”，类的两个不同实例就没有这种约束。

* **static sychronized**：

> 控制类的所有实例的并发访问，static sychronized是限制**==多线程中该类的所有实例==**访问JVM中该类所对应的代码块。

* 总结如下：sychronized相当于this.synchronized，而static synchronized相当于SomeThing.synchronized。