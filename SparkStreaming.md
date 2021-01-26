# SparkStreaming

### 1、概念

```
Spark Streaming用于流式数据的处理。Spark Streaming支持的数据输入源很多，例如：Kafka、Flume、Twitter、ZeroMQ和简单的TCP套接字等等。数据输入后可以用Spark的高度抽象原语如：map、reduce、join、window等进行运算。而结果也能保存在很多地方，如HDFS，数据库等
```

### 2、特点

```
易用，容错，易整合到spark体系，背压机制（默认不开启，可以使用spark.streaming.backpressure.enabled=true来启用）
```

### 3、DStream输入流

```
介绍：Discretized Stream(离散型数据流)，是SparkStreaming的基础抽象

依赖：
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-streaming_2.12</artifactId>
    <version>2.4.5</version>
</dependency>

创建：
1、RDD队列
val conf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("dstest01")

    val sc = new StreamingContext(conf,Seconds(3))//创建上下文环境

    val queue = new mutable.Queue[RDD[Int]]()//创建RDD队列

    for(i <- Range(0,500)){
      queue += sc.sparkContext.makeRDD(1 to 300, 10)
      Thread.sleep(2000)
    }//往队列中生成数据

    val is: InputDStream[Int] = sc.queueStream(queue,oneAtATime = false)//创建队列输入流
    val ds: DStream[(Int, Int)] = is.map((_,1)).reduceByKey(_+_)//处理流中的数据

    ds.print()//显示

    sc.start()
    sc.awaitTermination()
    
2、自定义数据源
3、Kafka数据源
首先导入依赖
<dependency>
     <groupId>org.apache.spark</groupId>
     <artifactId>spark-streaming-kafka-0-10_2.12</artifactId>
     <version>2.4.5</version>
</dependency>

	val conf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("dstest02")

    val sc = new StreamingContext(conf,Seconds(3))//创建上下文环境

    val kafka: Map[String, Object] = Map[String, Object](
      ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG -> "hadoop102:9092,hadoop103:9092,hadoop104:9092",
      ConsumerConfig.GROUP_ID_CONFIG -> "atguigu",
      "key.deserializer" -> "org.apache.kafka.common.serialization.StringDeserializer",
      "value.deserializer" -> "org.apache.kafka.common.serialization.StringDeserializer"
    )//定义kafka参数
    
    val kafkaDStream: InputDStream[ConsumerRecord[String, String]] = KafkaUtils.createDirectStream[String, String](sc,
      LocationStrategies.PreferConsistent,
      ConsumerStrategies.Subscribe[String, String](Set("atguigu"), kafka)
    )//读取kafka数据创建DStream
    
    val ds: DStream[String] = kafkaDStream.map(s=>s.value())//取出value值

    ds.flatMap(_.split(" ")).map((_,1)).reduceByKey(_+_).print()//处理数据并显示
    
    sc.start()
    sc.awaitTermination()
    
注意：先启动消费程序，然后使用kafka-console-producer.sh --broker-list hadoop102:9092 --topic atguigu生产数据

4、通过监控端口创建DStream，读进来的数据为一行行
val lineStreams = ssc.socketTextStream("hadoop102", 9999)
启动程序后，通过命令nc -lk 9999向端口发送数据
```

### 4、DStream转换

#### 无状态转换

把简单的RDD转化操作应用到每个批次上，也就是转化DStream中的每一个RDD

注意：针对键值对的DStream转化操作(比如 reduceByKey())要添加import StreamingContext._才能在Scala中使用

```
map,filter,flatMap,reduceByKey,GroupByKey,repartition

transform:允许DStream上执行任意的RDD-to-RDD函数。即使这些函数并没有在DStream的API中暴露出来
	//创建DStream
    val lineDStream: ReceiverInputDStream[String] = ssc.socketTextStream("hadoop102", 9999)

    //转换为RDD操作
    val wordAndCountDStream: DStream[(String, Int)] = lineDStream.transform(rdd => {

      val words: RDD[String] = rdd.flatMap(_.split(" "))

      val wordAndOne: RDD[(String, Int)] = words.map((_, 1))

      val value: RDD[(String, Int)] = wordAndOne.reduceByKey(_ + _)

      value
    })

join:两个流之间的join需要两个流的批次大小一致，这样才能做到同时触发计算。计算过程就是对当前批次的两个流中各自的RDD进行join，与两个RDD的join效果相同

	//从端口获取数据创建流
    val lineDStream1: ReceiverInputDStream[String] = ssc.socketTextStream("hadoop102", 9999)
    val lineDStream2: ReceiverInputDStream[String] = ssc.socketTextStream("hadoop103", 8888)

    //4.将两个流转换为KV类型
    val wordToOneDStream: DStream[(String, Int)] = lineDStream1.flatMap(_.split(" ")).map((_, 1))
    val wordToADStream: DStream[(String, String)] = lineDStream2.flatMap(_.split(" ")).map((_, "a"))

    //5.流的JOIN
    val joinDStream: DStream[(String, (Int, String))] = wordToOneDStream.join(wordToADStream)

```

#### 有状态转换

```
updateStateByKey：记录历史记录，跨批次维护状态，新信息更新时保持任意状态
1、需要设置检查点ssc.checkpoint("output")否则Please set it by StreamingContext.checkpoint()
val conf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("dstest03")

    val sc = new StreamingContext(conf,Seconds(3))//创建上下文环境
    sc.checkpoint("output")
    val rids: ReceiverInputDStream[String] = sc.socketTextStream("hadoop102",9999)//拉取数据创建DSt
    
    val ds: DStream[(String, Int)] = rids.flatMap(_.split(" ")).map((_,1))//处理数据
    
    val ds1: DStream[(String, Int)] = ds.updateStateByKey[Int](
      ( seq:Seq[Int], buffer:Option[Int] ) => {
          val newBuffer = buffer.getOrElse(0) + seq.sum
          Option(newBuffer)
      }
    )//使用updateStateByKey保存历史状态
    ds1.print()//显示打印

    sc.start()
    sc.awaitTermination()

注意：先启动消费端，然后在hadoop102上使用nc -lk 9999 回车生产数据

Window Operations：3秒一个批次，窗口12秒，滑步6秒，窗口和滑步都应该是批次的整数倍

pairs.reduceByKeyAndWindow((a:Int,b:Int) => (a + b),Seconds(1200), Seconds(6))//近1200秒内，每6秒更新一次数据状态


另外的一些：
（1）window(windowLength, slideInterval): 基于对源DStream窗化的批次进行计算返回一个新的Dstream；
（2）countByWindow(windowLength, slideInterval): 返回一个滑动窗口计数流中的元素个数；
（3）reduceByWindow(func, windowLength, slideInterval): 通过使用自定义函数整合滑动区间流元素来创建一个新的单元素流；
（4）reduceByKeyAndWindow(func, windowLength, slideInterval, [numTasks]): 当在一个(K,V)对的DStream上调用此函数，会返回一个新(K,V)对的DStream，此处通过对滑动窗口中批次数据使用reduce函数来整合每个key的value值。
（5）reduceByKeyAndWindow(func, invFunc, windowLength, slideInterval, [numTasks]): 这个函数是上述函数的变化版本，每个窗口的reduce值都是通过用前一个窗的reduce值来递增计算。通过reduce进入到滑动窗口数据并”反向reduce”离开窗口的旧数据来实现这个操作。一个例子是随着窗口滑动对keys的“加”“减”计数。通过前边介绍可以想到，这个函数只适用于”可逆的reduce函数”，也就是这些reduce函数有相应的”反reduce”函数(以参数invFunc形式传入)。如前述函数，reduce任务的数量通过可选参数来配置。

```

### 5、DStream输出流

```
print()：在运行流程序的驱动结点上打印DStream中每一批次数据的最开始10个元素。这用于开发和调试。在Python API中，同样的操作叫print()

saveAsTextFiles(prefix, [suffix])：以text文件形式存储这个DStream的内容。每一批次的存储文件名基于参数中的prefix和suffix。”prefix-Time_IN_MS[.suffix]”

saveAsObjectFiles(prefix, [suffix])：以Java对象序列化的方式将Stream中的数据保存为 SequenceFiles . 每一批次的存储文件名基于参数中的为"prefix-TIME_IN_MS[.suffix]". Python中目前不可用

saveAsHadoopFiles(prefix, [suffix])：将Stream中的数据保存为 Hadoop files. 每一批次的存储文件名基于参数中的为"prefix-TIME_IN_MS[.suffix]"。Python API 中目前不可用

foreachRDD(func)：这是最通用的输出操作，即将函数 func 用于产生于 stream的每一个RDD。其中参数传入的函数func应该实现将每一个RDD中数据推送到外部系统，如将RDD存入文件或者通过网络将其写入数据库
```

### 6、优雅的关闭

```
class MonitorStop(ssc: StreamingContext) extends Runnable {

  override def run(): Unit = {

    val fs: FileSystem = FileSystem.get(new URI("hdfs://linux1:9000"), new Configuration(), "atguigu")

    while (true) {
      try
        Thread.sleep(5000)
      catch {
        case e: InterruptedException =>
          e.printStackTrace()
      }
      val state: StreamingContextState = ssc.getState

      val bool: Boolean = fs.exists(new Path("hdfs://linux1:9000/stopSpark"))

      if (bool) {
        if (state == StreamingContextState.ACTIVE) {
          ssc.stop(stopSparkContext = true, stopGracefully = true)
          System.exit(0)
        }
      }
    }
  }
}

object SparkTest {

  def createSSC(): _root_.org.apache.spark.streaming.StreamingContext = {

    val update: (Seq[Int], Option[Int]) => Some[Int] = (values: Seq[Int], status: Option[Int]) => {

      //当前批次内容的计算
      val sum: Int = values.sum

      //取出状态信息中上一次状态
      val lastStatu: Int = status.getOrElse(0)

      Some(sum + lastStatu)
    }

    val sparkConf: SparkConf = new SparkConf().setMaster("local[4]").setAppName("SparkTest")

    //设置优雅的关闭
    sparkConf.set("spark.streaming.stopGracefullyOnShutdown", "true")

    val ssc = new StreamingContext(sparkConf, Seconds(5))

    ssc.checkpoint("./ck")

    val line: ReceiverInputDStream[String] = ssc.socketTextStream("linux1", 9999)

    val word: DStream[String] = line.flatMap(_.split(" "))

    val wordAndOne: DStream[(String, Int)] = word.map((_, 1))

    val wordAndCount: DStream[(String, Int)] = wordAndOne.updateStateByKey(update)

    wordAndCount.print()

    ssc
  }

  def main(args: Array[String]): Unit = {

    val ssc: StreamingContext = StreamingContext.getActiveOrCreate("./ck", () => createSSC())

    new Thread(new MonitorStop(ssc)).start()

    ssc.start()
    ssc.awaitTermination()
  }
}

```

### 7、练习

练习一：广告黑名单

```
1）读取Kafka数据之后，并对MySQL中存储的黑名单数据做校验；
2）校验通过则对给用户点击广告次数累加一并存入MySQL；
3）在存入MySQL之后对数据做校验，如果单日超过100次则将该用户加入黑名单
```

练习二：广告点击量的实时统计

```
1）单个批次内对数据进行按照天维度的聚合统计;
2）结合MySQL数据跟当前批次数据更新原有的数据
```

练习三：最近一个小时广告点击量

```
1）开窗确定时间范围；
2）在窗口内将数据转换数据结构为((adid,hm),count);
3）按照广告id进行分组处理，组内按照时分排序
```

