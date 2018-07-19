## 一、Kafka概述

​	Kafka 是分布式发布-订阅消息系统。它最初由 LinkedIn 公司开发，使用 Scala语言编写,之后成为 Apache 项目的一部分。
	Kafka是一个分布式的、可分区的、可复制的消息系统。它提供了普通消息系统的功能，但具有自己独特的设计。
	它提供了类似于JMS的特性，但是在设计实现上完全不同，此外它并不是JMS规范的实现。
	kafka对消息保存时根据Topic进行归类，发送消息者称为Producer,消息接受者称为Consumer,此外kafka集群由多个kafka实例组成，每个实例(server)称为broker。
	无论是kafka集群，还是producer和consumer都依赖于zookeeper来保证系统可用性集群保存一些meta信息。

总结：

> 分布式消息队列
>
> 按topic分类开放数据
>
> Producer Consumer
>
> Broker
>
> 使用zookeeper做为集群的协调工具

kafka的特点：

> 高吞吐量
>
> ​	Kafka 每秒可以生产约 25 万消息（50 MB），每秒处理 55 万消息（110 MB）
>
> 持久化数据存储
>
> ​	可进行持久化操作。将消息持久化到磁盘，因此可用于批量消费，例如 ETL，以及实时应用程序。通过将数据持久化到硬盘以及 replication 防止数据丢失。
>
> 分布式系统易于扩展
>
> ​	所有的 producer、broker 和 consumer 都会有多个，均为分布式的。无需停机即可扩展机器。
>
> 客户端状态维护
>
> ​	消息被处理的状态是在 consumer 端维护，而不是由 server 端维护。当失败时能自动平衡。



## 二、Kafka中的基本概念

Kafka将消息以`topic`为单位进行归纳。
将向Kafka topic发布消息的程序称为`producers`.
将预订`topics`并消费消息的程序称为`consumer`.
Kafka以集群的方式运行，可以由一个或多个服务组成，每个服务叫做一个broker.
producers通过网络将消息发送到Kafka集群，集群向消费者提供消息。 
客户端和服务端通过TCP协议通信。Kafka提供了Java客户端，并且对多种语言都提供了支持。



## 三、Topics、Producers、Consumers

### 1. Topics

一个topic是对一组消息的归纳。对每个topic，Kafka 对它的日志进行了分区。每个分区都由一系列有序的、不可变的消息组成，这些消息被连续的追加到分区中。

分区中的每个消息都有一个连续的序列号叫做offset,用来在分区中唯一的标识这个消息。

在一个可配置的时间段内，Kafka集群保留所有发布的消息，不管这些消息有没有被消费。比如，如果消息的保存策略被设置为2天，那么在一个消息被发布的两天时间内，它都是可以被消费的。之后它将被丢弃以释放空间。Kafka的性能是和数据量无关的常量级的，所以保留太多的数据并不是问题。

每个分区在Kafka集群的若干服务中都有副本，这样这些持有副本的服务可以共同处理数据和请求，副本数量是可以配置的。副本使Kafka具备了容错能力。

kafka中的分区是负载均衡和失败恢复的基本单位

每个分区都由一个服务器作为“leader”，零或若干服务器作为“followers”,leader负责处理消息的读和写，followers则去复制leader.如果leader down了，followers中的一台则会自动成为leader。集群中的每个服务器都会同时扮演两个角色：作为它所持有的一部分分区的leader，同时作为其他分区的followers，这样集群就会据有较好的负载均衡。

**将日志分区可以达到以下目的：首先这使得每个日志的数量不会太大，可以在单个服务上保存。另外每个分区可以单独发布和消费，为并发操作topic提供了一种可能。
**分区是负载均衡失败恢复分布式数据存储的基本单元

​	

### 2. Producers

​	Producer将消息发布到它指定的topic中,并负责决定发布到哪个分区。通常简单的由负载均衡机制随机选择分区，但也可以通过特定的分区函数选择分区。使用的更多的是第二种。

​	

### 3. Consumers

​	**实际上每个consumer唯一需要维护的数据是消息在日志中的位置，也就是offset.这个offset由consumer来维护：一般情况下随着consumer不断的读取消息，这offset的值不断增加，但其实consumer可以以任意的顺序读取消息，比如它可以将offset设置成为一个旧的值来重读之前的消息。
	**以上特点的结合，使Kafka consumers非常的轻量级：它们可以在不对集群和其他consumer造成影响的情况下读取消息。你可以使用命令行来"tail"消息而不会对其他正在消费消息的consumer造成影响。



**消费消息通常有两种模式：队列模式（queuing）和发布-订阅模式(publish-subscribe)。

(1) 队列模式
	队列模式中，多个consumers可以同时从服务端读取消息，每个消息只被其中一个consumer读到；

(2) 发布订阅模式
	发布-订阅模式中消息被广播到所有的consumer中。		

> Consumers可以加入一个consumer group，组内的Consumer是一个竞争的关系，共同竞争一个topic内的消息，topic中的消息将被分发到组中的一个成员中，同一条消息只发往其中的一个消费者。同一组中的consumer可以在不同的程序中，也可以在不同的机器上。而如果有多个Consumer group来消费相同的Topic中的消息，则组和组之间是一个共享数据的状态，每一个组都可以获取到这个主题中的所有消息。
>
> 如果所有的consumer都在一个组中，这就成为了传统的队列模式，在各consumer中实现负载均衡。
>
> 如果所有的consumer都不在不同的组中，这就成为了发布-订阅模式，所有的消息都被分发到所有的consumer中。
>
> 更常见的是，每个topic都有若干数量的consumer组来消费，每个组都是一个逻辑上的“订阅者”，为了容错和更好的稳定性，每个组都由若干consumer组成，在组内竞争实现负载均衡。实现了组内竞争负载均衡，组间共享互不影响，这其实就是一个发布-订阅模式，只不过订阅者是个组而不是单个consumer。



相比传统的消息系统，Kafka可以很好的保证有序性。
	传统的队列在服务器上保存有序的消息，如果多个consumers同时从这个服务器消费消息，服务器就会以消息存储的顺序向consumer分发消息。虽然服务器按顺序发布消息，但是消息是被异步的分发到各consumer上，所以当消息到达时可能已经失去了原来的顺序，这意味着并发消费将导致顺序错乱。为了避免故障，这样的消息系统通常使用“专用consumer”的概念，其实就是只允许一个消费者消费消息，当然这就意味着失去了并发性。
	在这方面Kafka做的更好，通过分区的概念，Kafka可以在多个consumer组并发的情况下提供较好的有序性和负载均衡。将每个分区分只分发给一个consumer组，这样一个分区就只被这个组的一个consumer消费，就可以顺序的消费这个分区的消息。因为有多个分区，依然可以在多个consumer组之间进行负载均衡。注意consumer组的数量不能多于分区的数量，也就是有多少分区就允许多少并发消费。
	Kafka只能保证一个分区之内消息的有序性，在不同的分区之间是不可以的，这已经可以满足大部分应用的需求。如果需要topic中所有消息的有序性，那就只能让这个topic只有一个分区，当然也就只有一个consumer组消费它。



为什么大数据环境下的消息队列常选择kafka？
	分布式存储数据，提供了更好的性能 可靠性 可扩展能力
	利用磁盘存储数据，且按照主题、分区来分布式存放数据，持久化存储，提供海量数据存储能力
	采用磁盘存储数据，连续进行读写保证性能，性能和磁盘的性能相关和数据量的大小无关
			

## 四、Kafka安装配置

### 1. 伪分布式安装:

下载Kafka安装包
解压：

```shell
tar -zxvf kafka_2.9.2-0.8.1.1.tgz
```

修改`server.properties`文件

```properties
log.dirs=/tmp/kafka-logs
```

修改`zookeeper.properties`

```properties
dataDir=/tmp/zookeeper
```

启动Kafka:

```shell
#启动zookeeper：
bin/zookeeper-server-start.sh config/zookeeper.properties &

#启动kafka：
bin/kafka-server-start.sh config/server.properties
```



### 2. kafka完全分布式安装

下载Kafka安装包
解压：

```shell
tar -zxvf kafka_2.9.2-0.8.1.1.tgz
```

配置：
在config目录下，修改server.properties，在文件中修改如下参数：

```properties
broker.id=0 #当前server编号
port=9092 #使用的端口
log.dirs=/tmp/kafka-logs-1 #日志存储目录
zookeeper.connect=localhost:2181
```

启动zookeeper

```shell
#在各个机器上执行：bin/kafka-server-start.sh config/server.properties
#启动ZooKeeper
zkServer.sh start
#启动kafka
bin/kafka-server-start.sh config/server.properties

```

测试：
创建一个拥有3个副本的topic:

```shell
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 1 --topic my-replicated-topic
```

查看主题：

```bash
bin/kafka-topics.sh --list --zookeeper localhost:2181
```

查看每个节点的信息：

```shell
bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic my-replicated-topic
```

向topic发送消息：

```shell
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic my-replicated-topic

bin/kafka-console-producer.sh --broker-list hadoop01:2181,hadoop02:2181,hadoop03:2181 --topic my-replicated-topic
```

消费消息：

```shell
bin/kafka-console-consumer.sh --zookeeper localhost:2181 --from-beginning --topic my-replicated-topic

bin/kafka-console-consumer.sh --zookeeper hadoop01:2181,hadoop02:2181,hadoop03:2181 --from-beginning --topic my-replicated-topic
```

实验 - 容错性：

```shell
kill -9 pid
bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic my-replicated-topic
bin/kafka-console-consumer.sh --zookeeper localhost:2181 --from-beginning --topic my-replicated-topic

```

启动kafka:

```shell
#启动zookeeper：
zkServer.sh start

#启动kafka：
bin/kafka-server-start.sh config/server.properties
```



## 五、使用kafka

创建topic：

```shell
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
```

查看topic：

```shell
bin/kafka-topics.sh --list --zookeeper localhost:2181
```

使用命令行producer从文件中或者从标准输入中读取消息并发送到服务端：

```shell
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
```

启动命令行consumer读取消息并输出到标准输出：

```shell
bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic test --from-beginning
```



## 六、Kafka的javaApi操作

### 1.搭建开发环境

创建java工程，导入kafka相关包

```java
/**
		接收数据
	*/
	@Test
	public void ConsumerReceive() throws Exception{
		Properties properties = new Properties();  
		properties.put("zookeeper.connect", "192.168.242.101:2181,192.168.242.102:2181,192.168.242.103:2181");//声明zk  
		properties.put("group.id", "group2xx");// 必须要使用别的组名称， 如果生产者和消费者都在同一组，则不能访问同一组内的topic数据  
		properties.put("auto.offset.reset", "smallest");
//		properties.put("zookeeper.session.timeout.ms", "400");
//		properties.put("zookeeper.sync.time.ms", "200");
//		properties.put("auto.commit.interval.ms", "1000");
//		properties.put("serializer.class", "kafka.serializer.StringEncoder");
		ConsumerConnector consumer = Consumer.createJavaConsumerConnector(new ConsumerConfig(properties));  

		Map<String, Integer> topicCountMap = new HashMap<String, Integer>();  
		topicCountMap.put("my-replicated-topic", 1); // 一次从主题中获取一个数据  
		Map<String, List<KafkaStream<byte[], byte[]>>>  messageStreams = consumer.createMessageStreams(topicCountMap);  
		KafkaStream<byte[], byte[]> stream = messageStreams.get("my-replicated-topic").get(0);// 获取每次接收到的这个数据  
		ConsumerIterator<byte[], byte[]> iterator =  stream.iterator();  
		
		while(iterator.hasNext()){
			System.out.println("receive：" + new String(iterator.next().message()));
		}
	}

	/**
		发送数据
	*/
	@Test
	public void ProducerSend(){
		Properties props = new Properties();
		props.put("serializer.class", "kafka.serializer.StringEncoder");
        props.put("metadata.broker.list", "192.168.242.101:9092");
		Producer<Integer, String> producer = new Producer<Integer, String>(new ProducerConfig(props ));
		producer.send(new KeyedMessage<Integer, String>("my-replicated-topic","message~xxx123asdf"));
		producer.close();
	}
```

