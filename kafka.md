# kafka

### 1.概述

```
Kafka是一个分布式的基于发布/订阅模式的消息队列，主要应用于大数据实时处理领域。

架构组成：生产者，broker，消费者，zk

概念理解：
producer:生产者，向kafka broker发消息的客户端
consumer：消费者，从kafka broker取消息的客户端
consumergroup：多个消费者组成一个消费者组，是逻辑上的一个订阅者，每个消费者消费一个分区的数据
broker：一台kafka服务器就是一个broker
topic:一个队列，消费者和生产者面向的都是一个topic
partition：一个topic可以分成多个partition,每个partition是一个有序队列
replica：副本，为了防止某节点故障导致partition数据丢失，一个topic的每个分区都有若干副本
leader：副本的主，生产者和消费者生产和消费的对象
follower：副本，保持同步，leader发生故障时，某个follower会成为新的leader
segment:partition相当于一个巨型文件，里面有多个segment file小文件，多个segment的消息数量不一定想等，方便旧的segment快速删除
```

### 2、kafka集群搭建

```
1、上传压缩包至/opt/software：rz kafka_2.11-2.4.1.tgz

2、解压：tar -zxvf kafka_2.11-2.4.1.tgz -C /opt/module/

3、改名：mv kafka_2.11-2.4.1.tgz kafka

4、更改配置文件：vim /opt/module/kafka/config/server.properties
输入以下内容：
#broker的全局唯一编号，不能重复
broker.id=0
#删除topic功能使能
delete.topic.enable=true
#处理网络请求的线程数量
num.network.threads=3
#用来处理磁盘IO的现成数量
num.io.threads=8
#发送套接字的缓冲区大小
socket.send.buffer.bytes=102400
#接收套接字的缓冲区大小
socket.receive.buffer.bytes=102400
#请求套接字的缓冲区大小
socket.request.max.bytes=104857600
#kafka运行日志存放的路径
log.dirs=/opt/module/kafka/logs
#topic在当前broker上的分区个数
num.partitions=1
#用来恢复和清理data下数据的线程数量
num.recovery.threads.per.data.dir=1
#segment文件保留的最长时间，超时将被删除
log.retention.hours=168
#配置连接Zookeeper集群地址
zookeeper.connect=hadoop102:2181,hadoop103:2181,hadoop104:2181/kafka

5、更改环境变量：vim /etc/profile.d/my_env.sh
#KAFKA_HOME
export KAFKA_HOME=/opt/module/kafka
export PATH=$PATH:$KAFKA_HOME/bin

6、环境变量生效：source /etc/profile

7、分发kafka和环境变量配置至hadoop103和hadoop104

8、改hadoop103和hadoop104配置文件：vim /opt/module/kafka/config/server.properties
分别改id为：broker.id=1、broker.id=2

9、分发配置文件/etc/profile.d/my_env.sh并执行source /etc/profile

10、三台机器分别验证：echo $KAFKA_HOME
```

### 3、集群启动

```
1、三个机器上：kafka-server-start.sh -daemon $KAFKA_HOME/config/server.properties

2、查看集群：jps

3、关闭集群：kafka-server-stop.sh
```

### 4、启停脚本

```
#!/bin/bash
if [ $# -lt 1 ]
then
echo "参数不能为空"
exit;
fi
for i in hadoop102 hadoop103 hadoop104
do
case $1 in
"start")
echo "开始启动 $i 的kafka"
ssh $i /opt/module/kafka/bin/kafka-server-start.sh -daemon /opt/module/kafka/config/server.properties
;;
"stop")
echo "开始关闭 $i 的kafka"
ssh $i /opt/module/kafka/bin/kafka-server-stop.sh
;;
*)
echo "参数错误"
;;
esac
done

```

### 5、命令操作

```
1、显示所有topic:	kafka-topics.sh --list --bootstrap-server hadoop102:9092

2、创建topic:		 kafka-topics.sh --create --bootstrap-server hadoop102:9092 --topic GMALL_START --partitions 4 --replication-factor 1
说明：partitions用来指定分区，replication-factor用来指定副本数

3、显示topic详细信息：kafka-topics.sh --bootstrap-server hadoop102:9092 --describe --topic first

4、删除topic:kafka-topics.sh --delete --bootstrap-server hadoop102:9092 --topic first


6、生产消息：kafka-console-producer.sh --broker-list hadoop102:9092 --topic first

7、消费消息：kafka-console-consumer.sh --bootstrap-server hadoop102:9092 --from-beginning --topic first

8、修改分区数：kafka-topics.sh --bootstrap-server --alter --topic first --partitions 6

9、消费数据时指明消费者组：kafka-console-consumer.sh --bootstrap-server hadoop102:9092 --topic first --consumer-property group.id=group_mytest

10、kafka生产者压力测试：kafka-producer-perf-test.sh --topic test --record-size 100 --num-records 100000 --throughput -1 --producer-props bootstrap.servers=hadoop102:9092,hadoop103:9092,hadoop104:9092

11、kafka消费者压力测试:kafka-consumer-perf-test.sh --broker-list hadoop102:9092,hadoop103:9092,hadoop104:9092 --topic test --fetch-size 10000 --messages 10000000 --threads 1


说明：指明同一个消费者组时，可以开两个及多个进程充当这个消费者组的多个消费者实例，共同消费一个topic下的数据
```

### 6、topic 和 partition

```
1、kafka中消息均以topic分类：消费者和生产者都是面向topic的

2、topic是逻辑概念，partition是物理概念，每个partition对应一个文件，文件中包含生产的数据，每条数据都有自己的offset，消费者根据这个记录自己消费到了哪个offset

3、一个partition包含多个segment，每个段中有.index文件记录索引信息，.log记录真实数据

4、partition文件命名规则：topic名字 + 分区号 例如：first-0,first-1,first-2

```

### 7、分区的好处

```
1、集群方便扩展：
每个Partition可以通过调整segment以适应它所在的机器，而一个topic又可以有多个Partition组成，因此整个集群就可以适应任意大小的数据了；

2、提高并发性：
以partition为单位，多个节点并发读写
```

### 8、分区原则

```
首先将发送的数据封装成ProducerRecord对象
1、指明 partition 的情况下，直接将指明的值直接作为 partiton 值

2、没有指明 partition 值但有 key 的情况下，将 key 的 hash 值与 topic 的 partition 数进行取余得到 partition 值

3、既没有 partition 值又没有 key 值的情况下，第一次调用时随机生成一个整数（后面每次调用在这个整数上自增），将这个值与 topic 可用的 partition 总数取余得到 partition 值，也就是常说的 round-robin 算法。
```

### 9、数据同步策略

| **方案**                        | **优点**                                           | **缺点**                                            |
| ------------------------------- | -------------------------------------------------- | --------------------------------------------------- |
| **半数以上完成同步，就发送ack** | 延迟低                                             | 选举新的leader时，容忍n台节点的故障，需要2n+1个副本 |
| **全部完成同步，才发送ack**     | 选举新的leader时，容忍n台节点的故障，需要n+1个副本 | 延迟高（kafka选用此策略）                           |

### 10、kafka的数据同步策略

```
全部同步
原因：
（1）同样为了容忍n台节点的故障，第一种方案需要2n+1个副本，而第二种方案只需要n+1个副本，而Kafka的每个分区都有大量的数据，第一种方案会造成大量数据的冗余。
（2）虽然第二种方案的网络延迟会比较高，但网络延迟对Kafka的影响较小。万一有个节点迟迟同步不上，怎么办？使用ISR机制
```

### 11、ISR机制

```
1、Leader维护了一个动态的in-sync replica set (ISR)，意为和leader保持同步的follower集合。
2、当ISR中的follower完成数据的同步之后，leader就会给producer发送ack。如果follower长时间未向leader同步数据，则该follower将被踢出ISR，该时间阈值由replica.lag.time.max.ms参数设定。
3、Leader发生故障之后，就会从ISR中选举新的leader。
```

### 12、ack三种应答机制

#### 1、acks=0

```
producer不等待broker的ack，这一操作提供了一个最低的延迟，broker一接收到还没有写入磁盘就已经返回，当broker故障时有可能丢失数据
```

#### 2、ack=1

```
producer等待broker的ack，partition的leader落盘成功后返回ack，如果在follower同步成功之前leader故障，那么将会丢失数据

数据不会重复，但可能丢失
```

#### 3、ack=-1

```
producer等待broker的ack，partition的leader和follower全部落盘成功后才返回ack。但是如果在follower同步完成后，broker发送ack之前，leader发生故障，那么会造成数据重复

数据会重复，但不会丢失
```

### 13、幂等性解决数据重复

```
启用：设置参数enable.idompotence=true

1、Kafka的幂等性实现其实就是将原来下游需要做的去重放在了数据上游。开启幂等性的Producer在初始化的时候会被分配一个PID，发往同一Partition的消息会附带Sequence Number。而Broker端会对<PID, Partition, SeqNumber>做缓存，当具有相同主键的消息提交时，Broker只会持久化一条。

2、但是PID重启就会变化，同时不同的Partition也具有不同主键，所以幂等性无法保证跨分区跨会话的Exactly Once。

3、at least once  + 幂等性 = exactly once
```



### 14、消费方式

```
push（推）模式很难适应消费速率不同的消费者，因为消息发送速率是由broker决定的。

pull（拉）模式则可以根据consumer的消费能力以适当的速率消费消息

consumer采用pull（拉）模式从broker中读取数据。
```

### 15、consumer group消费策略

#### roundrobin

```
两个topic：
topic1:	a1,a2,a3,a4,a5,a6,a7
topic2:	b1,b2,b3,b4,b5

一个消费者组包含三个消费者c1,c2,c3：
c1:	a1,a4,a7,b3
c2:	a2,a5,b1,b4
c3:	a3,a6,b2,b5

一个消费者组包含四个消费者d1,d2,d3,d4:
d1:	a1,a5,b2
d2:	a2,a6,b3
d3:	a3,a7,b4
d4:	a4,b1,b5

优点：数据均衡，缺点增加节点时分区调整动作过大
```

#### range

```
两个topic：
topic1:	a1,a2,a3,a4,a5,a6,a7
topic2:	b1,b2,b3,b4,b5

一个消费者包含三个消费者c1,c2,c3:
c1:	a1,a2,a3,b1,b2
c2:	a4,a5,b3,b4
c3:	a6,a7,b5

优点：增加节点，数据调整动作小，缺点是数据倾斜
```

#### sticky

```
两个topic：
topic1:	a1,a2,a3,a4,a5,a6,a7
topic2:	b1,b2,b3,b4,b5

一个消费者组包含三个消费者c1,c2,c3：
c1:	a1,a4,a7,b3
c2:	a2,a5,b1,b4
c3:	a3,a6,b2,b5

增加一个节点c4:
c1:	a1,a4,a7
c2:	a2,a5,b1
c3:	a3,a6,b2
c4:	b3,b4,b5
```

### 16、offset的维护

```
作用：记录consumer在消费到什么地方了，方便在断电宕机后继续原先的位置消费

原理：kafka0.9之前维护在zookeeper中，之后维护在kafka内置的topic中，名字叫_concumer_offsets

```

### 17、kafka高效读写的秘密

```
1、顺序读写，省去了磁头寻址时间

2、使用pagecache
Kafka数据持久化是直接持久化到Pagecache中，这样会产生以下几个好处： 
1、I/O Scheduler 会将连续的小块写组装成大块的物理写从而提高性能
2、I/O Scheduler 会尝试将一些写操作重新按顺序排好，从而减少磁盘头的移动时间
3、充分利用所有空闲内存（非 JVM 内存）。如果使用应用层 Cache（即 JVM 堆内存），会增加 GC 负担
4、读操作可直接在 Page Cache 内进行。如果消费和生产速度相当，甚至不需要通过物理磁盘（直接通过 Page Cache）交换数据
5、如果进程重启，JVM 内的 Cache 会失效，但 Page Cache 仍然可用
尽管持久化到Pagecache上可能会造成宕机丢失数据的情况，但这可以被Kafka的Replication机制解决。如果为了保证这种情况下数据不丢失而强制将 Page Cache 中的数据 Flush 到磁盘，反而会降低性能。

3、零拷贝技术

如果有10个消费者，传统方式下，数据复制次数为4*10=40次，而使用“零拷贝技术”只需要1+10=11次，一次为从磁盘复制到页面缓存，10次表示10个消费者各自读取一次页面缓存。

传统方式消费者读取数据：
磁盘拷贝到内核缓冲区（1）=》内核缓冲区拷贝到用户缓冲区（2）=》用户缓冲区拷贝到socket缓冲区（3）=》socket缓冲区拷贝到网卡接口（4）

零拷贝：磁盘拷贝到内核缓冲区（1），其他消费者也可以用=》网卡接口
```

### 18、zookeeper在kafka中的作用

```
1、在kafka集群中选举出controller
2、controller管理broker上下线（通过监听集群isr）
3、controller管理topic的分区副本分配
4、controller管理leader的选举
以上都依赖于zookeeper
```

### 19、事务

#### Producer事务

```
作用：实现跨分区跨会话的事务，引入全局id:Transaction ID

原理：Transaction ID和PID绑定，当Producer重启后，可以通过唯一的Transaction ID找到PID。

```

#### Consumer事务

```
作用：实现精准一次性消费

原理：将消费过程和offset提交过程进行原子绑定
```

### 20、kafka api

#### producer api

```
导入依赖：
<dependency>
<groupId>org.apache.kafka</groupId>
<artifactId>kafka-clients</artifactId>
<version>2.4.1</version>
</dependency>



package com.zhangyubo.producer;

import org.apache.kafka.clients.producer.Callback;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;

import java.util.Properties;

/**
 * @author zhangyubo
* @create 2020-05-08 10:51
 */
public class Myproducer {

    public static void main(String[] args) {
        Properties properties = new Properties();//kafka生产者参数

        properties.setProperty("key.serializer","org.apache.kafka.common.serialization.StringSerializer");
        properties.setProperty("value.serializer","org.apache.kafka.common.serialization.StringSerializer");
        properties.setProperty("acks","all");//ack原则
        properties.setProperty("bootstrap.servers","hadoop102:9092");//集群地址
        properties.setProperty("batch.size","5");//批次大小
        properties.setProperty("linger.ms","500");//等待时间
        properties.setProperty("retries","1");//重试次数
        properties.setProperty("buffer.memory","33554432");//缓冲区大小


        KafkaProducer<String, String> producer = new KafkaProducer<String, String>(properties);//构建生产者

        for (int i = 0; i < 20; i++) {

            ProducerRecord<String, String> first = new ProducerRecord<String, String>("first", "这是第" + i + "条数据");
            Callback callback = new Callback() {
                public void onCompletion(RecordMetadata recordMetadata, Exception e) {

                    if(recordMetadata != null){
                        System.out.println(recordMetadata.topic() + "话题" + recordMetadata.partition() + "分区" + recordMetadata.offset()
                        + "发送成功");
                    }
                }
            };
            producer.send(first,callback);
        }

        producer.close();
    }
}

```

#### consumer api

```
<dependency>
<groupId>org.apache.kafka</groupId>
<artifactId>kafka-clients</artifactId>
<version>2.4.1</version>
</dependency>




package com.zhangyubo.consumer;

import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;

import java.time.Duration;
import java.util.Collections;
import java.util.Properties;

/**
 * @author zhangyubo
 * @create 2020-05-08 10:51
 */
public class Myconsumer {
    public static void main(String[] args) {

        Properties properties = new Properties();

        properties.setProperty("key.deserializer","org.apache.kafka.common.serialization.StringDeserializer");
        properties.setProperty("value.deserializer","org.apache.kafka.common.serialization.StringDeserializer");
        properties.setProperty("bootstrap.servers","hadoop102:9092");
        properties.setProperty("group.id","Idea01");//消费者组id
        properties.setProperty("auto.offset.reset","earliest");

        KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(properties);

        consumer.subscribe(Collections.singleton("first"));//订阅主题

        Duration millis = Duration.ofMillis(500);

        ConsumerRecords<String, String> poll = consumer.poll(millis);//0.5秒拉取一次

        for (ConsumerRecord<String, String> record : poll) {
            System.out.println(record);
        }

        consumer.close();

    }
}

说明：
auto.offset.reset
这个配置项，是告诉Kafka Broker在发现kafka在没有初始offset，或者当前的offset是一个不存在的值（如果一个record被删除，就肯定不存在了）时，该如何处理。它有4种处理方式：

1） earliest：自动重置到最早的offset。

2） latest：看上去重置到最晚的offset。

3） none：如果边更早的offset也没有的话，就抛出异常给consumer，告诉consumer在整个consumer group中都没有发现有这样的offset。

4） 如果不是上述3种，只抛出异常给consumer。

默认值是latest。
```

### 21、kafka监控

```

Version 1.4.5 -- Copyright 2016-2020
*******************************************************************
* Kafka Eagle Service has started success.
* Welcome, Now you can visit 'http://192.168.75.102:8048/ke'
* Account:admin ,Password:123456
*******************************************************************
* <Usage> ke.sh [start|status|stop|restart|stats] </Usage>
* <Usage> https://www.kafka-eagle.org/ </Usage>
*******************************************************************

```

### 22、故障

```
follower故障：
leader会将故障的follower提出ISR，当故障的follower重启后，先截取掉hw之后的内容，然后和leader同步，当自身的leo大于等于leader时，就可以重新加入ISR

leader故障：
会从其余的follower中选举出新的leader，其他follower会截取掉hw之后的内容，重新和新的leader同步
```



### 23、面试题

```
1、ISR和AR是什么
ISR：与leader保持同步的follower集合
AR:分区所有副本

2、HW和LEO分别代表什么
HW:一个分区所有副本最小的offset
LEO:每个副本最后一条消息的offset

leader:0,1,2,3,4,5,6,7,8,9	LEO是9，HW是7
f1:1,2,3,4,5,6,7			LEO是7，HW是7
f2:1,2,3,4,5,6,7,8			LEO是8，HW是7

说明：只有HW之前的数据对consumer可见

3、Kafka中是怎么体现消息顺序性的
分区内，每条消息有一个offset，可实现分区内有序，分区间无序

4、Kafka中的分区器、序列化器、拦截器是否了解？它们之间的处理顺序是什么
分区器：分区字段有值时，分区器不起作用，直接将消息分往指定分区，如果没有值，分区器起作用，如果指定了key,按照key进行hash操作，取模指定分区，如果没有key,先产生随机数，之后在这个数基础上自增1产生新的数，优先对可用分区取模后选择一个可用分区进入，如果没有可用分区，就对不可用分区取模后选择一个分区进入

序列化器：serializer 和deserializer

拦截器：拦截器允许修改key和value,多个拦截器组成拦截器链，依次对k-v进行修改
顺序：拦截器 > 序列化器 > 分区器

5、Kafka生产者客户端的整体结构是什么样子的？使用了几个线程来处理？分别是什么？
两个线程:main线程和sender线程
main线程：生产Producer——》interceptors拦截器封装——》Serializer——》partitioner——多个封装成recordbatch交给recordaccumulate生成队列，
sender线程：从recordbatch队列中读取数据写入topic的某个分区

6、“消费组中的消费者个数如果超过topic的分区，那么就会有消费者消费不到数据”这句话是否正确？
正确

7、消费者提交消费位移时提交的是当前消费到的最新消息的offset还是offset+1？
offset + 1

8、有哪些情形会造成重复消费？
消费端在提交offset时宕机，导致消费最新数据的offset未及时同步给_consumer_offsets,重启后又消费了一次未同步的offset数据

9、那些情景会造成消息漏消费
先提交offset，后消费

10、当你使用kafka-topics.sh创建（删除）了一个topic之后，Kafka背后会执行什么逻辑？
1）会在zookeeper中的/brokers/topics节点下创建一个新的topic节点，如：/brokers/topics/first
2）触发Controller的监听程序
3）kafka Controller 负责topic的创建工作，并更新metadata cache

11、topic的分区数可不可以增加？如果可以怎么增加？如果不可以，那又是为什么
可以，使用kafka-topics.sh --bootstrap-server --alter --topic first --partitions 6

12、topic的分区数可不可以减少？如果可以怎么减少？如果不可以，那又是为什么
不可以，被删除的分区数据难以处理

13、Kafka有内部的topic吗？如果有是什么？有什么所用
有_consumer_offsets,用来保存消费者返回的offset

14、简述Kafka的日志目录结构
每个分区对应一个文件夹，文件夹命名规则是topic名+分区编号，内部是.log文件存储真实数据，.index存储索引

15、如果我指定了一个offset，Kafka Controller怎么查找到对应的消息？
例如offset为3，先找到对应的segement，从.index文件中找到3对应的索引，然后从.log文件中找到对应的消息

16、Kafka Controller的作用
1）broker的上下线管理
2）topic删除新建，分区和副本的分配
3）leader选举

17、Kafka中有那些地方需要选举？这些地方的选举策略又有哪些
1）controller选举（先到先得）
2）partition leader选举（isr列表顺序继承）

18、失效副本是指什么？有那些应对措施
不能及时与leader同步的副本，暂时将失效副本踢出isr列表，等失效副本追上leader后再重新加入

19、Kafka的那些设计让它有如此高的性能
分区，pagecache的使用，顺序读写，零拷贝技术


20、Kafka分区分配的概念
roundrobin、range和sticky策略
```

### 24、kafka和flume对比

```
共同点：
1、都是日志系统的中间件，

区别：
1、kafka是分布式中间件，自带存储，提供push和pull存取数据，
2、flume分为agent采集，collector数据简单处理，存储storage
3、kafka作为日志缓存比较合适，但是flume的数据采集部分做的比较好，因为可以有很多数据源，减少开发量
4、比较流行的是flume+kafka的方式，如果是写入到hdfs中会使用kafka+flume的方式
5、kafka是一个可持久化的分布式的消息对列
6、flume可以使用拦截器对数据进行处理，kafka需要外部的流处理系统才能做到
7、flume和kafka都是可靠的系统，但是flume不支持副本机制，所以节点崩毁会丢数据，除非恢复这些磁盘，如果需要高可用则考虑使用kafka
8、使用flume开发量少，而使用kafka开发量比较大，因为他对社区支持的不是很好

```

