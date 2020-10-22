

# Zookeeper

[思维导图](./Zookeeper.xmind)

## 一、介绍

* ZooKeeper简单易用，能够很好的解决分布式框架在运行中，出现的各种协调问题。比如集群master主备切换、节点的上下线感知等等。
* ZooKeeper是一个分布式的，开放源码的，用于分布式应用程序的协调服务（service）；是Google的Chubby的一个开源实现版。

![ecosystem](http://img.hurenjieee.com/uPic/ecosystem.png)

## 二、Zookeeper架构

### 2.1 Zookeeper外部

* ZooKeeper服务端有两种不同的运行模式。单机的称为"**独立模式**"(standalone mode)；

* 集群的称为“**仲裁模式**(quorum mode)”

  ![zkservice](http://img.hurenjieee.com/uPic/zkservice.jpg)

* 主从架构：Master + Slave

* 客户端读

* 客户端写：类比存钱

  * ![img](http://img.hurenjieee.com/uPic/1931096-20200522214737147-1268449607.png)
  * 1. 用户发起写请求
    2. 节点发送到主节点
    3. 主节点分发到所有节点进行投票
    4. 开始投票
    5. 超过半数认可，即认同写请求
    6. 发起读请求

* leader在通知follower执行某条命令时，如何保障每个follower都收到？

  * 队列结构。
    * ![](http://img.hurenjieee.com/uPic/1931096-20200522214736458-1925378554-20200803151758143.png)

* CAP：

  * Consistency一致性；
    * 一致性是指数据在多个副本之间是否能够保持数据一致的特性。
  * Availability可用性；
    * 可用性是指系统提供的服务必须一直处于可用的状态。
  * Partition Tolerance分区容错；
    * **分布式系统在遇到任何网络分区故障的时候，仍然需要能够保证对外提供满足一致性和可用性的服务，除非是整个网络环境都发生了故障**。

  * 这三个基本需求，最多只能同时满足其中的两项，**因为P是必须的,因此往往选择就在CP或者AP中**。ZooKeeper保证的是CP
    * **不能保证每次服务请求的可用性**。任何时刻对ZooKeeper的访问请求能得到一致的数据结果，同时系统对网络分割具备容错性；但是它不能保证每次服务请求的可用性（注：也就是在极端环境下，ZooKeeper可能会丢弃一些请求，消费者程序需要重新请求才能获得结果）。所以说，ZooKeeper不能保证服务可用性。
    * **进行leader选举时集群都是不可用**。在使用ZooKeeper获取服务列表时，当master节点因为网络故障与其他节点失去联系时，剩余节点会重新进行leader选举。问题在于，选举leader的时间太长，30 ~ 120s, 且选举期间整个zk集群都是不可用的，这就导致在选举期间注册服务瘫痪，虽然服务能够最终恢复，但是漫长的选举时间导致的注册长期不可用是不能容忍的。所以说，ZooKeeper不能保证服务可用性。

### 2.2 Zookeeper内部

* **Zookeeper架构：**

* ![img](http://img.hurenjieee.com/uPic/1931096-20200522214723046-165405261.jpg)

* leader很重要？如果挂了怎么办？开始选举新的leader



* **ZooKeeper服务器四种状态：**
  * looking：服务器处于寻找Leader群首的状态
  * leading：服务器作为群首时的状态
  * following：服务器作为follower跟随者时的状态
  * observing：服务器作为观察者时的状态

> leader选举分**两种情况**
>
> - 集群初始启动时
> - 集群运行中leader挂了时



* **集群启动时的Leader选举**

  * ![img](http://img.hurenjieee.com/uPic/1931096-20200522214735204-894755671.png)

  * 以3台机器组成的ZooKeeper集群为例

  * 原则：集群中过半数Server启动后，才能选举出Leader；

  * 此处quorum数是多少？

  * 每个server投票信息**vote信息**结构为(sid, zxid)；

     server1~3初始投票信息分别为：

     server1 -> (1, 0)
    ​ server2 -> (2, 0)
    ​ server3 -> (3, 0)

  * **leader选举公式**：

     server1 (zxid1, sid1)

     server2 (zxid2, sid2)

     **zxid大的server胜出；若zxid相等，再根据判断sid判断，sid大的胜出**

  * 流程：

    1. ZK1和ZK2票投给自己；ZK1的投票为(1, 0)，ZK2的投票为(2, 0)，并各自将投票信息分发给其他机器。

    2. 处理投票。每个server将收到的投票和自己的投票对比；ZK1更新自己的投票为(2, 0)，并将投票重新发送给ZK2。

    3. 统计投票。server统计投票信息，是否有半数server投同一个服务器为leader；

    4. 改变服务器状态。确定Leader后，各服务器更新自己的状态，Follower变为FOLLOWING；Leader变为LEADING。

    5. 当K3启动时，发现已有Leader，不再选举，直接从LOOKING改为FOLLOWING。

* 集群运行时新leader选举

  * ![img](http://img.hurenjieee.com/uPic/1931096-20200522214734661-976925736.png)

  * 流程：

    1. ZK1和ZK3票投给自己；ZK1的投票为（1,3），ZK的投票为（3，2），并各自将投票信息分发给其他机器。

    2. 处理投票。每个server将收到的投票和自己的投票对比；ZK1更新自己的投票为(1, 3)，并将投票重新发送给ZK3。

    3. 统计投票。server统计投票信息，是否有半数server投同一个服务器为leader；

    4. 改变服务器状态。确定Leader后，各服务器更新自己的状态，Follower变为FOLLOWING；Leader变为LEADING。

  

  

  * **网络分区、脑裂**
    * 网络分区：网络通信故障，集群被分成了2部分
    * 脑裂：原leader处于一个分区；另外一个分区选举出新的leader -> 2个leader
    * 动图地址：http://thesecretlivesofdata.com/raft/#replication

  



## 三、Zookeeper命令行操作

1. 启动、停止、状态

   ```bash
   zkServer.sh start/stop/status
   ```

2. 连接Zookeeper	

   ```bash
   zkCli.sh -server node1:2181,node2:2181,node3:2181
   ```

3. 连接后命令

   ```bash
   # 列出列表
   ls /
   #[zookeeper, zk_test]
   
   # 创建了一个新的znode 节点“ zk ”以及与它关联的字符串
   create /zk myData
   # Created /zk
   
   # 获取节点
   get /zk
   #myData
   #cZxid = 0x300000002  节点创建时的zxid.
   #ctime = Fri Jul 26 23:48:10 EDT 2019 节点创建时的时间戳.
   #mZxid = 0x300000002 节点最新一次更新发生时的zxid.
   #mtime = Fri Jul 26 23:48:10 EDT 2019 节点最新一次更新发生时的时间戳.
   #pZxid = 0x300000002 
   #cversion = 0  其子节点的更新次数.
   #dataVersion = 0  节点数据的更新次数.
   #aclVersion = 0 节点ACL(授权信息)的更新次数.
   #ephemeralOwner = 0x0  如果该节点为ephemeral节点, ephemeralOwner值表示与该节点绑定的session id. 如果该节点不是ephemeral节点, ephemeralOwner值为0. 
   #dataLength = 6 节点数据的字节数.
   #numChildren = 0 子节点个数.
   
   # 删除节点
   delete /zk
   # 
   
   # 创建有序节点
   create -s /zk_order 1
   # Created /zk_order0000000003
   
   get /zk_order0000000003
   #1
   #cZxid = 0x300000006
   #ctime = Fri Jul 26 23:58:10 EDT 2019
   #mZxid = 0x300000006
   #mtime = Fri Jul 26 23:58:10 EDT 2019
   #pZxid = 0x300000006
   #cversion = 0
   #dataVersion = 0
   #aclVersion = 0
   #ephemeralOwner = 0x0
   #dataLength = 1
   #numChildren = 0
   
   create -s /zk_order 2
   # Created /zk_order0000000004
   
   # 创建临时节点
   create -e /zk_tem 1
   #Created /zk_tem
   get /zk_tem        
   #1
   #cZxid = 0x300000008
   #ctime = Sat Jul 27 00:01:16 EDT 2019
   #mZxid = 0x300000008
   #mtime = Sat Jul 27 00:01:16 EDT 2019
   #pZxid = 0x300000008
   #cversion = 0
   #dataVersion = 0
   #aclVersion = 0
   #ephemeralOwner = 0x100071e8e970000 临时节点
   #dataLength = 1
   #numChildren = 0
   
   # quit在zkClient
   get /zk_tem 
   # Ctrl+C不会节点不会马上消失，需要过一段时间
   # 当你客户端会话失效后，所产生的节点也不是一下子就消失了，也要过一段时间，大概是 10 秒以内，可以试一下，本机操作生成节点，在服务器端用命令来查看当前的节点数目，你会发现客户端已经 stop，但是产生的节点还在
   
   # 临时有序节点
   create -e -s /zk_tem_order 1
   #Created /zk_tem_order0000000009
   ```

## 四、Zookeeper特殊概念

### 4.1 数据节点

* ZooKeeper所提供的服务主要是通过以下三个部分来实现的
  * **ZooKeeper=简版文件系统(Znode)+原语+通知机制(Watcher)。**

  ![img](http://img.hurenjieee.com/uPic/znode.jpg)

### 4.2 Watch监视通知

* 客户端如何获取ZooKeeper服务器上的最新数据？

  - **方式一**轮询：ZooKeeper以远程服务的方式，被客户端访问；客户端以轮询的方式获得znode数据，效率会比较低（代价比较大）

    ![](/Users/jack/Documents/Github/MyNote/%25E5%25A4%25A7%25E6%2595%25B0%25E6%258D%25AE/zookeeper/img/watcher1.png)

  - **方式二**基于通知的机制：客户端在znode上注册一个Watcher监视器，当znode上数据出现变化，watcher监测到此变化，通知客户端

    ![](/Users/jack/Documents/Github/MyNote/%25E5%25A4%25A7%25E6%2595%25B0%25E6%258D%25AE/zookeeper/img/watcher2.png)

  - **监听事件**：

    - ![watcher_command](http://img.hurenjieee.com/uPic/watcher_command.png)

  - ​	示例1:

    ```bash
    #ls pat	h [watch]
    #node-01 上执行
    ls /zk_test watch
    
    #node-02 上执行
    create /zk_test/dir01 dir01-data
    
    #node-01上
    [zk: node-01:2181,node-02:2181,node-03:2181(CONNECTED) 87] 
    WATCHER::
    WatchedEvent state:SyncConnected type:NodeChildrenChanged path:/zk_test
    ```

    ​	示例2：

    ```bash
    # node1
    get /zk_test watch
    
    # node2
    set /zk_test zk_test_new
    
    # node1
    WATCHER::
    WatchedEvent state:SyncConnected type:NodeDataChanged path:/zk_test
    ```

  **上下线感知**

  1. 原理

     1. 节点1（client1）创建临时节点
     2. 节点2（client2）注册监听器watcher
     3. 当client1与zk集群断开连接，临时节点会被删除
     4. watcher发送消息，通知client2，临时节点被删除的事件

  2. 用到的zk特性：

     1. Watcher+临时节点

  3. 好处

     1. 通过这种方式，检测和被检测系统不需要直接关联（如client1与client2），而是通过ZK上的某个节点进行关联，大大减少了系统**耦合**。

  4. 实现

     1. client1操作

        ```shell
        # 创建临时节点
        create -e /zk_tmp tmp-data
        ```

     2. client2操作

        ```shell
        # 在/zk_tmp注册监听器
        ls /zk_tmp watch
        ```

     3. client1操作

        ```shell
        # 模拟节点下线
        close
        ```

     4. 观察client2

        ```shell
        WATCHER::
        WatchedEvent state:SyncConnected type:NodeDeleted path:/zk_tmp
        ```

### 4.3 会话Session

![img](http://img.hurenjieee.com/uPic/session.png)

1. 长连接：客户端与某一服务器建立TCP长连接，建立一个会话Session。

2. 特点

   1. 客户端打开一个Session中的请求以FIFO（先进先出）的顺序执行；
   2. 若打开两个Session，无法保证Session间，请求FIFO执行

3. 会话的生命周期

   ![img](http://img.hurenjieee.com/uPic/session_period.png)

   1. Session从NOT_CONNECTED状态开始，并随着Zookeeper客户端初始化，转移到CONNECTING状态。
   2. 正常情况下，客户端会与Zookeeper服务器连接成功，并且转移到CONNECTED状态。
   3. 当客户端失去了与ZooKeeper服务器的连接或者不能听到服务器，它会转移回CONNECTING
   4. 并且尝试寻找另一个ZooKeeper服务器。如果它能找到另一个服务器或者重新连接到之前的服务器，并确认了这个Session仍然有效，它会转移回CONNECTED状态。否则，它会定义这个Session失效，并转移到CLOSED
   5. 应用可以显示关闭Session。

   

### 4.4 请求

读写请求
![zkservice](http://img.hurenjieee.com/uPic/zkservice.jpg)



### 4.5 事务zxid(ZooKeeper Transaction Id)

* 增删改的操作，会形成一次事务。
* ACID：
  * 原子性atomicity
  * 一致性consistency
  * 隔离性isolation
  * 持久性durability

* 每个事务有一个全局唯一的事务ID，用 ZXID 表示，全局有序。
  ZXID通常是一个64位的数字。由**epoch（32位）+counter（32位）**组成
  ![](http://img.hurenjieee.com/uPic/zxid.png)

* epoch：leader周期的epoch编号
* counter：事务编号

### 4.6 Zookeeper工作原理

![](http://img.hurenjieee.com/uPic/zookeeper3.png)

1. 在Client向Follwer发出一个写的请求
2. Follwer把请求发送给Leader
3. Leader接收到以后开始发起投票并通知Follwer进行投票
4. Follwer把投票结果发送给Leader
5. Leader将结果汇总后如果需要写入，则开始写入同时把写入操作通知给Follwer，然后commit
6. Follwer把请求结果返回给Client

### 4.7 ZooKeeper状态同步

完成leader选举后，zk就进入ZooKeeper之间状态同步过程

1. leader等待server连接；
2. Follower连接leader，将最大的zxid发送给leader；
3. Leader根据follower的zxid确定同步点；
4. 完成同步后通知follower 已经成为uptodate状态；
5. Follower收到uptodate消息后，又可以重新接受client的请求进行服务了。

![](http://img.hurenjieee.com/uPic/sync.png)

## 五、应用场景

1. NameNode使用ZooKeeper实现高可用.
2. Yarn ResourceManager使用ZooKeeper实现高可用.
3. 利用ZooKeeper对HBase集群做高可用配置、元数据存储
4. Kafka使用ZooKeeper
   - 保存消息消费信息比如offset.
   - 用于检测崩溃
   - 主题topic发现
   - 保持主题的生产和消费状态

5. 分布式锁的实现
6. 定时任务争夺
7. ...

![scenes](http://img.hurenjieee.com/uPic/scenes.png)