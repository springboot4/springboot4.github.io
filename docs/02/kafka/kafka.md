# kafka消息队列



## 概念和基本架构



### Kafka介绍

Kafka是最初由Linkedin公司开发，是一个分布式、分区的、多副本的、多生产者、多订阅者，基于zookeeper协调的分布式日志系统（也可以当做MQ系统），常见可以用于web/nginx日志、访问日志，消息服务等等，Linkedin于2010年贡献给了Apache基金会并成为顶级开源项目。

主要应用场景是：日志收集系统和消息系统。

Kafka主要设计目标如下：

- 以时间复杂度为O(1)的方式提供消息持久化能力，即使对TB级以上数据也能保证常数时间的访问性能。

- 高吞吐率。即使在非常廉价的商用机器上也能做到单机支持每秒100K条消息的传输。

- 支持Kafka Server间的消息分区，及分布式消费，同时保证每个partition内的消息顺序传输。

- 同时支持离线数据处理和实时数据处理。

- 支持在线水平扩展

![](img\1.png)

有两种主要的消息传递模式：**点对点传递模式、发布订阅模式**。大部分的消息系统选用发布-订阅模式。**Kafka就是一种发布订阅模式**。

对于消息中间件，消息分推拉两种模式。Kafka只有消息的拉取，没有推送，可以通过轮询实现消息的推送。

1.  Kafka在一个或多个可以跨越多个数据中心的服务器上作为集群运行。

2.  Kafka集群中按照主题分类管理，一个主题可以有多个分区，一个分区可以有多个副本分区。

3.  每个记录由一个键，一个值和一个时间戳组成。

Kafka具有四个核心API： 

1.  Producer API：允许应用程序将记录流发布到一个或多个Kafka主题。

2.  Consumer API：允许应用程序订阅一个或多个主题并处理为其生成的记录流。

3. Streams API：允许应用程序充当流处理器，使用一个或多个主题的输入流，并生成一个或多个输出主题的输出流，从而有效地将输入流转换为输出流。

4. Connector API：允许构建和运行将Kafka主题连接到现有应用程序或数据系统的可重用生产者或使用者。例如，关系数据库的连接器可能会捕获对表的所有更改。

###  Kafka优势

1. 高吞吐量：单机每秒处理几十上百万的消息量。即使存储了许多TB的消息，它也保持稳定的性

能。

2. 高性能：单节点支持上千个客户端，并保证零停机和零数据丢失。

3. 持久化数据存储：将消息持久化到磁盘。通过将数据持久化到硬盘以及replication防止数据丢

失。

- 零拷贝

- 顺序读，顺序写

- 利用Linux的页缓存

4. 分布式系统，易于向外扩展。所有的Producer、Broker和Consumer都会有多个，均为分布式的。无需停机即可扩展机器。多个Producer、Consumer可能是不同的应用。

5. 可靠性 - Kafka是分布式，分区，复制和容错的。

6. 客户端状态维护：消息被处理的状态是在Consumer端维护，而不是由server端维护。当失败时能自动平衡。

7. 支持online和offline的场景。

8. 支持多种客户端语言。Kafka支持Java、.NET、PHP、Python等多种语言。

### Kafka应用场景

**日志收集：**一个公司可以用Kafka可以收集各种服务的Log，通过Kafka以统一接口服务的方式开放给各种Consumer；

**消息系统：**解耦生产者和消费者、缓存消息等；

**用户活动跟踪：**Kafka经常被用来记录Web用户或者App用户的各种活动，如浏览网页、搜索、点击等活动，这些活动信息被各个服务器发布到Kafka的Topic中，然后消费者通过订阅这些Topic来做实时的监控分析，亦可保存到数据库；

**运营指标：**Kafka也经常用来记录运营监控数据。包括收集各种分布式应用的数据，生产各种操作的集中反馈，比如报警和报告；

**流式处理：**比如Spark Streaming和Storm。

### 基本架构

**消息和批次**

Kafka的数据单元称为消息。可以把消息看成是数据库里的一个“数据行”或一条“记录”。消息由字节数组组成。

消息有键，键也是一个字节数组。当消息以一种可控的方式写入不同的分区时，会用到键。

为了提高效率，消息被分批写入Kafka。批次就是一组消息，这些消息属于同一个主题和分区。

把消息分成批次可以减少网络开销。批次越大，单位时间内处理的消息就越多，单个消息的传输时间就越长。批次数据会被压缩，这样可以提升数据的传输和存储能力，但是需要更多的计算处理。

**模式**

消息模式（schema）有许多可用的选项，以便于理解。如JSON和XML，但是它们缺乏强类型处理能力。Kafka的许多开发者喜欢使用Apache Avro。Avro提供了一种紧凑的序列化格式，模式和消息体分开。当模式发生变化时，不需要重新生成代码，它还支持强类型和模式进化，其版本既向前兼容，也向后兼容。

数据格式的一致性对Kafka很重要，因为它消除了消息读写操作之间的耦合性。

**主题和分区**

Kafka的消息通过主题进行分类。主题可比是数据库的表或者文件系统里的文件夹。主题可以被分为若干分区，一个主题通过分区分布于Kafka集群中，提供了横向扩展的能力。

![](img\2.png)

**生产者和消费者**

生产者创建消息。消费者消费消息。

一个消息被发布到一个特定的主题上。

生产者在默认情况下把消息均衡地分布到主题的所有分区上：

1. 直接指定消息的分区

2. 根据消息的key散列取模得出分区
3. 轮询指定分区。

消费者通过偏移量来区分已经读过的消息，从而消费消息。

消费者是消费组的一部分。消费组保证每个分区只能被一个消费者使用，避免重复消费。

![](img\3.png)

**broker和集群**

一个独立的Kafka服务器称为broker。broker接收来自生产者的消息，为消息设置偏移量，并提交消息到磁盘保存。broker为消费者提供服务，对读取分区的请求做出响应，返回已经提交到磁盘上的消息。**单个broker**可以轻松处理**数千个分区**以及**每秒百万级**的消息量

![](img\4.png)

每个集群都有一个broker是集群控制器（自动从集群的活跃成员中选举出来）。

控制器负责管理工作：

- 将分区分配给broker

- 监控broker

集群中一个分区属于一个**broker**，该broker称为**分区首领**。

**一个分区**可以分配给**多个broker**，此时会发生分区复制。

分区的复制提供了**消息冗余，高可用**。**副本分区**不负责处理消息的读写。

### 核心概念

#### **Producer**

生产者创建消息。

该角色将消息发布到Kafka的topic中。broker接收到生产者发送的消息后，broker将该消息**追加**到当前用于追加数据的 segment 文件中。

一般情况下，一个消息会被发布到一个特定的主题上。

1. 默认情况下通过轮询把消息均衡地分布到主题的所有分区上。

2. 在某些情况下，生产者会把消息直接写到指定的分区。这通常是通过消息键和分区器来实现的，分区器为键生成一个散列值，并将其映射到指定的分区上。这样可以保证包含同一个键的消息会被写到同一个分区上。

3. 生产者也可以使用自定义的分区器，根据不同的业务规则将消息映射到分区。

#### Consumer

消费者读取消息。

1. 消费者订阅一个或多个主题，并按照消息生成的顺序读取它们。

2. 消费者通过检查消息的**偏移量**来区分已经读取过的消息。偏移量是另一种元数据，它是一个**不断递增的整数值**，在创建消Kafka 会把它添加到消息里。在给定的**分区**里，每个消息的偏移量都是**唯一**的。消费者把每个分区最后读取的消息偏移量保存在Zookeeper 或**Kafka**上，如果消费者关闭或重启，它的读取状态不会丢失。

3. 消费者是**消费组**的一部分。群组保证每个分区只能被一个消费者使用。

4. 如果一个消费者失效，消费组里的其他消费者可以接管失效消费者的工作，再平衡，分区重新分配。

![](img\5.png)

####  Broker

一个独立的Kafka 服务器被称为broker。

broker 为消费者提供服务，对读取分区的请求作出响应，返回已经提交到磁盘上的消息。

1. 如果某topic有N个partition，集群有N个broker，那么每个broker存储该topic的一个partition。 

2. 如果某topic有N个partition，集群有(N+M)个broker，那么其中有N个broker存储该topic的一个partition，剩下的M个broker不存储该topic的partition数据。

3. 如果某topic有N个partition，集群中broker数目少于N个，那么一个broker存储该topic的一个或多个partition。在实际生产环境中，尽量避免这种情况的发生，这种情况容易导致Kafka集群数据不均衡。

broker 是集群的组成部分。每个集群都有一个broker 同时充当了集群控制器的角色（自动从集群的活跃成员中选举出来）。

控制器负责管理工作，包括将分区分配给broker 和监控broker。

在集群中，一个分区从属于一个broker，该broker 被称为分区的首领。

![](img\6.png)

#### Topic

每条发布到Kafka集群的消息都有一个类别，这个类别被称为Topic。

物理上不同Topic的消息分开存储。

主题就好比数据库的表，尤其是分库分表之后的逻辑表。



####  Partition

1. 主题可以被分为若干个分区，一个分区就是一个提交日志。

2. 消息以追加的方式写入分区，然后以先入先出的顺序读取。

3. 无法在整个主题范围内保证消息的顺序，但可以保证消息在单个分区内的顺序。

4. Kafka 通过分区来实现数据冗余和伸缩性。

5. 在需要严格保证消息的消费顺序的场景下，需要将partition数目设为1。

![](img\7.png)

#### Replicas

Kafka 使用主题来组织数据，每个主题被分为若干个分区，每个分区有多个副本。那些副本被保存在broker 上，每个broker 可以保存成百上千个属于不同主题和分区的副本。

副本有以下两种类型：

- 首领副本

  ​	每个分区都有一个首领副本。为了保证一致性，所有生产者请求和消费者请求都会经过这个副本。

- 跟随者副本

  ​	首领以外的副本都是跟随者副本。跟随者副本不处理来自客户端的请求，它们唯一的任务就是从首领那里复制消息，保持与首领一致的状态。如果首领发生崩溃，其中的一个跟随者会被提升为新首领。

####  Offset

**生产者Offset**

消息写入的时候，每一个分区都有一个offset，这个offset就是生产者的offset，同时也是这个分区的最新最大的offset。

有些时候没有指定某一个分区的offset，这个工作kafka帮我们完成。

![](img\8.png)

**消费者Offset**

![](img\9.png)

这是某一个分区的offset情况，生产者写入的offset是最新最大的值是12，而当Consumer A进行消费时，从0开始消费，一直消费到了9，消费者的offset就记录在9，Consumer B就纪录在了11。等下一次他们再来消费时，他们可以选择接着上一次的位置消费，当然也可以选择从头消费，或者跳到最近的记录并从“现在”开始消费。

#### 副本

Kafka通过副本保证高可用。副本分为首领副本(Leader)和跟随者副本(Follower)。

跟随者副本包括同步副本和不同步副本，在发生首领副本切换的时候，只有同步副本可以切换为首领副本。

##### AR

分区中的所有副本统称为**AR**（Assigned Repllicas）。

**AR=ISR+OSR**



#####  ISR

所有与leader副本保持一定程度同步的副本（包括Leader）组成**ISR**（In-Sync Replicas），ISR集合是AR集合中的一个子集。消息会先发送到leader副本，然后follower副本才能从leader副本中拉取消息进行同步，同步期间内follower副本相对于leader副本而言会有一定程度的滞后。前面所说的“一定程度”是指可以忍受的滞后范围，这个范围可以通过参数进行配置。

##### OSR

与leader副本同步滞后过多的副本（不包括leader）副本，组成**OSR**(Out-Sync Relipcas)。在正常情况下，所有的follower副本都应该与leader副本保持一定程度的同步，即AR=ISR,OSR集合为空。

#####  HW

HW是High Watermak的缩写， 俗称高水位，它表示了一个特定消息的偏移量（offset），消费之

只能拉取到这个offset之前的消息。

#####  LEO

LEO是Log End Offset的缩写，它表示了当前日志文件中**下一条待写入**消息的offset。

![](img\10.png)

##  Kafka安装与配置

###  Java环境为前提

- 上传jdk-8u261-linux-x64.rpm到服务器并安装：

  ```shell
  rpm -ivh jdk-8u261-linux-x64.rpm
  ```

- 配置环境变量：

  ```shell
  vim /etc/profile
  ```

  ![](img\11.png)

![](img\12.png)

###  Zookeeper的安装配置

1、上传zookeeper-3.4.14.tar.gz到服务器

2、解压到/opt： 

```shell
tar -zxf zookeeper-3.4.14.tar.gz -C /opt
cd /opt/zookeeper-3.4.14/conf 
# 复制zoo_sample.cfg命名为zoo.cfg
cp zoo_sample.cfg zoo.cfg 
# 编辑zoo.cfg文件
vim zoo.cfg
```

3、修改Zookeeper保存数据的目录，dataDir：dataDir=/var/lagou/zookeeper/data

4、编辑/etc/profile：

- 设置环境变量ZOO_LOG_DIR，指定Zookeeper保存日志的位置；

- ZOOKEEPER_PREFIX指向Zookeeper的解压目录；

- 将Zookeeper的bin目录添加到PATH中：

![](img\13.png)

5、使配置生效：

```shell
source /etc/profile
```

6、验证：

![](img\14.png)

### Kafka的安装与配置

1、上传kafka_2.12-1.0.2.tgz到服务器并解压：

```shell
tar -zxf kafka_2.12-1.0.2.tgz -C /opt
```

2、配置环境变量并生效：

```shell
vim /etc/profile
```

![](img\15.png)

3、配置/opt/kafka_2.12-1.0.2/config中的server.properties文件：

Kafka连接Zookeeper的地址，此处使用本地启动的Zookeeper实例，连接地址是localhost:2181，后面的 myKafka 是Kafka在Zookeeper中的根节点路径：

![](img\16.png)

4、启动Zookeeper： 

```shell
zkServer.sh start
```

5、确认Zookeeper的状态：

![](img\17.png)

6、启动Kafka：

进入Kafka安装的根目录，执行如下命令：

![](img\18.png)

启动成功，可以看到控制台输出的最后一行的started状态：

![](img\19.png)

7、查看Zookeeper的节点：

![](img\20.png)

8、此时Kafka是前台模式启动，要停止，使用Ctrl+C。

如果要后台启动，使用命令：

```shell
kafka-server-start.sh -daemon config/server.properties
```

![](img\22.png)

查看Kafka的后台进程：

```shell
ps aux | grep kafka
```

停止后台运行的Kafka：

![](img\23.png)

### 生产与消费

1、kafka-topics.sh 用于管理主题。

```shell
# 列出现有的主题 
[root@node1 ~]# kafka-topics.sh --list --zookeeper localhost:2181/myKafka 
# 创建主题，该主题包含一个分区，该分区为Leader分区，它没有Follower分区副本。 
[root@node1 ~]# kafka-topics.sh --zookeeper localhost:2181/myKafka --create --topic topic_1 --partitions 1 --replication-factor 1 
# 查看分区信息
[root@node1 ~]# kafka-topics.sh --zookeeper localhost:2181/myKafka --list
# 查看指定主题的详细信息
[root@node1 ~]# kafka-topics.sh --zookeeper localhost:2181/myKafka -- describe --topic topic_1 
# 删除指定主题 
[root@node1 ~]# kafka-topics.sh --zookeeper localhost:2181/myKafka --delete --topic topic_1
```

2、kafka-console-producer.sh用于生产消息：

```shell
# 开启生产者 
[root@node1 ~]# kafka-console-producer.sh --topic topic_1 --broker-list localhost:9020
```

3、kafka-console-consumer.sh用于消费消息：

```shell
# 开启消费者 
[root@node1 ~]# kafka-console-consumer.sh --bootstrap-server localhost:9092 - -topic topic_1
# 开启消费者方式二，从头消费，不按照偏移量消费 
[root@node1 ~]# kafka-console-consumer.sh --bootstrap-server localhost:9092 - -topic topic_1 --from-beginning
```

###  Kafka开发实战

####  消息的发送与接收

![](img\24.png)

生产者主要的对象有： KafkaProducer ， ProducerRecord 。

其中 KafkaProducer 是用于发送消息的类， ProducerRecord 类用于封装Kafka的消息。

KafkaProducer 的创建需要指定的参数和含义：

![](img\25.png)

其他参数可以从 org.apache.kafka.clients.producer.ProducerConfig 中找到。我们后面的内容会介绍到。

消费者生产消息后，需要broker端的确认，可以同步确认，也可以异步确认。

同步确认效率低，异步确认效率高，但是需要设置回调对象。



生产者：

```java
public class MyProducer1 {
    public static void main(String[] args) throws InterruptedException, ExecutionException, TimeoutException {
        Map<String, Object> configs = new HashMap<>();
        // 设置连接Kafka的初始连接用到的服务器地址
        // 如果是集群，则可以通过此初始连接发现集群中的其他broker
        configs.put("bootstrap.servers", "node1:9092");
        // 设置key的序列化器
        configs.put("key.serializer", "org.apache.kafka.common.serialization.IntegerSerializer");
        // 设置value的序列化器
        configs.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        configs.put("acks", "1");
        KafkaProducer<Integer, String> producer = new KafkaProducer<Integer, String>(configs);
        // 用于封装Producer的消息
        ProducerRecord<Integer, String> record
                = new ProducerRecord<Integer, String>(
                "topic_1", // 主题名称 
                0,// 分区编号，现在只有一个分区，所以是0 
                0, // 数字作为key
                "message 0" // 字符串作为value 
        );
        // 发送消息，同步等待消息的确认 
        producer.send(record).get(3_000, TimeUnit.MILLISECONDS);
        // 关闭生产者 
        producer.close();
    }
}

```

生产者2： 

```java
public class MyProducer2 {
    public static void main(String[] args) {
        Map<String, Object> configs = new HashMap<>();
        configs.put("bootstrap.servers", "node1:9092");
        configs.put("key.serializer", "org.apache.kafka.common.serialization.IntegerSerializer");
        configs.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        KafkaProducer<Integer, String> producer = new KafkaProducer<Integer, String>(configs);
        ProducerRecord<Integer, String> record = new ProducerRecord<Integer, String>("topic_1", 0, 1, "lagou message 2");
        // 使用回调异步等待消息的确认
        producer.send(record, new Callback() {
            @Override
            public void onCompletion(RecordMetadata metadata, Exception exception) {
                if (exception == null) {
                    System.out.println("主题：" + metadata.topic() + "\n" + "分区：" + metadata.partition() + "\n" + "偏移量：" + metadata.offset() + "\n" + "序列化的key字节：" + metadata.serializedKeySize() + "\n" + "序列化的value字节：" + metadata.serializedValueSize() + "\n" + "时间戳：" + metadata.timestamp());
                } else {
                    System.out.println("有异常：" + exception.getMessage());
                }
            }
        });
        // 关闭连接
        producer.close();
    }
}
```

生产者3：

```java
public class MyProducer3 {
    public static void main(String[] args) {
        Map<String, Object> configs = new HashMap<>();
        configs.put("bootstrap.servers", "node1:9092");
        configs.put("key.serializer", "org.apache.kafka.common.serialization.IntegerSerializer");
        configs.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        KafkaProducer<Integer, String> producer = new KafkaProducer<Integer, String>(configs);
        for (int i = 100; i < 200; i++) {
            ProducerRecord<Integer, String> record = new ProducerRecord<Integer, String>("topic_1", 0, i, "lagou message " + i);
            // 使用回调异步等待消息的确认
            producer.send(record, new Callback() {
                @Override
                public void onCompletion(RecordMetadata metadata, Exception exception) {
                    if (exception == null) {
                        System.out.println("主题：" + metadata.topic() + "\n" + "分区：" + metadata.partition() + "\n" + "偏移量：" + metadata.offset() + "\n" + "序列化的key字节：" + metadata.serializedKeySize() + "\n" + "序列化的value字节：" + metadata.serializedValueSize() + "\n" + "时间戳：" + metadata.timestamp());
                    } else {
                        System.out.println("有异常：" + exception.getMessage());
                    }
                }
            });
        }
        // 关闭连接 
        producer.close();
    }
}
```

消息消费流程：

Kafka不支持消息的推送，我们可以自己实现。

Kafka采用的是消息的拉取（poll方法）

消费者主要的对象有： KafkaConsumer 用于消费消息的类。

KafkaConsumer 的创建需要指定的参数和含义：

![](img\26.png)

ConsumerConfig类中包含了所有的可以给KfakaConsumer配置的参数。



消费者：

```java
public class MyConsumer1 {
    public static void main(String[] args) {
        Map<String, Object> configs = new HashMap<>();
        // 指定bootstrap.servers属性作为初始化连接Kafka的服务器。
        // 如果是集群，则会基于此初始化连接发现集群中的其他服务器。
        configs.put("bootstrap.servers", "node1:9092");
        // key的反序列化器
        configs.put("key.deserializer", "org.apache.kafka.common.serialization.IntegerDeserializer");
        // value的反序列化器
        configs.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        configs.put("group.id", "consumer.demo");
        // 创建消费者对象
        KafkaConsumer<Integer, String> consumer = new KafkaConsumer<Integer, String>(configs);
        // final Pattern pattern = Pattern.compile("topic_\\d");
        final Pattern pattern = Pattern.compile("topic_[0-9]");
        // 消费者订阅主题或分区
        // consumer.subscribe(pattern);
        // consumer.subscribe(pattern, new ConsumerRebalanceListener() {
        final List<String> topics = Arrays.asList("topic_1");
        consumer.subscribe(topics, new ConsumerRebalanceListener() {
            @Override
            public void onPartitionsRevoked(Collection<TopicPartition> partitions) {
                partitions.forEach(tp -> {
                    System.out.println("剥夺的分区：" + tp.partition());
                });
            }

            @Override
            public void onPartitionsAssigned(Collection<TopicPartition> partitions) {
                partitions.forEach(tp -> {
                    System.out.println(tp.partition());
                });
            }
        });
        // 拉取订阅主题的消息
        final ConsumerRecords<Integer, String> records = consumer.poll(3_000);
        // 获取topic_1主题的消息
        final Iterable<ConsumerRecord<Integer, String>> topic1Iterable = records.records("topic_1");
        // 遍历topic_1主题的消息
        topic1Iterable.forEach(record -> {
            System.out.println("========================================");
            System.out.println("消息头字段：" + Arrays.toString(record.headers().toArray()));
            System.out.println("消息的key：" + record.key());
            System.out.println("消息的偏移量：" + record.offset());
            System.out.println("消息的分区号：" + record.partition());
            System.out.println("消息的序列化key字节数：" + record.serializedKeySize());
            System.out.println("消息的序列化value字节数：" + record.serializedValueSize());
            System.out.println("消息的时间戳：" + record.timestamp());
            System.out.println("消息的时间戳类型：" + record.timestampType());
            System.out.println("消息的主题：" + record.topic());
            System.out.println("消息的值：" + record.value());
        });
        // 关闭消费者
        consumer.close();
    }
}
```

#### SpringBoot Kafka

1. pom.xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.8.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.lagou.kafka.demo</groupId>
    <artifactId>demo-02-springboot</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>demo-02-springboot</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

2. application.properties

```properties
spring.application.name=springboot-kafka-02 
server.port=8080 
# 用于建立初始连接的broker地址 
spring.kafka.bootstrap-servers=node1:9092
# producer用到的key和value的序列化类 
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.IntegerSerializer spring.kafka.producer.value- serializer=org.apache.kafka.common.serialization.StringSerializer
# 默认的批处理记录数 
spring.kafka.producer.batch-size=16384 
# 32MB的总发送缓存
spring.kafka.producer.buffer-memory=33554432 
# consumer用到的key和value的反序列化类
spring.kafka.consumer.key- deserializer=org.apache.kafka.common.serialization.IntegerDeserializer spring.kafka.consumer.value- deserializer=org.apache.kafka.common.serialization.StringDeserializer 
# consumer的消费组id 
spring.kafka.consumer.group-id=spring-kafka-02-consumer
# 是否自动提交消费者偏移量 
spring.kafka.consumer.enable-auto-commit=true 
# 每隔100ms向broker提交一次偏移量 
spring.kafka.consumer.auto-commit-interval=100 
# 如果该消费者的偏移量不存在，则自动设置为最早的偏移量 
spring.kafka.consumer.auto-offset-reset=earliest
```

3.  KafkaConfig.java

```java
@Configuration
public class KafkaConfig {
    @Bean
    public NewTopic topic1() {
        return new NewTopic("ntp-01", 5, (short) 1);
    }

    @Bean
    public NewTopic topic2() {
        return new NewTopic("ntp-02", 3, (short) 1);
    }
}
```

4. KafkaSyncProducerController.java

```java
@RestController
public class KafkaSyncProducerController {
    @Autowired
    private KafkaTemplate template;

    @RequestMapping("send/sync/{message}")
    public String sendSync(@PathVariable String message) {
        ListenableFuture future = template.send(new ProducerRecord<Integer, String>("topic-spring-02", 0, 1, message));
        try {
            // 同步等待broker的响应
            Object o = future.get();
            SendResult<Integer, String> result = (SendResult<Integer, String>) o;
            System.out.println(result.getRecordMetadata().topic() + result.getRecordMetadata().partition() + result.getRecordMetadata().offset());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
        return "success";
    }
}
        
```

5. KafkaAsyncProducerController

   ```java
   @RestController
   public class KafkaAsyncProducerController {
       @Autowired
       private KafkaTemplate<Integer, String> template;
   
       @RequestMapping("send/async/{message}")
       public String asyncSend(@PathVariable String message) {
           ProducerRecord<Integer, String> record = new ProducerRecord<Integer, String>("topic-spring-02", 0, 3, message);
           ListenableFuture<SendResult<Integer, String>> future = template.send(record);
           // 添加回调，异步等待响应
           future.addCallback(new ListenableFutureCallback<SendResult<Integer, String>>() {
               @Override
               public void onFailure(Throwable throwable) {
                   System.out.println("发送失败: " + throwable.getMessage());
               }
   
               @Override
               public void onSuccess(SendResult<Integer, String> result) {
                   System.out.println("发送成功：" + result.getRecordMetadata().topic() + "\t" + result.getRecordMetadata().partition() + "\t" + result.getRecordMetadata().offset());
               }
           });
           return "success";
       }
   }
   ```

6.  MyConsumer.java

   ```java
   @Component
   public class MyConsumer {
       @KafkaListener(topics = "topic-spring-02")
       public void onMessage(ConsumerRecord<Integer, String> record) {
           Optional<ConsumerRecord<Integer, String>> optional = Optional.ofNullable(record);
           if (optional.isPresent()) {
               System.out.println(record.topic() + "\t" + record.partition() + "\t" + record.offset() + "\t" + record.key() + "\t" + record.value());
           }
       }
   }
   ```

   

### 服务端参数配置

$KAFKA_HOME/config/server.properties文件中的配置。

#### zookeeper.connect

该参数用于配置Kafka要连接的Zookeeper/集群的地址。

它的值是一个字符串，使用逗号分隔Zookeeper的多个地址。Zookeeper的单个地址是 host:port形式的，可以在最后添加Kafka在Zookeeper中的根节点路径。

如：

```shell
zookeeper.connect=node2:2181,node3:2181,node4:2181/myKafka
```

![](img\27.png)

#### listeners

用于指定当前Broker向外发布服务的地址和端口。

与 advertised.listeners 配合，用于做内外网隔离。

![](img\28.png)

**内外网隔离配置：**

**listener.security.protocol.map**

监听器名称和安全协议的映射配置。

比如，可以将内外网隔离，即使它们都使用SSL。

listener.security.protocol.map=INTERNAL:SSL,EXTERNAL:SSL

每个监听器的名称只能在map中出现一次。

**inter.broker.listener.name**

用于配置broker之间通信使用的监听器名称，该名称必须在advertised.listeners列表中。

inter.broker.listener.name=EXTERNAL

**listeners**

用于配置broker监听的URI以及监听器名称列表，使用逗号隔开多个URI及监听器名称。

如果监听器名称代表的不是安全协议，必须配置listener.security.protocol.map。

每个监听器必须使用不同的网络端口。

**advertised.listeners**

需要将该地址发布到zookeeper供客户端使用，如果客户端使用的地址与listeners配置不同。

可以在zookeeper的 get /myKafka/brokers/ids/<broker.id> 中找到。

在IaaS环境，该条目的网络接口得与broker绑定的网络接口不同。

如果不设置此条目，就使用listeners的配置。跟listeners不同，该条目不能使用0.0.0.0网络端口。

advertised.listeners的地址必须是listeners中配置的或配置的一部分。

**典型配置：**

![](img\29.png)

####  broker.id

该属性用于唯一标记一个Kafka的Broker，它的值是一个任意integer值。

当Kafka以分布式集群运行的时候，尤为重要。

最好该值跟该Broker所在的物理主机有关的，如主机名为 host1.lagou.com ，则 broker.id=1 ，

如果主机名为 192.168.100.101 ，则 broker.id=101 等等。

![](img\30.png)

#### log.dir

通过该属性的值，指定Kafka在磁盘上保存消息的日志片段的目录。

它是一组用逗号分隔的本地文件系统路径。

如果指定了多个路径，那么broker 会根据“最少使用”原则，把同一个分区的日志片段保存到同一个路径下。

broker 会往拥有最少数目分区的路径新增分区，而不是往拥有最小磁盘空间的路径新增分区。

![](img\31.png)



## Kafka高级特性解析

### 生产者

#### 消息发送

##### 数据生产流程解析

![](img\32.png)

1. Producer创建时，会创建一个Sender线程并设置为守护线程。

2. 生产消息时，内部其实是异步流程；生产的消息先经过拦截器->序列化器->分区器，然后将消息缓存在缓冲区（该缓冲区也是在Producer创建时创建）。

3. 批次发送的条件为：缓冲区数据大小达到batch.size或者linger.ms达到上限，哪个先达到就算哪个。

4. 批次发送后，发往指定分区，然后落盘到broker；如果生产者配置了retrires参数大于0并且失败原因允许重试，那么客户端内部会对该消息进行重试。

5. 落盘到broker成功，返回生产元数据给生产者。

6. 元数据返回有两种方式：一种是通过阻塞直接返回，另一种是通过回调返回。

##### 必要参数配置

###### broker配置

1. 配置条目的使用方式：

![](img\34.png)

2. 配置参数

![](img\35.png)

##### 序列化器

![](img\36.png)

由于Kafka中的数据都是字节数组，在将消息发送到Kafka之前需要先将数据序列化为字节数组。

序列化器的作用就是用于序列化要发送的消息的。



Kafka使用 org.apache.kafka.common.serialization.Serializer 接口用于定义序列化器，将泛型指定类型的数据转换为字节数组。

```java
/**
 * 将对象转换为byte数组的接口 ** 该接口的实现类需要提供无参构造器
 *
 * @param <T> 从哪个类型转换
 */
public interface Serializer<T> extends Closeable {
    /**
     * 类的配置信息
     *
     * @param configs key/value pairs
     * @param isKey   key的序列化还是value的序列化
     */
    void configure(Map<String, ?> configs, boolean isKey);

    /**
     * 将对象转换为字节数组 ** @param topic 主题名称
     *
     * @param data 需要转换的对象
     * @return 序列化的字节数组
     */
    byte[] serialize(String topic, T data);

    /**
     * 关闭序列化器 * 该方法需要提供幂等性，因为可能调用多次。
     */
    @Override
    void close();
}
```

系统提供了该接口的子接口以及实现类：org.apache.kafka.common.serialization.ByteArraySerializer 

![](img\37.png)

org.apache.kafka.common.serialization.ByteBufferSerializer

![](img\38.png)

org.apache.kafka.common.serialization.BytesSerializer

![](img\39.png)

org.apache.kafka.common.serialization.DoubleSerializer

![](img\40.png)

org.apache.kafka.common.serialization.FloatSerializer

![](img\41.png)

org.apache.kafka.common.serialization.IntegerSerializer 

![](img\42.png)

org.apache.kafka.common.serialization.StringSerializer

![](img\42.png)

org.apache.kafka.common.serialization.LongSerializer 

![](img\43.png)

org.apache.kafka.common.serialization.ShortSerializer

![](img\44.png)

###### 自定义序列化器

数据的序列化一般生产中使用avro。

自定义序列化器需要实现org.apache.kafka.common.serialization.Serializer<T>接口，并实现其中的 serialize 方法。



案例：

实体类：

```java
public class User {
    private Integer userId;
    private String username;

    public Integer getUserId() {
        return userId;
    }

    public void setUserId(Integer userId) {
        this.userId = userId;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }
}
```

序列化类：

```java
public class UserSerializer implements Serializer<User> {
    @Override
    public void configure(Map<String, ?> configs, boolean isKey) {
        // do nothing
    }

    @Override
    public byte[] serialize(String topic, User data) {
        try {
            // 如果数据是null，则返回null
            if (data == null) return null;
            Integer userId = data.getUserId();
            String username = data.getUsername();
            int length = 0;
            byte[] bytes = null;
            if (null != username) {
                bytes = username.getBytes("utf-8");
                length = bytes.length;
            }
            ByteBuffer buffer = ByteBuffer.allocate(4 + 4 + length);
            buffer.putInt(userId);
            buffer.putInt(length);
            buffer.put(bytes);
            return buffer.array();
        } catch (UnsupportedEncodingException e) {
            throw new SerializationException("序列化数据异常");
        }
    }

    @Override
    public void close() {
        // do nothing
    }
}
```

生产者：

```java
public class MyProducer {
    public static void main(String[] args) {
        Map<String, Object> configs = new HashMap<>();
        configs.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "node1:9092");
        configs.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        // 设置自定义的序列化类
        configs.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, UserSerializer.class);
        KafkaProducer<String, User> producer = new KafkaProducer<String, User>(configs);
        User user = new User();
        user.setUserId(1001);
        user.setUsername("张三");
        ProducerRecord<String, User> record = new ProducerRecord<>("tp_user_01", 0, user.getUsername(), user);
        producer.send(record, (metadata, exception) -> {
            if (exception == null) {
                System.out.println("消息发送成功：" + metadata.topic() + "\t" + metadata.partition() + "\t" + metadata.offset());
            } else {
                System.out.println("消息发送异常");
            }
        });
        // 关闭生产者
        producer.close();
    }
}
```

##### 分区器

![](img\45.png)

默认（DefaultPartitioner）分区计算：

1. 如果record提供了分区号，则使用record提供的分区号

2. 如果record没有提供分区号，则使用key的序列化后的值的hash值对分区数量取模

3. 如果record没有提供分区号，也没有提供key，则使用轮询的方式分配分区号。

​       1.  会首先在可用的分区中分配分区号

​       2. 如果没有可用的分区，则在该主题所有分区中分配分区号。

![](img\46.png)

![](img\47.png)

如果要自定义分区器，则需要

1. 首先开发Partitioner接口的实现类

2. 在KafkaProducer中进行设置：configs.put("partitioner.class", "xxx.xx.Xxx.class")

位于 org.apache.kafka.clients.producer 中的分区器接口：

```java
/**
 * 分区器接口
 */
public interface Partitioner extends Configurable, Closeable {
    /**
     * 为指定的消息记录计算分区值
     *
     * @param topic      主题名称
     * @param key        根据该key的值进行分区计算，如果没有则为null。
     * @param keyBytes   key的序列化字节数组，根据该数组进行分区计算。如果没有key，则为 null
     * @param value      根据value值进行分区计算，如果没有，则为null
     * @param valueBytes value的序列化字节数组，根据此值进行分区计算。如果没有，则为 null
     * @param cluster    当前集群的元数据
     */
    public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster);

    /**
     * 关闭分区器的时候调用该方法
     */
    public void close();
}
```

包 org.apache.kafka.clients.producer.internals 中分区器的默认实现：

```java
/**
 * 默认的分区策略：
 * 如果在记录中指定了分区，则使用指定的分区
 * 如果没有指定分区，但是有key的值，则使用key值的散列值计算分区
 * 如果没有指定分区也没有key的值，则使用轮询的方式选择一个分区
 */
public class DefaultPartitioner implements Partitioner {
    private final ConcurrentMap<String, AtomicInteger> topicCounterMap = new ConcurrentHashMap<>();

    public void configure(Map<String, ?> configs) {
    }

    /**
     * 为指定的消息记录计算分区值
     *
     * @param topic      主题名称
     * @param key        根据该key的值进行分区计算，如果没有则为null。
     * @param keyBytes   key的序列化字节数组，根据该数组进行分区计算。如果没有key，则为 null
     * @param value      根据value值进行分区计算，如果没有，则为null
     * @param valueBytes value的序列化字节数组，根据此值进行分区计算。如果没有，则为 null
     * @param cluster    当前集群的元数据
     */
    public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
        // 获取指定主题的所有分区信息
        List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
        // 分区的数量
        int numPartitions = partitions.size();
        // 如果没有提供key
        if (keyBytes == null) {
            int nextValue = nextValue(topic);
            List<PartitionInfo> availablePartitions = cluster.availablePartitionsForTopic(topic);
            if (availablePartitions.size() > 0) {
                int part = Utils.toPositive(nextValue) % availablePartitions.size();
                return availablePartitions.get(part).partition();
            } else {
                // no partitions are available, give a non-available partition
                return Utils.toPositive(nextValue) % numPartitions;
            }
        } else {
            // hash the keyBytes to choose a partition
            // 如果有，就计算keyBytes的哈希值，然后对当前主题的个数取模
            return Utils.toPositive(Utils.murmur2(keyBytes)) % numPartitions;
        }
    }

    private int nextValue(String topic) {
        AtomicInteger counter = topicCounterMap.get(topic);
        if (null == counter) {
            counter = new AtomicInteger(ThreadLocalRandom.current().nextInt());
            AtomicInteger currentCounter = topicCounterMap.putIfAbsent(topic, counter);
            if (currentCounter != null) {
                counter = currentCounter;
            }
        }
        return counter.getAndIncrement();
    }

    public void close() {
    }
}
```

![](img\48.png)

可以实现Partitioner接口自定义分区器：

![](img\49.png)

然后在生产者中配置：

![](img\50.png)

##### 拦截器

![](img\51.png)

Producer拦截器（interceptor）和Consumer端Interceptor是在Kafka 0.10版本被引入的，主要用

于实现Client端的定制化控制逻辑。

对于Producer而言，Interceptor使得用户在消息发送前以及Producer回调逻辑前有机会对消息做

一些定制化需求，比如修改消息等。同时，Producer允许用户指定多个Interceptor按序作用于同一条消

息从而形成一个拦截链(interceptor chain)。Intercetpor的实现接口是

org.apache.kafka.clients.producer.ProducerInterceptor，其定义的方法包括：

- onSend(ProducerRecord)：该方法封装进KafkaProducer.send方法中，即运行在用户主线程

  中。Producer确保在消息被序列化以计算分区前调用该方法。用户可以在该方法中对消息做任

  何操作，但最好保证不要修改消息所属的topic和分区，否则会影响目标分区的计算。

- onAcknowledgement(RecordMetadata, Exception)：该方法会在消息被应答之前或消息发送

  失败时调用，并且通常都是在Producer回调逻辑触发之前。onAcknowledgement运行在Producer的IO线程中，因此不要在该方法中放入很重的逻辑，否则会拖慢Producer的消息发

  送效率。

- close：关闭Interceptor，主要用于执行一些资源清理工作。

如前所述，Interceptor可能被运行在多个线程中，因此在具体实现时用户需要**自行确保线程安全**。

另外倘若指定了多个Interceptor，则Producer将按照指定顺序调用它们，并仅仅是捕获每个Interceptor可能抛出的异常记录到错误日志中而非在向上传递。这在使用过程中要特别留意。



自定义拦截器：

1. 实现ProducerInterceptor接口

2. 在KafkaProducer的设置中设置自定义的拦截器

![](img\52.png)

案例：

1. **消息实体类：**

```java
public class User {
    private Integer userId;
    private String username;

    public Integer getUserId() {
        return userId;
    }

    public void setUserId(Integer userId) {
        this.userId = userId;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }
}
```

2. **自定义序列化器**

```java
public class UserSerializer implements Serializer<User> {
    @Override
    public void configure(Map<String, ?> configs, boolean isKey) {
        // do nothing
    }

    @Override
    public byte[] serialize(String topic, User data) {
        try {
            // 如果数据是null，则返回null
            if (data == null) return null;
            Integer userId = data.getUserId();
            String username = data.getUsername();
            int length = 0;
            byte[] bytes = null;
            if (null != username) {
                bytes = username.getBytes("utf-8");
                length = bytes.length;
            }
            ByteBuffer buffer = ByteBuffer.allocate(4 + 4 + length);
            buffer.putInt(userId);
            buffer.putInt(length);
            buffer.put(bytes);
            return buffer.array();
        } catch (UnsupportedEncodingException e) {
            throw new SerializationException("序列化数据异常");
        }
    }

    @Override
    public void close() {
        // do nothing
    }
}
```

3. **自定义分区器**

```java
public class MyPartitioner implements Partitioner {
    @Overridepublic
    int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
        return 2;
    }

    @Override
    public void close() {
    }

    @Override
    public void configure(Map<String, ?> configs) {
    }
}
```

4. **自定义拦截器1**

```java
public class InterceptorOne<KEY, VALUE> implements ProducerInterceptor<KEY, VALUE> {
    private static final Logger LOGGER = LoggerFactory.getLogger(InterceptorOne.class);

    @Override
    public ProducerRecord<KEY, VALUE> onSend(ProducerRecord<KEY, VALUE> record) {
        System.out.println("拦截器1---go");
        // 此处根据业务需要对相关的数据作修改
        String topic = record.topic();
        Integer partition = record.partition();
        Long timestamp = record.timestamp();
        KEY key = record.key();
        VALUE value = record.value();
        Headers headers = record.headers();
        // 添加消息头
        headers.add("interceptor", "interceptorOne".getBytes());
        ProducerRecord<KEY, VALUE> newRecord = new ProducerRecord<KEY, VALUE>(topic, partition, timestamp, key, value, headers);
        return newRecord;
    }

    @Override
    public void onAcknowledgement(RecordMetadata metadata, Exception exception) {
        System.out.println("拦截器1---back");
        if (exception != null) {
            // 如果发生异常，记录日志中
            LOGGER.error(exception.getMessage());
        }
    }

    @Override
    public void close() {
    }

    @Override
    public void configure(Map<String, ?> configs) {
    }
}
```

5. **自定义拦截器2**

```java
public class InterceptorTwo<KEY, VALUE> implements ProducerInterceptor<KEY, VALUE> {
    private static final Logger LOGGER = LoggerFactory.getLogger(InterceptorTwo.class);

    @Override
    public ProducerRecord<KEY, VALUE> onSend(ProducerRecord<KEY, VALUE> record) {
        System.out.println("拦截器2---go");
        // 此处根据业务需要对相关的数据作修改
        String topic = record.topic();
        Integer partition = record.partition();
        Long timestamp = record.timestamp();
        KEY key = record.key();
        VALUE value = record.value();
        Headers headers = record.headers();
        // 添加消息头
        headers.add("interceptor", "interceptorTwo".getBytes());
        ProducerRecord<KEY, VALUE> newRecord = new ProducerRecord<KEY, VALUE>(topic, partition, timestamp, key, value, headers);
        return newRecord;
    }

    @Override
    public void onAcknowledgement(RecordMetadata metadata, Exception exception) {
        System.out.println("拦截器2---back");
        if (exception != null) {
            // 如果发生异常，记录日志中
            LOGGER.error(exception.getMessage());
        }
    }

    @Override
    public void close() {
    }

    @Override
    public void configure(Map<String, ?> configs) {
    }
}
```

6. **自定义拦截器3**

```java
public class InterceptorThree<KEY, VALUE> implements ProducerInterceptor<KEY, VALUE> {
    private static final Logger LOGGER = LoggerFactory.getLogger(InterceptorThree.class);

    @Override
    public ProducerRecord<KEY, VALUE> onSend(ProducerRecord<KEY, VALUE> record) {
        System.out.println("拦截器3---go");
        // 此处根据业务需要对相关的数据作修改
        String topic = record.topic();
        Integer partition = record.partition();
        Long timestamp = record.timestamp();
        KEY key = record.key();
        VALUE value = record.value();
        Headers headers = record.headers();
        // 添加消息头
        headers.add("interceptor", "interceptorThree".getBytes());
        ProducerRecord<KEY, VALUE> newRecord = new ProducerRecord<KEY, VALUE>(topic, partition, timestamp, key, value, headers);
        return newRecord;
    }

    @Override
    public void onAcknowledgement(RecordMetadata metadata, Exception exception) {
        System.out.println("拦截器3---back");
        if (exception != null) {
            // 如果发生异常，记录日志中
            LOGGER.error(exception.getMessage());
        }
    }

    @Override
    public void close() {
    }

    @Override
    public void configure(Map<String, ?> configs) {
    }
}
```

7. **生产者**

```java
public class MyProducer {
    public static void main(String[] args) {
        Map<String, Object> configs = new HashMap<>();
        configs.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "node1:9092");
        // 设置自定义分区器
        // configs.put(ProducerConfig.PARTITIONER_CLASS_CONFIG, MyPartitioner.class);
        configs.put("partitioner.class", "com.lagou.kafka.demo.partitioner.MyPartitioner");
        // 设置拦截器
        configs.put(ProducerConfig.INTERCEPTOR_CLASSES_CONFIG, "com.lagou.kafka.demo.interceptor.InterceptorOne," + "com.lagou.kafka.demo.interceptor.InterceptorTwo," + "com.lagou.kafka.demo.interceptor.InterceptorThree");
        configs.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        // 设置自定义的序列化类
        configs.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, UserSerializer.class);
        KafkaProducer<String, User> producer = new KafkaProducer<String, User>(configs);
        User user = new User();
        user.setUserId(1001);
        user.setUsername("张三");
        ProducerRecord<String, User> record = new ProducerRecord<>("tp_user_01", 0, user.getUsername(), user);
        producer.send(record, (metadata, exception) -> {
            if (exception == null) {
                System.out.println("消息发送成功：" + metadata.topic() + "\t" + metadata.partition() + "\t" + metadata.offset());
            } else {
                System.out.println("消息发送异常");
            }
        });
        producer.close();
    }
}
```

8. **运行结果：**

![](img\53.png)

#### 原理剖析

![](img\54.png)

由上图可以看出：KafkaProducer有两个基本线程：

- 主线程：负责消息创建，拦截器，序列化器，分区器等操作，并将消息追加到消息收集器

  RecoderAccumulator中；

  - 消息收集器RecoderAccumulator为**每个分区**都维护了一个Deque<ProducerBatch> 类型的双端队列
  - ProducerBatch 可以理解为是 ProducerRecord 的集合，批量发送有利于提升吞吐量，降低网络影响；
  - 由于生产者客户端使用 java.io.ByteBuffer 在发送消息之前进行消息保存，并维护了一个 BufferPool 实现 ByteBuffer 的复用；该缓存池只针对特定大小（ batch.size指定）的 ByteBuffer进行管理，对于消息过大的缓存，不能做到重复利用。
  - 每次追加一条ProducerRecord消息，会寻找/新建对应的双端队列，从其尾部获取一个ProducerBatch，判断当前消息的大小是否可以写入该批次中。若可以写入则写入；若不可以写入，则新建一个ProducerBatch，判断该消息大小是否超过客户端参数配置 batch.size 的值，不超过，则以 batch.size建立新的ProducerBatch，这样方便进行缓存重复利用；若超过，则以计算的消息大小建立对应的 ProducerBatch ，缺点就是该内存不能被复用了。

- Sender线程：
  - 该线程从消息收集器获取缓存的消息，将其处理为 <Node, List<ProducerBatch> 的形式， Node 表示集群的broker节点。
  - 进一步将<Node, List<ProducerBatch>转化为<Node, Request>形式，此时才可以向服务端发送数据。
  - 在发送之前，Sender线程将消息以 Map<NodeId, Deque<Request>> 的形式保存到InFlightRequests 中进行缓存，可以通过其获取 leastLoadedNode ,即当前Node中负载压力最小的一个，以实现消息的尽快发出。



#### 生产者参数配置补充

1. 参数设置方式：

   ![](img\55.png)

2. 补充参数：

![](img\56.png)

![](img\57.png)

![](img\58.png)

![](img\59.png)

### 消费者

#### 概念入门

#####  消费者、消费组

消费者从订阅的主题消费消息，消费消息的偏移量保存在Kafka的名字是 __consumer_offsets 的主题中。

消费者还可以将自己的偏移量存储到Zookeeper，需要设置offset.storage=zookeeper。

推荐使用**Kafka**存储消费者的偏移量。因为Zookeeper不适合高并发。

多个从同一个主题消费的消费者可以加入到一个消费组中。

消费组中的消费者共享group_id。

configs.put("group.id", "xxx");

group_id一般设置为应用的逻辑名称。比如多个订单处理程序组成一个消费组，可以设置group_id为"order_process"。

group_id通过消费者的配置指定： group.id=xxxxx

消费组均衡地给消费者分配分区，每个分区只由消费组中一个消费者消费。

![](img\60.png)

一个拥有四个分区的主题，包含一个消费者的消费组。

此时，消费组中的消费者消费主题中的所有分区。并且没有重复的可能。

如果在消费组中添加一个消费者2，则每个消费者分别从两个分区接收消息。

![](img\61.png)

如果消费组有四个消费者，则每个消费者可以分配到一个分区。

![](img\62.png)

如果向消费组中添加更多的消费者，超过主题分区数量，则有一部分消费者就会闲置，不会接收任何消息。

![](img\63.png)

向消费组添加消费者是横向扩展消费能力的主要方式。

必要时，需要为主题创建大量分区，在负载增长时可以加入更多的消费者。但是不要让消费者的数量超过主题分区的数量。

![](img\64.png)

除了通过增加消费者来横向扩展单个应用的消费能力之外，经常出现多个应用程序从同一个主题消费的情况。

此时，每个应用都可以获取到所有的消息。只要保证每个应用都有自己的消费组，就可以让它们获取到主题所有的消息。

横向扩展消费者和消费组不会对性能造成负面影响。

为每个需要获取一个或多个主题全部消息的应用创建一个消费组，然后向消费组添加消费者来横向扩展消费能力和应用的处理能力，则每个消费者只处理一部分消息。

##### 心跳机制

![](img\65.png)

消费者宕机，退出消费组，触发再平衡，重新给消费组中的消费者分配分区

![](img\66.png)

由于broker宕机，主题X的分区3宕机，此时分区3没有Leader副本，触发再平衡，消费者4没有对应的主题分区，则消费者4闲置。

![](img\67.png)

Kafka 的心跳是 Kafka Consumer 和 Broker 之间的健康检查，只有当 Broker Coordinator 正常时，Consumer 才会发送心跳。

Consumer 和 Rebalance 相关的 2 个配置参数：

| **参数**             | **字段**                          |
| -------------------- | --------------------------------- |
| session.timeout.ms   | MemberMetadata.sessionTimeoutMs   |
| max.poll.interval.ms | MemberMetadata.rebalanceTimeoutMs |

broker 端，sessionTimeoutMs 参数

broker 处理心跳的逻辑在 GroupCoordinator 类中：如果心跳超期， broker coordinator 会把消费者从 group 中移除，并触发 rebalance。 

```
private def completeAndScheduleNextHeartbeatExpiration(group:GroupMetadata,member:MemberMetadata){
        // complete current heartbeat expectation
        member.latestHeartbeat=time.milliseconds()
        val memberKey=MemberKey(member.groupId,member.memberId)
        heartbeatPurgatory.checkAndComplete(memberKey)
        // reschedule the next heartbeat expiration deadline
        // 计算心跳截止时刻
        val newHeartbeatDeadline=member.latestHeartbeat+member.sessionTimeoutMs
        val delayedHeartbeat=new DelayedHeartbeat(this,group,member,newHeartbeatDeadline,member.sessionTimeoutMs)
        heartbeatPurgatory.tryCompleteElseWatch(delayedHeartbeat,Seq(memberKey))
        }
// 心跳过期
        def onExpireHeartbeat(group:GroupMetadata,member:MemberMetadata,heartbeatDeadline:Long){
            group.inLock{
                if(!shouldKeepMemberAlive(member,heartbeatDeadline)){
                    info(s"Member ${member.memberId} in group ${group.groupId} has failed, removing it from the group")
                    removeMemberAndUpdateGroup(group,member)
                }
            }
}
private def shouldKeepMemberAlive(member: MemberMetadata, heartbeatDeadline: Long) = member.awaitingJoinCallback != null || member.awaitingSyncCallback != null || member.latestHeartbeat + member.sessionTimeoutMs > heartbeatDeadline
```

consumer 端：sessionTimeoutMs，rebalanceTimeoutMs 参数

如果客户端发现心跳超期，客户端会标记 coordinator 为不可用，并阻塞心跳线程；如果超过了poll 消息的间隔超过了 rebalanceTimeoutMs，则 consumer 告知 broker 主动离开消费组，也会触发rebalance

org.apache.kafka.clients.consumer.internals.AbstractCoordinator.HeartbeatThread

```
.................
```

#### 消息接收

##### 必要参数配置

![](img\68.png)

#####  订阅

###### 主题和分区

​	1.**Topic**，Kafka用于分类管理消息的逻辑单元，类似与MySQL的数据库。

2. **Partition**，是Kafka下数据存储的基本单元，这个是物理上的概念。**同一个****topic****的数据，会**被分散的存储到多partition**中，这些partition可以在同一台机器上，也可以是在多台机器上。优势在于：有利于水平扩展，避免单台机器在磁盘空间和性能上的限制，同时可以通过复制来增加数据冗余性，提高容灾能力。为了做到均匀分布，通常partition的数量通常是Broker

Server数量的整数倍。

​	3.**Consumer Group**，同样是逻辑上的概念，是**Kafka**实现单播和广播两种消息模型的手段。保证一个消费组获取到特定主题的全部的消息。在消费组内部，若干个消费者消费主题分区的消息，消费组可以保证一个主题的每个分区只被消费组中的一个消费者消费。

![](img\69.png)

consumer 采用 pull 模式从 broker 中读取数据。

采用 pull 模式，consumer 可自主控制消费消息的速率， 可以自己控制消费方式（批量消费/逐条消费)，还可以选择不同的提交方式从而实现不同的传输语义。

consumer.subscribe("tp_demo_01,tp_demo_02")

##### 反序列化

Kafka的broker中所有的消息都是字节数组，消费者获取到消息之后，需要先对消息进行反序列化处理，然后才能交给用户程序消费处理。

消费者的反序列化器包括key的和value的反序列化器。

key.deserializer

value.deserializer

IntegerDeserializer

StringDeserializer

需要实现 org.apache.kafka.common.serialization.Deserializer<T> 接口。

消费者从订阅的主题拉取消息：

consumer.poll(3_000);

在Fetcher类中，对拉取到的消息首先进行反序列化处理。

![](img\70.png)

Kafka默认提供了几个反序列化的实现：

org.apache.kafka.common.serialization 包下包含了这几个实现：

![](img\71.png)

![](img\72.png)

```
............
```

###### 自定义反序列化

自定义反序列化类，需要实现 org.apache.kafka.common.serialization.Deserializer<T> 接口。

com.lagou.kafka.demo.deserializer.UserDeserializer 

```java
public class UserDeserializer implements Deserializer<User> {
    @Override
    public void configure(Map<String, ?> configs, boolean isKey) {
    }

    @Override
    public User deserialize(String topic, byte[] data) {
        ByteBuffer allocate = ByteBuffer.allocate(data.length);
        allocate.put(data);
        allocate.flip();
        int userId = allocate.getInt();
        int length = allocate.getInt();
        System.out.println(length);
        String username = new String(data, 8, length);
        return new User(userId, username);
    }

    @Override
    public void close() {
    }
}
```

com.lagou.kafka.demo.consumer.MyConsumer

```java
public class MyConsumer {
    public static void main(String[] args) {
        Map<String, Object> configs = new HashMap<>();
        configs.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "node1:9092");
        configs.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        configs.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, UserDeserializer.class);
        configs.put(ConsumerConfig.GROUP_ID_CONFIG, "consumer1");
        configs.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
        configs.put(ConsumerConfig.CLIENT_ID_CONFIG, "con1");
        KafkaConsumer<String, User> consumer = new KafkaConsumer<String, User>(configs);
        consumer.subscribe(Collections.singleton("tp_user_01"));
        ConsumerRecords<String, User> records = consumer.poll(Long.MAX_VALUE);
        records.forEach(new Consumer<ConsumerRecord<String, User>>() {
            @Override
            public void accept(ConsumerRecord<String, User> record) {
                System.out.println(record.value());
            }
        });
        consumer.close();
    }
}
```

##### 位移提交

1. Consumer需要向Kafka记录自己的位移数据，这个汇报过程称为 提交位移(Committing Offsets) 

2. Consumer 需要为分配给它的每个分区提交各自的位移数据

3. 位移提交的由Consumer端负责的，Kafka只负责保管。__consumer_offsets

4. 位移提交分为自动提交和手动提交

5. 位移提交分为同步提交和异步提交

###### 自动提交

Kafka Consumer 后台提交

开启自动提交： enable.auto.commit=true

配置自动提交间隔：Consumer端： auto.commit.interval.ms ，默认 5s

```java
        Map<String, Object> configs = new HashMap<>();
        configs.put("bootstrap.servers", "node1:9092");
        configs.put("group.id", "mygrp");
		// 设置偏移量自动提交。自动提交是默认值。这里做示例。
        configs.put("enable.auto.commit", "true");
		// 偏移量自动提交的时间间隔
        configs.put("auto.commit.interval.ms", "3000");
        configs.put("key.deserializer", StringDeserializer.class);
        configs.put("value.deserializer", StringDeserializer.class);
        KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(configs);
        consumer.subscribe(Collections.singleton("tp_demo_01"));
        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(100);
            for (ConsumerRecord<String, String> record : records) {
                System.out.println(record.topic() + "\t" + record.partition() + "\t" + record.offset() + "\t" + record.key() + "\t" + record.value());
            }
        }
```

1. 自动提交位移的顺序

   ​	配置 enable.auto.commit = true

   ​	Kafka会保证在开始调用poll方法时，提交上次poll返回的所有消息

   ​	因此自动提交不会出现消息丢失，但会 重复消费

2. 重复消费举例

   ​	Consumer 每 5s 提交 offset

   ​	假设提交 offset 后的 3s 发生了 Rebalance

   ​	Rebalance 之后的所有 Consumer 从上一次提交的 offset 处继续消费

   ​	因此 Rebalance 发生前 3s 的消息会被重复消费

###### 异步提交

使用 KafkaConsumer#commitSync()：会提交 KafkaConsumer#poll() 返回的最新 offset

该方法为同步操作，等待直到 offset 被成功提交才返回

```java
while (true) {
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofSeconds(1));
            process(records);
            // 处理消息 
            try {
                consumer.commitSync();
            } catch (CommitFailedException e) {
                handle(e); // 处理提交失败异常
            }
        }
```

commitSync 在处理完所有消息之后

手动同步提交可以控制offset提交的时机和频率

手动同步提交会：

​		1.调用 commitSync 时，Consumer 处于阻塞状态，直到 Broker 返回结果

​		2.会影响 TPS

​		3.可以选择拉长提交间隔，但有以下问题:

​				会导致 Consumer 的提交频率下降

​				Consumer 重启后，会有更多的消息被消费

###### 异步提交

- KafkaConsumer#commitAsync()

  ```
  while (true) { 
  	ConsumerRecords<String, String> records = consumer.poll(3_000); 
  	process(records);
      // 处理消息 
      consumer.commitAsync((offsets, exception) -> { 
      if (exception != null) {
      	handle(exception);
         }
     	}); 
  }
  ```

- commitAsync出现问题不会自动重试

- 处理方式:

```
try {
	while(true) { 
		ConsumerRecords<String, String> records = consumer.poll(Duration.ofSeconds(1));
        process(records); // 处理消息 
        commitAysnc();  // 使用异步提交规避阻塞 
        } 
     } catch(Exception e) {
     	handle(e); // 处理异常 
     } finally {
     	try {
     		consumer.commitSync(); // 最后一次提交使用同步阻塞式提交
           } finally {
           		consumer.close();
           }
      }
```

##### 消费者位移管理

Kafka中，消费者根据消息的位移顺序消费消息。

消费者的位移由消费者管理，可以存储于zookeeper中，也可以存储于Kafka主题__consumer_offsets中。

Kafka提供了消费者API，让消费者可以管理自己的位移。

API如下：KafkaConsumer<K, V>

![](img\73.png)

![](img\74.png)

1. 准备数据

```
# 生成消息文件 
[root@node1 ~]# for i in `seq 60`; do echo "hello lagou $i" >> nm.txt; done 

# 创建主题，三个分区，每个分区一个副本 
[root@node1 ~]# kafka-topics.sh --zookeeper node1:2181/myKafka --create -- topic tp_demo_01 --partitions 3 --replication-factor 1 

# 将消息生产到主题中 
[root@node1 ~]# kafka-console-producer.sh --broker-list node1:9092 --topic tp_demo_01 < nm.txt
```

2. API实战

```java
public class MyConsumerMgr1 {
    public static void main(String[] args) {
        Map<String, Object> configs = new HashMap<>();
        configs.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "node1:9092");
        configs.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        configs.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(configs);
        /**
         *  给当前消费者手动分配一系列主题分区。
         *  手动分配分区不支持增量分配，如果先前有分配分区，则该操作会覆盖之前的分配。
         *  如果给出的主题分区是空的，则等价于调用unsubscribe方法。
         *  手动分配主题分区的方法不使用消费组管理功能。当消费组成员变了，或者集群或主题 的元数据改变了，不会触发分区分配的再平衡。
         *  手动分区分配assign(Collection)不能和自动分区分配subscribe(Collection, ConsumerRebalanceListener)一起使用。
         *  如果启用了自动提交偏移量，则在新的分区分配替换旧的分区分配之前，会对旧的分区 分配中的消费偏移量进行异步提交。
         **/
        // consumer.assign(Arrays.asList(new TopicPartition("tp_demo_01", 0)));
        // Set<TopicPartition> assignment = consumer.assignment();
        // for (TopicPartition topicPartition : assignment) {
        //      System.out.println(topicPartition);
        //  }
        // 获取对用户授权的所有主题分区元数据。该方法会对服务器发起远程调用。
        // Map<String, List<PartitionInfo>> stringListMap = consumer.listTopics();
        // stringListMap.forEach((k, v) -> {
        //      System.out.println("主题：" + k);
        //      v.forEach(info -> {System.out.println(info);});
        // });

        // Set<String> strings = consumer.listTopics().keySet();
        // strings.forEach(topicName -> {  System.out.println(topicName);  });
        // List<PartitionInfo> partitionInfos = consumer.partitionsFor("tp_demo_01");
        // for (PartitionInfo partitionInfo : partitionInfos) {
        //     Node leader = partitionInfo.leader();
        //     System.out.println(leader);
        //     System.out.println(partitionInfo);
        //     当前分区在线副本
        //     Node[] nodes = partitionInfo.inSyncReplicas();
        //     当前分区下线副本
        //     Node[] nodes1 = partitionInfo.offlineReplicas();
        // }
        // 手动分配主题分区给当前消费者
        consumer.assign(Arrays.asList(new TopicPartition("tp_demo_01", 0), new TopicPartition("tp_demo_01", 1), new TopicPartition("tp_demo_01", 2)));

        // 列出当前主题分配的所有主题分区
        // Set<TopicPartition> assignment = consumer.assignment();
        // assignment.forEach(k -> {  System.out.println(k);  });
        // 对于给定的主题分区，列出它们第一个消息的偏移量。
        // 注意，如果指定的分区不存在，该方法可能会永远阻塞。  该方法不改变分区的当前消费者偏移量。
        // Map<TopicPartition, Long> topicPartitionLongMap = consumer.beginningOffsets(consumer.assignment());
        // topicPartitionLongMap.forEach((k, v) -> {  System.out.println("主题：" + k.topic() + "\t分区：" + k.partition() + "偏移量\t" + v);  });
        // 将偏移量移动到每个给定分区的最后一个。
        // 该方法延迟执行，只有当调用过poll方法或position方法之后才可以使用。
        // 如果没有指定分区，则将当前消费者分配的所有分区的消费者偏移量移动到最后。
        // 如果设置了隔离级别为：isolation.level=read_committed，则会将分区的消费 偏移量移动到
        // 最后一个稳定的偏移量，即下一个要消费的消息现在还是未提交状态的事务消息。
        // consumer.seekToEnd(consumer.assignment());
        // 将给定主题分区的消费偏移量移动到指定的偏移量，即当前消费者下一条要消费的消息 偏移量。
        // 若该方法多次调用，则最后一次的覆盖前面的。
        // 如果在消费中间随意使用，可能会丢失数据。
        // consumer.seek(new TopicPartition("tp_demo_01", 1), 10);
        // 检查指定主题分区的消费偏移量
        // long position = consumer.position(new TopicPartition("tp_demo_01", 1));
        // System.out.println(position);
        consumer.seekToEnd(Arrays.asList(new TopicPartition("tp_demo_01", 1)));
        // 检查指定主题分区的消费偏移量
        long position = consumer.position(new TopicPartition("tp_demo_01", 1));
        System.out.println(position);
        // 关闭生产者
        consumer.close();
    }
}
```

##### 再均衡

重平衡可以说是kafka为人诟病最多的一个点了。

重平衡其实就是一个协议，它规定了如何让消费者组下的所有消费者来分配topic中的每一个分区。

比如一个topic有100个分区，一个消费者组内有20个消费者，在协调者的控制下让组内每一个消费者分配到5个分区，这个分配的过程就是重平衡。

重平衡的触发条件主要有三个：

1. 消费者组内成员发生变更，这个变更包括了增加和减少消费者，比如消费者宕机退出消费组。

2. 主题的分区数发生变更，kafka目前只支持增加分区，当增加的时候就会触发重平衡

3. 订阅的主题发生变化，当消费者组使用正则表达式订阅主题，而恰好又新建了对应的主题，就会触发重平衡

![](img\75.png)

消费者宕机，退出消费组，触发再平衡，重新给消费组中的消费者分配分区。

![](img\76.png)

由于broker宕机，主题X的分区3宕机，此时分区3没有Leader副本，触发再平衡，消费者4没有对应的主题分区，则消费者4闲置。

![](img\77.png)

主题增加分区，需要主题分区和消费组进行再均衡。

![](img\78.png)

由于使用正则表达式订阅主题，当增加的主题匹配正则表达式的时候，也要进行再均衡。

![](img\79.png)

为什么说重平衡为人诟病呢？**因为重平衡过程中，消费者无法从kafka消费消息，这对kafka的TPS影响极大，而如果kafka集内节点较多，比如数百个，那重平衡可能会耗时极多。数分钟到数小时都有可能，而这段时间kafka基本处于不可用状态。**所以在实际环境中，应该尽量避免重平衡发生。



**避免重平衡**

要说完全避免重平衡，是不可能，因为你无法完全保证消费者不会故障。而消费者故障其实也是最常见的引发重平衡的地方，所以我们需要保证**尽力避免消费者故障**。

而其他几种触发重平衡的方式，增加分区，或是增加订阅的主题，抑或是增加消费者，更多的是主动控制。

如果消费者真正挂掉了，就没办法了，但实际中，会有一些情况，**kafka错误地认为**一个正常的消费者已经挂掉了，我们要的就是避免这样的情况出现。

首先要知道哪些情况会出现错误判断挂掉的情况。

在分布式系统中，通常是通过心跳来维持分布式系统的，kafka也不例外。



在分布式系统中，由于网络问题你不清楚没接收到心跳，是因为对方真正挂了还是只是因为负载过重没来得及发生心跳或是网络堵塞。所以一般会约定一个时间，超时即判定对方挂了。**而在kafka消费者场景中，session.timout.ms参数就是规定这个超时时间是多少**。

还有一个参数，**heartbeat.interval.ms**，这个参数控制发送心跳的频率，频率越高越不容易被误判，但也会消耗更多资源。

此外，还有最后一个参数，**max.poll.interval.ms**，消费者poll数据后，需要一些处理，再进行拉取。如果两次拉取时间间隔超过这个参数设置的值，那么消费者就会被踢出消费者组。也就是说，拉取，然后处理，这个处理的时间不能超过 max.poll.interval.ms 这个参数的值。这个参数的默认值是5分钟，而如果消费者接收到数据后会执行耗时的操作，则应该将其设置得大一些。



三个参数，

session.timout.ms控制心跳超时时间，

heartbeat.interval.ms控制心跳发送频率，

max.poll.interval.ms控制poll的间隔。



这里给出一个相对较为合理的配置，如下：

1. session.timout.ms：设置为6s

2. heartbeat.interval.ms：设置2s

3. max.poll.interval.ms：推荐为消费者处理消息最长耗时再加1分钟

##### 消费者拦截器

消费者在拉取了分区消息之后，要首先经过反序列化器对key和value进行反序列化处理。

处理完之后，如果消费端设置了拦截器，则需要经过拦截器的处理之后，才能返回给消费者应用程序进行处理。

![](img\80.png)

消费端定义消息拦截器，需要实现org.apache.kafka.clients.consumer.ConsumerInterceptor<K, V> 接口。

1. 一个可插拔接口，允许拦截甚至更改消费者接收到的消息。首要的用例在于将第三方组件引入消费者应用程序，用于定制的监控、日志处理等。

2. 该接口的实现类通过configre方法获取消费者配置的属性，如果消费者配置中没有指定clientID，还可以获取KafkaConsumer生成的clientId。获取的这个配置是跟其他拦截器共享的，需要保证不会在各个拦截器之间产生冲突。

3. ConsumerInterceptor方法抛出的异常会被捕获、记录，但是不会向下传播。如果用户配置了错误的key或value类型参数，消费者不会抛出异常，而仅仅是记录下来。

4. ConsumerInterceptor回调发生在org.apache.kafka.clients.consumer.KafkaConsumer#poll(long)方法同一个线程



该接口中有如下方法：

```java
public interface ConsumerInterceptor<K, V> extends Configurable {
    /**
     * 该方法在poll方法返回之前调用。调用结束后poll方法就返回消息了。
     * 该方法可以修改消费者消息，返回新的消息。拦截器可以过滤收到的消息或生成新的消息。
     * 如果有多个拦截器，则该方法按照KafkaConsumer的configs中配置的顺序调用。
     *
     * @param records 由上个拦截器返回的由客户端消费的消息。
     */
    public ConsumerRecords<K, V> onConsume(ConsumerRecords<K, V> records);

    /**
     * 当消费者提交偏移量时，调用该方法。
     * 该方法抛出的任何异常调用者都会忽略。
     */
    public void onCommit(Map<TopicPartition, OffsetAndMetadata> offsets);

    public void close();
}
```

代码实现：

```java
public class MyConsumer {
    public static void main(String[] args) {
        Properties props = new Properties();
        props.setProperty(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "node1:9092");
        props.setProperty(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer");
        props.setProperty(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer");
        props.setProperty(ConsumerConfig.GROUP_ID_CONFIG, "mygrp");
        // props.setProperty(ConsumerConfig.CLIENT_ID_CONFIG, "myclient");
        // 如果在kafka中找不到当前消费者的偏移量，则设置为最旧的
        props.setProperty(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
        // 配置拦截器
        // One -> Two -> Three，接收消息和发送偏移量确认都是这个顺序
        props.setProperty(ConsumerConfig.INTERCEPTOR_CLASSES_CONFIG, "com.lagou.kafka.demo.interceptor.OneInterceptor" + ",com.lagou.kafka.demo.interceptor.TwoInterceptor" + ",com.lagou.kafka.demo.interceptor.ThreeInterceptor");
        KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(props);
        // 订阅主题
        consumer.subscribe(Collections.singleton("tp_demo_01"));
        while (true) {
            final ConsumerRecords<String, String> records = consumer.poll(3_000);
            records.forEach(record -> {
                System.out.println(record.topic() + "\t" + record.partition() + "\t" + record.offset() + "\t" + record.key() + "\t" + record.value());
            });
            // consumer.commitAsync();
            // consumer.commitSync();
        }
        // consumer.close();
    }
}
```

```java
public class OneInterceptor implements ConsumerInterceptor<String, String> {
    @Override
    public ConsumerRecords<String, String> onConsume(ConsumerRecords<String, String> records) {
        // poll方法返回结果之前最后要调用的方法
        System.out.println("One -- 开始");
        // 消息不做处理，直接返回
        return records;
    }

    @Override
    public void onCommit(Map<TopicPartition, OffsetAndMetadata> offsets) {
        // 消费者提交偏移量的时候，经过该方法
        System.out.println("One -- 结束");
    }

    @Override
    public void close() {
        // 用于关闭该拦截器用到的资源，如打开的文件，连接的数据库等
    }

    @Override
    public void configure(Map<String, ?> configs) {
        // 用于获取消费者的设置参数
        configs.forEach((k, v) -> {
            System.out.println(k + "\t" + v);
        });
    }
}
```

```java
public class TwoInterceptor implements ConsumerInterceptor<String, String> {
    @Override
    public ConsumerRecords<String, String> onConsume(ConsumerRecords<String, String> records) {
        // poll方法返回结果之前最后要调用的方法
        System.out.println("Two -- 开始");
        // 消息不做处理，直接返回
        return records;
    }

    @Override
    public void onCommit(Map<TopicPartition, OffsetAndMetadata> offsets) {
        // 消费者提交偏移量的时候，经过该方法
        System.out.println("Two -- 结束");
    }

    @Override
    public void close() {
        // 用于关闭该拦截器用到的资源，如打开的文件，连接的数据库等
    }

    @Override
    public void configure(Map<String, ?> configs) {
        // 用于获取消费者的设置参数
        configs.forEach((k, v) -> {
            System.out.println(k + "\t" + v);
        });
    }
}
```

```java
public class ThreeInterceptor implements ConsumerInterceptor<String, String> {
    @Override
    public ConsumerRecords<String, String> onConsume(ConsumerRecords<String, String> records) {
        // poll方法返回结果之前最后要调用的方法
        System.out.println("Three -- 开始");
        // 消息不做处理，直接返回
        return records;
    }

    @Override
    public void onCommit(Map<TopicPartition, OffsetAndMetadata> offsets) {
        // 消费者提交偏移量的时候，经过该方法
        System.out.println("Three -- 结束");
    }

    @Override
    public void close() {
        // 用于关闭该拦截器用到的资源，如打开的文件，连接的数据库等
    }

    @Override
    public void configure(Map<String, ?> configs) {
        // 用于获取消费者的设置参数
        configs.forEach((k, v) -> {
            System.out.println(k + "\t" + v);
        });
    }
}
```

##### 消费者参数配置补充

![](img\81.png)

![](img\82.png)

![](img\83.png)

![](img\84.png)

![](img\85.png)

#### 消费组管理

**一、消费者组** **(Consumer Group)**

**1** **什么是消费者组**

consumer group是kafka提供的可扩展且具有容错性的消费者机制。

三个特性：

1. 消费组有一个或多个消费者，消费者可以是一个进程，也可以是一个线程

2. group.id是一个字符串，唯一标识一个消费组

3. 消费组订阅的主题每个分区只能分配给消费组一个消费者。

   

**2** **消费者位移(consumer position)**

消费者在消费的过程中记录已消费的数据，即消费位移（offset）信息。

每个消费组保存自己的位移信息，那么只需要简单的一个整数表示位置就够了；同时可以引入checkpoint机制定期持久化。

**3** **位移管理(offset management)**

**3.1** **自动VS手动**

Kafka默认定期自动提交位移( enable.auto.commit = true )，也手动提交位移。另外kafka会定期把group消费情况保存起来，做成一个offset map，如下图所示：

![](img\86.png)

**3.2** **位移提交**

位移是提交到Kafka中的 __consumer_offsets 主题。 __consumer_offsets 中的消息保存了每个消费组某一时刻提交的offset信息。

```shell
[root@node1 __consumer_offsets-0]# kafka-console-consumer.sh --topic __consumer_offsets --bootstrap-server node1:9092 --formatter "kafka.coordinator.group.GroupMetadataManager\$OffsetsMessageFormatter" -- consumer.config /opt/kafka_2.12-1.0.2/config/consumer.properties --from- beginning
```

![](img\87.png)

上图中，标出来的，表示消费组为 test-consumer-group ，消费的主题为 __consumer_offsets ，消费的分区是4，偏移量为5。 

__consumers_offsets 主题配置了compact策略，使得它总是能够保存最新的位移信息，既控制了该topic总体的日志容量，也能实现保存最新offset的目的。

**4** **再谈再均衡**

**4.1** **什么是再均衡？**

再均衡（Rebalance）本质上是一种协议，规定了一个消费组中所有消费者如何达成一致来分配订阅主题的每个分区。

比如某个消费组有20个消费组，订阅了一个具有100个分区的主题。正常情况下，Kafka平均会为每个消费者分配5个分区。这个分配的过程就叫再均衡。

**4.2** **什么时候再均衡？**

再均衡的触发条件：

1. 组成员发生变更(新消费者加入消费组组、已有消费者主动离开或崩溃了) 

2. 订阅主题数发生变更。如果正则表达式进行订阅，则新建匹配正则表达式的主题触发再均衡。

3. 订阅主题的分区数发生变更

**4.3** **如何进行组内分区分配？**

三种分配策略：RangeAssignor和RoundRobinAssignor以及StickyAssignor。后面讲。

**4.4** **谁来执行再均衡和消费组管理？**

Kafka提供了一个角色：Group Coordinator来执行对于消费组的管理。

Group Coordinator——每个消费组分配一个消费组协调器用于组管理和位移管理。当消费组的第一个消费者启动的时候，它会去和Kafka Broker确定谁是它们组的组协调器。之后该消费组内所有消费者和该组协调器协调通信。

**4.5** **如何确定coordinator？**

两步：

1. 确定消费组位移信息写入 __consumers_offsets 的哪个分区。具体计算公式：__consumers_offsets partition# = Math.abs(groupId.hashCode() %groupMetadataTopicPartitionCount) 注意：groupMetadataTopicPartitionCount由 offsets.topic.num.partitions 指定，默认是50个分区。

2. 该分区leader所在的broker就是组协调器。

**4.6 Rebalance Generation**

它表示Rebalance之后主题分区到消费组中消费者映射关系的一个版本，主要是用于保护消费组，隔离无效偏移量提交的。如上一个版本的消费者无法提交位移到新版本的消费组中，因为映射关系变了，你消费的或许已经不是原来的那个分区了。每次group进行Rebalance之后，Generation号都会加1，表示消费组和分区的映射关系到了一个新版本，如下图所示： Generation 1时group有3个成员，随后成员2退出组，消费组协调器触发Rebalance，消费组进入Generation 2，之后成员4加入，再次触发Rebalance，消费组进入Generation 3.

![](img\88.png)

**4.7** **协议(protocol)**

kafka提供了5个协议来处理与消费组协调相关的问题：

1. Heartbeat请求：consumer需要定期给组协调器发送心跳来表明自己还活着

2. LeaveGroup请求：主动告诉组协调器我要离开消费组

3. SyncGroup请求：消费组Leader把分配方案告诉组内所有成员

4. JoinGroup请求：成员请求加入组

5. DescribeGroup请求：显示组的所有信息，包括成员信息，协议名称，分配方案，订阅信息等。通常该请求是给管理员使用

组协调器在再均衡的时候主要用到了前面4种请求。

**4.8 liveness**

消费者如何向消费组协调器证明自己还活着？ 通过定时向消费组协调器发送Heartbeat请求。如果超过了设定的超时时间，那么协调器认为该消费者已经挂了。一旦协调器认为某个消费者挂了，那么它就会开启新一轮再均衡，并且在当前其他消费者的心跳**响应中**添加“REBALANCE_IN_PROGRESS”，告诉其他消费者：重新分配分区。

**4.9** **再均衡过程**

再均衡分为2步：Join和Sync

1. Join， 加入组。所有成员都向消费组协调器发送JoinGroup请求，请求加入消费组。一旦所有成员都发送了JoinGroup请求，协调i器从中选择一个消费者担任Leader的角色，并把组成员信息以及订阅信息发给Leader。 

2. Sync，Leader开始分配消费方案，即哪个消费者负责消费哪些主题的哪些分区。一旦完成分配，Leader会将这个方案封装进SyncGroup请求中发给消费组协调器，非Leader也会发SyncGroup请求，只是内容为空。消费组协调器接收到分配方案之后会把方案塞进SyncGroup的response中发给各个消费者。

![](img\89.png)

注意：在协调器收集到所有成员请求前，它会把已收到请求放入一个叫purgatory(炼狱)的地方。然后是分发分配方案的过程，即SyncGroup请求：

![](img\90.png)

注意：消费组的分区分配方案在客户端执行。Kafka交给客户端可以有更好的灵活性。Kafka默认提供三种分配策略：range和round-robin和sticky。可以通过消费者的参数：

partition.assignment.strategy 来实现自己分配策略。

**4.10** **消费组状态机**

消费组组协调器根据状态机对消费组做不同的处理：

![](img\91.png)

说明：

1. Dead：组内已经没有任何成员的最终状态，组的元数据也已经被组协调器移除了。这种状态响应各种请求都是一个response： UNKNOWN_MEMBER_ID

2. Empty：组内无成员，但是位移信息还没有过期。这种状态只能响应JoinGroup请求

3. PreparingRebalance：组准备开启新的rebalance，等待成员加入

4. AwaitingSync：正在等待leader consumer将分配方案传给各个成员

5. Stable：再均衡完成，可以开始消费。

### 主题

#### 管理

使用kafka-topics.sh脚本：

![](img\92.png)

![](img\93.png)

![](img\94.png)

主题中可以使用的参数定义:

![](img\95.png)

![](img\96.png)

![](img\97.png)

##### 创建主题

```shell
kafka-topics.sh --zookeeper localhost:2181/myKafka --create --topic topic_x - -partitions 1 --replication-factor 1

kafka-topics.sh --zookeeper localhost:2181/myKafka --create --topic topic_test_02 --partitions 3 --replication-factor 1 --config max.message.bytes=1048576 --config segment.bytes=10485760
```

##### 查看主题

```shell
kafka-topics.sh --zookeeper localhost:2181/myKafka --list
kafka-topics.sh --zookeeper localhost:2181/myKafka --describe --topic topic_x
kafka-topics.sh --zookeeper localhost:2181/myKafka --topics-with-overrides -- describe
```

##### 修改主题

```
kafka-topics.sh --zookeeper localhost:2181/myKafka --create --topic topic_test_01 --partitions 2 --replication-factor 1
kafka-topics.sh --zookeeper localhost:2181/myKafka --alter --topic topic_test_01 --config max.message.bytes=1048576 
kafka-topics.sh --zookeeper localhost:2181/myKafka --describe --topic topic_test_01
kafka-topics.sh --zookeeper localhost:2181/myKafka --alter --topic topic_test_01 --config segment.bytes=10485760
kafka-topics.sh --zookeeper localhost:2181/myKafka --alter --delete-config max.message.bytes --topic topic_test_01
```

##### 删除主题

```shell
kafka-topics.sh --zookeeper localhost:2181/myKafka --delete --topic topic_x
```

![](img\98.png)

#### 增加分区

通过命令行工具操作，主题的分区只能增加，不能减少。否则报错：

```shell
ERROR org.apache.kafka.common.errors.InvalidPartitionsException: The number of partitions for a topic can only be increased. Topic myTop1 currently has 2 partitions, 1 would not be an increase.
```

通过--alter修改主题的分区数，增加分区

```shell
kafka-topics.sh --zookeeper localhost/myKafka --alter --topic myTop1 -- partitions 2
```

#### 必要参数配置

kafka-topics.sh --config xx=xx --config yy=yy

配置给主题的参数。

![](img\99.png)

![](img\100.png)

![](img\101.png)

####  KafkaAdminClient应用

**说明**

除了使用Kafka的bin目录下的脚本工具来管理Kafka，还可以使用管理Kafka的API将某些管理查看的功能集成到系统中。在Kafka0.11.0.0版本之前，可以通过kafka-core包（Kafka的服务端，采用Scala编写）中的AdminClient和AdminUtils来实现部分的集群管理操作。Kafka0.11.0.0之后，又多了一个AdminClient，在kafka-client包下，一个抽象类，具体的实现是org.apache.kafka.clients.admin.KafkaAdminClient。

**功能与原理介绍**

Kafka官网：The AdminClient API supports managing and inspecting topics, brokers, acls, and other Kafka objects。

KafkaAdminClient包含了一下几种功能（以Kafka1.0.2版本为准):

 创建主题：

1. createTopics(final Collection<NewTopic> newTopics, final CreateTopicsOptions options)

2. 删除主题：

​       1. deleteTopics(final Collection<String> topicNames, DeleteTopicsOptions options)

3. 列出所有主题：

​      1. listTopics(final ListTopicsOptions options)

4. 查询主题：

​       1. describeTopics(final Collection<String> topicNames, DescribeTopicsOptions options)

5. 查询集群信息： 

​       1. describeCluster(DescribeClusterOptions options)

6. 查询配置信息：

​       1. describeConfigs(Collection<ConfigResource> configResources, finalDescribeConfigsOptions options)

7. 修改配置信息：

​       1. alterConfigs(Map<ConfigResource, Config> configs, final AlterConfigsOptions options)

8. 修改副本的日志目录：

​       1. alterReplicaLogDirs(Map<TopicPartitionReplica, String> replicaAssignment, final AlterReplicaLogDirsOptions options)

9. 查询节点的日志目录信息：

​      1. describeLogDirs(Collection<Integer> brokers, DescribeLogDirsOptions options)

10. 查询副本的日志目录信息：

​      1. describeReplicaLogDirs(Collection<TopicPartitionReplica> replicas,DescribeReplicaLogDirsOptions options)

11. 增加分区：

​      1. createPartitions(Map<String, NewPartitions> newPartitions, final CreatePartitionsOptions options)

其内部原理是使用Kafka自定义的一套二进制协议来实现，详细可以参见Kafka协议。

**用到的参数：**

![](img\102.png)

![](img\103.png)

![](img\104.png)

主要操作步骤：

客户端根据方法的调用创建相应的协议请求，比如创建Topic的createTopics方法，其内部就是发送CreateTopicRequest请求。

客户端发送请求至Kafka Broker。

Kafka Broker处理相应的请求并回执，比如与CreateTopicRequest对应的是CreateTopicResponse。 客户端接收相应的回执并进行解析处理。

和协议有关的请求和回执的类基本都在org.apache.kafka.common.requests包中，AbstractRequest和AbstractResponse是这些请求和响应类的两个父类。

综上，如果要自定义实现一个功能，只需要三个步骤：

1. 自定义XXXOptions;

2. 自定义XXXResult返回值；

3. 自定义Call，然后挑选合适的XXXRequest和XXXResponse来实现Call类中的3个抽象方法。

```java
public class MyAdminClient {
    private KafkaAdminClient client;

    @Before
    public void before() {
        Map<String, Object> conf = new HashMap<>();
        conf.put("bootstrap.servers", "node1:9092");
        conf.put("client.id", "adminclient-1");
        client = (KafkaAdminClient) KafkaAdminClient.create(conf);
    }

    @After
    public void after() {
        client.close();
    }

    @Test
    public void testListTopics1() throws ExecutionException, InterruptedException {
        ListTopicsResult listTopicsResult = client.listTopics();
        // KafkaFuture<Collection<TopicListing>> listings = listTopicsResult.listings();
        // Collection<TopicListing> topicListings = listings.get();
        // topicListings.forEach(new Consumer<TopicListing>() {
        //        @Override
        //        public void accept(TopicListing topicListing) {
        //              boolean internal = topicListing.isInternal();
        //              String name = topicListing.name();
        //              String s = topicListing.toString();
        //              System.out.println(s + "\t" + name + "\t" + internal);
        //              }
        //              });
        //       KafkaFuture<Set<String>> names = listTopicsResult.names();
        //       Set<String> strings = names.get();
        //       strings.forEach(name -> {  System.out.println(name);  });
        //       KafkaFuture<Map<String, TopicListing>> mapKafkaFuture = listTopicsResult.namesToListings();
        //       Map<String, TopicListing> stringTopicListingMap = mapKafkaFuture.get();
        //       stringTopicListingMap.forEach((k, v) -> {  System.out.println(k + "\t" + v); });
        ListTopicsOptions options = new ListTopicsOptions();
        options.listInternal(false);
        options.timeoutMs(500);
        ListTopicsResult listTopicsResult1 = client.listTopics(options);
        Map<String, TopicListing> stringTopicListingMap = listTopicsResult1.namesToListings().get();
        stringTopicListingMap.forEach((k, v) -> {
            System.out.println(k + "\t" + v);
        });
        // 关闭管理客户端
        client.close();
    }

    @Test
    public void testCreateTopic() throws ExecutionException, InterruptedException {
        Map<String, String> configs = new HashMap<>();
        configs.put("max.message.bytes", "1048576");
        configs.put("segment.bytes", "1048576000");
        NewTopic newTopic = new NewTopic("adm_tp_01", 2, (short) 1);
        newTopic.configs(configs);
        CreateTopicsResult topics = client.createTopics(Collections.singleton(newTopic));
        KafkaFuture<Void> all = topics.all();
        Void aVoid = all.get();
        System.out.println(aVoid);
    }

    @Test
    public void testDeleteTopic() throws ExecutionException, InterruptedException {
        DeleteTopicsOptions options = new DeleteTopicsOptions();
        options.timeoutMs(500);
        DeleteTopicsResult deleteResult = client.deleteTopics(Collections.singleton("adm_tp_01"), options);
        deleteResult.all().get();
    }

    @Test
    public void testAlterTopic() throws ExecutionException, InterruptedException {
        NewPartitions newPartitions = NewPartitions.increaseTo(5);
        Map<String, NewPartitions> newPartitionsMap = new HashMap<>();
        newPartitionsMap.put("adm_tp_01", newPartitions);
        CreatePartitionsOptions option = new CreatePartitionsOptions();
        // Set to true if the request should be validated without creating new partitions.
        // 如果只是验证，而不创建分区，则设置为true // option.validateOnly(true);
        CreatePartitionsResult partitionsResult = client.createPartitions(newPartitionsMap, option);
        Void aVoid = partitionsResult.all().get();
    }

    @Test
    public void testDescribeTopics() throws ExecutionException, InterruptedException {
        DescribeTopicsOptions options = new DescribeTopicsOptions();
        options.timeoutMs(3000);
        DescribeTopicsResult topicsResult = client.describeTopics(Collections.singleton("adm_tp_01"), options);
        Map<String, TopicDescription> stringTopicDescriptionMap = topicsResult.all().get();
        stringTopicDescriptionMap.forEach((k, v) -> {
            System.out.println(k + "\t" + v);
            System.out.println("=======================================");
            System.out.println(k);
            boolean internal = v.isInternal();
            String name = v.name();
            List<TopicPartitionInfo> partitions = v.partitions();
            String partitionStr = Arrays.toString(partitions.toArray());
            System.out.println("内部的？" + internal);
            System.out.println("topic name = " + name);
            System.out.println("分区：" + partitionStr);
            partitions.forEach(partition -> {
                System.out.println(partition);
            });
        });
    }

    @Test
    public void testDescribeCluster() throws ExecutionException, InterruptedException {
        DescribeClusterResult describeClusterResult = client.describeCluster();
        KafkaFuture<String> stringKafkaFuture = describeClusterResult.clusterId();
        String s = stringKafkaFuture.get();
        System.out.println("cluster name = " + s);
        KafkaFuture<Node> controller = describeClusterResult.controller();
        Node node = controller.get();
        System.out.println("集群控制器：" + node);
        Collection<Node> nodes = describeClusterResult.nodes().get();
        nodes.forEach(node1 -> {
            System.out.println(node1);
        });
    }

    @Test
    public void testDescribeConfigs() throws ExecutionException, InterruptedException, TimeoutException {
        ConfigResource configResource = new ConfigResource(ConfigResource.Type.BROKER, "0");
        DescribeConfigsResult describeConfigsResult
        client.describeConfigs(Collections.singleton(configResource));
        Map<ConfigResource, Config> configMap = describeConfigsResult.all().get(15, TimeUnit.SECONDS);
        configMap.forEach(new BiConsumer<ConfigResource, Config>() {
            @Override
            public void accept(ConfigResource configResource, Config config) {
                ConfigResource.Type type = configResource.type();
                String name = configResource.name();
                System.out.println("资源名称：" + name);
                Collection<ConfigEntry> entries = config.entries();
                entries.forEach(new Consumer<ConfigEntry>() {
                    @Override
                    public void accept(ConfigEntry configEntry) {
                        boolean aDefault = configEntry.isDefault();
                        boolean readOnly = configEntry.isReadOnly();
                        boolean sensitive = configEntry.isSensitive();
                        String name1 = configEntry.name();
                        String value = configEntry.value();
                        System.out.println("是否默认：" + aDefault + "\t是否 只读？" + readOnly + "\t是否敏感？" + sensitive + "\t" + name1 + " --> " + value);
                    }
                });
                ConfigEntry retries = config.get("retries");
                if (retries != null) {
                    System.out.println(retries.name() + " -->" + retries.value());
                } else {
                    System.out.println("没有这个属性");
                }
            }
        });
    }

    @Test
    public void testAlterConfig() throws ExecutionException, InterruptedException {
        // 这里设置后，原来资源中不冲突的属性也会丢失，直接按照这里的配置设置
        Map<ConfigResource, Config> configMap = new HashMap<>();
        ConfigResource resource = new ConfigResource(ConfigResource.Type.TOPIC, "adm_tp_01");
        Config config = new Config(Collections.singleton(new ConfigEntry("segment.bytes", "1048576000")));
        configMap.put(resource, config);
        AlterConfigsResult alterConfigsResult = client.alterConfigs(configMap);
        Void aVoid = alterConfigsResult.all().get();
    }

    @Test
    public void testDescribeLogDirs() throws ExecutionException, InterruptedException {
        DescribeLogDirsOptions option = new DescribeLogDirsOptions();
        option.timeoutMs(1000);
        DescribeLogDirsResult describeLogDirsResult = client.describeLogDirs(Collections.singleton(0), option);
        Map<Integer, Map<String, DescribeLogDirsResponse.LogDirInfo>> integerMapMap = describeLogDirsResult.all().get();
        integerMapMap.forEach(new BiConsumer<Integer, Map<String, DescribeLogDirsResponse.LogDirInfo>>() {
            @Override
            public void accept(Integer integer, Map<String, DescribeLogDirsResponse.LogDirInfo> stringLogDirInfoMap) {
                System.out.println("broker.id = " + integer);
                stringLogDirInfoMap.forEach(new BiConsumer<String, DescribeLogDirsResponse.LogDirInfo>() {
                    @Override
                    public void accept(String s, DescribeLogDirsResponse.LogDirInfo logDirInfo) {
                        System.out.println("log.dirs：" + s);
                        // 查看该broker上的主题/分区/偏移量等信息
                        // logDirInfo.replicaInfos.forEach(new BiConsumer<TopicPartition, DescribeLogDirsResponse.ReplicaInfo>() {
                        //           @Override
                        //           public void accept(TopicPartition topicPartition, DescribeLogDirsResponse.ReplicaInfo replicaInfo) {
                        //              int partition = topicPartition.partition();
                        //              String topic = topicPartition.topic();
                        //              boolean isFuture = replicaInfo.isFuture;
                        //              long offsetLag = replicaInfo.offsetLag;
                        //              long size = replicaInfo.size;
                        //              System.out.println("partition:" + partition + "\ttopic:" + topic // + "\tisFuture:" + isFuture // + "\toffsetLag:" + offsetLag // + "\tsize:" + size);
                        //            }
                        //            });
                    });
                }
            });
        }
    }
```

#### 偏移量管理

Kafka 1.0.2，__consumer_offsets主题中保存各个消费组的偏移量。

早期由zookeeper管理消费组的偏移量。

**查询方法：**

通过原生 kafka 提供的工具脚本进行查询。

工具脚本的位置与名称为 bin/kafka-consumer-groups.sh

首先运行脚本，查看帮助：

![](img\105.png)

![](img\106.png)

![](img\107.png)

![](img\108.png)

这里我们先编写一个生产者，消费者的例子：

我们先启动消费者，再启动生产者， 再通过 bin/kafka-consumer-groups.sh 进行消费偏移量查询，

由于kafka 消费者记录group的消费偏移量有两种方式 ： 

1）kafka 自维护 （新）

2）zookpeer 维护 (旧) ，已经逐渐被废弃

所以 ，脚本只查看由broker维护的，由zookeeper维护的可以将 --bootstrap-server 换成 -- zookeeper 即可。

1**.** **查看有那些** **group ID** **正在进行消费：**

```
[root@node11 ~]# kafka-consumer-groups.sh --bootstrap-server node1:9092 -- list Note: This will not show information about old Zookeeper-based consumers. group
```

![](img\109.png)

**注意：**

1. **这里面是没有指定** **topic****，查看的是所有****topic****消费者的** **group.id** **的列表。**

2. **注意： 重名的** **group.id** **只会显示一次**

2.**查看指定group.id的消费者消费情况**

```
[root@node11 ~]# kafka-consumer-groups.sh --bootstrap-server node1:9092 -- describe --group group
Note: This will not show information about old Zookeeper-based consumers. TOPIC PARTITION CURRENT-OFFSET LOG-END-OFFSET LAG CONSUMER-ID HOST CLIENT-ID tp_demo_02 0 923 923 0 consumer-1-6d88cc72-1bf1-4ad7-8c6c-060d26dc1c49 /192.168.100.1 consumer-1 tp_demo_02 1 872 872 0 consumer-1-6d88cc72-1bf1-4ad7-8c6c-060d26dc1c49 /192.168.100.1 consumer-1 tp_demo_02 2 935 935 0 consumer-1-6d88cc72-1bf1-4ad7-8c6c-060d26dc1c49 /192.168.100.1 consumer-1 
[root@node11 ~]#
```

如果消费者停止，查看偏移量信息：

![](img\110.png)

将偏移量设置为最早的：

![](img\111.png)

```
......
```

### 分区

#### 副本机制

![](img\112.png)

Kafka在一定数量的服务器上对主题分区进行复制。

当集群中的一个broker宕机后系统可以自动故障转移到其他可用的副本上，不会造成数据丢失。

--replication-factor 3 1leader+2follower

1. 将复制因子为1的未复制主题称为复制主题。

2. 主题的分区是复制的最小单元。

3. 在非故障情况下，Kafka中的每个分区都有一个Leader副本和零个或多个Follower副本。

4. 包括Leader副本在内的副本总数构成复制因子。

5. 所有读取和写入都由Leader副本负责。

6. 通常，分区比broker多，并且Leader分区在broker之间平均分配。

**Follower分区像普通的Kafka消费者一样，消费来自Leader分区的消息，并将其持久化到自己的日**

**志中。**

允许Follower对日志条目拉取进行**批处理**。

同步节点定义：

1. 节点必须能够维持与ZooKeeper的会话（通过ZooKeeper的心跳机制）

2. 对于Follower副本分区，它复制在Leader分区上的写入，并且不要延迟太多

Kafka提供的保证是，只要有至少一个同步副本处于活动状态，提交的消息就不会丢失。

宕机如何恢复

（1）少部分副本宕机

当leader宕机了，会从follower选择一个作为leader。当宕机的重新恢复时，会把之前commit的数据清空，重新从leader里pull数据。

（2）全部副本宕机

当全部副本宕机了有两种恢复方式

1、等待ISR中的一个恢复后，并选它作为leader。（等待时间较长，降低可用性）

2、选择第一个恢复的副本作为新的leader，无论是否在ISR中。（并未包含之前leader commit的数据，因此造成数据丢失）

#### Leader选举

下图中

分区P1的Leader是0，ISR是0和1

分区P2的Leader是2，ISR是1和2

分区P3的Leader是1，ISR是0，1，2。

![](img\113.png)

生产者和消费者的请求都由Leader副本来处理。Follower副本只负责消费Leader副本的数据和Leader保持同步。

对于P1，如果0宕机会发生什么？

Leader副本和Follower副本之间的关系并不是固定不变的，在Leader所在的broker发生故障的时候，就需要进行分区的Leader副本和Follower副本之间的切换，需要选举Leader副本。

**如何选举？**

如果某个分区所在的服务器除了问题，不可用，kafka会从该分区的其他的副本中选择一个作为新的Leader。之后所有的读写就会转移到这个新的Leader上。现在的问题是应当选择哪个作为新的Leader.

只有那些跟Leader保持同步的Follower才应该被选作新的Leader。

Kafka会在Zookeeper上针对每个Topic维护一个称为ISR（in-sync replica，已同步的副本）的集合，该集合中是一些分区的副本。

只有当这些副本都跟Leader中的副本同步了之后，kafka才会认为消息已提交，并反馈给消息的生产者。

如果这个集合有增减，kafka会更新zookeeper上的记录。

如果某个分区的Leader不可用，Kafka就会从ISR集合中选择一个副本作为新的Leader。

显然通过ISR，kafka需要的**冗余度较低**，可以容忍的失败数比较高。

假设某个topic有N+1个副本，kafka可以容忍N个服务器不可用。



**为什么不用少数服从多数的方法**

少数服从多数是一种比较常见的一致性算发和Leader选举法。

它的含义是只有超过半数的副本同步了，系统才会认为数据已同步；

选择Leader时也是从超过半数的同步的副本中选择。

这种算法需要较高的冗余度，跟Kafka比起来，浪费资源。

譬如只允许一台机器失败，需要有三个副本；而如果只容忍两台机器失败，则需要五个副本。

而kafka的ISR集合方法，分别只需要两个和三个副本。

**如果所有的ISR副本都失败了怎么办？**

此时有两种方法可选，

1. 等待ISR集合中的副本复活，

2. 选择任何一个立即可用的副本，而这个副本不一定是在ISR集合中
   - 需要设置 unclean.leader.election.enable=tr

这两种方法各有利弊，实际生产中按需选择。 如果要等待ISR副本复活，虽然可以保证一致性，但可能需要很长时间。而如果选择立即可用的副本，则很可能该副本并不一致。



**总结：**

Kafka中Leader分区选举，通过维护一个动态变化的ISR集合来实现，一旦Leader分区丢掉，则从ISR中随机挑选一个副本做新的Leader分区。

如果ISR中的副本都丢失了，则：

1. 可以等待ISR中的副本任何一个恢复，接着对外提供服务，需要时间等待。

2. 从OSR中选出一个副本做Leader副本，此时会造成数据丢失

#### 分区重新分配

向已经部署好的Kafka集群里面添加机器，我们需要从已经部署好的Kafka节点中复制相应的配置文件，然后把里面的broker id修改成全局唯一的，最后启动这个节点即可将它加入到现有Kafka集群中。

问题：新添加的Kafka节点并不会自动地分配数据，无法分担集群的负载，除非我们新建一个topic。

需要手动将部分分区移到新添加的Kafka节点上，Kafka内部提供了相关的工具来重新分布某个topic的分区。

在重新分布topic分区之前，我们先来看看现在topic的各个分区的分布位置：

1. 创建主题：

```
[root@node1 ~]# kafka-topics.sh --zookeeper node1:2181/myKafka -- create --topic tp_re_01 --partitions 5 --replication-factor 1
```

2. 查看主题信息：

   ```
   [root@node1 ~]# kafka-topics.sh --zookeeper node1:2181/myKafka -- describe --topic tp_re_01 Topic:tp_re_01 PartitionCount:5 ReplicationFactor:1 Configs: 
   Topic: tp_re_01 Partition: 0 Leader: 0 Replicas: 0 Isr: 0 
   Topic: tp_re_01 Partition: 1 Leader: 0 Replicas: 0 Isr: 0
   Topic: tp_re_01 Partition: 2 Leader: 0 Replicas: 0 Isr: 0
   Topic: tp_re_01 Partition: 3 Leader: 0 Replicas: 0 Isr: 0 
   Topic: tp_re_01 Partition: 4 Leader: 0 Replicas: 0 Isr: 0
   ```

3. 在node11搭建Kafka

   - 拷贝JDK并安装

```
[root@node1 opt]# scp jdk-8u261-linux-x64.rpm node11:~
```

此处不需要zookeeper，切记！！！

![](img\114.png)

让配置生效：

```
. /etc/profile
```

拷贝node1上安装的Kafka

```
[root@node1 opt]# scp -r kafka_2.12-1.0.2/ node11:/opt
```

修改node11上Kafka的配置：

![](img\115.png)

启动Kafka：

```
[root@node11 ~]# kafka-server-start.sh /opt/kafka_2.12- 1.0.2/config/server.properties
```

注意观察node11上节点启动的时候的ClusterId，看和zookeeper节点上的ClusterId是否一致，如果是，证明node11和node1在同一个集群中。 node11启动的Cluster ID：

![](img\116.png)

4. 现在我们在现有集群的基础上再添加一个Kafka节点，然后使用Kafka自带的 kafka-reassign- partitions.sh 工具来重新分布分区。该工具有三种使用模式：

   1、generate模式，给定需要重新分配的Topic，自动生成reassign plan（并不执行） 

   2、execute模式，根据指定的reassign plan重新分配Partition 3、verify模式，验证重新分配Partition是否成功

   

5. 我们将分区3和4重新分布到broker1上，借助kafka-reassign-partitions.sh工具生成reassignplan，不过我们先得按照要求定义一个文件，里面说明哪些topic需要重新分区，文件内容如下:

   ```
   [root@node1 ~]# cat topics-to-move.json { "topics": [ { "topic":"tp_re_01" } ],"version":1 }
   ```

   然后使用 kafka-reassign-partitions.sh 工具生成reassign plan

```
............
```

####  自动再均衡

我们可以在新建主题的时候，手动指定主题各个Leader分区以及Follower分区的分配情况，即什么分区副本在哪个broker节点上。

随着系统的运行，broker的宕机重启，会引发Leader分区和Follower分区的角色转换，最后可能Leader大部分都集中在少数几台broker上，由于Leader负责客户端的读写操作，此时集中Leader分区的少数几台服务器的网络I/O，CPU，以及内存都会很紧张。Leader和Follower的角色转换会引起Leader副本在集群中分布的不均衡，此时我们需要一种手段，让Leader的分布重新恢复到一个均衡的状态。

执行脚本：

```
[root@node11 ~]# kafka-topics.sh --zookeeper node1:2181/myKafka --create -- topic tp_demo_03 --replica-assignment "0:1,1:0,0:1"
```

上述脚本执行的结果是：创建了主题tp_demo_03，有三个分区，每个分区两个副本，Leader副本在列表中第一个指定的brokerId上，Follower副本在随后指定的brokerId上。

![](img\117.png)

然后模拟broker0宕机的情况：

![](img\118.png)

![](img\119.png)

是否有一种方式，可以让Kafka自动帮我们进行修改？改为初始的副本分配？

此时，用到了Kafka提供的自动再均衡脚本： kafka-preferred-replica-election.sh

先看介绍：

![](img\120.png)

该工具会让每个分区的Leader副本分配在合适的位置，让Leader分区和Follower分区在服务器之间均衡分配。

如果该脚本仅指定zookeeper地址，则会对集群中所有的主题进行操作，自动再平衡。

具体操作：

1. 创建preferred-replica.json，内容如下：

```json
{ "partitions": [ { "topic":"tp_demo_03", "partition":0 },{ "topic":"tp_demo_03", "partition":1 },{ "topic":"tp_demo_03", "partition":2 } ] }
```

2. 执行操作：

![](img\121.png)

3. 查看操作的结果：

![](img\122.png)

恢复到最初的分配情况。

之所以是这样的分配，是因为我们在创建主题的时候：

```
--replica-assignment "0:1,1:0,0:1"
```

在逗号分割的每个数值对中排在前面的是Leader分区，后面的是副本分区。那么所谓的preferred

replica，就是排在前面的数字就是Leader副本应该在的brokerId。

#### 修改分区副本

实际项目中，我们可能由于主题的副本因子设置的问题，需要重新设置副本因子或者由于集群的扩展，需要重新设置副本因子。

topic一旦使用又不能轻易删除重建，因此动态增加副本因子就成为最终的选择。

**说明**：kafka 1.0版本配置文件默认没有default.replication.factor=x， 因此如果创建topic时，不指定–replication-factor 想， 默认副本因子为1. 我们可以在自己的server.properties中配置上常用的副本因子，省去手动调整。例如设置default.replication.factor=3， 详细内容可参考官方文档https://kafka.apache.org/documentation/#replication

**原因分析：**

假设我们有2个kafka broker分别broker0，broker1。 

1. 当我们创建的topic有2个分区partition时并且replication-factor为1，基本上一个broker上一个分区。当一个broker宕机了，该topic就无法使用了，因为两个个分区只有一个能用。

2. 当我们创建的topic有3个分区partition时并且replication-factor为2时，可能分区数据分布情况是 broker0， partiton0，partiton1，partiton2， broker1， partiton1，partiton0，partiton2，

每个分区有一个副本，当其中一个broker宕机了，kafka集群还能完整凑出该topic的两个分区，例如当broker0宕机了，可以通过broker1组合出topic的两个分区。

1. 创建主题：

```
[root@node1 ~]# kafka-topics.sh --zookeeper node1:2181/myKafka -- create --topic tp_re_02 --partitions 3 --replication-factor 1
```

2. 查看主题细节：

```
[root@node1 ~]# kafka-topics.sh --zookeeper node1:2181/myKafka -- describe --topic tp_re_02
Topic:tp_re_02 PartitionCount:3 ReplicationFactor:1 Configs: 
Topic: tp_re_02 Partition: 0 Leader: 1 Replicas: 1 Isr: 1
Topic: tp_re_02 Partition: 1 Leader: 0 Replicas: 0 Isr: 0
Topic: tp_re_02 Partition: 2 Leader: 1 Replicas: 1 Isr: 1 
[root@node1 ~]#
```

3. 修改副本因子：错误

4. 使用 kafka-reassign-partitions.sh 修改副本因子：

   -. 创建increment-replication-factor.json

```json
{ "version":1, 
  "partitions":[ 
  	{"topic":"tp_re_02","partition":0,"replicas":[0,1]},
    {"topic":"tp_re_02","partition":1,"replicas":[0,1]},
    {"topic":"tp_re_02","partition":2,"replicas":[1,0]} 
    ]
}
```

5. 执行分配：

![](img\123.png)

6. 查看主题细节：

![](img\124.png)

7. 搞定！

####  分区分配策略

![](img\125.png)

在Kafka中，每个Topic会包含多个分区，默认情况下一个分区只能被一个消费组下面的一个消费者消费，这里就产生了分区分配的问题。Kafka中提供了多重分区分配算法（PartitionAssignor）的实现：

RangeAssignor、RoundRobinAssignor、StickyAssignor。

#####  RangeAssignor

PartitionAssignor接口用于用户定义实现分区分配算法，以实现Consumer之间的分区分配。

消费组的成员订阅它们感兴趣的Topic并将这种订阅关系传递给作为订阅组协调者的Broker。协调者选择其中的一个消费者来执行这个消费组的分区分配并将分配结果转发给消费组内所有的消费者。

**Kafka默认采用RangeAssignor的分配算法**。

RangeAssignor对每个Topic进行独立的分区分配。对于每一个Topic，首先对分区按照分区ID进行数值排序，然后订阅这个Topic的消费组的消费者再进行字典排序，之后尽量均衡的将分区分配给消费者。这里只能是尽量均衡，因为分区数可能无法被消费者数量整除，那么有一些消费者就会多分配到一些分区。

![](img\126.png)

![](img\127.png)

RangeAssignor策略的原理是按照消费者总数和分区总数进行整除运算来获得一个跨度，然后将分区按照跨度进行平均分配，以保证分区尽可能均匀地分配给所有的消费者。对于每一个Topic，RangeAssignor策略会将消费组内所有订阅这个Topic的消费者按照名称的字典序排序，然后为每个消费者划分固定的分区范围，如果不够平均分配，那么字典序靠前的消费者会被多分配一个分区。

这种分配方式明显的一个问题是随着消费者订阅的Topic的数量的增加，不均衡的问题会越来越严重，比如上图中4个分区3个消费者的场景，C0会多分配一个分区。如果此时再订阅一个分区数为4的Topic，那么C0又会比C1、C2多分配一个分区，这样C0总共就比C1、C2多分配两个分区了，而且随着Topic的增加，这个情况会越来越严重。

字典序靠前的消费组中的消费者比较“**贪婪**”

![](img\128.png)

##### RoundRobinAssignor

RoundRobinAssignor的分配策略是将消费组内订阅的所有Topic的分区及所有消费者进行排序后尽量均衡的分配（RangeAssignor是针对单个Topic的分区进行排序分配的）。如果消费组内，消费者订阅的Topic列表是相同的（每个消费者都订阅了相同的Topic），那么分配结果是尽量均衡的（消费者之间分配到的分区数的差值不会超过1）。如果订阅的Topic列表是不同的，那么分配结果是不保证“尽量均衡”的，因为某些消费者不参与一些Topic的分配。

![](img\129.png)

相对于RangeAssignor，在订阅多个Topic的情况下，RoundRobinAssignor的方式能消费者之间尽量均衡的分配到分区（分配到的分区数的差值不会超过1——RangeAssignor的分配策略可能随着订阅的Topic越来越多，差值越来越大）。

对于消费组内消费者订阅Topic不一致的情况：假设有两个个消费者分别为C0和C1，有2个TopicT1、T2，分别拥有3和2个分区，并且C0订阅T1和T2，C1订阅T2，那么RoundRobinAssignor的分配结果如下：

![](img\130.png)

看上去分配已经尽量的保证均衡了，不过可以发现C0承担了4个分区的消费而C1订阅了T2一个分区，是不是把T2P0交给C1消费能更加的均衡呢

##### StickyAssignor

**动机**

尽管RoundRobinAssignor已经在RangeAssignor上做了一些优化来更均衡的分配分区，但是在一些情况下依旧会产生严重的分配偏差，比如消费组中订阅的Topic列表不相同的情况下。

更核心的问题是无论是RangeAssignor，还是RoundRobinAssignor，当前的分区分配算法都没有考虑**上一次的分配结果**。显然，在执行一次新的分配之前，如果能考虑到上一次分配的结果，尽量少的调整分区分配的变动，显然是能节省很多开销的。

**目标**

从字面意义上看，Sticky是“粘性的”，可以理解为分配结果是带“粘性的”： 

1. **分区的分配尽量的均衡**

2. **每一次重分配的结果尽量与上一次分配结果保持一致**

当这两个目标发生冲突时，优先保证第一个目标。第一个目标是每个分配算法都尽量尝试去完成的，而第二个目标才真正体现出StickyAssignor特性的。

我们先来看预期分配的结构，后续再具体分析StickyAssignor的算法实现。

例如：

1. 有3个Consumer：C0、C1、C2

2. 有4个Topic：T0、T1、T2、T3，每个Topic有2个分区

3. 所有Consumer都订阅了这4个分区

StickyAssignor的分配结果如下图所示（增加RoundRobinAssignor分配作为对比）：

![](img\131.png)

如果消费者1宕机，则按照RoundRobin的方式分配结果如下：

打乱从新来过，轮询分配：

![](img\132.png)

按照Sticky的方式：

仅对消费者1分配的分区进行重分配，红线部分。最终达到均衡的目的。

![](img\133.png)

再举一个例子：

1. 有3个Consumer：C0、C1、C2

2. 3个Topic：T0、T1、T2，它们分别有1、2、3个分区

3. C0订阅T0；C1订阅T0、T1；C2订阅T0、T1、T2

分配结果如下图所示：

![](img\134.png)

消费者0下线，则按照轮询的方式分配:

![](img\135.png)

按照Sticky方式分配分区，仅仅需要动的就是红线部分，其他部分不动。

![](img\136.png)

StickyAssignor分配方式的实现稍微复杂点儿，我们可以先理解图示部分即可。感兴趣的同学可以研究一下。

自定义的分配策略必须要实现org.apache.kafka.clients.consumer.internals.PartitionAssignor接口。PartitionAssignor接口的定义如下：

![](img\137.png)

PartitionAssignor接口中定义了两个内部类：Subscription和Assignment。

Subscription类用来表示消费者的订阅信息，类中有两个属性：topics和userData，分别表示消费者所订阅topic列表和用户自定义信息。PartitionAssignor接口通过subscription()方法来设置消费者自身相关的Subscription信息，注意到此方法中只有一个参数topics，与Subscription类中的topics的相互呼应，但是并没有有关userData的参数体现。为了增强用户对分配结果的控制，可以在subscription()方法内部添加一些影响分配的用户自定义信息赋予userData，比如：权重、ip地址、host或者机架（rack）等等。再来说一下Assignment类，它是用来表示分配结果信息的，类中也有两个属性：partitions和userData，分别表示所分配到的分区集合和用户自定义的数据。可以通过PartitionAssignor接口中的onAssignment()方法是在每个消费者收到消费组leader分配结果时的回调函数，例如在StickyAssignor策略中就是通过这个方法保存当前的分配方案，以备在下次消费组再平衡（rebalance）时可以提供分配参考依据。

接口中的name()方法用来提供分配策略的名称，对于Kafka提供的3种分配策略而言，RangeAssignor对应的protocol_name为“range”，RoundRobinAssignor对应的protocol_name为“roundrobin”，StickyAssignor对应的protocol_name为“sticky”，所以自定义的分配策略中要注意命名的时候不要与已存在的分配策略发生冲突。这个命名用来标识分配策略的名称，在后面所描述的加入消费组以及选举消费组leader的时候会有涉及。

真正的分区分配方案的实现是在assign()方法中，方法中的参数metadata表示集群的元数据信息，而subscriptions表示消费组内各个消费者成员的订阅信息，最终方法返回各个消费者的分配信息。

Kafka中还提供了一个抽象类org.apache.kafka.clients.consumer.internals.AbstractPartitionAssignor，它可以简化PartitionAssignor接口的实现，对assign()方法进行了实现，其中会将Subscription中的userData信息去掉后，在进行分配。Kafka提供的3种分配策略都是继承自这个抽象类。如果开发人员在自定义分区分配策略时需要使用userData信息来控制分区分配的结果，那么就不能直接继承AbstractPartitionAssignor这个抽象类，而需要直接实现PartitionAssignor接口。

![](img\138.png)

### 物理存储

#### 日志存储概述

Kafka 消息是以主题为单位进行归类，各个主题之间是彼此独立的，互不影响。

每个主题又可以分为一个或多个分区。

每个分区各自存在一个记录消息数据的日志文件

![](img\139.png)

图中，创建了一个 tp_demo_01 主题，其存在6个 Parition，对应的每个Parition下存在一个[Topic-Parition] 命名的消息日志文件。在理想情况下，数据流量分摊到各个 Parition 中，实现了负载均衡的效果。在分区日志文件中，你会发现很多类型的文件，比如： .index、.timestamp、.log、.snapshot 等。

其中，文件名一致的文件集合就称为 LogSement。

![](img\140.png)

**LogSegment**

1. 分区日志文件中包含很多的 LogSegment

2. Kafka 日志追加是顺序写入的

3. LogSegment 可以减小日志文件的大小

4. 进行日志删除的时候和数据查找的时候可以快速定位。

5. ActiveLogSegment 是活跃的日志分段，拥有文件拥有写入权限，其余的 LogSegment 只有只读的权限。

日志文件存在多种后缀文件，重点需要关注 .index、.timestamp、.log 三种类型。

**类别作用**

![](img\141.png) 

每个 LogSegment 都有一个基准偏移量，表示当前 LogSegment 中**第一条消息的** **offset**。

偏移量是一个 64 位的长整形数，固定是20位数字，长度未达到，用 0 进行填补，索引文件和日志文件都由该作为文件名命名规则（00000000000000000000.index、00000000000000000000.timestamp、00000000000000000000.log）。

如果日志文件名为 00000000000000000121.log ，则当前日志文件的一条数据偏移量就是121（偏移量从 0 开始）。

![](img\142.png)

**日志与索引文件**

![](img\143.png)

**配置项默认值说明**

偏移量索引文件用于记录消息偏移量与物理地址之间的映射关系。

时间戳索引文件则根据时间戳查找对应的偏移量。

Kafka 中的索引文件是以稀疏索引的方式构造消息的索引，并不保证每一个消息在索引文件中都有对应的索引项。

每当写入一定量的消息时，偏移量索引文件和时间戳索引文件分别增加一个偏移量索引项和时间戳索引项。

通过修改 log.index.interval.bytes 的值，改变索引项的密度。

**切分文件**

当满足如下几个条件中的其中之一，就会触发文件的切分：

1. 当前日志分段文件的大小超过了 broker 端参数 log.segment.bytes 配置的值。log.segment.bytes 参数的默认值为 1073741824，即 1GB。 

2. 当前日志分段中消息的最大时间戳与当前系统的时间戳的差值大于 log.roll.ms 或 log.roll.hours 参数配置的值。如果同时配置了 log.roll.ms 和 log.roll.hours 参数，那么 log.roll.ms 的优先级高。默认情况下，只配置了 log.roll.hours 参数，其值为168，即 7 天。

3. 偏移量索引文件或时间戳索引文件的大小达到 broker 端参数 log.index.size.max.bytes配置的值。 log.index.size.max.bytes 的默认值为 10485760，即 10MB。 

4. 追加的消息的偏移量与当前日志分段的偏移量之间的差值大于 Integer.MAX_VALUE ，即要追加的消息的偏移量不能转变为相对偏移量。

**为什么是Integer.MAX_VALUE**

1024 * 1024 * 1024=1073741824

在偏移量索引文件中，每个索引项共占用 8 个字节，并分为两部分。

相对偏移量和物理地址。

相对偏移量：表示消息相对与基准偏移量的偏移量，占 4 个字节

物理地址：消息在日志分段文件中对应的物理位置，也占 4 个字节

4 个字节刚好对应 Integer.MAX_VALUE ，如果大于 Integer.MAX_VALUE ，则不能用 4 个字节进行表示了。

**索引文件切分过程**

索引文件会根据 log.index.size.max.bytes 值进行**预先分配空间**，即文件创建的时候就是最大值

当真正的进行索引文件切分的时候，才会将其裁剪到实际数据大小的文件。

这一点是跟日志文件有所区别的地方。其意义**降低了代码逻辑的复杂性**

#### 日志存储

##### 索引

偏移量索引文件用于记录**消息偏移量与物理地址**之间的映射关系。时间戳索引文件则根据**时间戳查找对应的偏移量**。



**文件：**

查看一个topic分区目录下的内容，发现有log、index和timeindex三个文件：

1. log文件名是以文件中第一条message的offset来命名的，实际offset长度是64位，但是这里只使用了20位，应付生产是足够的。

2. 一组index+log+timeindex文件的名字是一样的，并且log文件默认写满1G后，会进行logrolling形成一个新的组合来记录消息，这个是通过broker端 log.segment.bytes =1073741824指定的。

3. index和timeindex在刚使用时会分配10M的大小，当进行 log rolling 后，它会修剪为实际的大小。

![](img\144.png)



1、创建主题：

![](img\145.png)

![](img\146.png)

如果想查看这些文件，可以使用kafka提供的shell来完成，几个关键信息如下：

（1）offset是逐渐增加的整数，每个offset对应一个消息的偏移量。

（2）position：消息批字节数，用于计算物理地址。

（3）CreateTime：时间戳。

（4）magic：2代表这个消息类型是V2，如果是0则代表是V0类型，1代表V1类型。

（5）compresscodec：None说明没有指定压缩类型，kafka目前提供了4种可选择，0-None、1-GZIP、2-snappy、3-lz4。 

（6）crc：对所有字段进行校验后的crc值。

![](img\147.png)

**关于消息偏移量：**

一、消息存储

1. 消息内容保存在log日志文件中。

2. 消息封装为Record，追加到log日志文件末尾，采用的是**顺序写**模式。

3. 一个topic的不同分区，可认为是queue，顺序写入接收到的消息。

![](img\148.png)

消费者有offset。下图中，消费者A消费的offset是9，消费者B消费的offset是11，不同的消费者offset是交给一个内部公共topic来记录的。

![](img\149.png)

（3）时间戳索引文件，它的作用是可以让用户查询某个时间段内的消息，它一条数据的结构是时间戳（8byte）+相对offset（4byte），如果要使用这个索引文件，首先需要通过时间范围，找到对应的相对offset，然后再去对应的index文件找到position信息，然后才能遍历log文件，它也是需要使用上面说的index文件的。

但是由于producer生产消息可以指定消息的时间戳，这可能将导致消息的时间戳不一定有先后顺序，因此**尽量不要生产消息时指定时间戳**。

###### 偏移量

1. 位置索引保存在index文件中

2. log日志默认每写入4K（log.index.interval.bytes设定的），会写入一条索引信息到index文件中，因此索引文件是稀疏索引，它**不会为每条日志都建立索引信息**。 

3. log文件中的日志，是顺序写入的，由message+实际offset+position组成

4. 索引文件的数据结构则是由相对offset（4byte）+position（4byte）组成，由于保存的是相对第一个消息的相对offset，只需要4byte就可以了，可以节省空间，在实际查找后还需要计算回实际的offset，这对用户是透明的。

稀疏索引，索引密度不高，但是offset有序，二分查找的时间复杂度为O(lgN)，如果从头遍历时间复杂度是O(N)。

示意图如下：

![](img\150.png)

偏移量索引由相对偏移量和物理地址组成。

![](img\151.png)

可以通过如下命令解析 .index 文件

```shell
kafka-run-class.sh kafka.tools.DumpLogSegments --files 00000000000000000000.index --print-data-log | head
```

注意：offset 与 position 没有直接关系，因为会删除数据和清理日志。

![](img\152.png)

在偏移量索引文件中，索引数据都是顺序记录 offset ，但时间戳索引文件中每个追加的索引时间戳必须大于之前追加的索引项，否则不予追加。在 Kafka 0.11.0.0 以后，消息元数据中存在若干的时间戳信息。如果 broker 端参数 log.message.timestamp.type 设置为 LogAppendTIme ，那么时间戳必定能保持单调增长。反之如果是 CreateTime 则无法保证顺序。

注意：timestamp文件中的 offset 与 index 文件中的 relativeOffset 不是一一对应的。因为数据的写入是各自追加。

思考：如何查看偏移量为23的消息？

Kafka 中存在一个 ConcurrentSkipListMap 来保存在每个日志分段，通过跳跃表方式，定位到在00000000000000000000.index ，通过二分法在偏移量索引文件中找到不大于 23 的**最大索引项**，即offset 20 那栏，然后从日志分段文件中的物理位置为320 开始顺序查找偏移量为 23 的消息。

###### 时间戳

在偏移量索引文件中，索引数据都是顺序记录 offset ，但时间戳索引文件中每个追加的索引时间戳必须大于之前追加的索引项，否则不予追加。在 Kafka 0.11.0.0 以后，消息信息中存在若干的时间戳信息。如果 broker 端参数 log.message.timestamp.type 设置为 LogAppendTIme ，那么时间戳必定能保持单调增长。反之如果是 CreateTime 则无法保证顺序。

通过时间戳方式进行查找消息，需要通过查找时间戳索引和偏移量索引两个文件。

时间戳索引索引格式：前八个字节表示**时间戳**，后四个字节表示**偏移量**。

![](img\153.png)

**思考：查找时间戳为** **1557554753430** **开始的消息？**

1. 查找该时间戳应该在哪个日志分段中。将1557554753430和每个日志分段中最大时间戳largestTimeStamp逐一对比，直到找到不小于1557554753430所对应的日志分段。日志分段中的largestTimeStamp的计算是：先查询该日志分段所对应**时间戳索引文件**，找到**最后一条索引项**，若最后一条索引项的时间戳字段值大于0，则取该值，否则取该日志分段的最近修改

时间。

2. 查找该日志分段的偏移量索引文件，查找该偏移量对应的物理地址。

3. 日志文件中从 320 的物理位置开始查找不小于 1557554753430 数据。

注意：timestamp文件中的 offset 与 index 文件中的 relativeOffset 不是一一对应的，因为数据的写入是各自追加。

##### 清理

Kafka 提供两种日志清理策略：

日志删除：按照一定的删除策略，将不满足条件的数据进行数据删除

日志压缩：针对每个消息的 Key 进行整合，对于有相同 Key 的不同 Value 值，只保留最后一个版本。

Kafka 提供 log.cleanup.policy 参数进行相应配置，默认值： delete ，还可以选择compact 。

主题级别的配置项是 cleanup.policy 。

######  日志删除

**基于时间**

日志删除任务会根据 log.retention.hours/log.retention.minutes/log.retention.ms 设定

日志保留的时间节点。如果超过该设定值，就需要进行删除。默认是 7 天， log.retention.ms 优先级最高。

Kafka 依据日志分段中最大的时间戳进行定位。

首先要查询该日志分段所对应的时间戳索引文件，查找时间戳索引文件中最后一条索引项，若最后一条索引项的时间戳字段值大于 0，则取该值，否则取最近修改时间。

**为什么不直接选最近修改时间呢？**

因为日志文件可以有意无意的被修改，并不能真实的反应日志分段的最大时间信息。

**删除过程**

1. 从日志对象中所维护日志分段的跳跃表中移除待删除的日志分段，保证没有线程对这些日志分段进行读取操作。

2. 这些日志分段所有文件添加 上 .delete 后缀。

3. 交由一个以 "delete-file" 命名的延迟任务来删除这些 .delete 为后缀的文件。延迟执行时间可以通过 file.delete.delay.ms 进行设置

**如果活跃的日志分段中也存在需要删除的数据时?**

Kafka 会先切分出一个新的日志分段作为活跃日志分段，该日志分段不删除，删除原来的日志分段。

先腾出地方，再删除。

**基于日志大小**

日志删除任务会检查当前日志的大小是否超过设定值。设定项为 log.retention.bytes ，单个日志分段的大小由 log.segment.bytes 进行设定。

**删除过程**

1. 计算需要被删除的日志总大小 (当前日志文件大小（所有分段）减去retention值)。 

2. 从日志文件第一个 LogSegment 开始查找可删除的日志分段的文件集合。

3. 执行删除。

**基于偏移量**

根据日志分段的下一个日志分段的起始偏移量是否大于等于日志文件的起始偏移量，若是，则可以删除此日志分段。

注意：日志文件的起始偏移量并不一定等于第一个日志分段的基准偏移量，存在数据删除，可能与之相等的那条数据已经被删除了。

![](img\154.png)

**删除过程**

1. 从头开始遍历每个日志分段，日志分段1的下一个日志分段的起始偏移量为21，小于logStartOffset，将日志分段1加入到删除队列中

2. 日志分段 2 的下一个日志分段的起始偏移量为35，小于 logStartOffset，将 日志分段 2 加入到删除队列中

3. 日志分段 3 的下一个日志分段的起始偏移量为57，小于logStartOffset，将日志分段3加入删除集合中

4. 日志分段4的下一个日志分段的其实偏移量为71，大于logStartOffset，则不进行删除。

###### 日志压缩策略

**1.** **概念**

日志压缩是Kafka的一种机制，可以提供较为细粒度的记录保留，而不是基于粗粒度的基于时间的保留。

对于具有相同的Key，而数据不同，只保留最后一条数据，前面的数据在合适的情况下删除。

**2.** **应用场景**

日志压缩特性，就实时计算来说，可以在异常容灾方面有很好的应用途径。比如，我们在Spark、Flink中做实时计算时，需要长期在内存里面维护一些数据，这些数据可能是通过聚合了一天或者一周的日志得到的，这些数据一旦由于异常因素（内存、网络、磁盘等）崩溃了，从头开始计算需要很长的时间。一个比较有效可行的方式就是定时将内存里的数据备份到外部存储介质中，当崩溃出现时，再从外部存储介质中恢复并继续计算。

使用日志压缩来替代这些外部存储有哪些优势及好处呢？这里为大家列举并总结了几点：

- Kafka即是数据源又是存储工具，可以简化技术栈，降低维护成本

- 使用外部存储介质的话，需要将存储的Key记录下来，恢复的时候再使用这些Key将数据取回，实现起来有一定的工程难度和复杂度。使用Kafka的日志压缩特性，只需要把数据写进Kafka，等异常出现恢复任务时再读回到内存就可以了
- Kafka对于磁盘的读写做了大量的优化工作，比如磁盘顺序读写。相对于外部存储介质没有索引查询等工作量的负担，可以实现高性能。同时，Kafka的日志压缩机制可以充分利用廉价的磁盘，不用依赖昂贵的内存来处理，在性能相似的情况下，实现非常高的性价比（这个观点仅仅针对于异常处理和容灾的场景来说）

**日志压缩方式的实现细节**

主题的 cleanup.policy 需要设置为compact。

Kafka的后台线程会定时将Topic遍历两次：

1. 记录每个key的hash值最后一次出现的偏移量

2. 第二次检查每个offset对应的Key是否在后面的日志中出现过，如果出现了就删除对应的日志。

日志压缩允许删除，除最后一个key之外，删除先前出现的所有该key对应的记录。在一段时间后从日志中清理，以释放空间。

注意：日志压缩与key有关，**确保每个消息的key不为null**。

压缩是在Kafka后台通过**定时重新打开**Segment来完成的，Segment的压缩细节如下图所示：

![](img\155.png)

日志压缩可以确保：

1. 任何保持在日志头部以内的使用者都将看到所写的每条消息，这些消息将具有顺序偏移量。可以使用Topic的min.compaction.lag.ms属性来保证消息在被压缩之前必须经过的最短时间。也就是说，它为每个消息在（未压缩）头部停留的时间提供了一个下限。可以使用Topic的max.compaction.lag.ms属性来保证从收到消息到消息符合压缩条件之间的最大延时

- 消息始终保持顺序，压缩永远不会重新排序消息，只是删除一些而已

- 消息的偏移量永远不会改变，它是日志中位置的永久标识符

- 从日志开始的任何使用者将至少看到所有记录的最终状态，按记录的顺序写入。另外，如果使用者在比Topic的log.cleaner.delete.retention.ms短的时间内到达日志的头部，则会看到已删除记录的所有delete标记。保留时间默认是24小时。

默认情况下，启动日志清理器，若需要启动特定Topic的日志清理，请添加特定的属性。配置日志清理器，这里为大家总结了以下几点：

1. log.cleanup.policy 设置为 compact ，Broker的配置，影响集群中所有的Topic。 

2. log.cleaner.min.compaction.lag.ms ，用于防止对更新超过最小消息进行压缩，如果没有设置，除最后一个Segment之外，所有Segment都有资格进行压缩log.cleaner.max.compaction.lag.ms ，用于防止低生产速率的日志在无限制的时间内不压缩。

Kafka的日志压缩原理并不复杂，就是定时把所有的日志读取两遍，写一遍，而CPU的速度超过磁盘完全不是问题，只要日志的量对应的读取两遍和写入一遍的时间在可接受的范围内，那么它的性能就是可以接受的。

#### 磁盘存储

##### 零拷贝

kafka高性能，是多方面协同的结果，包括宏观架构、分布式partition存储、ISR数据同步、以及“无所不用其极”的高效利用磁盘/操作系统特性。

零拷贝并不是不需要拷贝，而是减少不必要的拷贝次数。通常是说在IO读写过程中。

nginx的高性能也有零拷贝的身影。

**传统IO**

比如：读取文件，socket发送

传统方式实现：先读取、再发送，实际经过1~4四次copy。

```
buffer = File.read 
Socket.send(buffer)
```

![](img\156.png)

实际IO读写，需要进行IO中断，需要CPU响应中断(内核态到用户态转换)，尽管引入DMA(Direct

Memory Access，直接存储器访问)来接管CPU的中断请求，但四次copy是存在“不必要的拷贝”的。

实际上并不需要第二个和第三个数据副本。数据可以直接从读缓冲区传输到套接字缓冲区。

kafka的两个过程：

1、网络数据持久化到磁盘 (Producer 到 Broker)

2、磁盘文件通过网络发送（Broker 到 Consumer）

数据落盘通常都是非实时的，Kafka的数据并不是实时的写入硬盘，它充分利用了现代操作系统分页存储来利用内存提高I/O效率

**磁盘文件通过网络发送（Broker到 Consumer）**

磁盘数据通过DMA(Direct Memory Access，直接存储器访问)拷贝到内核态 Buffer

直接通过 DMA 拷贝到 NIC Buffer(socket buffer)，无需 CPU 拷贝。

除了减少数据拷贝外，整个读文件 ==> 网络发送由一个 sendfile 调用完成，整个过程只有两次上下文切换，因此大大提高了性能。

Java NIO对sendfile的支持就是FileChannel.transferTo()/transferFrom()。

fileChannel.transferTo( position, count, socketChannel);

把磁盘文件读取OS内核缓冲区后的fileChannel，直接转给socketChannel发送；底层就是sendfile。消费者从broker读取数据，就是由此实现。

具体来看，Kafka 的数据传输通过 TransportLayer 来完成，其子类 PlaintextTransportLayer 通过Java NIO 的 FileChannel 的 transferTo 和 transferFrom 方法实现零拷贝

![](img\157.png)

注： transferTo 和 transferFrom 并不保证一定能使用零拷贝，需要操作系统支持。

Linux 2.4+ 内核通过 sendfile 系统调用，提供了零拷贝

##### 页缓存

页缓存是操作系统实现的一种主要的磁盘缓存，以此用来减少对磁盘 I/O 的操作。

具体来说，就是把磁盘中的数据缓存到内存中，把对磁盘的访问变为对内存的访问。

Kafka接收来自socket buffer的网络数据，应用进程不需要中间处理、直接进行持久化时。可以使用mmap内存文件映射。

Memory Mapped Files简称mmap，简单描述其作用就是：将磁盘文件映射到内存, 用户通过修改内存就能修改磁盘文件。

它的工作原理是直接利用操作系统的Page来实现磁盘文件到物理内存的直接映射。完成映射之后你对物理内存的操作会被同步到硬盘上（操作系统在适当的时候）

![](img\158.png)

通过mmap，进程像读写硬盘一样读写内存（当然是虚拟机内存）。使用这种方式可以获取很大的I/O提升，省去了用户空间到内核空间复制的开销。

mmap也有一个很明显的缺陷：不可靠，写到mmap中的数据并没有被真正的写到硬盘，操作系统会在程序主动调用flush的时候才把数据真正的写到硬盘。

Kafka提供了一个参数 producer.type 来控制是不是主动flush；

如果Kafka写入到mmap之后就立即flush然后再返回Producer叫同步(sync)；

写入mmap之后立即返回Producer不调用flush叫异步(async)。

Java NIO对文件映射的支持

Java NIO，提供了一个MappedByteBuffer 类可以用来实现内存映射。

MappedByteBuffer只能通过调用FileChannel的map()取得，再没有其他方式。

FileChannel.map()是抽象方法，具体实现是在 FileChannelImpl.map()可自行查看JDK源码，其map0()方法就是调用了Linux内核的mmap的API。

![](img\159.png)

使用 MappedByteBuffer类要注意的是

mmap的文件映射，在full gc时才会进行释放。当close时，需要手动清除内存映射文件，可以

反射调用sun.misc.Cleaner方法。

当一个进程准备读取磁盘上的文件内容时：

1. 操作系统会先查看待读取的数据所在的页 (page)是否在页缓存(pagecache)中，如果存在(命中)则直接返回数据，从而避免了对物理磁盘的 I/O 操作；

2. 如果没有命中，则操作系统会向磁盘发起读取请求并将读取的数据页存入页缓存，之后再将数据返回给进程。

如果一个进程需要将数据写入磁盘：

1. 操作系统也会检测数据对应的页是否在页缓存中，如果不存在，则会先在页缓存中添加相应的页，最后将数据写入对应的页。

2. 被修改过后的页也就变成了脏页，操作系统会在合适的时间把脏页中的数据写入磁盘，以保持数据的一致性。



对一个进程而言，它会在进程内部缓存处理所需的数据，然而这些数据有可能还缓存在操作系统的页缓存中，因此同一份数据有可能被缓存了两次。并且，除非使用Direct I/O的方式， 否则页缓存很难被禁止。

当使用页缓存的时候，即使Kafka服务重启， 页缓存还是会保持有效，然而进程内的缓存却需要重建。这样也极大地简化了代码逻辑，因为维护页缓存和文件之间的一致性交由操作系统来负责，这样会比进程内维护更加安全有效。

Kafka中大量使用了页缓存，这是 Kafka 实现高吞吐的重要因素之一。

**消息先被写入页缓存，由操作系统负责刷盘任务。**

##### 顺序写入

操作系统可以针对线性读写做深层次的优化，比如预读(read-ahead，提前将一个比较大的磁盘块读入内存) 和后写(write-behind，将很多小的逻辑写操作合并起来组成一个大的物理写操作)技术。

![](img\160.png)

Kafka 在设计时采用了文件追加的方式来写入消息，即只能在日志文件的尾部追加新的消 息，并且也不允许修改已写入的消息，这种方式属于典型的顺序写盘的操作，所以就算 Kafka 使用磁盘作为存储介质，也能承载非常大的吞吐量。

mmap和sendfile： 

1. Linux内核提供、实现零拷贝的API； 

2. sendfile 是将读到内核空间的数据，转到socket buffer，进行网络发送；

3. mmap将磁盘文件映射到内存，支持读和写，对内存的操作会反映在磁盘文件上。

4. RocketMQ 在消费消息时，使用了 mmap。kafka 使用了 sendFile。

Kafka速度快是因为：

1. partition顺序读写，充分利用磁盘特性，这是基础；

2. Producer生产的数据持久化到broker，采用mmap文件映射，实现顺序的快速写入；

3. Customer从broker读取数据，采用sendfile，将磁盘文件读到OS内核缓冲区后，直接转到socket buffer进行网络发送。

### 稳定性

#### 事务

**一、事务场景**

1. 如producer发的多条消息组成一个事务这些消息需要对consumer同时可见或者同时不可见 。 

2. producer可能会给多个topic，多个partition发消息，这些消息也需要能放在一个事务里面，这就形成了一个典型的分布式事务。

3. kafka的应用场景经常是应用先消费一个topic，然后做处理再发到另一个topic，这个consume-transform-produce过程需要放到一个事务里面，比如在消息处理或者发送的过程中如果失败了，消费偏移量也不能提交。

4. producer或者producer所在的应用可能会挂掉，新的producer启动以后需要知道怎么处理之前未完成的事务 。

**二、几个关键概念和推导**

1. 因为producer发送消息可能是分布式事务，所以引入了常用的2PC，所以有事务协调者(Transaction Coordinator)。Transaction Coordinator和之前为了解决脑裂和惊群问题引入的Group Coordinator在选举和failover上面类似。

2. 事务管理中事务日志是必不可少的，kafka使用一个内部topic来保存事务日志，这个设计和之前使用内部topic保存偏移量的设计保持一致。事务日志是Transaction Coordinator管理的状态的持久化，因为不需要回溯事务的历史状态，所以事务日志只用保存最近的事务状态。

3. 因为事务存在commit和abort两种操作，而客户端又有read committed和read uncommitted两种隔离级别，所以消息队列必须能标识事务状态，这个被称作Control Message。 

4. producer挂掉重启或者漂移到其它机器需要能关联的之前的未完成事务所以需要有一个唯一标识符来进行关联，这个就是TransactionalId，一个producer挂了，另一个有相同TransactionalId的producer能够接着处理这个事务未完成的状态。kafka目前没有引入全局序，所以也没有transaction id，这个TransactionalId是用户提前配置的。

5. TransactionalId能关联producer，也需要避免两个使用相同TransactionalId的producer同时存在，所以引入了producer epoch来保证对应一个TransactionalId只有一个活跃的producerepoch

**三、事务语义**

**多分区原子写入**

事务能够保证Kafka topic下每个分区的原子写入。事务中所有的消息都将被成功写入或者丢弃。

首先，我们来考虑一下原子 读取-处理-写入 周期是什么意思。简而言之，这意味着如果某个应用程序在某个topic tp0的偏移量X处读取到了消息A，并且在对消息A进行了一些处理（如B = F（A））之后将消息B写入topic tp1，则只有当消息A和B被认为被成功地消费并一起发布，或者完全不发布时，整个读取过程写入操作是原子的。

现在，只有当消息A的偏移量X被标记为已消费，消息A才从topic tp0消费，消费到的数据偏移量（record offset）将被标记为提交偏移量（Committing offset）。在Kafka中，我们通过写入一个名为offsets topic的内部Kafka topic来记录offset commit。消息仅在其offset被提交给offsets topic时才被认为成功消费。

由于offset commit只是对Kafkatopic的另一次写入，并且由于消息仅在提交偏移量时被视为成功消费，所以跨多个主题和分区的原子写入也启用原子 读取-处理-写入 循环：提交偏移量X到offset topic和消息B到tp1的写入将是单个事务的一部分，所以整个步骤都是原子的。

**2.2.** **粉碎僵尸实例**

我们通过为每个事务Producer分配一个称为transactional.id的唯一标识符来解决僵尸实例的问题。

在进程重新启动时能够识别相同的Producer实例。

API要求事务性Producer的第一个操作应该是在Kafka集群中显示注册transactional.id。 当注册的时候，Kafka broker用给定的transactional.id检查打开的事务并且完成处理。 Kafka也增加了一个与transactional.id相关的epoch。Epoch存储每个transactional.id内部元数据。

一旦epoch被触发，任何具有相同的transactional.id和旧的epoch的生产者被视为僵尸，Kafka拒绝来自这些生产者的后续事务性写入。

简而言之：Kafka可以保证Consumer最终只能消费非事务性消息或已提交事务性消息。它将保留来自未完成事务的消息，并过滤掉已中止事务的消息。

**1、事务使用场景**

在一个原子操作中，根据包含的操作类型，可以分为三种情况，**前两种情况是事务引入的场景**，最后一种没用。

1. 只有Producer生产消息；

2. 消费消息和生产消息并存，**这个是事务场景中最常用的情况**，就是我们常说的 consume- transform-produce 模式

3. 只有consumer消费消息，这种操作其实没有什么意义，跟使用手动提交效果一样，而且也不是事务属性引入的目的，所以一般不会使用这种情况

**2、事务配置**

1、创建消费者代码，需要：

将配置中的自动提交属性（auto.commit）进行关闭

而且在代码里面也不能使用手动提交commitSync( )或者commitAsync( )

设置isolation.level

2、创建生成者，代码如下,需要：

配置transactional.id属性

配置enable.idempotence属性

**五、事务工作原理**

![](img\161.png)

**1、事务协调器和事务日志**

事务协调器是每个Kafka内部运行的一个模块。事务日志是一个内部的主题。每个协调器拥有事务日志所在分区的子集，即, 这些borker中的分区都是Leader。

每个transactional.id都通过一个简单的哈希函数映射到事务日志的特定分区，事务日志文件__transaction_state-0。这意味着只有一个Broker拥有给定的transactional.id。

通过这种方式，我们利用Kafka可靠的复制协议和Leader选举流程来确保事务协调器始终可用，并且所有事务状态都能够持久化。

值得注意的是，事务日志只保存事务的最新状态而不是事务中的实际消息。消息只存储在实际的Topic的分区中。事务可以处于诸如“Ongoing”，“prepare commit”和“Completed”之类的各种状态中。

正是这种状态和关联的元数据存储在事务日志中。

**2、事务数据流**

数据流在抽象层面上有四种不同的类型。

**A. producer和事务coordinator的交互**

​	执行事务时，Producer向事务协调员发出如下请求：

1. initTransactions API向coordinator注册一个transactional.id。 此时，coordinator使用该transactional.id关闭所有待处理的事务，并且会避免遇到僵尸实例，由具有相同的transactional.id的Producer的另一个实例启动的任何事务将被关闭和隔离。每个Producer会话只发生一次。

2. 当Producer在事务中第一次将数据发送到分区时，首先向coordinator注册分区。

3. 当应用程序调用commitTransaction或abortTransaction时，会向coordinator发送一个请求以开始两阶段提交协议。

**B. Coordinator和事务日志交互**

随着事务的进行，Producer发送上面的请求来更新Coordinator上事务的状态。事务Coordinator会在内存中保存每个事务的状态，并且把这个状态写到事务日志中（这是以三种方式复制的，因此是持久保存的）。

事务Coordinator是读写事务日志的唯一组件。如果一个给定的Borker故障了，一个新的Coordinator会被选为新的事务日志的Leader，这个事务日志分割了这个失效的代理，它从传入的分区中读取消息并在内存中重建状态。

**C.Producer****将数据写入目标****Topic****所在分区**

在Coordinator的事务中注册新的分区后，Producer将数据正常地发送到真实数据所在分区。这与producer.send流程完全相同，但有一些额外的验证，以确保Producer不被隔离。

**D.Topic分区和Coordinator的交互**

1. 在Producer发起提交（或中止）之后，协调器开始两阶段提交协议。

2. 在第一阶段，Coordinator将其内部状态更新为“prepare_commit”并在事务日志中更新此状态。一旦完成了这个事务，无论发生什么事，都能保证事务完成。

3. Coordinator然后开始阶段2，在那里它将事务提交标记写入作为事务一部分的Topic分区。

4. 这些事务标记不会暴露给应用程序，但是在read_committed模式下被Consumer使用来过滤掉被中止事务的消息，并且不返回属于开放事务的消息（即那些在日志中但没有事务标记与他们相关联）。

5. 一旦标记被写入，事务协调器将事务标记为“完成”，并且Producer可以开始下一个事务

**六、事务相关配置**

**1、Broker configs**

![](img\162.png)

**2、Producer configs**

![](img\163.png)

**3、Consumer configs**

![](img\164.png)

##### 幂等性

Kafka在引入幂等性之前，Producer向Broker发送消息，然后Broker将消息追加到消息流中后给Producer返回Ack信号值。实现流程如下：

![](img\165.png)

![](img\166.png)

**幂等性**

保证在消息重发的时候，消费者不会重复处理。即使在消费者收到重复消息的时候，重复处理，也要保证最终结果的一致性。

所谓幂等性，数学概念就是： f(f(x)) = f(x) 。f函数表示对消息的处理。

比如，银行转账，如果失败，需要重试。不管重试多少次，都要保证最终结果一定是一致的。

**幂等性实现**

添加唯一ID，类似于数据库的主键，用于唯一标记一个消息。

Kafka为了实现幂等性，它在底层设计架构中引入了ProducerID和SequenceNumber。

- ProducerID：在每个新的Producer初始化时，会被分配一个唯一的ProducerID，这个ProducerID对客户端使用者是不可见的。

- SequenceNumber：对于每个ProducerID，Producer发送数据的每个Topic和Partition都对应一个从0开始单调递增的SequenceNumber值。

![](img\167.png)

当Producer发送消息(x2,y2)给Broker时，Broker接收到消息并将其追加到消息流中。此时，Broker返回Ack信号给Producer时，发生异常导致Producer接收Ack信号失败。对于Producer来说，会触发重试机制，将消息(x2,y2)再次发送，但是，由于引入了幂等性，在每条消息中附带了PID（ProducerID）和SequenceNumber。相同的PID和SequenceNumber发送给Broker，而之前Broker缓存过之前发送的相同的消息，那么在消息流中的消息就只有一条(x2,y2)，不会出现重复发送的情况。

客户端在生成Producer时，会实例化如下代码:

```
// 实例化一个Producer对象
Producer<String, String> producer = new KafkaProducer<>(props);
```

在org.apache.kafka.clients.producer.internals.Sender类中，在run()中有一个

maybeWaitForPid()方法，用来生成一个ProducerID，实现代码如下：

![](img\168.png)

##### 事务操作

在Kafka事务中，一个原子性操作，根据操作类型可以分为3种情况。情况如下：

- 只有Producer生产消息，这种场景需要事务的介入；

- 消费消息和生产消息并存，比如Consumer&Producer模式，这种场景是一般Kafka项目中比

较常见的模式，需要事务介入；

- 只有Consumer消费消息，这种操作在实际项目中意义不大，和手动Commit Offsets的结果一样，而且这种场景不是事务的引入目的。

![](img\169.png)

案例1：单个Producer，使用事务保证消息的仅一次发送：

![](img\170.png)

案例2：在 消费-转换-生产 模式，使用事务保证仅一次发送。

```java
public class MyTransactional {
    public static KafkaProducer<String, String> getProducer() {
        Map<String, Object> configs = new HashMap<>();
        configs.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "node1:9092");
        configs.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        configs.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        // 设置client.id
        configs.put(ProducerConfig.CLIENT_ID_CONFIG, "tx_producer_01");
        // 设置事务id
        configs.put(ProducerConfig.TRANSACTIONAL_ID_CONFIG, "tx_id_02");
        // 需要所有的ISR副本确认
        configs.put(ProducerConfig.ACKS_CONFIG, "all");
        // 启用幂等性
        configs.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, true);
        KafkaProducer<String, String> producer = new KafkaProducer<String, String>(configs);
        return producer;
    }

    public static KafkaConsumer<String, String> getConsumer(String consumerGroupId) {
        Map<String, Object> configs = new HashMap<>();
        configs.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "node1:9092");
        configs.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        configs.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        // 设置消费组ID
        configs.put(ConsumerConfig.GROUP_ID_CONFIG, "consumer_grp_02");
        // 不启用消费者偏移量的自动确认，也不要手动确认
        configs.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);
        configs.put(ConsumerConfig.CLIENT_ID_CONFIG, "consumer_client_02");
        configs.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
        // 只读取已提交的消息
        // configs.put(ConsumerConfig.ISOLATION_LEVEL_CONFIG, "read_committed");
        KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(configs);
        return consumer;
    }

    public static void main(String[] args) {
        String consumerGroupId = "consumer_grp_id_101";
        KafkaProducer<String, String> producer = getProducer();
        KafkaConsumer<String, String> consumer = getConsumer(consumerGroupId);
        // 事务的初始化
        producer.initTransactions();
        //订阅主题
        consumer.subscribe(Collections.singleton("tp_tx_01"));
        final ConsumerRecords<String, String> records = consumer.poll(1_000);
        // 开启事务
        producer.beginTransaction();
        try {
            Map<TopicPartition, OffsetAndMetadata> offsets = new HashMap<>();
            for (ConsumerRecord<String, String> record : records) {
                System.out.println(record);
                producer.send(new ProducerRecord<String, String>("tp_tx_out_01", record.key(), record.value()));
                offsets.put(new TopicPartition(record.topic(), record.partition()), new OffsetAndMetadata(record.offset() + 1));
                // 偏 移量表示下一条要消费的消息
            }
            // 将该消息的偏移量提交作为事务的一部分，随事务提交和回滚（不提交消费偏移 量）
            producer.sendOffsetsToTransaction(offsets, consumerGroupId);
            // int i = 1 / 0;
            // 提交事务
            producer.commitTransaction();
        } catch (Exception e) {
            e.printStackTrace();
            // 回滚事务
            producer.abortTransaction();
        } finally {
            // 关闭资源
            producer.close();
            consumer.close();
        }
    }
}
```

#### 控制器

Kafka集群包含若干个broker，broker.id指定broker的编号，编号不要重复。

Kafka集群上创建的主题，包含若干个分区。

每个分区包含若干个副本，副本因子包括了Follower副本和Leader副本。

副本又分为ISR（同步副本分区）和OSR（非同步副本分区）。

![](img\171.png)

控制器就是一个broker。

控制器除了一般broker的功能，还负责Leader分区的选举。

#####  broker选举

集群里第一个启动的broker在Zookeeper中创建临时节点 <KafkaZkChroot>/controller 。

其他broker在该控制器节点创建Zookeeper watch对象，使用Zookeeper的监听机制接收该节点的变更。

即：Kafka通过Zookeeper的分布式锁特性选举**集群控制器**。

下图中，节点 /myKafka/controller 是一个zookeeper临时节点，其中 "brokerid":0 ，表示当前控制器是broker.id为0的broker。

![](img\172.png)

每个新选出的控制器通过 Zookeeper 的条件递增操作获得一个全新的、数值更大的 controller epoch。其他 broker 在知道当前 controller epoch 后，如果收到由控制器发出的包含较旧epoch 的消息，就会忽略它们，以防止“**脑裂**”。

比如当一个Leader副本分区所在的broker宕机，需要选举新的Leader副本分区，有可能两个具有不同纪元数字的控制器都选举了新的Leader副本分区，如果选举出来的Leader副本分区不一样，听谁的？脑裂了。有了纪元数字，直接使用纪元数字最新的控制器结果。

![](img\173.png)

当控制器发现一个 broker 已经离开集群，那些失去Leader副本分区的Follower分区需要一个新Leader（这些分区的首领刚好是在这个 broker 上）。

1. 控制器需要知道哪个broker宕机了？

2. 控制器需要知道宕机的broker上负责的时候哪些分区的Leader副本分区？

下图中， <KafkaChroot>/brokers/ids/0 保存该broker的信息，此节点为临时节点，如果broker节点宕机，该节点丢失。集群控制器负责监听 ids 节点，一旦节点子节点发送变化，集群控制器得到通知。

![](img\174.png)

控制器遍历这些Follower副本分区，并确定谁应该成为新Leader分区，然后向所有包含新Leader分区和现有Follower的 broker 发送请求。该请求消息包含了谁是新Leader副本分区以及谁是Follower副本分区的信息。随后，新Leader分区开始处理来自生产者和消费者的请求，而跟随者开始从新Leader副本分区消费消息。

当控制器发现一个 broker 加入集群时，它会使用 broker ID 来检查新加入的 broker 是否包含现有分区的副本。如果有，控制器就把变更通知发送给新加入的 broker 和其他 broker，新 broker上的副本分区开始从Leader分区那里消费消息，与Leader分区保持同步。

**结论：**

1. Kafka 使用 Zookeeper 的分布式锁选举控制器，并在节点加入集群或退出集群时通知控制器。

2. 控制器负责在节点加入或离开集群时进行分区Leader选举。

3. 控制器使用epoch 来避免“脑裂”。“脑裂”是指两个节点同时认为自己是当前的控制器。

####  可靠性保证

**概念**

1. 创建Topic的时候可以指定 --replication-factor 3 ，表示分区的副本数，不要超过broker的数量。

2. Leader是负责读写的节点，而其他副本则是Follower。Producer只把消息发送到Leader，Follower定期地到Leader上Pull数据。

3. ISR是Leader负责维护的与其保持同步的Replica列表，即当前活跃的副本列表。如果一个Follow落后太多，Leader会将它从ISR中移除。落后太多意思是该Follow复制的消息落后于Leader的条数超过预定值(参数： replica.lag.max.messages 默认值：4000)或者Follow长时间没有向Leader发送fetch请求(参数： replica.lag.time.max.ms 默认值：10000)。 

4. 为了保证可靠性，可以设置 acks=all 。Follower收到消息后，会像Leader发送ACK。一旦Leader收到了ISR中所有Replica的ACK，Leader就commit，那么Leader就向Producer发送ACK。

**副本的分配：**

当某个topic的 --replication-factor 为N(N>1)时，每个Partition都有N个副本，称作replica。原则上是将replica均匀的分配到整个集群上。不仅如此，partition的分配也同样需要均匀分配，为了更好的负载均衡。

副本分配的三个目标：

1. 均衡地将副本分散于各个broker上 

2. 对于某个broker上分配的分区，它的其他副本在其他broker上 

3. 如果所有的broker都有机架信息，尽量将分区的各个副本分配到不同机架上的broker。

在不考虑机架信息的情况下：

1. 第一个副本分区通过轮询的方式挑选一个broker，进行分配。该轮询从broker列表的随机位置进行轮询。

2. 其余副本通过增加偏移进行分配。

![](img\175.png)

Leader的选举

如果Leader宕机了该怎么办？很容易想到我们在Follower中重新选举一个Leader，但是选举哪个作为leader呢？Follower可能已经落后许多了，因此我们要选择的是”最新”的Follow：新的Leader必须拥有与原来Leader commit过的所有信息。 kafka动态维护一组同步leader数据的副本（ISR），只有这个组的成员才有资格当选leader，kafka副本写入不被认为是已提交，直到所有的同步副本已经接收才认为。这组ISR保存在zookeeper，正因为如此，在ISR中的任何副本都有资格当选leader。

基于Zookeeper的选举方式

大数据很多组件都有Leader选举的概念，如HBASE等。它们大都基于ZK进行选举，所有Follow都在ZK上面注册一个Watch,一旦Leader宕机，Leader对于的Znode会自动删除，那些Follow由于在Leader节点上注册了Watcher,故可以得到通知，就去参与下一轮选举，尝试去创建该节点，zK会保证只有一个Follow创建成功，成为新的Leader。

**但是这种方式有几个缺点：**

- split-brain。这是由ZooKeeper的特性引起的，虽然ZooKeeper能保证所有Watch按顺序触发，但并不能保证同一时刻所有Replica“看”到的状态是一样的，这就可能造成不同Replica的响应不一致

- herd effect。如果宕机的那个Broker上的Partition比较多，会造成多个Watch被触发，造成集群内大量的调整

- ZooKeeper负载过重。每个Replica都要为此在ZooKeeper上注册一个Watch，当集群规模增加到几千个Partition时ZooKeeper负载会过重。

基于Controller的选举方式

Kafka 0.8后的Leader Election方案解决了上述问题，它在所有broker中选出一个controller，所有Partition的Leader选举都由controller决定。controller会将Leader的改变直接通过RPC的方式（比ZooKeeper Queue的方式更高效）通知需为为此作为响应的Broker。同时controller也负责增删Topic以及Replica的重新分配。

优点：极大缓解了Herd Effect问题、减轻了ZK的负载，Controller与Leader/Follower之间通过RPC通信，高效且实时。

缺点：引入Controller增加了复杂度，且需要考虑Controller的Failover如何处理Replica的恢复





如何处理Replica的恢复

![](img\176.png)

1：只有当ISR列表中所有列表都确认接收数据后，该消息才会被commit。因此只有m1被

commit了。即使leader上有m1,m2,m3，consumer此时只能读到m1。 

2：此时A宕机了。B变成了新的leader了，A从ISR列表中移除。B有m2，B会发给C,C收到m2

后，m2被commit。 

3：B继续commit消息4和5 

4：A回来了。注意A并不能马上在isr列表中存在，因为它落后了很多，只有当它接受了一些数据，比如m2 m4 m5,它不落后太多的时候，才会回到ISR列表中。

思考：m3怎么办呢？

两种情况：

1. A重试，重试成功了，m3就恢复了，但是乱序了。

2. A重试不成功，此时数据就可能丢失了。



如果Replica都死了怎么办？



只要至少有一个replica，就能保证数据不丢失，可是如果某个partition的所有replica都死了怎么办？有两种方案：

1. 等待在ISR中的副本恢复，并选择该副本作为Leader

2. 选择第一个活过来的副本（不一定在 ISR中)，作为Leader。

可用性和一致性的矛盾：如果一定要等待副本恢复，等待的时间可能比较长，甚至可能永远不可用。如果是第二种，不能保证所有已经commit的消息不丢失，但有可用性。

Kafka默认选用第二种方式，支持选择不能保证一致的副本。

可以通过参数 unclean.leader.election.enable 禁用它。

Broker宕机怎么办？

Controller在Zookeeper的/brokers/ids节点上注册Watch。一旦有Broker宕机，其在Zookeeper对应的Znode会自动被删除，Zookeeper会fire Controller注册的Watch，Controller即可获取最新的幸存的Broker列表。

Controller决定set_p，该集合包含了宕机的所有Broker上的所有Partition。 

对set_p中的每一个Partition： 

1. 从/brokers/topics/[topic]/partitions/[partition]/state读取该Partition当前的ISR。 

2. 决定该Partition的新Leader。如果当前ISR中有至少一个Replica还幸存，则选择其中一个作为新Leader，新的ISR则包含当前ISR中所有幸存的Replica。否则选择 该Partition中任意一个幸存的Replica作为新的Leader以及ISR（该场景下可能会有潜在的数据丢失）。如果该Partition的所有Replica都宕机了，则将新的Leader设置为-1。 

3. 将新的Leader，ISR和新的leader_epoch及controller_epoch写 入/brokers/topics/[topic]/partitions/[partition]/state。

```shell
[zk: localhost:2181(CONNECTED) 13] get /brokers/topics/bdstar/partitions/0/state{"controller_epoch":1272,"le ader":0,"version":1,"leader_epoch":4,"isr":[0,2]}
```

直接通过RPC向set_p相关的Broker发送LeaderAndISRRequest命令。Controller可以在一个RPC操作中发送多个命令从而提高效率。

Controller宕机怎么办？

每个Broker都会在/controller上注册一个Watch。

```shell
[zk: localhost:2181(CONNECTED) 19] get /controller {"version":1,"brokerid":1...............}
```

当前Controller宕机时，对应的/controller会自动消失。所有“活”着的Broker竞选成为新的Controller，会创建新的Controller Path

```shell
[zk: localhost:2181(CONNECTED) 19] get /controller {"version":1,"brokerid":2...............}
```

注意：只会有一个竞选成功（这点由Zookeeper保证）。竞选成功者即为新的Leader，竞选失败者则重新在新的Controller Path上注册Watch。因为Zookeeper的Watch是一次性的，被fire一次之后即失效，所以需要重新注册

##### 失效副本

Kafka中，一个主题可以有多个分区，增强主题的可扩展性，为了保证靠可用，可以为每个分区设置副本数。

只有Leader副本可以对外提供读写服务，Follower副本只负责poll Leader副本的数据，与Leader副本保持数据的同步。

系统维护一个ISR副本集合，即所有与Leader副本保持同步的副本列表。

当Leader宕机找不到的时候，就从ISR列表中挑选一个分区做Leader。 如果ISR列表中的副本都找不到了，就剩下OSR的副本了。

此时，有两个选择：要么选择OSR的副本做Leader，优点是可以立即恢复该分区的服务。 缺点是可能会丢失数据。

要么选择等待，等待ISR列表中的分区副本可用，就选择该可用ISR分区副本做Leader。优点是不会丢失数据 缺点是会影响当前分区的可用性。因为

#####  副本复制

#### 一致性保证



**一、概念**

1. **水位标记**
水位或水印（watermark）一词，表示位置信息，即位移（offset）。Kafka源码中使用的名字是高水位，HW（high watermark）。
2. **副本角色**
Kafka分区使用多个副本(replica)提供高可用。
3. **LEO和HW**
每个分区副本对象都有两个重要的属性：LEO和HW

- LEO：即日志末端位移(log end offset)，记录了该副本日志中下一条消息的位移值。如果
  LEO=10，那么表示该副本保存了10条消息，位移值范围是[0, 9]。另外，Leader LEO和
  Follower LEO的更新是有区别的。
- HW：即上面提到的水位值。对于同一个副本对象而言，其HW值不会大于LEO值。小于等于
  HW值的所有消息都被认为是“已备份”的（replicated）。Leader副本和Follower副本的HW更
  新不同。

![](img\177.png)

上图中，HW值是7，表示位移是 0~7 的所有消息都已经处于“已提交状态”（committed），而LEO值是14，8~13的消息就是未完全备份（fully replicated）——为什么没有14？LEO指向的是下一条消息到来时的位移。

消费者无法消费分区下Leader副本中位移大于分区HW的消息。

**二、Follower副本何时更新LEO**

Follower副本不停地向Leader副本所在的broker发送FETCH请求，一旦获取消息后写入自己的日志中进行备份。那么Follower副本的LEO是何时更新的呢？首先我必须言明，Kafka有两套Follower副本

LEO： 

1. 一套LEO保存在Follower副本所在Broker的**副本管理机**中；

2. 另一套LEO保存在Leader副本所在Broker的副本管理机中。Leader副本机器上保存了所有的follower副本的LEO。

Kafka使用前者帮助Follower副本更新其HW值；利用后者帮助Leader副本更新其HW。 

1. Follower副本的本地LEO何时更新？ Follower副本的LEO值就是日志的LEO值，每当新写入一条消息，LEO值就会被更新。当Follower发送FETCH请求后，Leader将数据返回给Follower，此时Follower开始Log写数据，从而自动更新LEO值。

2. Leader端Follower的LEO何时更新？ Leader端的Follower的LEO更新发生在Leader在处理Follower FETCH请求时。一旦Leader接收到Follower发送的FETCH请求，它先从Log中读取相应的数据，给Follower返回数据前，先更新Follower的LEO。

**三、Follower副本何时更新HW**

Follower更新HW发生在其更新LEO之后，一旦Follower向Log写完数据，尝试更新自己的HW值。

比较当前LEO值与FETCH响应中Leader的HW值，取两者的小者作为新的HW值。

即：如果Follower的LEO大于Leader的HW，Follower HW值不会大于Leader的HW值。

![](img\178.png)

**四、Leader副本何时更新LEO**

和Follower更新LEO相同，Leader写Log时自动更新自己的LEO值。

**五、Leader副本何时更新HW值**

Leader的HW值就是分区HW值，直接影响分区数据对消费者的可见性 。



Leader会**尝试**去更新分区HW的四种情况：

1. Follower副本成为Leader副本时：Kafka会尝试去更新分区HW。 

2. Broker崩溃导致副本被踢出ISR时：检查下分区HW值是否需要更新是有必要的。

3. 生产者向Leader副本写消息时：因为写入消息会更新Leader的LEO，有必要检查HW值是否需要更新

4. Leader处理Follower FETCH请求时：首先从Log读取数据，之后尝试更新分区HW值

**结论：**

当Kafka broker都正常工作时，分区HW值的更新时机有两个：

1. Leader处理PRODUCE请求时

2. Leader处理FETCH请求时。

Leader如何更新自己的HW值？Leader broker上保存了一套Follower副本的LEO以及自己的LEO。

当尝试确定分区HW时，它会选出所有**满足条件的副本**，比较它们的LEO(包括Leader的LEO)，并**选择最小的LEO值作为HW值**。

需要满足的条件，（二选一）：

1. 处于ISR中 

2. 副本LEO落后于Leader LEO的时长不大于 replica.lag.time.max.ms 参数值(默认是10s)

如果Kafka只判断第一个条件的话，确定分区HW值时就不会考虑这些未在ISR中的副本，但这些副本已经具备了“立刻进入ISR”的资格，因此就可能出现分区HW值越过ISR中副本LEO的情况——不允许。

因为分区HW定义就是ISR中所有副本LEO的最小值。

**六、HW和LEO正常更新案例**

我们假设有一个topic，单分区，副本因子是2，即一个Leader副本和一个Follower副本。我们看下当producer发送一条消息时，broker端的副本到底会发生什么事情以及分区HW是如何被更新的。

1. **初始状态**

初始时Leader和Follower的HW和LEO都是0(严格来说源代码会初始化LEO为-1，不过这不影响之后的讨论)。Leader中的Remote LEO指的就是Leader端保存的Follower LEO，也被初始化成0。此时，生产者没有发送任何消息给Leader，而Follower已经开始不断地给Leader发送FETCH请求了，但因为没有数据因此什么都不会发生。值得一提的是，Follower发送过来的FETCH请求因为无数据而暂时会被寄存到Leader端的purgatory中，待500ms ( replica.fetch.wait.max.ms 参数)超时后会强制完成。倘若在寄存期间生产者发来数据，则Kafka会自动唤醒该FETCH请求，让Leader继续处理。

![](img\179.png)

2. **Follower发送FETCH请求在Leader处理完PRODUCE请求之后**

producer给该topic分区发送了一条消息

此时的状态如下图所示：

![](img\180.png)

如上图所示，Leader接收到PRODUCE请求主要做两件事情：

1. 把消息写入Log，同时自动更新Leader自己的LEO

2. 尝试更新Leader HW值。假设此时Follower尚未发送FETCH请求，Leader端保存的Remote LEO依然是0，因此Leader会比较它自己的LEO值和Remote LEO值，发现最小值是0，与当前HW值相同，故不会更新分区HW值（仍为0）、

PRODUCE请求处理完成后各值如下，Leader端的HW值依然是0，而LEO是1，Remote LEO也是0。

![](img\181.png)

**假设此时follower发送了FETCH请求**，则状态变更如下：

![](img\182.png)

本例中当follower发送FETCH请求时，Leader端的处理依次是：

1. 读取Log数据

2. 更新remote LEO = 0（为什么是0？ 因为此时Follower还没有写入这条消息。Leader如何确认Follower还未写入呢？这是通过Follower发来的FETCH请求中的Fetch offset来确定的）

3. 尝试更新分区HW：此时Leader LEO = 1，Remote LEO = 0，故分区HW值= min(Leader LEO,Follower Remote LEO) = 0

4. 把数据和当前分区HW值（依然是0）发送给Follower副本

而Follower副本接收到FETCH Response后依次执行下列操作：

1. 写入本地Log，同时更新Follower自己管理的 LEO为1

2. 更新Follower HW：比较本地LEO和 FETCH Response 中的当前Leader HW值，取较小者，Follower HW = 0

此时，第一轮FETCH RPC结束，我们会发现虽然Leader和Follower都已经在Log中保存了这条消息，但分区HW值尚未被更新，仍为0。

![](img\183.png)

**Follower第二轮FETCH**

分区HW是在第二轮FETCH RPC中被更新的，如下图所示：

![](img\184.png)

Follower发来了第二轮FETCH请求，Leader端接收到后仍然会依次执行下列操作：

1. 读取Log数据

2. 更新Remote LEO = 1（这次为什么是1了？ 因为这轮FETCH RPC携带的fetch offset是1，那么为什么这轮携带的就是1了呢，因为上一轮结束后Follower LEO被更新为1了）

3. 尝试更新分区HW：此时leader LEO = 1，Remote LEO = 1，故分区HW值= min(Leader LEO,Follower Remote LEO) = **1**。 

4. 把数据（实际上没有数据）和当前分区HW值（已更新为1）发送给Follower副本作为Response

同样地，Follower副本接收到FETCH response后依次执行下列操作：

1. 写入本地Log，当然没东西可写，Follower LEO也不会变化，依然是1。 

2. 更新Follower HW：比较本地LEO和当前Leader LEO取小者。由于都是1，故更新follower HW = 1 。

![](img\185.png)

此时消息已经成功地被复制到Leader和Follower的Log中且分区HW是1，表明消费者能够消费offset = 0的消息。

3. **FETCH请求保存在purgatory中，PRODUCE请求到来。**

当Leader无法立即满足FECTH返回要求的时候(比如没有数据)，那么该FETCH请求被暂存到Leader端的purgatory中（炼狱），待时机成熟尝试再次处理。Kafka不会无限期缓存，默认有个超时时间（500ms），一旦超时时间已过，则这个请求会被强制完成。当寄存期间还没超时，生产者发送PRODUCE请求从而使之满足了条件以致被唤醒。此时，Leader端处理流程如下：

1. Leader写Log（自动更新Leader LEO） 

2. 尝试唤醒在purgatory中寄存的FETCH请求

3. 尝试更新分区HW

**七、HW和LEO异常案例**

Kafka使用HW值来决定副本备份的进度，而HW值的更新通常需要额外一轮FETCH RPC才能完成。

但这种设计是有问题的，可能引起的问题包括：

1. 备份数据丢失

2. 备份数据不一致



1. **数据丢失**

使用HW值来确定备份进度时其值的更新是在**下一轮**RPC中完成的。如果Follower副本在标记上方的

的第一步与第二步之间发生崩溃，那么就有可能造成数据的丢失

![](img\186.png)

上图中有两个副本：A和B。开始状态是A是Leader。

假设生产者 min.insync.replicas 为1，那么当生产者发送两条消息给A后，A写入Log，此时Kafka会通知生产者这两条消息写入成功。

![](img\187.png)

但是在broker端，Leader和Follower的Log虽都写入了2条消息且分区HW已经被更新到2，但Follower HW尚未被更新还是1，也就是上面标记的第二步尚未执行，表中最后一条未执行。

倘若此时副本B所在的broker宕机，那么重启后B会自动把LEO调整到之前的HW值1，故副本B会做日志截断(log truncation)，将offset = 1的那条消息从log中删除，并调整LEO = 1。此时follower副本底层log中就只有一条消息，即offset = 0的消息！

B重启之后需要给A发FETCH请求，但若A所在broker机器在此时宕机，那么Kafka会令B成为新的Leader，而当A重启回来后也会执行日志截断，将HW调整回1。这样，offset=1的消息就从两个副本的log中被删除，也就是说这条已经被生产者认为发送成功的数据丢失。

丢失数据的前提是 min.insync.replicas=1 时，一旦消息被写入Leader端Log即被认为是committed 。**延迟一轮** FETCH RPC **更新HW值的设计使follower HW**值是异步延迟更新，若在这个过程中Leader发生变更，那么成为新Leader的Follower的HW值就有可能是过期的，导致生产者本是成功提交的消息被删除。

2. **Leader**和**Follower**数据离散

除了可能造成的数据丢失以外，该设计还会造成Leader的Log和Follower的Log数据不一致。如Leader端记录序列：m1,m2,m3,m4,m5,…；Follower端序列可能是m1,m3,m4,m5,…。

看图：

![](img\188.png)

假设：A是Leader，A的Log写入了2条消息，但B的Log只写了1条消息。分区HW更新到2，但B的HW还是1，同时生产者 min.insync.replicas 仍然为1。

假设A和B所在Broker同时宕机，B先重启回来，因此B成为Leader，分区HW = 1。假设此时生产者发送了第3条消息(红色表示)给B，于是B的log中offset = 1的消息变成了红框表示的消息，同时分区HW更新到2（A还没有回来，就B一个副本，故可以直接更新HW而不用理会A）之后A重启回来，需要**执行日志截断**，但发现此时分区HW=2而A之前的HW值也是2，故**不做任何调整**。此后A和B将以这种状态继续正常工作。

显然，这种场景下，A和B的Log中保存在offset = 1的消息是不同的记录，从而引发不一致的情形出现。

**八、Leader Epoch使用**

0. **Kafka****解决方案**

造成上述两个问题的根本原因在于

1. HW值被用于衡量副本备份的成功与否。

2. 在出现失败重启时作为日志截断的依据。

但HW值的更新是异步延迟的，特别是需要额外的FETCH请求处理流程才能更新，故这中间发生的任何崩溃都可能导致HW值的过期。

Kafka从0.11引入了 leader epoch 来取代HW值。Leader端使用**内存**保存Leader的epoch信息，即使出现上面的两个场景也能规避这些问题。

所谓Leader epoch实际上是一对值：<epoch, offset>： 

1. epoch表示Leader的版本号，从0开始，Leader变更过1次，epoch+1

2. offset对应于该epoch版本的Leader写入第一条消息的offset。因此假设有两对值

```
<0, 0> 
<1, 120>
```

则表示第一个Leader从位移0开始写入消息；共写了120条[0, 119]；而第二个Leader版本号是1，从位移120处开始写入消息。

1. Leader broker中会保存这样的一个缓存，并定期地写入到一个 checkpoint 文件中。

2. 当Leader写Log时它会尝试更新整个缓存：如果这个Leader首次写消息，则会在缓存中增加一个条目；否则就不做更新。

3. 每次副本变为Leader时会查询这部分缓存，获取出对应Leader版本的位移，则不会发生数据

不一致和丢失的情况。

1. **规避数据丢失**

![](img\189.png)

只需要知道每个副本都引入了新的状态来保存自己当leader时开始写入的第一条消息的offset以及leader版本。这样在恢复的时候完全使用这些信息而非HW来判断是否需要截断日志。

2. **规避数据不一致**

![](img\190.png)

依靠Leader epoch的信息可以有效地规避数据不一致的问题。

#### 消息重复的场景及解决方案

消息重复主要发生在以下三个阶段：

1. 生产者阶段

2. broke阶段

3. 消费者阶段

##### 根本原因

生产发送的消息没有收到正确的broke响应，导致producer重试。

producer发出一条消息，broke落盘以后因为网络等种种原因发送端得到一个发送失败的响应或者网络中断，然后producer收到一个可恢复的Exception重试消息导致消息重复。

####  __consumer_offsets

```
。。。。。。
```

### 延时队列

TimingWheel是kafka时间轮的实现，内部包含了一个TimerTaskList数组，每个数组包含了一些链表组成的TimerTaskEntry事件，每个TimerTaskList表示时间轮的某一格，这一格的时间跨度为tickMs，同一个TimerTaskList中的事件都是相差在一个tickMs跨度内的，整个时间轮的时间跨度为interval = tickMs * wheelSize，该时间轮能处理的时间范围在cuurentTime到currentTime + interval之间的事件。

当添加一个时间他的超时时间大于整个时间轮的跨度时， expiration >= currentTime + interval，则会将该事件向上级传递，上级的tickMs是下级的interval，传递直到某一个时间轮满足expiration <currentTime + interval，

然后计算对应位于哪一格，然后将事件放进去，重新设置超时时间，然后放进jdk延迟队列

![](img\191.png)

SystemTimer会取出queue中的TimerTaskList，根据expiration将currentTime往前推进，然后把里面所有的事件重新放进时间轮中，因为ct推进了，所以有些事件会在第0格，表示到期了，直接返回。

else if (expiration < currentTime + tickMs) {

然后将任务提交到java线程池中处理.



服务端在处理客户端的请求，针对不同的请求，可能不会立即返回响应结果给客户端。在处理这类请求时，服务端会为这类请求创建延迟操作对象放入延迟缓存队列中。延迟缓存的数据结构类似MAP，

延迟操作对象从延迟缓存队列中完成并移除有两种方式：

1，延迟操作对应的外部事件发生时，外部事件会尝试完成延迟缓存中的延迟操作 。 

2，如果外部事件仍然没有完成延迟操作，超时时间达到后，会强制完成延迟的操作 。

**延迟操作接口**

DelayedOperation接口表示延迟的操作对象。此接口的实现类包括延迟加入，延迟心跳，延迟生产，延迟拉取。延迟接口相关的方法：

​	*tryComplete*：尝试完成，外部事件发生时会尝试完成延迟的操作。该方法返回值为true，表示可以完成延迟操作，会调用强制完成的方法（forceComplete）。返回值为false，表示不可以完成延迟操作。

*forceComplete*：强制完成，两个地方调用，尝试完成方法（tryComplete）返回true时；延迟操作超时时。

*run*：线程运行，延迟操作超时后，会调用线程的运行方法，只会调用一次，因为超时就会发生一次。超时后会调用强制完成方法（forceComplete），如果返回true，会调用超时的回调方法。

*onComplete*：完成的回调方法。

*onExpiration*：超时的回调方法。

外部事件触发完成和超时完成都会调用forceComplete()，并调用onComplete()。forceComplete和onComplete只会调用一次。多线程下用原子变量来控制只有一个线程会调用onComplete和forceComplete。

**延迟生产和延迟拉取完成时的回调方法，尝试完成的延迟操作**

副本管理器在创建延迟操作时，会把回调方法传给延迟操作对象。当延迟操作完成时，在

onComplete方法中会调用回调方法，返回响应结果给客户端。

创建延迟操作对象需要提供请求对应的元数据。**延迟生产元数据是分区的生产结果；延迟拉取元数据是分区的拉取信息。**

创建延迟的生产对象之前，将消息集写入分区的主副本中，每个分区的生产结果会作为延迟生产的元数据。创建延迟的拉取对象之前，从分区的主副本中读取消息集，但并不会使用分区的拉取结果作为延迟拉取的元数据，因为延迟生产返回给客户端的响应结果可以直接从分区的生产结果中获取，而延迟的拉取返回给客户端的响应结果不能直接从分区的拉取结果中获取。

元数据包含返回结果的条件是：从创建延迟操作对象到完成延迟操作对象，元数据的含义不变。对于延迟的生产，服务端写入消息集到主副本返回的结果是确定的。是因为ISR中的备份副本还没有全部发送应答给主副本，才会需要创建延迟的生产。服务端在处理备份副本的拉取请求时，不会改变分区的生产结果。最后在完成延迟生产的操作对象时，服务端就可以把 “创建延迟操作对象” 时传递给它的分区生产结果直接返回给生产者 。对应延迟的拉取，读取了主副本的本地日志，但是因为消息数量不够，才会需要创建延迟的拉取，而不用分区的拉取结果而是用分区的拉取信息作为延迟拉取的元数据，是因为在尝试完成延迟拉取操作对象时，会再次读取主副本的本地日志，这次的读取有可能会让消息数量达到足够或者超时，从而完成延迟拉取操作对象。这样创建前和完成时延迟拉取操作对象的返回结果是不同的。但是拉取信息不管读取多少次都是一样的。延迟的生产的外部事件是：ISR的所有备份副本发送了拉取请求；备份副本的延迟拉取的外部事件是：追加消息集到主副本；消费者的延迟拉取的外部事件是：增加主副本的最高水位。

**尝试完成延迟的生产**

服务端处理生产者客户端的生产请求，将消息集追加到对应主副本的本地日志后，会等待ISR中所有的备份刚本都向主副本发送应答 。生产请求包括多个分区的消息集，每个分区都有对应的ISR集合。当所有分区的ISR副本都向对应分区的主副本发送了应答，生产请求才能算完成。生产请求中虽然有多个分区，但是延迟的生产操作对象只会创建一个。

判断分区的ISR副本是否都已经向主副本发送了应答，需要检查ISR中所有备份副本的偏移量是否到了延迟生产元数据的指定偏移量（延迟生产的元数据是分区的生产结果中包含有追加消息集到本地日志返回下一个偏移量）。所以ISR所有副本的偏移量只要等于元数据的偏移量，就表示备份副本向主副本发送了应答。由于当备份副本向主副本发送拉取请求，服务端读取日志后，会更新对应备份副本的偏移量数据。所以在具体的实现上，备份副本并不需要真正发送应答给主副本，因为主副本所在消息代理节点的分区对象已经记录了所有副本的信息，所以尝试完成延迟的生产时，根据副本的偏移量就可以判断备份副本是否发送了应答。进而检查分区是否有足够的副本赶上指定偏移量，只需要判断主副本的最高水位是否等于指定偏移量（最高水位的值会选择ISR中所有备份副本中最小的偏移量来设置，最小的值都等于了指定偏移量，那么就代表所有的ISR都发送了应答）。

**总结：服务端创建的延迟生产操作对象，在尝试完成时根据主副本的最高水位是否等于延迟生产操作对象中元数据的指定偏移量来判断。**具体步骤：

1，服务端处理生产者的生产请求，写入消息集到Leader副本的本地日志。

2，服务端返回追加消息集的下一个偏移量，并且创建一个延迟生产操作对象。元数据为分区的生产结果（其中就包含下一个偏移量的值）。

3，服务端处理备份副本的拉取请求，首先读取主副本的本地日志。

4，服务端返回给备份副本读取消息集，并更新备份副本的偏移量。

5，选择ISR备份副本中最小的偏移量更新主副本的最高水位。

6，如果主副本的最高水位等于指定的下一个偏移量的值，就完成延迟的生产。

**尝试完成延迟的拉取**

服务端处理消费者或备份副本的拉取请求，如果创建了延迟的拉取操作对象，一般都是客户端的消费进度能够一直赶上主副本。比如备份副本同步主副本的数据，备份副本如果一直能赶上主副本，那么主副本有新消息写入，备份副本就会马上同步。但是针对备份副本已经消费到主副本的最新位置，而主副本并没有新消息写入时：服务端没有立即返回空的拉取结果给备份副本，这时会创建一个延迟的拉取操作对象，如果有新的消息写入，服务端会等到收集足够的消息集后，才返回拉取结果给备份副本，有新的消息写入，但是还没有收集到足够的消息集，等到延迟操作对象超时后，服务端会读取新写入主副本的消息后，返回拉取结果给备份副本（完成延迟的拉取时，服务端还会再读取一次主副本的本地日志，返回新读取出来的消息集）。

客户端的拉取请求包含多个分区，服务端判断拉取的消息大小时，会收集拉取请求涉及的所有分区。**只要消息的总大小超过拉取请求设置的最少字节数，就会调用forceComplete()方法完成延迟的拉取**。

外部事件尝试完成延迟的生产和拉取操作时的判断条件：

![](img\192.png)

拉取偏移量是指拉取到消息大小。对于备份副本的延迟拉取，主副本的结束偏移量是它的最新偏移量（LEO）。对于消费者的拉取延迟，主副本的结束偏移量是它的最高水位（HW）。备份副本要时刻与主副本同步，消费者只能消费到主副本的最高水位。

**生产请求和拉取请求的延迟缓存**

客户端的一个请求包括多个分区，服务端为每个请求都会创建一个延迟操作对象。而不是为每个分区创建一个延迟操作对象。服务端的“延迟操作缓存”管理了所有的“延迟操作对象”，缓存的键是每一个分区，缓存的值是分区对应的延迟操作列表。

一个客户端请求对应一个延迟操作，一个延迟操作对应多个分区。在延迟缓存中，一个分区对应多个延迟操作。延迟缓存中保存了分区到延迟操作的映射关系。

根据分区尝试完成延迟的操作，因为生产者和消费者是以分区为最小单位来追加消息和消费消息。虽然延迟操作的创建是针对一个请求，但是一个请求中会有多个分区，在生产者追加消息时，一个生产请求总的不同分区包含的消息是不一样的。这样追加到分区对应的主副本的本地日志中，有的分区就可以去完成延迟的拉取，但是有的分区有可能还达不到完成延迟拉取操作的条件。同样完成延迟的生产也一样。所以在延迟缓存中要以分区为键来存储各个延迟操作。

由于一个请求创建一个延迟操作，一个请求又会包含多个分区，所以不同的延迟操作可能会有相同的分区。在加入到延迟缓存时，每个分区都对应相同的延迟操作。外部事件发生时，服务端会以分区为粒度，尝试完成这个分区中的所有延迟操作 。 如果指定分区对应的某个延迟操作可以被完成，那么延迟操作会从这个分区的延迟操作列表中移除。但这个延迟操作还有其他分区，其他分区中已经被完成的延迟操作也需要从延迟缓存中删除。但是不会立即被删除，因为分区作为延迟缓存的键，在服务端的数量会很多。只要分区对应的延迟操作完成了一个，就要立即检查所有分区，对服务端的性能影响比较大。所以采用一个清理器，会负责定时地清理所有分区中已经完成的延迟操作。

副本管理器针对生产请求和拉取请求都分别有一个全局的延迟缓存。生产请求对应延迟缓存中存储了延迟的生产。拉取请求对应延迟缓存中存储了延迟的拉取。

延迟缓存提供了两个方法：

tryCompleteElseWatch()：尝试完成延迟的操作，如果不能完成，将延迟操作加入延迟缓存中。一旦将延迟操作加入延迟缓存的监控，延迟操作的每个分区都会监视该延迟操作。换句话说就是每个分区发生了外部事件后，都会去尝试完成延迟操作。

checkAndComplete()：参数是延迟缓存的键，外部事件调用该方法，根据指定的键尝试完成延迟缓存中的延迟操作。

延迟缓存在调用tryCompleteElseWatch方法将延迟操作加入延迟缓存之前，会先尝试一次完成延迟的操作，如果不能完成，会调用方法将延迟操作加入到分区对应的监视器，之后还会尝试完成一次延迟操作，如果还不能完成，会将延迟操作加入定时器。如果前面的加入过程中，可以完成延迟操作后，那么就可以不用加入到其他分区的延迟缓存了。

延迟操作不仅存在于延迟缓存中，还会被定时器监控。定时器的目的是在延迟操作超时后，服务端可以强制完成延迟操作返回结果给客户端。延迟缓存的目的是让外部事件去尝试完成延迟操作。

**监视器**

延迟缓存的每个键都有一个监视器（类似每个分区有一个监视器），以链表结构来管理延迟操作。当外部事件发生时，会根据给定的键，调用这个键的对应监视器的tryCompleteWatch()方法，尝试完成监视器中所有的延迟操作。监视器尝试完成所有延迟操作的过程中，会调用每个延迟操作的tryComplete()方法，判断能否完成延迟的操作。如果能够完成，就从链表中删除对应的延迟操作。

**清理线程**

清理线程的作用是清理所有监视器中已经完成的延迟操作。

**定时器**

服务端创建的延迟操作会作为一个定时任务，加入定时器的延迟队列中。当延迟操作超时后，定时器会将延迟操作从延迟队列中弹出，并调用延迟操作的运行方法，强制完成延迟的操作。

定时器使用延迟队列管理服务端创建的所有延迟操作，延迟队列的每个元素是定时任务列表，一个定时任务列表可以存放多个定时任务条目。服务端创建的延迟操作对象，会先包装成定时任务条目，然后加入延迟队列指定的一个定时任务列表。延迟队列是定时器中保存定时任务列表的全局数据结构，服务端创建的延迟操作不是直接加入定时任务列表，而是加入时间轮。

时间轮和延迟队列的关系：

1，定时器拥有一个全局的延迟队列和时间轮，所有时间轮公用一个计数器。

2，时间轮持有延迟队列的引用。

3，定时任务条目添加到时间轮对应的时间格（槽）（槽中是定时任务列表）中，并且把该槽表也会加入到延迟队列中。

3，一个线程会将超时的定时任务列表会从延迟队列的poll方法弹出。定时任务列表超时并不一定代表定时任务超时，将定时任务重新加入时间轮，如果加入失败，说明定时任务确实超时，提交给线程池执行。

4，延迟队列的poll方法只会弹出超时的定时任务列表，队列中的每个元素（定时任务列表）按照超时时间排序，如果第一个定时任务列表都没有过期，那么其他定时任务列表也一定不会超时。

延迟操作本身的失效时间是客户端请求设置的，延迟队列的元素（每个定时任务列表）也有失效时间，当定时任务列表中的getDelay()方法返回值小于等于0，就表示定时任务列表已经过期，需要立即执行。

如果当前的时间轮放不下加入的时间时，就会创建一个更高层的时间轮。定时器只持有第一层的时间轮的引用，并不会持有更高层的时间轮。因为第一层的时间轮会持有第二层的时间轮的引用，第二层会持有第三层的时间轮的引用。定时器将定时任务加入到当前时间轮，要判断定时任务的失效时间首是否在当前时间轮的范围内，如果不在当前时间轮的范围内，则要将定时任务上升到更高一层的时间轮中。时间轮包含了定时器全局的延迟队列。

时间轮中的变量：tickMs=1：表示一格的长度是1毫秒；wheelSize=20表示一共20格，时间轮的范围就是20毫秒，定时任务的失效时间小于等于20毫秒的都会加入到这一层的时间轮中；

interval=tickMs*wheelSize=20，如果需要创建更高一层的时间轮，那么低一层的时间轮的interval的值作为高一层数据轮的tickMs值；currentTime当前时间轮的当前时间，往前移动时间轮，主要就是更新当前时间轮的当前时间，更新后重新加入定时任务条目。

本文起源于之前去面试的一道面试题，面试题大致上是这样的：消费者去Kafka里拉去消息，但是目前Kafka中又没有新的消息可以提供，那么Kafka会如何处理？

如下图所示，两个follower副本都已经拉取到了leader副本的最新位置，此时又向leader副本发送拉取请求，而leader副本并没有新的消息写入，那么此时leader副本该如何处理呢？可以直接返回空的拉取结果给follower副本，不过在leader副本一直没有新消息写入的情况下，follower副本会一直发送拉取请求，并且总收到空的拉取结果，这样徒耗资源，显然不太合理。

![](img\193.png)

这里就涉及到了Kafka延迟操作的概念。Kafka在处理拉取请求时，会先读取一次日志文件，如果收集不到足够多（fetchMinBytes，由参数fetch.min.bytes配置，默认值为1）的消息，那么就会创建一个延时拉取操作（DelayedFetch）以等待拉取到足够数量的消息。当延时拉取操作执行时，会再读取一次日志文件，然后将拉取结果返回给follower副本。

延迟操作不只是拉取消息时的特有操作，在Kafka中有多种延时操作，比如延时数据删除、延时生产等。

对于延时生产（消息）而言，如果在使用生产者客户端发送消息的时候将acks参数设置为-1，那么就意味着需要等待ISR集合中的所有副本都确认收到消息之后才能正确地收到响应的结果，或者捕获超时异常。

![](img\194.png)

假设某个分区有3个副本：leader、follower1和follower2，它们都在分区的ISR集合中。为了简化说明，这里我们不考虑ISR集合伸缩的情况。Kafka在收到客户端的生产请求后，将消息3和消息4写入leader副本的本地日志文件，如上图所示。

由于客户端设置了acks为-1，那么需要等到follower1和follower2两个副本都收到消息3和消息4后才能告知客户端正确地接收了所发送的消息。如果在一定的时间内，follower1副本或follower2副本没能够完全拉取到消息3和消息4，那么就需要返回超时异常给客户端。生产请求的超时时间由参数request.timeout.ms配置，默认值为30000，即30s。

![](img\195.png)

那么这里等待消息3和消息4写入follower1副本和follower2副本，并返回相应的响应结果给客户端的动作是由谁来执行的呢？在将消息写入leader副本的本地日志文件之后，Kafka会创建一个延时的生产操作（DelayedProduce），用来处理消息正常写入所有副本或超时的情况，以返回相应的响应结果给客户端。

延时操作需要延时返回响应的结果，首先它必须有一个超时时间（delayMs），如果在这个超时时间内没有完成既定的任务，那么就需要强制完成以返回响应结果给客户端。其次，延时操作不同于定时操作，定时操作是指在特定时间之后执行的操作，而延时操作可以在所设定的超时时间之前完成，所以延时操作能够支持外部事件的触发。

就延时生产操作而言，它的外部事件是所要写入消息的某个分区的HW（高水位）发生增长。也就是说，随着follower副本不断地与leader副本进行消息同步，进而促使HW进一步增长，HW每增长一次都会检测是否能够完成此次延时生产操作，如果可以就执行以此返回响应结果给客户端；如果在超时时间内始终无法完成，则强制执行。

回顾一下文中开头的延时拉取操作，它也同样如此，也是由超时触发或外部事件触发而被执行的。超时触发很好理解，就是等到超时时间之后触发第二次读取日志文件的操作。外部事件触发就稍复杂了一些，因为拉取请求不单单由follower副本发起，也可以由消费者客户端发起，两种情况所对应的外部事件也是不同的。如果是follower副本的延时拉取，它的外部事件就是消息追加到了leader副本的本地日志文件中；如果是消费者客户端的延时拉取，它的外部事件可以简单地理解为HW的增长。

###  重试队列

kafka没有重试机制不支持消息重试，也没有死信队列，因此使用kafka做消息队列时，需要自己实现消息重试的功能。

**实现**

创建新的kafka主题作为重试队列：

1. 创建一个topic作为重试topic，用于接收等待重试的消息。

2. 普通topic消费者设置待重试消息的下一个重试topic。 

3. 从重试topic获取待重试消息储存到redis的zset中，并以下一次消费时间排序

4. 定时任务从redis获取到达消费事件的消息，并把消息发送到对应的topic

5. 同一个消息重试次数过多则不再重试

**代码实现**

重试消息的*javaBean*

```java
@Data
public class KafkaRetryRecord {
    public static final String KEY_RETRY_TIMES = "retryTimes";
    private String key;
    private String value;
    private Integer retryTimes;
    private String topic;
    private Long nextTime;
    
    public ProducerRecord parse() {
        Integer partition = null;
        Long timestamp = System.currentTimeMillis();
        List<Header> headers = new ArrayList<>();
        ByteBuffer retryTimesBuffer = ByteBuffer.allocate(4);
        retryTimesBuffer.putInt(retryTimes);
        retryTimesBuffer.flip();
        headers.add(new RecordHeader(KafkaRetryRecord.KEY_RETRY_TIMES, retryTimesBuffer));
        ProducerRecord sendRecord = new ProducerRecord(topic, partition, timestamp, key, value, headers);
        return sendRecord;
    }
}
```

**消费端的消息发送到重试队列**

```java
public class KafkaRetryService {
    private static final Logger log = LoggerFactory.getLogger(KafkaRetryService.class);
    /*** 消息消费失败后下一次消费的延迟时间(秒) * 第一次重试延迟10秒;第 二次延迟30秒,第三次延迟1分钟... */
    private static final int[] RETRY_INTERVAL_SECONDS = {10, 30, 1 * 60, 2 * 60, 5 * 60, 10 * 60, 30 * 60, 1 * 60 * 60, 2 * 60 * 60};
    /*** 重试topic */
    @Value("${spring.kafka.topics.retry}")
    private String retryTopic;
    @Autowired
    private KafkaTemplate<String, String> template;

    public void consumerLater(ConsumerRecord<String, String> record) {
        // 获取消息的已重试次数
        int retryTimes = getRetryTimes(record);
        Date nextConsumerTime = getNextConsumerTime(retryTimes);
        if (nextConsumerTime == null) {
            return;
        }
        KafkaRetryRecord retryRecord = new KafkaRetryRecord();
        retryRecord.setNextTime(nextConsumerTime.getTime());
        retryRecord.setTopic(record.topic());
        retryRecord.setRetryTimes(retryTimes);
        retryRecord.setKey(record.key());
        retryRecord.setValue(record.value());
        String value = JSON.toJSONString(retryRecord);
        template.send(retryTopic, null, value);
    }

    /*** 获取消息的已重试次数 */
    private int getRetryTimes(ConsumerRecord record) {
        int retryTimes = -1;
        for (Header header : record.headers()) {
            if (KafkaRetryRecord.KEY_RETRY_TIMES.equals(header.key())) {
                ByteBuffer buffer = ByteBuffer.wrap(header.value());
                retryTimes = buffer.getInt();
            }
        }
        retryTimes++;
        return retryTimes;
    }

    /*** 获取待重试消息的下一次消费时间 */
    private Date getNextConsumerTime(int retryTimes) {
        // 重试次数超过上限,不再重试
        if (RETRY_INTERVAL_SECONDS.length < retryTimes) {
            return null;
        }
        Calendar calendar = Calendar.getInstance();
        calendar.add(Calendar.SECOND, RETRY_INTERVAL_SECONDS[retryTimes]);
        return calendar.getTime();
    }
}
```

**处理待消费的消息**

```java
public class RetryListener {
    private static final Logger log = LoggerFactory.getLogger(RetryListener.class);
    private static final String RETRY_KEY_ZSET = "_retry_key";
    private static final String RETRY_VALUE_MAP = "_retry_value";
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;

    @KafkaListener(topics = "${spring.kafka.topics.retry}")
    public void consume(List<ConsumerRecord<String, String>> list) {
        for (ConsumerRecord<String, String> record : list) {
            KafkaRetryRecord retryRecord = JSON.parseObject(record.value(), KafkaRetryRecord.class);
            /*** TODO 防止待重试消息太多撑爆redis,可以将待重试消息按下一次重试时间分开存 储放到不同介质 * 例如下一次重试时间在半小时以后的消息储存到mysql,并定时从mysql读取即将重 试的消息储储存到redis */
            // 通过redis的zset进行时间排序
            String key = UUID.randomUUID().toString();
            redisTemplate.opsForHash().put(RETRY_VALUE_MAP, key, record.value());
            redisTemplate.opsForZSet().add(RETRY_KEY_ZSET, key, retryRecord.getNextTime());
            /*** 定时任务从redis读取到达重试时间的消息,发送到对应的topic */
            @Scheduled(cron = "0/2 * * * * *") public void retryFormRedis () {
                long currentTime = System.currentTimeMillis();
                Set<ZSetOperations.TypedTuple<Object>> typedTuples = redisTemplate.opsForZSet().reverseRangeByScoreWithScores(RETRY_KEY_ZSET, 0, currentTime);
                redisTemplate.opsForZSet().removeRangeByScore(RETRY_KEY_ZSET, 0, currentTime);
                for (ZSetOperations.TypedTuple<Object> tuple : typedTuples) {
                    String key = tuple.getValue().toString();
                    String value = redisTemplate.opsForHash().get(RETRY_VALUE_MAP, key).toString();
                    redisTemplate.opsForHash().delete(RETRY_VALUE_MAP, key);
                    KafkaRetryRecord retryRecord = JSON.parseObject(value, KafkaRetryRecord.class);
                    ProducerRecord record = retryRecord.parse();
                    kafkaTemplate.send(record);
                }
                // TODO 发生异常将发送失败的消息重新扔回redis
            }
        }
```

消息重试

```java
public class ConsumeListener {
    private static final Logger log = LoggerFactory.getLogger(ConsumeListener.class);
    @Autowired
    private KafkaRetryService kafkaRetryService;

    @KafkaListener(topics = "${spring.kafka.topics.test}")
    public void consume(List<ConsumerRecord<String, String>> list) {
        for (ConsumerRecord record : list) {
            try {
                // 业务处理 
            } catch (Exception e) {
                log.error(e.getMessage());
                // 消息重试 kafkaRetryService.consumerLater(record); 
            }
        }
    }
}
```

