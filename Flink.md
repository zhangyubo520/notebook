# Flink

### 1、概述

```
分布式，高性能，流式处理，开源

应用场景：电商广告推送、物联网数据采集、电信业基站流量调配、银行和金融业

特点：事件驱动、流处理
```

### 2、安装

#### standalone

```
1、解压缩：tar -zxvf flink-1.10.1-bin-scala_2.12.tgz -C /opt/module/

2、改名字：mv flink-1.10.1 flink

3、修改配置文件：cd /opt/module/flink/conf  vim flink-conf.yaml
jobmanager.rpc.address: hadoop102

4、分发flink
xsync /opt/module/flink

5、修改masters 和 slaves配置
vim masters
hadoop102:8081

vim slaves
hadoop103
hadoop104

6、分发配置文件：xsync /opt/module/flink/conf

7、启动：/opt/module/flink/bin/start-cluster.sh
效果：
[atguigu@hadoop102 bin]$ jps.sh 
==================== hadoop102 ====================
3040 Jps
2930 StandaloneSessionClusterEntrypoint
==================== hadoop103 ====================
2782 TaskManagerRunner
2863 Jps
==================== hadoop104 ====================
2913 Jps
2825 TaskManagerRunner

8、web查看：http://hadoop102:8081
```

#### yarn

```
1、启动hadoop集群

2、配置环境变量：
#flink环境变量配置
export FLINK_HOME=/opt/module/flink
export PATH=$PATH:$FLINK_HOME/bin

#flink on yarn配置
HADOOP_CONF_DIR=$HADOOP_HOME
export HADOOP_CLASSPATH=`hadoop classpath`

说明：测试：bin/yarn-session.sh -nm wordCount  -n 2

3、启动yarn-session
./yarn-session.sh -n 2 -s 2 -jm 1024 -tm 1024 -nm test -d
执行任务：./flink run -c com.zhangyubo.wordcount.WordCount1 -p 2 /opt/module/flink/myjob/wordcount-1.0-SNAPSHOT.jar --host hadoop102 --port 7777

3、或者直接使用：./flink run -m yarn-cluster -c com.zhangyubo.wordcount.WordCount1 -p 2 /opt/module/flink/myjob/wordcount-1.0-SNAPSHOT.jar --host hadoop102 --port 7777

4、web查看：hadoop103:8088

https://www.cnblogs.com/hxuhongming/p/12873010.html
```

### 3、api简单使用

```
依赖导入：
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>flink</artifactId>
        <groupId>com.atguigu.flink</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>wordcount</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-scala_2.12</artifactId>
            <version>1.10.1</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.flink/flink-streaming-scala -->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-streaming-scala_2.12</artifactId>
            <version>1.10.1</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <!-- 该插件用于将Scala代码编译成class文件 -->
            <plugin>
                <groupId>net.alchim31.maven</groupId>
                <artifactId>scala-maven-plugin</artifactId>
                <version>3.4.6</version>
                <executions>
                    <execution>
                        <!-- 声明绑定到maven的compile阶段 -->
                        <goals>
                            <goal>compile</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>3.0.0</version>
                <configuration>
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                </configuration>
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

批处理

```
package com.zhangyubo.wordcount
//导入的不仅有类，还有转换操作类，所以直接用下划线导入所有
import org.apache.flink.api.scala._

object WordCount {
  def main(args: Array[String]): Unit = {

    val env: ExecutionEnvironment = ExecutionEnvironment.getExecutionEnvironment//  创建执行环境

    val input = "input/hello.txt"

    val dS: DataSet[String] = env.readTextFile(input)//从文件中读取数据

//    val result: AggregateDataSet[(String, Int)] = dS.flatMap(_.split(" ")).map((_,1)).groupBy(0).sum(1)
//执行逻辑
    val result: AggregateDataSet[(String, Int)] = dS.flatMap(_.split(" ")).map((_,1)).groupBy(0).sum(1)
//输出
    result.print()
  }

}
```

流处理

```
package com.zhangyubo.wordcount

import org.apache.flink.api.java.utils.ParameterTool
import org.apache.flink.streaming.api.scala._

object WordCount1 {
  def main(args: Array[String]): Unit = {

    val params: ParameterTool =  ParameterTool.fromArgs(args)//获取外部参数
    val host: String = params.get("host")//保存参数，ip地址
    val port: Int = params.getInt("port")//保存参数，端口号

    val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment//获取执行环境

    val DS: DataStream[String] = env.socketTextStream(host,port)//从http端接受流式数据

    val result: DataStream[(String, Int)] = DS.flatMap(_.split(" ")).map((_,1)).keyBy(0).sum(1)

    result.print()//输出

    env.execute("first stream")//一直等待执行
  }
//  在算子后面使用点setParallelism(1)的方式可以手动指定某个算子的并行度，也可以在获取执行环境时使用点setParallelism(1)定义全局并行度
}

```

### 4、客户端操作

```
1、从本机某端口发送数据：nc -lk 7777

2、查看正在运行的任务：./flink list

3、查看所有任务（包括历史任务和正在运行的任务）：./flink list -a
说明：可以查到任务id和任务的名字

4、删除指定任务 ./flink cancel 8960d3b8bcbaf39fe328cb491d573caf 
说明：后面是任务id,可以通过命令行查询到，也可以通过web端看到

5、启动standalone集群：/opt/module/flink/bin/start-cluster.sh

6、关闭standalone集群：/opt/module/flink/bin/stop-cluster.sh

7、提交任务：./flink run -c com.zhangyubo.wordcount.WordCount1 -p 2 /opt/module/flink/myjob/wordcount-1.0-SNAPSHOT.jar --host hadoop102 --port 7777 指定主类，并行度，jar包，参数

8、
```

### 5、web端操作

```
1、总览:overview

2、查看正在执行的任务：running jobs

3、查看已经完成的任务：completed jobs

4、任务管理（可以查看slave的执行及输出情况，执行输出在这里查看）：task managers

5、工作管理：job managers

6、添加新任务：submit new job
```

### 6、参数生效顺序

```
代码中的 》 全局的 》 客户端设定的 》 配置文件
```

### 7、Flink运行框架

```
jobmanager:
1、作业的管理，控制一个程序的主进程，
2、将jobGraph转换成excutionGraph，
3、向resourcemanager申请资源
4、中央协调工作，例如管理检查点

resourcemanager：
1、管理所有的插槽slot
2、资源不够时向资源提供平台申请资源
3、taskmanager执行完毕时，回收slot

taskmanager：
1、包含有一个或多个slot，并向resourcemanager注册
2、收到resourcemanager指令后，会将自己的slot交给jobmanager调用，jobmanager就可以分配个任务给这些slot

dispatcher:
1、当一个应用被提交时，分发器启动并将应用移交给jobmanager
```

### 8、Flink任务调度流程

```
抽象视角：
1、app将任务提交到dispatcher(分发器)
2、dispatcher将应用移交给jobmanager
3、jobmanager向resourcemanager申请资源slots
4、resourcemanager接受taskmanager所有的slot注册，分配slot供给jobmanager条用
5、jobmanager将任务分配给某些slot执行
6、taskmanager中的slot执行任务


yarn上具体实现：
1、app提交任务给resourcemanager
2、resourcemanager启动applicationmaster
3、applicationmaster启动jobmanager
4、jobmanager向resourcemanager申请资源
5、resourcemanager启动taskmanager
6、jobmanager和taskmanager交互完成任务
```

### 9、流式处理API

#### Environment

```
val env: ExecutionEnvironment = ExecutionEnvironment.getExecutionEnvironment

val env = StreamExecutionEnvironment.getExecutionEnvironment

说明：会根据查询运行的方式决定返回什么样的运行环境，是最常用的一种创建执行环境的方式
```

#### source

```
集合：
env.fromCollection(new List())

文件：
env.readTextFile("path")

kafka:
1、导依赖：
<!-- https://mvnrepository.com/artifact/org.apache.flink/flink-connector-kafka-0.11 -->
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-kafka-0.11_2.12</artifactId>
    <version>1.10.1</version>
</dependency>

2、
val properties = new Properties()
properties.setProperty("bootstrap.servers", "hadoop102:9092")
properties.setProperty("group.id", "consumer-group")
properties.setProperty("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer")
properties.setProperty("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer")
properties.setProperty("auto.offset.reset", "latest")

val stream3 = env.addSource(new FlinkKafkaConsumer011[String]("sensor", new SimpleStringSchema(), properties))

自定义：
env.addSource(new MySensorSource())

class MySensorSource extends SourceFunction[SensorReading]{

// flag: 表示数据源是否还在正常运行
var running: Boolean = true

override def cancel(): Unit = {
running = false
}

override def run(ctx: SourceFunction.SourceContext[SensorReading]): Unit = {
// 初始化一个随机数发生器
val rand = new Random()

var curTemp = 1.to(10).map(
i => ( "sensor_" + i, 65 + rand.nextGaussian() * 20 )
)

while(running){
// 更新温度值
curTemp = curTemp.map(
t => (t._1, t._2 + rand.nextGaussian() )
)
// 获取当前时间戳
val curTime = System.currentTimeMillis()

curTemp.foreach(
t => ctx.collect(SensorReading(t._1, curTime, t._2))
)
Thread.sleep(100)
}
}
}

```

#### 转换算子

```
1、map:转换

2、filter：过滤

3、flatmap：扁平化处理

4、keyBy:DataStream -> KeyedStream,根据key的hash值，将一个流的数据分成不同的分区，

5、滚动聚合算子：sum()\min()\max()\minBy()\maxBy()

6、reduce：reduce( (x, y) => SensorReading(x.id, y.timestamp, x.temperature.min(y.temperature)) )
说明：x是历史状态，y是最新数据

7、split：拆分流 select ：分别获取拆分的流
DataStream -> SplitStream -> DataStream
例如：val splitstream = sourcestream.split(x => if(x>60) Seq("good") else Seq("bad"))
val good = splitstream.select("good")
val bad = splitstream.select("bad")
val all = splitstream.select("good","bad")

8、connect和coMap:聚合流，对聚合到的两个流分别操作
DataStream,DataStream -> ConnectedStreams -> DataStream
例如：val connectstream = good.connect(bad)
val result = connectstream.map(
x=>(x._1,"成绩异常优秀，奖励大红花一枚")
y=>(y._1,"成绩异常糟糕，没有任何奖励")
)

9、union 合并流
例如：good.union(bad)

说明：connect可以合并两个完全不同的流，然后分别处理得到一致的流，仅仅能够实现两个流的合并
union可以合并多个相同的流，不同的流无法合并
```

### 10、UDF

```
1、需要实现接口或抽象类：如MapFunction, FilterFunction, ProcessFunction

例子：
class FilterFilter extends FilterFunction[String] {
      override def filter(value: String): Boolean = {
        value.contains("flink")
      }
}

使用dataStream.filter( new FilterFilter )

2、使用匿名函数：
dataStream.filter(_.contains("flink"))

3、富函数可以获得更多细节，如open方法：初始化，获取连接等，close方法：结束后的清理工作，getRuntimeContext，获得运行时上下文环境
例子：
class MyMap extands RichMapFuntion[String]{
	override def open(configuration : Configuration) = {
		//建立各种连接，初始化等操作
	}
	
	override def map(in : String ,out : Collector[(String, Int)]) = {
		out.collect((in,1))
	}
	
	override def close() = {
		//关闭资源
	}
}
```

### 11、sink

#### kafka_sink

```
<!-- https://mvnrepository.com/artifact/org.apache.flink/flink-connector-kafka-0.11 -->
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-kafka-0.11_2.12</artifactId>
    <version>1.10.1</version>
</dependency>

使用：ds2.addSink(new FlinkKafkaProducer011[String]("hadoop102:9092","test",new SimpleStringSchema()))
```

#### Redis_sink

```
<!-- https://mvnrepository.com/artifact/org.apache.bahir/flink-connector-redis -->
<dependency>
    <groupId>org.apache.bahir</groupId>
    <artifactId>flink-connector-redis_2.11</artifactId>
    <version>1.0</version>
</dependency>

使用：
class MyRedisSink extends RedisMapper[Student]{
  override def getCommandDescription: RedisCommandDescription = {

    new RedisCommandDescription(RedisCommand.HSET,"redis_sink")
  }

  override def getKeyFromData(data: Student): String = data.id.toString

  override def getValueFromData(data: Student): String = data.score.toInt.toString
}

val conf = new FlinkJedisPoolConfig.Builder().setHost("hadoop102").setPort(6379).build()
    
ds1.addSink(new RedisSink[Student](conf,new MyRedisSink))

```

####  Elasticsearch_sink

```
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-elasticsearch6_2.12</artifactId>
    <version>1.10.1</version>
</dependency>

使用：
   val httpHosts = new util.ArrayList[HttpHost]()
    httpHosts.add(new HttpHost("hadoop102", 9200))

    val elastic: ElasticsearchSinkFunction[String] = new ElasticsearchSinkFunction[String] {
      override def process(t: String, runtimeContext: RuntimeContext, requestIndexer: RequestIndexer): Unit = {
        val request: IndexRequest = Requests.indexRequest().index("sensor").`type`("readingData").source(t)
      }
    }

    val eSink = new ElasticsearchSink.Builder[String](httpHosts,elastic)

    result.addSink(eSink.build())
```

#### Mysql_sink

```
class MyJdbcSink() extends RichSinkFunction[SensorReading]{

  var conn: Connection = _
  var insertStmt: PreparedStatement = _
  var updateStmt: PreparedStatement = _

  override def open(parameters: Configuration): Unit = {
    super.open(parameters)

    conn = DriverManager.getConnection("jdbc:mysql://hadoop102:3306/test", "root", "123456")

    insertStmt = conn.prepareStatement("INSERT INTO temperatures (sensor, temp) VALUES (?, ?)")

    updateStmt = conn.prepareStatement("UPDATE temperatures SET temp = ? WHERE sensor = ?")
  }

  override def invoke(value: SensorReading, context: SinkFunction.Context[_]): Unit = {

    updateStmt.setDouble(1, value.temperature)
    updateStmt.setString(2, value.id)
    updateStmt.execute()

    if (updateStmt.getUpdateCount == 0) {
      insertStmt.setString(1, value.id)
      insertStmt.setDouble(2, value.temperature)
      insertStmt.execute()
    }

  }

  override def close(): Unit = {
    insertStmt.close()
    updateStmt.close()
    conn.close()
  }
  
  result.addSink(new MyJdbcSink())
```

### 12、window

```
滚动窗口：ds.timeWindow(Time.seconds(15)) 解释：每15秒计算一次窗口内的数据并输出

滑动窗口：ds.timeWindow(Time.hours(1),Time.seconds(15)) 解释：每隔15秒计算一次过去1小时内的数据

会话窗口：ds.window( EventTimeSessionWindows.withGap(Time.seconds(2))) 解释：同key的数据相隔时间超过2秒就是两个会话，

滚动计数：ds.countWindow(10) 解释：来10个相同key的数据触发一次计算

滑动计数：ds.countWindow(10,2) 解释：每来2个相同的key便计算一下近10个元素的聚合值


其他可用api:
	.trigger() —— 触发器
定义 window 什么时候关闭，触发计算并输出结果
	.evitor() —— 移除器
定义移除某些数据的逻辑
	.allowedLateness() —— 允许处理迟到的数据
	.sideOutputLateData() —— 将迟到的数据放入侧输出流
	.getSideOutput() —— 获取侧输出流

```

### 13、时间语义

时间语义

```
1、概念：
事件时间event time：事情发生的时间，通常使用flink的时间戳分配器访问数据的时间戳获得事情发生的时间

进入flink的时间ingestion time：数据进入flink的时间

处理时间processing time：数据被处理的时间，机器默认的时间语义

2、应用：
引入时间语义的方式：env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
```

水位线

```
1、概念：

解决事件时间乱序进入flink时，无法确定数据是否全部到位，需要使用waterMark解决：
一种衡量事件时间进展的机制，会触发窗口的计算

有序数据的watermark(0)
-|-15-14-13-12-11-|-10-9-8-7-6-|-5-4-3-2-1

无序数据的waterMark(2)
-15-16-14-13-|-12-10-8-11-9-6-|-7-3-2-5-4-1

watermark = maxeventTime - t(t是数据的乱序时间，有网络等多种因素决定)

2、应用：
周期产生时间戳Assigner with periodic watermarks：
	// 每隔5秒产生一个watermark
	env.getConfig.setAutoWatermarkInterval(5000)

	//定义水位线抽取方法：处理乱序
    val waterMask: BoundedOutOfOrdernessTimestampExtractor[SensorReading] 
    = new BoundedOutOfOrdernessTimestampExtractor[SensorReading](Time.milliseconds(2000)) {//延时时间2s
      override def extractTimestamp(element: SensorReading): Long = {
        element.timestamp * 1000
      }//获取时间事件戳
    }//定义水位线
    ds.assignTimestampsAndWatermarks(watermask)//使用水位线处理乱序事件
    
    //定义水位线抽取方法：处理有序(直接使用时间戳)
    ds.assignAscendingTimestamps(x => x.timestamp)

间断式地生成时间戳：Assigner with punctuated watermarks

class PunctuatedAssigner extends AssignerWithPunctuatedWatermarks[SensorReading] {
val bound: Long = 60 * 1000

override def checkAndGetNextWatermark(r: SensorReading, extractedTS: Long): Watermark = {
if (r.id == "sensor_1") {
new Watermark(extractedTS - bound)
} else {
null
}
}
override def extractTimestamp(r: SensorReading, previousTS: Long): Long = {
r.timestamp
}
}

```



### 14、process api

```
1、概念：更为底层的api,可以访问时间戳、watermark以及注册定时事件。还可以输出特定的一些事件.

2、包括：
•	ProcessFunction
•	KeyedProcessFunction
•	CoProcessFunction
•	ProcessJoinFunction
•	BroadcastProcessFunction
•	KeyedBroadcastProcessFunction
•	ProcessWindowFunction
•	ProcessAllWindowFunction

3、以KeyedProcessFunction为例子，说明方法的使用
processElement(v: IN, ctx: Context, out: Collector[OUT])：流中每一个元素都会调用这个方法，调用的结果放在Collect输出
而上下文可以获取元素时间戳，元素的key,TimerService时间服务，还可以将结果输出到别的流

onTimer(timestamp: Long, ctx: OnTimerContext, out: Collector[OUT])

Context和OnTimerContext所持有的TimerService对象拥有以下方法:
•	currentProcessingTime(): Long 返回当前处理时间
•	currentWatermark(): Long 返回当前watermark的时间戳
•	registerProcessingTimeTimer(timestamp: Long): Unit 会注册当前key的processing time的定时器。当processing time到达定时时间时，触发timer。
•	registerEventTimeTimer(timestamp: Long): Unit 会注册当前key的event time 定时器。当水位线大于等于定时器注册的时间时，触发定时器执行回调函数。
•	deleteProcessingTimeTimer(timestamp: Long): Unit 删除之前注册处理时间定时器。如果没有这个时间戳的定时器，则不执行。
•	deleteEventTimeTimer(timestamp: Long): Unit 删除之前注册的事件时间定时器，如果没有此时间戳的定时器，则不执行。

4、侧输出流：
class FreezingMonitor extends ProcessFunction[SensorReading, SensorReading] {
  // 定义一个侧输出标签
  lazy val freezingAlarmOutput: OutputTag[String] =
    new OutputTag[String]("freezing-alarms")

  override def processElement(r: SensorReading,
                              ctx: ProcessFunction[SensorReading, SensorReading]#Context,
                              out: Collector[SensorReading]): Unit = {
    // 温度在32F以下时，输出到侧输出流
    if (r.temperature < 32.0) {
      ctx.output(freezingAlarmOutput, s"Freezing Alarm for ${r.id}")
    }
    // 所有数据直接常规输出到主流
    out.collect(r)
  }
}
```

### 15、状态编程

```
1、概念：
无状态流：向map,filter等不需要保存中间结果的
有状态流：向reduce等需要保存中间结果的

算子状态：有三种基本数据结构：列表状态，联合列表状态，广播状态
键控状态：有多个数据类型：ValueState保存单个值，（get,set方法)；ListState保存一个列表，（get,add,addall,update方法)
MapState保存一个键值；ReducingState;AggregatingState

2、状态一致性
at-most-once
at-least-once
exactly-once
Flink的一个重大价值在于，它既保证了exactly-once，也具有低延迟和高吞吐的处理能力

3、检查点：

4、状态后端(默认为内存级别)：
内存级别：MemoryStateBackend
远程磁盘：FsStateBackend
序列化：RocksDBStateBackend

应用：
导入依赖：
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-statebackend-rocksdb_2.12</artifactId>
    <version>1.10.1</version>
</dependency>

val env = StreamExecutionEnvironment.getExecutionEnvironment//环境
val checkpointPath: String = "inputpath"
val backend = new RocksDBStateBackend(checkpointPath)

env.setStateBackend(backend)//设置状态后端为RocksDB
env.setStateBackend(new FsStateBackend("file:///tmp/checkpoints"))//设置状态后端为Fs
env.enableCheckpointing(1000)//允许设置检查点
// 配置重启策略
env.setRestartStrategy(RestartStrategies.fixedDelayRestart(60, Time.of(10, TimeUnit.SECONDS)))


Flink的状态介绍
**

Flink的状态指的是
1.维护的状态变量,键控状态
值状态（Value state）
为每个键存储一个任意类型的单个值。复杂数据结构也可以存储为值状态。
列表状态（List state）
为每个键存储一个值的列表。列表里的每个数据可以是任意类型。
映射状态（Map state）
为每个键存储一个键值映射（map）。map的key和value可以是任意类型。

2.算子的状态保存，即checkPoint
Apache Flink将应用程序状态，存储在内存或者嵌入式数据库中。由于Flink是一个分布式系统，因此需要保护本地状态以防止在应用程序或计算机故障时数据丢失。 Flink通过定期将应用程序状态的一致性检查点（check point）写入远程且持久的存储，来保证这一点
列表状态（List state）
将状态表示为一组数据的列表。
联合列表状态（Union list state）
也将状态表示为数据的列表。它与常规列表状态的区别在于，在发生故障时，或者从保存点（savepoint）启动应用程序时如何恢复。我们将在后面继续讨论。
广播状态（Broadcast state）
如果一个算子有多项任务，而它的每项任务状态又都相同，那么这种特殊情况最适合应用广播状态。在保存检查点和重新调整算子并行度时，会用到这个特性。

状态后端（State Backends）
每传入一条数据，有状态的算子任务都会读取和更新状态。由于有效的状态访问对于处理数据的低延迟至关重要，因此每个并行任务都会在本地维护其状态，以确保快速的状态访问。
状态后端主要负责两件事：本地的状态管理，以及将检查点（checkpoint）状态写入远程存储。
Flink提供了默认的状态后端，会将键控状态作为内存中的对象进行管理，将它们存储在JVM堆上。另一种状态后端则会把状态对象进行序列化，并将它们放入RocksDB中，然后写入本地硬盘。第一种方式可以提供非常快速的状态访问，但它受内存大小的限制；而访问RocksDB状态后端存储的状态速度会较慢，但其状态可以增长到非常大。
状态检查点的写入也非常重要，这是因为Flink是一个分布式系统，而状态只能在本地维护。 TaskManager进程（所有任务在其上运行）可能在任何时间点挂掉。因此，它的本地存储只能被认为是不稳定的。状态后端负责将任务的状态检查点写入远程的持久存储。写入检查点的远程存储可以是分布式文件系统，也可以是数据库。不同的状态后端在状态检查点的写入机制方面有所不同。例如，RocksDB状态后端支持增量的检查点，这对于非常大的状态来说，可以显著减少状态检查点写入的开销。

**

有状态的计算
**

有状态计算是指在程序计算过程中，在Flink程序内部存储计算产生的中间结果，并提供给后续Function或算子计算结果使用。状态数据可以维系在本地存储中，这里的存储可以是Flink的堆内存或者堆外内存，也可以借助第三方的存储介质，例如Flink中已经实现的RocksDB，当然用户也可以自己实现相应的缓存系统去存储状态信息，以完成更加复杂的计算逻辑。
和状态计算不同的是，无状态计算不会存储计算过程中产生的结果，也不会将结果用于下一步计算过程中，程序只会在当前的计算流程中实行计算，计算完成就输出结果，然后下一条数据接入，然后再处理
```

不同 Source 和 Sink 的一致性保证可以用下表说明：

| sink\source | 不可重置     | 可重置        |
| ----------- | ------------ | ------------- |
| 任意        | at-most-once | at-least-once |
| 幂等        | at-most-once | exactly-once  |
| 预写日志    | at-most-once | at-least-once |
| 两阶段提交  | at-most-once | exactly-once  |



### 16、table api

操作流程

```
val tableEnv = ...     // 创建表的执行环境

// 创建一张表，用于读取数据
tableEnv.connect(...).createTemporaryTable("inputTable")
// 注册一张表，用于把计算结果输出
tableEnv.connect(...).createTemporaryTable("outputTable")

// 通过 Table API 查询算子，得到一张结果表
val result = tableEnv.from("inputTable").select(...)
// 通过 SQL查询语句，得到一张结果表
val sqlResult  = tableEnv.sqlQuery("SELECT ... FROM inputTable ...")

// 将结果表写入输出表中
result.insertInto("outputTable")

```

创建表环境

```
1、流处理环境：val tableEnv = StreamTableEnvironment.create(env)

2、老版本流处理：
val settings = EnvironmentSettings.newInstance()
  .useOldPlanner()      // 使用老版本planner
  .inStreamingMode()    // 流处理模式
  .build()
val tableEnv = StreamTableEnvironment.create(env, settings)

3、老板本批处理：
val batchEnv = ExecutionEnvironment.getExecutionEnvironment
val batchTableEnv = BatchTableEnvironment.create(batchEnv)

4、blink流处理：
val bsSettings = EnvironmentSettings.newInstance()
.useBlinkPlanner()
.inStreamingMode().build()
val bsTableEnv = StreamTableEnvironment.create(env, bsSettings)

5、blink批处理：
val bbSettings = EnvironmentSettings.newInstance()
.useBlinkPlanner()
.inBatchMode().build()
val bbTableEnv = TableEnvironment.create(bbSettings)
```

表的概念

```
1、概念
表（Table）是由一个“标识符”（identifier）来指定的，由3部分组成：Catalog名、数据库（database）名和对象名

常规表可以用来描述一个外部数据，如文件，数据库表，消息队列的数据等，也可以是从DataStream转换而来

视图可以从现有的表中创建，通常是一个结果集

```

表的创建

```
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-csv</artifactId>
    <version>1.10.1</version>
</dependency>
```

```
1、创建表从文件系统：
tableEnv
.connect( new FileSystem().path("sensor.txt"))  // 定义表数据来源，外部连接
  .withFormat(new Csv())    // 定义从外部系统读取数据之后的格式化方法
  .withSchema( new Schema()
    .field("id", DataTypes.STRING())
    .field("timestamp", DataTypes.BIGINT())
    .field("temperature", DataTypes.DOUBLE())
  )    // 定义表结构
  .createTemporaryTable("inputTable")    // 创建临时表

2、创建表从kafka:
tableEnv.connect(
  new Kafka()
    .version("0.11") // 定义kafka的版本
    .topic("sensor") // 定义主题
    .property("zookeeper.connect", "hadoop102:2181") 
    .property("bootstrap.servers", "hadoop102:9092")
)
  .withFormat(new Csv()) //定义格式化方法
  .withSchema(new Schema()
  .field("id", DataTypes.STRING())
  .field("timestamp", DataTypes.BIGINT())
  .field("temperature", DataTypes.DOUBLE())
)
  .createTemporaryTable("kafkaInputTable") //创建临时表
  
3、创建表从DataStream：
val sensorTable: Table = tableEnv.fromDataStream(dataStream)//将流直接转换成表

val sensorTable = tableEnv.fromDataStream(dataStream,'id, 'timestamp, 'temperature)//将流中字段单独指定

val sensorTable = tableEnv.fromDataStream(dataStream, 'timestamp as 'ts, 'id as 'myId, 'temperature)//名称

val sensorTable = tableEnv.fromDataStream(dataStream, 'myId, 'ts)//基于位置

4、创建临时视图：
tableEnv.createTemporaryView("sensorView", dataStream)

tableEnv.createTemporaryView("sensorView", 
        dataStream, 'id, 'temperature, 'timestamp as 'ts)//基于DataStream创建临时视图
        
tableEnv.createTemporaryView("sensorView", sensorTable)//基于表创建临时视图
```

表的使用

```
链式调用：

val sensorTable: Table = tableEnv.from("inputTable")
val resultTable: Table = sensorTable
    .select("id, temperature")
    .filter("id = 'sensor_1'")

sqlQuery:

tblEnv.createTemporaryView("table01",table)

val table1: Table = tblEnv.sqlQuery("select id, avg(score) as avg_score from table01 group by id")

```

表的输出

```
tableEnv.connect(...).createTemporaryTable("outputTable")
val resultSqlTable: Table = ...
resultTable.insertInto("outputTable")

例子：1、如输出到文件：
    /*
    输入
     */
    tblEnv.connect(new FileSystem().path("input/student.txt"))
      .withFormat(new Csv())
      .withSchema(new Schema()
        .field("id", DataTypes.STRING())
        .field("score", DataTypes.INT())
      )
      .createTemporaryTable("student")

    val table: Table = tblEnv.from("student").select("*")

    /*
    输出
     */
    tblEnv.connect(new FileSystem().path("input/output.txt")) // 定义到文件系统的连接
      .withFormat(new Csv())
      .withSchema(new Schema()
        .field("id", DataTypes.STRING())
        .field("score", DataTypes.INT()))
        .createTemporaryTable("output")

    table.insertInto("output")

2、输出到kafka:
//    输出到kafka
        tblEnv.connect(
          new Kafka()
            .version("0.11")
            .topic("sink_test")
            .property("zookeeper.connect","hadoop102:2181")
            .property("bootstrap.servers","hadoop102:9092")
        )
            .withFormat(new Csv())
            .withSchema(new Schema()
            .field("id",DataTypes.STRING())
            .field("score",DataTypes.INT())
          )
          .createTemporaryTable("student")

    table.insertInto("student")//输出到表

3、输出到es
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-json</artifactId>
    <version>1.10.1</version>
</dependency>

    //    输出到es
    tblEnv.connect(
      new Elasticsearch()
        .version("6")
        .host("hadoop", 9200, "http")
        .index("sensor")
        .documentType("temp")
    )
      .inUpsertMode()           // 指定是 Upsert 模式
      .withFormat(new Json())
      .withSchema( new Schema()
        .field("id", DataTypes.STRING())
        .field("score", DataTypes.BIGINT())
      )
      .createTemporaryTable("esOutputTable")

4、输出到SQL
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-jdbc_2.12</artifactId>
    <version>1.10.1</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.44</version>
</dependency>	

// 输出到 Mysql
val sinkDDL: String =
  """
    |create table jdbcOutputTable (
    |  id varchar(20) not null,
    |  cnt bigint not null
    |) with (
    |  'connector.type' = 'jdbc',
    |  'connector.url' = 'jdbc:mysql://hadoop102:3306/test',
    |  'connector.table' = 'student_count',
    |  'connector.driver' = 'com.mysql.jdbc.Driver',
    |  'connector.username' = 'root',
    |  'connector.password' = '123456'
    |)
  """.stripMargin

tableEnv.sqlUpdate(sinkDDL)//相当于注册中间桥梁
table.insertInto("jdbcOutputTable")//将动态表table插入到表student_count中，用的中间桥梁就是jdbcOutputTable


5、输出到DataStream
val resultStream: DataStream[Row] = tableEnv.toAppendStream[Row](resultTable)//追加模式

val aggResultStream: DataStream[(Boolean, (String, Long))] = tableEnv.toRetractStream[(String, Long)](aggResultTable)

```

### 17、查看执行计划

```
val explaination: String = tableEnv.explain(resultTable)
println(explaination)
```

### 18、动态表的时间特性

指定处理时间

```
1、在DataStream转换成表的时候指定：
// 将 DataStream转换为 Table，并指定时间字段
val sensorTable = tableEnv.fromDataStream(dataStream, 'id, 'temperature, 'timestamp, 'pt.proctime)

2、在定义table schema时指定：
tableEnv.connect(
  new FileSystem().path("..\\sensor.txt"))
  .withFormat(new Csv())
  .withSchema(new Schema()
    .field("id", DataTypes.STRING())
    .field("timestamp", DataTypes.BIGINT())
    .field("temperature", DataTypes.DOUBLE())
    .field("pt", DataTypes.TIMESTAMP(3))
      .proctime()    // 指定 pt字段为处理时间
  ) // 定义表结构
  .createTemporaryTable("inputTable") // 创建临时表

3、在创建表的DDL中指定：
val sinkDDL: String =
  """
    |create table dataTable (
    |  id varchar(20) not null,
    |  ts bigint,
    |  temperature double,
    |  pt AS PROCTIME()
    |) with (
    |  'connector.type' = 'filesystem',
    |  'connector.path' = 'file:///D:\\..\\sensor.txt',
    |  'format.type' = 'csv'
    |)
  """.stripMargin

tableEnv.sqlUpdate(sinkDDL) // 执行 DDL

```

指定事件事件

```
1、DataStream转换成表时指定：
// 将 DataStream转换为 Table，并指定时间字段
val sensorTable = tableEnv.fromDataStream(dataStream, 'id, 'timestamp.rowtime, 'temperature)
// 或者，直接追加字段
val sensorTable2 = tableEnv.fromDataStream(dataStream, 'id, 'temperature, 'timestamp, 'rt.rowtime)

2、定义表结构时指定：
tableEnv.connect(
  new FileSystem().path("sensor.txt"))
  .withFormat(new Csv())
  .withSchema(new Schema()
    .field("id", DataTypes.STRING())
    .field("timestamp", DataTypes.BIGINT())
      .rowtime(
        new Rowtime()
          .timestampsFromField("timestamp")    // 从字段中提取时间戳
          .watermarksPeriodicBounded(1000)    // watermark延迟1秒
      )
    .field("temperature", DataTypes.DOUBLE())
  ) // 定义表结构
  .createTemporaryTable("inputTable") // 创建临时表

3、DDL
val sinkDDL: String =
"""
|create table dataTable (
|  id varchar(20) not null,
|  ts bigint,
|  temperature double,
|  rt AS TO_TIMESTAMP( FROM_UNIXTIME(ts) ),
|  watermark for rt as rt - interval '1' second
|) with (
|  'connector.type' = 'filesystem',
|  'connector.path' = 'file:///D:\\..\\sensor.txt',
|  'format.type' = 'csv'
|)
""".stripMargin
tableEnv.sqlUpdate(sinkDDL) // 执行 DDL

```

### 19、开窗sql

滚动窗口

```
// Tumbling Event-time Window（事件时间字段rowtime）
.window(Tumble over 10.minutes on 'rowtime as 'w)

// Tumbling Processing-time Window（处理时间字段proctime）
.window(Tumble over 10.minutes on 'proctime as 'w)

// Tumbling Row-count Window (类似于计数窗口，按处理时间排序，10行一组)
.window(Tumble over 10.rows on 'proctime as 'w)

```

滑动窗口

```
// Sliding Event-time Window
.window(Slide over 10.minutes every 5.minutes on 'rowtime as 'w)

// Sliding Processing-time window 
.window(Slide over 10.minutes every 5.minutes on 'proctime as 'w)

// Sliding Row-count window
.window(Slide over 10.rows every 5.rows on 'proctime as 'w)

```

会话窗口

```
// Session Event-time Window
.window(Session withGap 10.minutes on 'rowtime as 'w)

// Session Processing-time Window 
.window(Session withGap 10.minutes on 'proctime as 'w)

```

over windows

```
// 无界的事件时间over window (时间字段 "rowtime")
.window(Over partitionBy 'a orderBy 'rowtime preceding UNBOUNDED_RANGE as 'w)

//无界的处理时间over window (时间字段"proctime")
.window(Over partitionBy 'a orderBy 'proctime preceding UNBOUNDED_RANGE as 'w)

// 无界的事件时间Row-count over window (时间字段 "rowtime")
.window(Over partitionBy 'a orderBy 'rowtime preceding UNBOUNDED_ROW as 'w)

//无界的处理时间Row-count over window (时间字段 "rowtime")
.window(Over partitionBy 'a orderBy 'proctime preceding UNBOUNDED_ROW as 'w)


// 有界的事件时间over window (时间字段 "rowtime"，之前1分钟)
.window(Over partitionBy 'a orderBy 'rowtime preceding 1.minutes as 'w)

// 有界的处理时间over window (时间字段 "rowtime"，之前1分钟)
.window(Over partitionBy 'a orderBy 'proctime preceding 1.minutes as 'w)

// 有界的事件时间Row-count over window (时间字段 "rowtime"，之前10行)
.window(Over partitionBy 'a orderBy 'rowtime preceding 10.rows as 'w)

// 有界的处理时间Row-count over window (时间字段 "rowtime"，之前10行)
.window(Over partitionBy 'a orderBy 'proctime preceding 10.rows as 'w)

```

窗口函数

```
	TUMBLE(time_attr, interval)
定义一个滚动窗口，第一个参数是时间字段，第二个参数是窗口长度。
	HOP(time_attr, interval, interval)
定义一个滑动窗口，第一个参数是时间字段，第二个参数是窗口滑动步长，第三个是窗口长度。
	SESSION(time_attr, interval)
定义一个会话窗口，第一个参数是时间字段，第二个参数是窗口间隔（Gap）。

另外还有一些辅助函数，可以用来选择Group Window的开始和结束时间戳，以及时间属性。
这里只写TUMBLE_*，滑动和会话窗口是类似的（HOP_*，SESSION_*）。
	TUMBLE_START(time_attr, interval)
	TUMBLE_END(time_attr, interval)
	TUMBLE_ROWTIME(time_attr, interval)
	TUMBLE_PROCTIME(time_attr, interval)

```

20、函数

自带的一些

```
	比较函数
SQL：
value1 = value2
value1 > value2
Table API：
ANY1 === ANY2
ANY1 > ANY2
	逻辑函数
SQL：
boolean1 OR boolean2
boolean IS FALSE
NOT boolean
Table API：
BOOLEAN1 || BOOLEAN2
BOOLEAN.isFalse
!BOOLEAN
	算术函数
SQL：
numeric1 + numeric2
POWER(numeric1, numeric2)
Table API：
NUMERIC1 + NUMERIC2
NUMERIC1.power(NUMERIC2)
	字符串函数
SQL：
string1 || string2
UPPER(string)
CHAR_LENGTH(string)
Table API：
STRING1 + STRING2
STRING.upperCase()
STRING.charLength()
	时间函数
SQL：
DATE string
TIMESTAMP string
CURRENT_TIME
INTERVAL string range
Table API：
STRING.toDate
STRING.toTimestamp
currentTime()
NUMERIC.days
NUMERIC.minutes
	聚合函数
SQL：
COUNT(*)
SUM([ ALL | DISTINCT ] expression)
RANK()
ROW_NUMBER()
Table API：
FIELD.count
FIELD.sum0    

```

标量函数ScalarFunction

```
// 自定义一个标量函数
class HashCode( factor: Int ) extends ScalarFunction {
  def eval( s: String ): Int = {
    s.hashCode * factor
  }
}

注册并使用：
val addScore = new AddScore(10)
tblEnv.registerFunction("addScore",addScore)

val table1: Table = tblEnv.sqlQuery("select id, score, addScore(score) as new_score from table01")
```

表函数

```
// 自定义TableFunction
class Split(separator: String) extends TableFunction[(String, Int)]{
  def eval(str: String): Unit = {
    str.split(separator).foreach(
      word => collect((word, word.length))
    )
  }
}

注册并使用：
tableEnv.createTemporaryView("sensor", sensorTable)
    tableEnv.registerFunction("split", split)
    
    val resultSqlTable = tableEnv.sqlQuery(
      """
        |select id, word, length
        |from
        |sensor, LATERAL TABLE(split(id)) AS newsensor(word, length)
      """.stripMargin)

    // 或者用左连接的方式
    val resultSqlTable2 = tableEnv.sqlQuery(
      """
        |SELECT id, word, length
        |FROM
        |sensor
|  LEFT JOIN 
|  LATERAL TABLE(split(id)) AS newsensor(word, length) 
|  ON TRUE
      """.stripMargin
    )

直接使用：
// Table API中调用，需要用joinLateral
    val resultTable = sensorTable
      .joinLateral(split('id) as ('word, 'length))   // as对输出行的字段重命名
      .select('id, 'word, 'length)
    
// 或者用leftOuterJoinLateral
    val resultTable2 = sensorTable
      .leftOuterJoinLateral(split('id) as ('word, 'length))
      .select('id, 'word, 'length)

```

### 20、热门商品

```
package com.zhangyubo.hotitem


import java.sql.Timestamp
import java.util.Properties
import java.{lang, util}

import org.apache.flink.api.common.functions.AggregateFunction
import org.apache.flink.api.common.serialization.SimpleStringSchema
import org.apache.flink.api.common.state.{ListState, ListStateDescriptor}
import org.apache.flink.api.java.tuple._
import org.apache.flink.configuration.Configuration
import org.apache.flink.streaming.api.TimeCharacteristic
import org.apache.flink.streaming.api.functions.KeyedProcessFunction
import org.apache.flink.streaming.api.scala._
import org.apache.flink.streaming.api.scala.function.WindowFunction
import org.apache.flink.streaming.api.windowing.time.Time
import org.apache.flink.streaming.api.windowing.windows.TimeWindow
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer09
import org.apache.flink.util.Collector

import scala.collection.mutable.ListBuffer

/**
 * 封装用户行为数据
 *
 * @param userId     用户id
 * @param itemId     商品id
 * @param categoryId 商品类别id
 * @param behavior   用户行为
 * @param timestamp  操作时间
 */
case class UserBehavior(userId: Long, itemId: Long, categoryId: Int, behavior: String, timestamp: Long)

/**
 * 封装结果数据
 *
 * @param itemId    商品id
 * @param windowEnd 窗口结束时间
 * @param count     计数
 */
case class ItemViewCount(itemId: Long, windowEnd: Long, count: Long)

object HotItems {
  def main(args: Array[String]): Unit = {

    val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment

    env.setParallelism(1)

    env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
    //读取文件
    val inputStream: DataStream[String] = env.readTextFile("D:\\data\\idea\\wordspace\\UserBehaviorAnalysis\\HotItemsAnalysis\\src\\main\\resources\\UserBehavior.csv")
    //读取kafka
    val properties = new Properties()
    properties.setProperty("bootstrap.servers", "hadoop102:9092")
    properties.setProperty("group.id", "consumer-group")
    properties.setProperty("key.deserializer",
      "org.apache.kafka.common.serialization.StringDeserializer")
    properties.setProperty("value.deserializer",
      "org.apache.kafka.common.serialization.StringDeserializer")
    properties.setProperty("auto.offset.reset", "latest")

    val kafkaStream: DataStream[String] = env.addSource(new FlinkKafkaConsumer09[String]("test",new SimpleStringSchema(),properties))

    val dataStream: DataStream[UserBehavior] = inputStream.map(
      data => {
        val arr = data.split(",")
        UserBehavior(arr(0).toLong, arr(1).toLong, arr(2).toInt, arr(3), arr(4).toLong)
      }
    ).assignAscendingTimestamps(_.timestamp * 1000L)

    val aggStream: KeyedStream[ItemViewCount, Tuple] = dataStream
      .filter(_.behavior == "pv")
      .keyBy("itemId")
      .timeWindow(Time.hours(1), Time.minutes(5))
      .aggregate(new CountAgg, new WindowResultFunction)
      .keyBy(1)

    aggStream
      .process(new TopNHotItems(3))
      .print()


    env.execute("TopN")

  }

}

/**
 * 预聚合函数 ，输入，状态，输出
 */
class CountAgg() extends AggregateFunction[UserBehavior, Long, Long] {
  override def createAccumulator(): Long = 0L

  override def add(value: UserBehavior, acc: Long): Long = acc + 1

  override def getResult(acc: Long): Long = acc

  override def merge(a: Long, b: Long): Long = a + b
}

/**
 * 输入，输出，key,窗口类型
 */
class WindowResultFunction extends WindowFunction[Long, ItemViewCount, Tuple, TimeWindow] {
  /**
   * Evaluates the window and outputs none or several elements.
   *
   * @param key    The key for which this window is evaluated.
   * @param window The window that is being evaluated.
   * @param input  The elements in the window being evaluated.
   * @param out    A collector for emitting elements.
   * @throws Exception The function may throw exceptions to fail the program and trigger recovery.
   */
  override def apply(key: Tuple,
                     window: TimeWindow,
                     input: Iterable[Long],
                     out: Collector[ItemViewCount]): Unit = {
    val itemId: Long = key.asInstanceOf[Tuple1[Long]].f0
    val windowEnd: Long = window.getEnd
    val count: Long = input.iterator.next()

    out.collect(ItemViewCount(itemId, windowEnd, count))
  }
}

class TopNHotItems(topSize: Int) extends KeyedProcessFunction[Tuple, ItemViewCount, String] {

  var itemState: ListState[ItemViewCount] = _

  override def open(parameters: Configuration): Unit = {
    super.open(parameters)

    // 命名状态变量的名字和状态变量的类型
    val itemsStateDesc = new ListStateDescriptor[ItemViewCount]("itemState-state", classOf[ItemViewCount])
    // 定义状态变量
    itemState = getRuntimeContext.getListState(itemsStateDesc)
  }

  override def processElement(value: ItemViewCount,
                              ctx: KeyedProcessFunction[Tuple, ItemViewCount, String]#Context,
                              out: Collector[String]): Unit = {
    itemState.add(value)
    ctx.timerService().registerEventTimeTimer(value.windowEnd + 1)
  }

  override def onTimer(timestamp: Long,
                       ctx: KeyedProcessFunction[Tuple, ItemViewCount, String]#OnTimerContext,
                       out: Collector[String]): Unit = {
    val allItems: ListBuffer[ItemViewCount] = ListBuffer()
    val iter: util.Iterator[ItemViewCount] = itemState.get().iterator()
    while (iter.hasNext) {
      allItems += iter.next()
    }

    itemState.clear()

    val sortedItems: ListBuffer[ItemViewCount] = allItems.sortBy(_.count)(Ordering.Long.reverse).take(topSize)

    val result = new StringBuilder
    result.append("====================================\n")
    result.append("时间: ").append(new Timestamp(timestamp - 1)).append("\n")

    for (i <- sortedItems.indices) {
      val curr: ItemViewCount = sortedItems(i)

      // e.g.  No1：  商品ID=12224  浏览量=2413
      result.append("No").append(i + 1).append(":")
        .append("  商品ID=").append(curr.itemId)
        .append("  浏览量=").append(curr.count).append("\n")
    }

    result.append("====================================\n\n")
    // 控制输出频率，模拟实时滚动结果
    Thread.sleep(1000)
    out.collect(result.toString())
  }
}
```

