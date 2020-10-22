# Flink DataStream操作

## 一、常见Transformation操作

### 1.1 map和filter（转换和过滤）

```java
/**
 * 数据源：1 2 3 4 5.....源源不断过来
 * 通过map打印一下接受到数据
 * 通过filter过滤一下数据，我们只需要偶数
 */
public class MapDemo {
        public static void main(String[] args) throws Exception {
            StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
            DataStreamSource<Long> numberStream = env.addSource(new MyNoParalleSource()).setParallelism(1);
            SingleOutputStreamOperator<Long> dataStream = numberStream.map(new MapFunction<Long, Long>() {
                @Override
                public Long map(Long value) throws Exception {
                    System.out.println("接受到了数据："+value);
                    return value;
                }
            });
            SingleOutputStreamOperator<Long> filterDataStream = dataStream.filter(new FilterFunction<Long>() {
                @Override
                public boolean filter(Long number) throws Exception {
                    return number % 2 == 0;
                }
            });

            filterDataStream.print().setParallelism(1);
            env.execute("StreamingDemoWithMyNoPralalleSource");
        }
}
```

### 1.2 flatMap，keyBy和sum（过滤，统计）

```java
/**
 * 滑动窗口实现单词计数
 * 数据源：socket
 * 需求：每隔1秒计算最近2秒单词出现的次数
 *
 * 练习算子：
 * flatMap
 * keyBy:
 *    dataStream.keyBy("someKey") // 指定对象中的 "someKey"字段作为分组key
 *    dataStream.keyBy(0) //指定Tuple中的第一个元素作为分组key
 * sum
 */
public class WindowWordCountJava {
    public static void main(String[] args) throws  Exception {
        int port;
        try{
            ParameterTool parameterTool = ParameterTool.fromArgs(args);
            port = parameterTool.getInt("port");
        }catch (Exception e){
            System.err.println("no port set,user default port 9988");
            port=9988;
        }
        //步骤一：获取flink运行环境（stream）
        StreamExecutionEnvironment env= StreamExecutionEnvironment.getExecutionEnvironment();
        String hostname="10.126.88.226";
        String delimiter="\n";
        //步骤二：获取数据源
        DataStreamSource<String> textStream = env.socketTextStream(hostname, port, delimiter);
        //步骤三：执行transformation操作
        SingleOutputStreamOperator<WordCount> wordCountStream = textStream.flatMap(new FlatMapFunction<String, WordCount>() {
            public void flatMap(String line, Collector<WordCount> out) throws Exception {
                String[] fields = line.split("\t");
                for (String word : fields) {
                    out.collect(new WordCount(word, 1L));
                }
            }
        }).keyBy("word")
                .timeWindow(Time.seconds(2), Time.seconds(1))//每隔1秒计算最近2秒
                .sum("count");

        wordCountStream.print().setParallelism(1);//打印并设置并行度
        //步骤四：运行程序
        env.execute("socket word count");


    }

    public static class WordCount{
        public String word;
        public long count;
        public WordCount(){

        }
        public WordCount(String word,long count){
            this.word=word;
            this.count=count;
        }

        @Override
        public String toString() {
            return "WordCount{" +
                    "word='" + word + '\'' +
                    ", count=" + count +
                    '}';
        }
    }
}
```

### 1.3 union（连接相同类型）

```java
/**
 * 合并多个流，新的流会包含所有流中的数据，但是union是一个限制，就是所有合并的流类型必须是一致的
 */
public class unionDemo {
    public static void main(String[] args) throws Exception {
        //获取Flink的运行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        //获取数据源
        DataStreamSource<Long> text1 = env.addSource(new MyNoParalleSource()).setParallelism(1);//注意：针对此source，并行度只能设置为1
        DataStreamSource<Long> text2 = env.addSource(new MyNoParalleSource()).setParallelism(1);

        //把text1和text2组装到一起
        DataStream<Long> text = text1.union(text2);

        DataStream<Long> num = text.map(new MapFunction<Long, Long>() {
            @Override
            public Long map(Long value) throws Exception {
                System.out.println("原始接收到数据：" + value);
                return value;
            }
        });
        //每2秒钟处理一次数据
        DataStream<Long> sum = num.timeWindowAll(Time.seconds(2)).sum(0);
        //打印结果
        sum.print().setParallelism(1);
        String jobName = unionDemo.class.getSimpleName();
        env.execute(jobName);
    }
}
```



### 1.4 connect(conMapFunction和conFlatMapFunction)（连接不同类型）

```java
/**
 * 和union类似，但是只能连接两个流，两个流的数据类型可以不同，会对两个流中的数据应用不同的处理方法
 */
public class ConnectionDemo {
    public static void main(String[] args) throws Exception {
        //获取Flink的运行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        //获取数据源
        DataStreamSource<Long> text1 = env.addSource(new MyNoParalleSource()).setParallelism(1);//注意：针对此source，并行度只能设置为1

        DataStreamSource<Long> text2 = env.addSource(new MyNoParalleSource()).setParallelism(1);
        SingleOutputStreamOperator<String> text2_str = text2.map(new MapFunction<Long, String>() {
            @Override
            public String map(Long value) throws Exception {
                return "str_" + value;
            }
        });

        ConnectedStreams<Long, String> connectStream = text1.connect(text2_str);

        SingleOutputStreamOperator<Object> result = connectStream.map(new CoMapFunction<Long, String, Object>() {
            @Override
            public Object map1(Long value) throws Exception {
                return value;
            }

            @Override
            public Object map2(String value) throws Exception {
                return value;
            }
        });

        //打印结果
        result.print().setParallelism(1);
        String jobName = ConnectionDemo.class.getSimpleName();
        env.execute(jobName);
    }
}
```

### 1.5 Split和Select（分区）

```java
/**
 *  根据规则把一个数据流切分为多个流
 应用场景：
 * 可能在实际工作中，源数据流中混合了多种类似的数据，多种类型的数据处理规则不一样，所以就可以在根据一定的规则，
 * 把一个数据流切分成多个数据流，这样每个数据流就可以使用不用的处理逻辑了
 */
public class SplitDemo {
    public static void main(String[] args) throws  Exception {
        //获取Flink的运行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        //获取数据源
        DataStreamSource<Long> text = env.addSource(new MyNoParalleSource()).setParallelism(1);//注意：针对此source，并行度只能设置为1
        //对流进行切分，按照数据的奇偶性进行区分
        SplitStream<Long> splitStream = text.split(new OutputSelector<Long>() {
            @Override
            public Iterable<String> select(Long value) {
                ArrayList<String> outPut = new ArrayList<>();
                if (value % 2 == 0) {
                    outPut.add("even");//偶数
                } else {
                    outPut.add("odd");//奇数
                }
                return outPut;
            }
        });

        //选择一个或者多个切分后的流
        DataStream<Long> evenStream = splitStream.select("even");
        DataStream<Long> oddStream = splitStream.select("odd");
        DataStream<Long> moreStream = splitStream.select("odd","even");

        //打印结果
        evenStream.print().setParallelism(1);
        String jobName = SplitDemo.class.getSimpleName();
        env.execute(jobName);

    }
}
```

## 二、常见sink操作

### 2.1 print() / printToErr() 

打印每个元素的toString()方法的值到标准输出或者标准错误输出流中

### 2.2 writeAsText()

```java
/**
 * 数据源：1 2 3 4 5.....源源不断过来
 * 通过map打印一下接受到数据
 * 通过filter过滤一下数据，我们只需要偶数
 */
public class WriteTextDemo {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        DataStreamSource<Long> numberStream = env.addSource(new MyNoParalleSource()).setParallelism(1);
        SingleOutputStreamOperator<Long> dataStream = numberStream.map(new MapFunction<Long, Long>() {
            @Override
            public Long map(Long value) throws Exception {
                System.out.println("接受到了数据："+value);
                return value;
            }
        });
        SingleOutputStreamOperator<Long> filterDataStream = dataStream.filter(new FilterFunction<Long>() {
            @Override
            public boolean filter(Long number) throws Exception {
                return number % 2 == 0;
            }
        });
   filterDataStream.writeAsText("D:\\flinklesson\\src\\output\\test").setParallelism(1);
        env.execute("StreamingDemoWithMyNoPralalleSource");
    }
}
```

### 2.3 Flink提供的sink

- [Apache Kafka](https://ci.apache.org/projects/flink/flink-docs-release-1.9/dev/connectors/kafka.html) (source/sink)
- [Apache Cassandra](https://ci.apache.org/projects/flink/flink-docs-release-1.9/dev/connectors/cassandra.html) (sink)
- [Amazon Kinesis Streams](https://ci.apache.org/projects/flink/flink-docs-release-1.9/dev/connectors/kinesis.html) (source/sink)
- [Elasticsearch](https://ci.apache.org/projects/flink/flink-docs-release-1.9/dev/connectors/elasticsearch.html) (sink)
- [Hadoop FileSystem](https://ci.apache.org/projects/flink/flink-docs-release-1.9/dev/connectors/filesystem_sink.html) (sink)
- [RabbitMQ](https://ci.apache.org/projects/flink/flink-docs-release-1.9/dev/connectors/rabbitmq.html) (source/sink)
- [Apache NiFi](https://ci.apache.org/projects/flink/flink-docs-release-1.9/dev/connectors/nifi.html) (source/sink)
- [Twitter Streaming API](https://ci.apache.org/projects/flink/flink-docs-release-1.9/dev/connectors/twitter.html) (source)
- [Google PubSub](https://ci.apache.org/projects/flink/flink-docs-release-1.9/dev/connectors/pubsub.html) (source/sink)

### 2.4 自定义sink

增加pom依赖

```java
     <dependency>
            <groupId>org.apache.bahir</groupId>
            <artifactId>flink-connector-redis_2.11</artifactId>
            <version>1.0</version>
     </dependency>
```

自定义redis sink

```java
/**
 * 把数据写入redis
 */
public class SinkForRedisDemo {
    public static void main(String[] args) throws  Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        DataStreamSource<String> text = env.socketTextStream("hadoop100", 9000, "\n");
        //lpsuh l_words word
        //对数据进行组装,把string转化为tuple2<String,String>
        DataStream<Tuple2<String, String>> l_wordsData = text.map(new MapFunction<String, Tuple2<String, String>>() {
            @Override
            public Tuple2<String, String> map(String value) throws Exception {
                return new Tuple2<>("l_words", value);
            }
        });
        //创建redis的配置
        FlinkJedisPoolConfig conf = new FlinkJedisPoolConfig.Builder().setHost("hadoop110").setPort(6379).build();

        //创建redissink
        RedisSink<Tuple2<String, String>> redisSink = new RedisSink<>(conf, new MyRedisMapper());
        l_wordsData.addSink(redisSink);
        env.execute("StreamingDemoToRedis");

    }

    public static class MyRedisMapper implements RedisMapper<Tuple2<String, String>> {
        //表示从接收的数据中获取需要操作的redis key
        @Override
        public String getKeyFromData(Tuple2<String, String> data) {
            return data.f0;
        }
        //表示从接收的数据中获取需要操作的redis value
        @Override
        public String getValueFromData(Tuple2<String, String> data) {
            return data.f1;
        }

        @Override
        public RedisCommandDescription getCommandDescription() {
            return new RedisCommandDescription(RedisCommand.LPUSH);
        }
    }
}
```
