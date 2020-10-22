# afka

## 一、Kafka基础知识

### 1.1 为什么会有消息系统

1. ==**解耦**==

   允许你独立的扩展或修改两边的处理过程，只要确保它们遵守同样的接口约束。

2. ==**冗余**==

   消息队列把数据进行持久化直到它们已经被完全处理，通过这一方式规避了数据丢失风险。许多消息队列所采用的"插入-获取-删除"范式中，在把一个消息从队列中删除之前，需要你的处理系统明确的指出该消息已经被处理完毕，从而确保你的数据被安全的保存直到你使用完毕。

3. ==**扩展性**==

   因为消息队列解耦了你的处理过程，所以增大消息入队和处理的频率是很容易的，只要另外增加处理过程即可。

4. ==**灵活性 & 峰值处理能力**==

   在访问量剧增的情况下，应用仍然需要继续发挥作用，但是这样的突发流量并不常见。如果为以能处理这类峰值访问为标准来投入资源随时待命无疑是巨大的浪费。使用消息队列能够使关键组件顶住突发的访问压力，而不会因为突发的超负荷的请求而完全崩溃。

5. ==**可恢复性**==

   系统的一部分组件失效时，不会影响到整个系统。消息队列降低了进程间的耦合度，所以即使一个处理消息的进程挂掉，加入队列中的消息仍然可以在系统恢复后被处理。

6. ==**顺序保证**==

   在大多使用场景下，数据处理的顺序都很重要。大部分消息队列本来就是排序的，并且能保证数据会按照特定的顺序来处理。（Kafka 保证一个 Partition 内的消息的有序性）

7. ==**缓冲**==

   有助于控制和优化数据流经过系统的速度，解决生产消息和消费消息的处理速度不一致的情况。

8. ==**异步通信**==

   很多时候，用户不想也不需要立即处理消息。消息队列提供了异步处理机制，允许用户把一个消息放入队列，但并不立即处理它。想向队列中放入多少消息就放多少，然后在需要的时候再去处理它们。

### 1.2 kafka核心概念

![Kafka集群架构](http://img.hurenjieee.com/uPic/Kafka集群架构.png)

* ==**Producer**==

  消息生产者，发布消息到 Kafka集群的终端或服务。

* ==**Broker**==

  Kafka集群包含的服务器

* ==**Topic**==

  每条发布到Kafka集群的消息属于的类别，即Kafka是面向topic的

* ==**partition**==

  partition是物理上的概念，每个Topic包含一个或者多个partition。Kafka分配的单位就是partition。

* ==**Consumer**==

  Kafka集群中的消费者

* ==**Consumer Group**==

  high-level consumer api中，每个Consumer都属于一个consumer greoup，每条消息只能被consumer group中的一个Consiume消费，但是可以被多个consumer group 消费

* ==**Replica**==

  partition的副本，保障了partition的高可用

* ==**Leader**==

  replica的一个角色，prodicer和consumer只能leader交互。

* ==**Follower**==

  replica的一个角色，从leader中复制数据

* ==**Controller**==

  **负责管理整个Kafka集群范围内的各种东西。**

  * Kafka集群中某个broker宕机之后，是谁负责感知到他的宕机，以及负责进行Leader Partition的选举？
  * 在Kafka集群里新加入了一些机器，此时谁来负责把集群里的数据进行负载均衡的迁移？
  * Kafka集群的各种元数据，比如说每台机器上有哪些partition，谁是leader，谁是follower，是谁来管理的？
  * 删除一个topic，那么背后的各种partition如何删除，是谁来控制？
  * Kafka集群扩容加入一个新的broker，是谁负责监听这个broker的加入？
  * 某个broker崩溃了，是谁负责监听这个broker崩溃？

* ==**Zookeeper**==

  1. Kafka 通过 zookeeper 来存储集群的 meta 信息。
  2. 一旦controller所在broker宕机了，此时临时节点消失，集群里其他broker会一直监听这个临时节点，发现临时节点消失了，就争抢再次创建临时节点，保证有一台新的broker会成为controller角色。

## 三、Kafka集群常用命令

### 2.1 启动服务

```bash
服务启动：
./kafka-server-start.sh -daemon ../config/server.properties 
停止服务：
kafka-server-stop.sh

如果报错：
No kafka server to stop
修改kafka-server-stop.sh
将 PIDS=$(ps ax | grep -i 'kafka.Kafka' | grep java | grep -v grep | awk '{print $1}')
修改为 PIDS=$(jps -lm | grep -i 'kafka.Kafka' | awk '{print $1}')
```

### 2.2 创建主题

```bash
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
```

### 2.3 查看主题

```bash
bin/kafka-topics.sh --list --zookeeper localhost:2181
```

### 2.4 发送消息

```bash
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
This is a message
This is another message
```

### 2.5 消费消息

```
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
This is a message
This is another message
```

### 2.6 自带的脚本测试

```
测试生产数据
bin/kafka-producer-perf-test.sh --topic test-topic --num-records 500000 --record-size 200 --throughput -1 --producer-props bootstrap.servers=hadoop03:9092,hadoop04:9092,hadoop05:9092 acks=-1
测试消费数据
bin/kafka-consumer-perf-test.sh --broker-list hadoop03:9092,hadoop04:9092,hadoop53:9092 --fetch-size 2000 --messages 500000 --topic test-topic
```

### 2.7 查看Group消费情况

```sh
bin/kafka-consumer-groups.sh --describe --group test-topic --bootstrap-server hadoop03:9092,hadoop04:9092,hadoop53:9092
```

## 三、Kafka生产者

### 3.1 Producer发送原理

1. Producer发送一条消息，首先需要选择一个topic的分区，默认是轮询来负载均衡，但是如果指定了一个分区key，那么根据这个key的hash值来分发到指定的分区，这样可以让相同的key分发到同一个分区里去，还可以自定义partitioner来实现分区策略。

   ```java
   producer.send(msg); // 用类似这样的方式去发送消息，就会把消息给你均匀的分布到各个分区上去
   producer.send(key, msg); // 订单id，或者是用户id，他会根据这个key的hash值去分发到某个分区上去，他可以保证相同的key会路由分发到同一个分区上去
   ```

2. 每次发送消息都必须先把数据封装成一个ProducerRecord对象，里面包含了要发送的topic，具体在哪个分区，分区key，消息内容，timestamp时间戳，然后这个对象交给序列化器，变成自定义协议格式的数据，接着把数据交给partitioner分区器，对这个数据选择合适的分区，默认就轮询所有分区，或者根据key来hash路由到某个分区，这个topic的分区信息，都是在客户端会有缓存的，当然会提前跟broker去获取。

3. 接着这个数据会被发送到producer内部的一块缓冲区里，然后producer内部有一个Sender线程，会从缓冲区里提取消息封装成一个一个的batch，然后每个batch发送给分区的leader副本所在的broker。

   ![微信图片_20191019101345](http://img.hurenjieee.com/uPic/微信图片_20191019101345.png)

### 3.2 Producer代码演示

```java
import java.util.Properties;

import org.apache.kafka.clients.producer.Callback;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;

public class ProducerDemo {
	public static void main(String[] args) throws Exception {
		Properties props = new Properties();
		// 这里可以配置几台broker即可，他会自动从broker去拉取元数据进行缓存
		props.put("bootstrap.servers", "hadoop03:9092,hadoop04:9092,hadoop05:9092");  
		// 这个就是负责把发送的key从字符串序列化为字节数组
		props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
		// 这个就是负责把你发送的实际的message从字符串序列化为字节数组
		props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
		props.put("acks", "-1");
		props.put("retries", 3);
		props.put("batch.size", 323840);
		props.put("linger.ms", 10);
		props.put("buffer.memory", 33554432);
		props.put("max.block.ms", 3000);	
		// 创建一个Producer实例：线程资源，跟各个broker建立socket连接资源
		KafkaProducer<String, String> producer = new KafkaProducer<String, String>(props);
		ProducerRecord<String, String> record = new ProducerRecord<>(
				"test-topic", "test-key", "test-value");
		
		// 这是异步发送的模式
		producer.send(record, new Callback() {
			@Override
			public void onCompletion(RecordMetadata metadata, Exception exception) {
				if(exception == null) {
					// 消息发送成功
					System.out.println("消息发送成功");  
				} else {
					// 消息发送失败，需要重新发送
				}
			}
		});
		Thread.sleep(10 * 1000); 
		
		// 这是同步发送的模式
    // producer.send(record).get(); 
		// 你要一直等待人家后续一系列的步骤都做完，发送消息之后
		// 有了消息的回应返回给你，你这个方法才会退出来
		producer.close();
	}
	
}
```



### 3.3 核心参数

#### 3.3.1 异常处理

```
不管是异步还是同步，都可能让你处理异常，常见的异常如下：
1）LeaderNotAvailableException：这个就是如果某台机器挂了，此时leader副本不可用，会导致你写入失败，要等待其他follower副本切换为leader副本之后，才能继续写入，此时可以重试发送即可。如果说你平时重启kafka的broker进程，肯定会导致leader切换，一定会导致你写入报错，是LeaderNotAvailableException
2）NotControllerException：这个也是同理，如果说Controller所在Broker挂了，那么此时会有问题，需要等待Controller重新选举，此时也是一样就是重试即可
3）NetworkException：网络异常，重试即可
我们之前配置了一个参数，retries，他会自动重试的，但是如果重试几次之后还是不行，就会提供Exception给我们来处理了。
参数：retries 默认值是3
参数：retry.backoff.ms  两次重试之间的时间间隔
```

#### 3.3.2 提升消息吞吐量

```
1）buffer.memory：设置发送消息的缓冲区，默认值是33554432，就是32MB
如果发送消息出去的速度小于写入消息进去的速度，就会导致缓冲区写满，此时生产消息就会阻塞住，所以说这里就应该多做一些压测，尽可能保证说这块缓冲区不会被写满导致生产行为被阻塞住

Long startTime=System.currentTime();
producer.send(record, new Callback() {

			@Override
			public void onCompletion(RecordMetadata metadata, Exception exception) {
				if(exception == null) {
					// 消息发送成功
					System.out.println("消息发送成功");  
				} else {
					// 消息发送失败，需要重新发送
				}
			}
		});
Long endTime=System.currentTime();
If(endTime - startTime > 100){//说明内存被压满了
        说明有问题
}

2）compression.type，默认是none，不压缩，但是也可以使用lz4压缩，效率还是不错的，压缩之后可以减小数据量，提升吞吐量，但是会加大producer端的cpu开销

3）batch.size，设置meigebatch的大小，如果batch太小，会导致频繁网络请求，吞吐量下降；如果batch太大，会导致一条消息需要等待很久才能被发送出去，而且会让内存缓冲区有很大压力，过多数据缓冲在内存里
默认值是：16384，就是16kb，也就是一个batch满了16kb就发送出去，一般在实际生产环境，这个batch的值可以增大一些来提升吞吐量，可以自己压测一下

4）linger.ms，这个值默认是0，意思就是消息必须立即被发送，但是这是不对的，一般设置一个100毫秒之类的，这样的话就是说，这个消息被发送出去后进入一个batch，如果100毫秒内，这个batch满了16kb，自然就会发送出去。但是如果100毫秒内，batch没满，那么也必须把消息发送出去了，不能让消息的发送延迟时间太长，也避免给内存造成过大的一个压力。

```

#### 3.3.3 请求超时

```
1）max.request.size：这个参数用来控制发送出去的消息的大小，默认是1048576字节，也就1mb，这个一般太小了，很多消息可能都会超过1mb的大小，所以需要自己优化调整，把他设置更大一些（企业一般设置成10M）
2）request.timeout.ms：这个就是说发送一个请求出去之后，他有一个超时的时间限制，默认是30秒，如果30秒都收不到响应，那么就会认为异常，会抛出一个TimeoutException来让我们进行处理
```

#### 3.3.4 ACK参数

```
acks参数，其实是控制发送出去的消息的持久化机制的
1）如果acks=0，那么producer根本不管写入broker的消息到底成功没有，发送一条消息出去，立马就可以发送下一条消息，这是吞吐量最高的方式，但是可能消息都丢失了，你也不知道的，但是说实话，你如果真是那种实时数据流分析的业务和场景，就是仅仅分析一些数据报表，丢几条数据影响不大的。会让你的发送吞吐量会提升很多，你发送弄一个batch出，不需要等待人家leader写成功，直接就可以发送下一个batch了，吞吐量很大的，哪怕是偶尔丢一点点数据，实时报表，折线图，饼图。

2）acks=all，或者acks=-1：这个leader写入成功以后，必须等待其他ISR中的副本都写入成功，才可以返回响应说这条消息写入成功了，此时你会收到一个回调通知


3）acks=1：只要leader写入成功，就认为消息成功了，默认给这个其实就比较合适的，还是可能会导致数据丢失的，如果刚写入leader，leader就挂了，此时数据必然丢了，其他的follower没收到数据副本，变成leader

如果要想保证数据不丢失，得如下设置：
a)min.insync.replicas = 2，ISR里必须有2个副本，一个leader和一个follower，最最起码的一个，不能只有一个leader存活，连一个follower都没有了

b)acks = -1，每次写成功一定是leader和follower都成功才可以算做成功，leader挂了，follower上是一定有这条数据，不会丢失

c) retries = Integer.MAX_VALUE，无限重试，如果上述两个条件不满足，写入一直失败，就会无限次重试，保证说数据必须成功的发送给两个副本，如果做不到，就不停的重试，除非是面向金融级的场景，面向企业大客户，或者是广告计费，跟钱的计算相关的场景下，才会通过严格配置保证数据绝对不丢失

```

#### 3.3.5 重试乱序

```
消息重试是可能导致消息的乱序的，因为可能排在你后面的消息都发送出去了，你现在收到回调失败了才在重试，此时消息就会乱序，所以可以使用“max.in.flight.requests.per.connection”参数设置为1，这样可以保证producer同一时间只能发送一条消息
```





## 四、Kafka消费者

### 4.1 Consumer架构

#### 4.1.1 offset管理

每个consumer内存里数据结构保存对每个topic的每个分区的消费offset，定期会提交offset，老版本是写入zk，但是那样高并发请求zk是不合理的架构设计，zk是做分布式系统的协调的，轻量级的元数据存储，不能负责高并发读写，作为数据存储。所以后来就是提交offset发送给内部topic：__consumer_offsets，提交过去的时候，key是group.id+topic+分区号，value就是当前offset的值，每隔一段时间，kafka内部会对这个topic进行compact。也就是每个group.id+topic+分区号就保留最新的那条数据即可。而且因为这个 __consumer_offsets可能会接收高并发的请求，所以默认分区50个，这样如果你的kafka部署了一个大的集群，比如有50台机器，就可以用50台机器来抗offset提交的请求压力，就好很多。

#### 4.1.2 Coordinator

**Coordinator的作用**

每个consumer group都会选择一个broker作为自己的coordinator，他是负责监控这个消费组里的各个消费者的心跳，以及判断是否宕机，然后开启rebalance，根据内部的一个选择机制，会挑选一个对应的Broker，Kafka总会把你的各个消费组均匀分配给各个Broker作为coordinator来进行管理的，consumer group中的每个consumer刚刚启动就会跟选举出来的这个consumer group对应的coordinator所在的broker进行通信，然后由coordinator分配分区给你的这个consumer来进行消费。coordinator会尽可能均匀的分配分区给各个consumer来消费。

**如何选择哪台是coordinator**

首先对消费组的groupId进行hash，接着对__consumer_offsets的分区数量取模，默认是50，可以通过offsets.topic.num.partitions来设置，找到你的这个consumer group的offset要提交到__consumer_offsets的哪个分区。比如说：groupId，“membership-consumer-group” -> hash值（数字）-> 对50取模 -> 就知道这个consumer group下的所有的消费者提交offset的时候是往哪个分区去提交offset，找到__consumer_offsets的一个分区，__consumer_offset的分区的副本数量默认来说1，只有一个leader，然后对这个分区找到对应的leader所在的broker，这个broker就是这个consumer group的coordinator了，consumer接着就会维护一个Socket连接跟这个Broker进行通信。

![39 GroupCoordinator原理剖析](http://img.hurenjieee.com/uPic/39%20GroupCoordinator原理剖析.png)

### 4.2 Rebalance策略

比如我们消费的一个主题有12个分区：
p0,p1,p2,p3,p4,p5,p6,p7,p8,p9,p10,p11
假设我们的消费者组里面有三个消费者
**1.range策略**
range策略就是按照partiton的序号范围
	p0~3             consumer1
	p4~7             consumer2
	p8~11            consumer3
	默认就是这个策略；

**2.round-robin策略**
consumer1:0,3,6,9
consumer2:1,4,7,10
consumer3:2,5,8,11

但是前面的这两个方案有个问题：
	假设consuemr1挂了:p0-5分配给consumer2,p6-11分配给consumer3
	这样的话，原本在consumer2上的的p6,p7分区就被分配到了 consumer3上。

**3.sticky策略**
最新的一个sticky策略，就是说尽可能保证在rebalance的时候，让原本属于这个consumer
的分区还是属于他们，
然后把多余的分区再均匀分配过去，这样尽可能维持原来的分区分配的策略

consumer1：0-3
consumer2:  4-7
consumer3:  8-11 
假设consumer3挂了
consumer1：0-3，+8,9
consumer2: 4-7，+10,11

**Rebalance分代机制**
在rebalance的时候，可能你本来消费了partition3的数据，结果有些数据消费了还没提交offset，结果此时rebalance，把partition3分配给了另外一个cnosumer了，此时你如果提交partition3的数据的offset，能行吗？必然不行，所以每次rebalance会触发一次consumer group generation，分代，每次分代会加1，然后你提交上一个分代的offset是不行的，那个partiton可能已经不属于你了，大家全部按照新的partiton分配方案重新消费数据。

### 4.3 Consumer代码演示

```java
import java.util.Arrays;
import java.util.Properties;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;

import com.alibaba.fastjson.JSONObject;

public class ConsumerDemo {

	private static ExecutorService threadPool = Executors.newFixedThreadPool(20);
	
	public static void main(String[] args) throws Exception {
		KafkaConsumer<String, String> consumer = createConsumer();
		consumer.subscribe(Arrays.asList("order-topic"));  
		try {
			while(true) {  
				ConsumerRecords<String, String> records = consumer.poll(Integer.MAX_VALUE); 
				for(ConsumerRecord<String, String> record : records) {
					JSONObject order = JSONObject.parseObject(record.value()); 
					threadPool.submit(new CreditManageTask(order));
				}
			}
		} catch(Exception e) {
			e.printStackTrace();
			consumer.close();
		}
	}
	
	private static KafkaConsumer<String, String> createConsumer() {
		Properties props = new Properties();
		props.put("bootstrap.servers", "hadoop03:9092,hadoop04:9092,hadoop05:9092");
		props.put("group.id", "test-group");
		props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
		props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
		props.put("heartbeat.interval.ms", 1000); // 这个尽量时间可以短一点
		props.put("session.timeout.ms", 10 * 1000); // 如果说kafka broker在10秒内感知不到一个consumer心跳
		props.put("max.poll.interval.ms", 30 * 1000); // 如果30秒才去执行下一次poll
		// 就会认为那个consumer挂了，此时会触发rebalance
		// 如果说某个consumer挂了，kafka broker感知到了，会触发一个rebalance的操作，就是分配他的分区
		// 给其他的cosumer来消费，其他的consumer如果要感知到rebalance重新分配分区，就需要通过心跳来感知
		// 心跳的间隔一般不要太长，1000，500
		props.put("fetch.max.bytes", 10485760);
		props.put("max.poll.records", 500); // 如果说你的消费的吞吐量特别大，此时可以适当提高一些
		props.put("connection.max.idle.ms", -1); // 不要去回收那个socket连接
		// 开启自动提交，他只会每隔一段时间去提交一次offset
		// 如果你每次要重启一下consumer的话，他一定会把一些数据重新消费一遍
		props.put("enable.auto.commit", "true");
		// 每次自动提交offset的一个时间间隔
		props.put("auto.commit.ineterval.ms", "1000");
		// 每次重启都是从最早的offset开始读取，不是接着上一次
		props.put("auto.offset.reset", "earliest"); 
		

		KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(props);
		return consumer;
	}
	
	static class CreditManageTask implements Runnable {
		private JSONObject order;
		public CreditManageTask(JSONObject order) {
			this.order = order;
		}
		
		@Override
		public void run() {
			System.out.println("对订单进行积分的维护......" + order.toJSONString());    
			// 就可以做一系列的数据库的增删改查的事务操作
		}
	}
}

```

### 4.4 Consumer核心参数

```
【heartbeat.interval.ms】
consumer心跳时间，必须得保持心跳才能知道consumer是否故障了，然后如果故障之后，就会通过心跳下发rebalance的指令给其他的consumer通知他们进行rebalance的操作

【session.timeout.ms】
kafka多长时间感知不到一个consumer就认为他故障了，默认是10秒

【max.poll.interval.ms】
如果在两次poll操作之间，超过了这个时间，那么就会认为这个consume处理能力太弱了，会被踢出消费组，分区分配给别人去消费，一遍来说结合你自己的业务处理的性能来设置就可以了

【fetch.max.bytes】
获取一条消息最大的字节数，一般建议设置大一些

【max.poll.records】
一次poll返回消息的最大条数，默认是500条

【connection.max.idle.ms】
consumer跟broker的socket连接如果空闲超过了一定的时间，此时就会自动回收连接，但是下次消费就要重新建立socket连接，这个建议设置为-1，不要去回收

【auto.offset.reset】
	earliest
		当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，从头开始消费
		topica -> partition0:1000   
				  partitino1:2000  			  
	latest
		当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，从当前位置开始消费
	none
		topic各分区都存在已提交的offset时，从offset后开始消费；只要有一个分区不存在已提交的offset，则抛出异常
		
注：我们生产里面一般设置的是latest

【enable.auto.commit】
这个就是开启自动提交唯一

【auto.commit.ineterval.ms】
这个指的是多久条件一次偏移量
```

## 五、生产环境集群部署

### 5.1 方案背景

假设每天集群需要承载10亿数据。一天24小时，晚上12点到凌晨8点几乎没多少数据。使用二八法则估计，也就是80%的数据（8亿）会在16个小时涌入，而且8亿的80%的数据（6.4亿）会在这16个小时的20%时间（3小时）涌入。QPS计算公式=640000000÷(3*60*60)=6万，也就是说高峰期的时候咱们的Kafka集群要抗住每秒6万的并发。
磁盘空间计算，每天10亿数据，每条50kb，也就是46T的数据。保存2副本，46*2=92T，保留最近3天的数据。故需要 92 * 3 = 276T

### 5.2 QPS角度

部署Kafka，Hadoop，MySQL，大数据核心分布式系统，一般建议大家直接采用物理机，不建议用一些低配置的虚拟机。QPS这个东西，不可能是说，你只要支撑6万QPS，你的集群就刚好支撑6万QPS就可以了。假如说你只要支撑6w QPS，2台物理机绝对绝对够了，单台物理机部署kafka支撑个几万QPS是没问题的。但是这里有一个问题，我们通常是建议，公司预算充足，尽量是让高峰QPS控制在集群能承载的总QPS的30%左右，所以我们搭建的kafka集群能承载的总QPS为20万~30万才是安全的。所以大体上来说，需要5~7台物理机来部署，基本上就很安全了，每台物理机要求吞吐量在每秒几万条数据就可以了，物理机的配置和性能也不需要特别高。

### 5.3 磁盘角度

我们现在需要5台物理机，需要存储276T的数据，所以大概需要每台存储60T的数据，公司一般的配置是11块盘，这样的话，我们一个盘7T就搞定。

SSD就是固态硬盘，比机械硬盘要快，那么到底是快在哪里呢？其实SSD的快主要是快在磁盘随机读写，就要对磁盘上的随机位置来读写的时候，SSD比机械硬盘要快。像比如说是MySQL这种系统，就应该使用SSD了。比如说我们在规划和部署线上系统的MySQL集群的时候，一般来说必须用SSD，性能可以提高很多，这样MySQL可以承载的并发请求量也会高很多，而且SQL语句执行的性能也会提高很多。
Kafka集群，物理机是用昂贵的SSD呢？还是用普通的机械硬盘呢？因为==**写磁盘的时候kafka是顺序写的**==。机械硬盘顺序写的性能机会跟内存读写的性能是差不多的，所以对于Kafka集群来说我们使用机械硬盘就可以了。

### 5.4 内存角度

kafka自身的jvm是用不了过多堆内存的，因为kafka设计就是规避掉用jvm对象来保存数据，避免频繁fullgc导致的问题，所以一般kafka自身的jvm堆内存，分配个10G左右就够了，剩下的内存全部留给os cache。
那服务器需要多少内存够呢？我们估算一下，大概有100个topic，所以要保证有100个topic的leader partition的数据在os chache里。100个topic，一个topic有5个partition。那么总共会有500个partition。每个partition的大小是1G，我们有2个副本，也就是说要把100个topic的leader partition数据都驻留在内存里需要1000G的内存。我们现在有5台服务器，所以平均下来每天服务器需要200G的内存，但是其实partition的数据我们没必要所有的都要驻留在内存里面，只需要25%的数据在内存就行，200G * 0.25 = 50G就可以了。所以一共需要60G的内存，故我们可以挑选64G内存的服务器也行，大不了partition的数据再少一点在内存，当然如果是128G内存那就更好。

### 5.5 CPU角度

CPU规划，主要是看你的这个进程里会有多少个线程，线程主要是依托多核CPU来执行的，如果你的线程特别多，但是CPU核很少，就会导致你的CPU负载很高，会导致整体工作线程执行的效率不太高，我们前面讲过Kafka的Broker的模型。acceptor线程负责去接入客户端的连接请求，但是他接入了之后其实就会把连接分配给多个processor，默认是3个，但是说实话一般生产环境的话呢 ，建议大家还是多加几个，整体可以提升kafka的吞吐量比如说你可以增加到6个，或者是9个。另外就是负责处理请求的线程，是一个线程池，默认是8个线程，在生产集群里，建议大家可以把这块的线程数量稍微多加个2倍~3倍，其实都正常，比如说搞个16个工作线程，24个工作线程。后台会有很多的其他的一些线程，比如说定期清理7天前数据的线程，Controller负责感知和管控整个集群的线程，副本同步拉取数据的线程，这样算下来每个broker起码会有上百个线程。根据经验4个cpu core，一般来说几十个线程，在高峰期CPU几乎都快打满了。8个cpu core，也就能够比较宽裕的支撑几十个线程繁忙的工作。所以Kafka的服务器一般是建议16核，基本上可以hold住一两百线程的工作。当然如果可以给到32 cpu core那就最好不过了。

### 5.6 网卡角度

现在的网基本就是千兆网卡（1GB / s），还有万兆网卡（10GB / s）。kafka集群之间，broker和broker之间是会做数据同步的，因为leader要同步数据到follower上去，他们是在不同的broker机器上的，broker机器之间会进行频繁的数据同步，传输大量的数据。每秒两台broker机器之间大概会传输多大的数据量？高峰期每秒大概会涌入6万条数据，约每天处理10000个请求，每个请求50kb，故每秒约进来488M数据，我们还有副本同步数据，故高峰期的时候需要488M * 2 = 976M/s的网络带宽，所以在高峰期的时候，使用千兆带宽，网络还是非常有压力的。

### 5.7 Kafka集群配置

10亿数据，6w/s的吞吐量，276T的数据，5台物理机
硬盘：11（SAS） * 7T，7200转
内存：64GB/128GB，JVM分配10G，剩余的给os cache
CPU：16核/32核
网络：千兆网卡，万兆更好

### 5.8 集群配置

1. server.properties配置文件核心参数配置

   ```
   【broker.id】
   每个broker都必须自己设置的一个唯一id
   
   【log.dirs】
   这个极为重要，kafka的所有数据就是写入这个目录下的磁盘文件中的，如果说机器上有多块物理硬盘，那么可以把多个目录挂载到不同的物理硬盘上，然后这里可以设置多个目录，这样kafka可以数据分散到多块物理硬盘，多个硬盘的磁头可以并行写，这样可以提升吞吐量。
   
   【zookeeper.connect】
   连接kafka底层的zookeeper集群的
   
   【Listeners】
   broker监听客户端发起请求的端口号
   
   【unclean.leader.election.enable】
   默认是false，意思就是只能选举ISR列表里的follower成为新的leader，1.0版本后才设为false，之前都是true，允许非ISR列表的follower选举为新的leader
   
   【delete.topic.enable】
   默认true，允许删除topic
   
   【log.retention.hours】
   可以设置一下，要保留数据多少个小时，这个就是底层的磁盘文件，默认保留7天的数据，根据自己的需求来就行了
   
   【min.insync.replicas】
   acks=-1（一条数据必须写入ISR里所有副本才算成功），你写一条数据只要写入leader就算成功了，不需要等待同步到follower才算写成功。但是此时如果一个follower宕机了，你写一条数据到leader之后，leader也宕机，会导致数据的丢失。
   ```

2. kafka-start-server.sh中的jvm设置 

   ```
   export KAFKA_HEAP_OPTS=”-Xmx6g -Xms6g -XX:MetaspaceSize=96m -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:G1HeapRegionSize=16M -XX:MinMetaspaceFreeRatio=50 -XX:MaxMetaspaceFreeRatio=80”
   ```

