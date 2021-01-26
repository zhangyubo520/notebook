### 1、概述

```
spark基于内存，通用，可扩展的大数据计算引擎
```

### 2、模块介绍

```
Spark Core ： 核心模块
Spark SQL : 操作结构化数据的组件
Spark Stream：流式计算
Spark MLlib : 机器学习
Spark GraphX : 图形计算
```

### 3、简单实用用

```
1、增加scala插件
2、pom文件依赖关系：
<dependency>
        <groupId>org.apache.spark</groupId>
        <artifactId>spark-core_2.12</artifactId>
        <version>2.4.5</version>
    </dependency>

<build>
    <plugins>
        <!-- 该插件用于将Scala代码编译成class文件 -->
        <plugin>
            <groupId>net.alchim31.maven</groupId>
            <artifactId>scala-maven-plugin</artifactId>
            <version>3.2.2</version>
            <executions>
                <execution>
                    <!-- 声明绑定到maven的compile阶段 -->
                    <goals>
                        <goal>testCompile</goal>
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

```

### 4、运行环境

#### local

```
1、解压缩并改名字
tar -zxvf spark-2.4.5-bin-without-hadoop-scala-2.12.tgz -C /opt/module
cd /opt/module 
mv spark-2.4.5-bin-without-hadoop-scala-2.12 spark-local
2、修改spark-env.sh.template文件名为spark-env.sh配置文件添加：
SPARK_DIST_CLASSPATH=$(/opt/module/hadoop-3.1.3/bin/hadoop classpath)
3、启动：
bin/spark-shell --master local[*]
4、web访问：
http://hadoop102:4040

5、简单实用
sc.textFile("data/word.txt").flatMap(_.split("")).map((_,1)).reduceByKey(_+_).collect
6、提交jar包执行
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master local[2] \
./examples/jars/spark-examples_2.12-2.4.5.jar \
10
1)	--class表示要执行程序的主类
2)	--master local[2] 部署模式，默认为本地模式，数字表示分配的虚拟CPU核数量
3)	spark-examples_2.12-2.4.5.jar 运行的应用类所在的jar包
4)	数字10表示程序的入口参数，用于设定当前应用的任务数量

```

#### standalone

|       | hadoop102        | hadoop103 | hadoop104 |
| ----- | ---------------- | --------- | --------- |
| Spark | Worker( Master ) | Worker    | Worker    |

```
1、解压缩并改名字
tar -zxvf spark-2.4.5-bin-without-hadoop-scala-2.12.tgz -C /opt/module
cd /opt/module 
mv spark-2.4.5-bin-without-hadoop-scala-2.12 spark-local

2、修改spark-env.sh.template文件名为spark-env.sh配置文件添加：
SPARK_DIST_CLASSPATH=$(/opt/module/hadoop-3.1.3/bin/hadoop classpath)
export JAVA_HOME=/opt/module/jdk1.8.0_212
SPARK_MASTER_HOST=hadoop102
SPARK_MASTER_PORT=7077

slaves.template文件名为slaves并添加：
hadoop102
hadoop103
hadoop104

4、分发

5、启动
sbin/start-all.sh

6、web查看：
http://hadoop102:8080

7、提交jar包
bin/spark-submit \
--class <main-class>
--master <master-url> \
... # other options
<application-jar> \
[application-arguments]

```

| 参数                      | 解释                                                         |
| ------------------------- | ------------------------------------------------------------ |
| --class                   | Spark程序中包含主函数的类                                    |
| --master                  | Spark程序运行的模式  本地模式：local[*]、spark://linux1:7077、yarn |
| --executor-memory  1G     | 指定每个executor可用内存为1G                                 |
| --total-executor-cores  2 | 指定所有executor使用的cpu核数为2个                           |
| --executor-cores          | 指定每个executor使用的cpu核数                                |
| application-jar           | 打包好的应用jar，包含依赖。这个URL在集群中全局可见。 比如hdfs:// 共享存储系统，如果是file://  path，那么所有的节点的path都包含同样的jar |
| application-arguments     | 传给main()方法的参数                                         |

#### yarn

```
1、解压缩并改名字
tar -zxvf spark-2.4.5-bin-without-hadoop-scala-2.12.tgz -C /opt/module
cd /opt/module 
mv spark-2.4.5-bin-without-hadoop-scala-2.12 spark-local

2、修改spark-env.sh.template文件名为spark-env.sh配置文件添加：
SPARK_DIST_CLASSPATH=$(/opt/module/hadoop3/bin/hadoop classpath)
export JAVA_HOME=/opt/module/jdk1.8.0_144
YARN_CONF_DIR=/opt/module/hadoop/etc/hadoop

3、yarn-site.xml
<!--是否启动一个线程检查每个任务正使用的物理内存量，如果任务超出分配值，则直接将其杀掉，默认是true -->
<property>
     <name>yarn.nodemanager.pmem-check-enabled</name>
     <value>false</value>
</property>

<!--是否启动一个线程检查每个任务正使用的虚拟内存量，如果任务超出分配值，则直接将其杀掉，默认是true -->
<property>
     <name>yarn.nodemanager.vmem-check-enabled</name>
     <value>false</value>
</property>

4、启动hadoop集群

5、提交应用：
默认文件路径是hdfs中的
如果想本地需要增加协议，sc.textFile(“files:///opt/module/spark/datas”)
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master yarn \
./examples/jars/spark-examples_2.12-2.4.5.jar \
10

6、web查看
http://hadoop102:8088
```

#### K8S & Mesos模式

```
https://spark.apache.org/docs/latest/running-on-kubernetes.html
```



#### Windows模式

```
1、将文件spark-2.4.5-bin-without-hadoop-scala-2.12.tgz解压缩到无中文无空格的路径中，将hadoop3依赖jar包拷贝到jars目录中

2、启动bin/spark-shell.cmd

```



### 5、配置历史服务

```
1、mv spark-defaults.conf.template spark-defaults.conf
spark.eventLog.enabled          true
spark.eventLog.dir               hdfs://hadoop102:9820/directory

hadoop fs -mkdir /directory

2、spark-env.sh
export SPARK_HISTORY_OPTS="
-Dspark.history.ui.port=18080 
-Dspark.history.fs.logDirectory=hdfs://hadoop102:9820/directory 
-Dspark.history.retainedApplications=30"

3、分发

4、启动
sbin/start-all.sh
sbin/start-history-server.sh

6、web查看
	http://hadoop102:18080
```

### 6、高可用的配置

|       | hadoop102                 | hadoop103                 | hadoop104         |
| ----- | ------------------------- | ------------------------- | ----------------- |
| Spark | Master  Zookeeper  Worker | Master  Zookeeper  Worker | Zookeeper  Worker |

```
1、启动zookeeper

2、配置文件
注释如下内容：
#SPARK_MASTER_HOST=linux1
#SPARK_MASTER_PORT=7077
SPARK_MASTER_WEBUI_PORT=8989

添加如下内容:
export SPARK_DAEMON_JAVA_OPTS="
-Dspark.deploy.recoveryMode=ZOOKEEPER 
-Dspark.deploy.zookeeper.url=hadoop102,hadoop103,hadoop104 
-Dspark.deploy.zookeeper.dir=/spark"

3、分发

4、启动
sbin/start-all.sh

5、web查看
http://hadoop102:8989

6、提交应用
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master spark://hadoop102:7070,hadoop103:7070 \
--deploy-mode cluster \
./examples/jars/spark-examples_2.12-2.4.5.jar \
10


```

### 7、端口号

```
	Spark查看当前Spark-shell运行任务情况端口号：4040（计算）
	Spark Master内部通信服务端口号：7077
	Standalone模式下，Spark Master Web端口号：8080（资源）
	Spark历史服务器端口号：18080
	Hadoop YARN任务运行情况查看端口号：8088

```

### 8、运行架构

```
master：资源调度，集群监控，类似于yarn的resourcemanager
worker：一个计算节点的管理

applicationmaster：向master申请资源，监控任务执行，处理任务失败等

driver：将用户程序转换成job,任务调度，跟踪任务执行情况
executer:负责执行任务，并将计算结果返回，为rdd提供内存存储

```

运行参数说明

| 名称              | 说明                                |
| ----------------- | ----------------------------------- |
| --num-executors   | 配置Executor的数量                  |
| --executor-memory | 配置每个Executor的内存大小          |
| --executor-cores  | 配置每个Executor的虚拟CPU  core数量 |

### 9、提交流程

Client

```
1、在任务提交的本地启动driver
2、driver与resourcemanager通信，申请container，启动applicationmaster，负责向resourcemanager申请executer内存
3、applicationmaster向resourcemanager申请container启动executer,applicationmaster在指定nodemanager上启动executer
4、启动后的executer会向driver反向注册，driver等全部的executer注册完毕后，启动main线程
5、执行到action算子时触发一个job,并根据宽依赖划分多个stage，每个stage对应一个taskset,之后将taskset分发到机器上运行
```

cluster

```
1、任务提交后会和resourcemanager通信，申请container启动applicationmaster，
2、分配container后在合适的nodemanager上启动applicationmaster，也就是driver，
3、applicationmaster向resourcemanager申请container启动executer,applicationmaster在指定nodemanager上启动executer
4、启动后的executer会向driver反向注册，driver等全部的executer注册完毕后，启动main线程
5、执行到action算子时触发一个job,并根据宽依赖划分多个stage，每个stage对应一个taskset,之后将taskset分发到机器上运行
```

### 10、数据结构

#### RDD

```
弹性式，分布式，数据集，不可变
弹性的理解：
	存储的弹性：内存与磁盘的自动切换；
	容错的弹性：数据丢失可以自动恢复；
	计算的弹性：计算出错重试机制；
	分片的弹性：可根据需要重新分片。

读取数据的方式：

读取内存数据时，数据按如下进行分区：如果能平分，就平分，如果不能平分，余数均分给后面的几个区
读取文件数据时，读取文件采用hadoop的规则，切片按照字节的方式，读取按行读取，
如果读取到一行的第一个字节，那么整行都会被读取到放到一个分区。
回车换行是两个字节


创建RDD的方式：
从集合: sc.makeRDD(List("hello spark","hello scala"))
从文件 ：sc.textFile("input")
从其他RDD转换 ：运算完后产生新的RDD
new : 框架自身使用
```

#### 累加器

```
分布式，共享，只写变量

实现原理：将executer端的变量信息聚合到driver端
具体实现：声明在driver端的变量，在executer端的每个task都会受到此变量的副本，每个task更新完副本后，传回driver进行聚合

使用系统累加器：
val conf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("wordcount03")

    val sc = new SparkContext(conf)

    val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4,5))

    var sum = sc.longAccumulator("sum")

    rdd.foreach(sum.add(_))

    println(s"sum=$sum")

    sc.stop()

自定义累加器：
class Count extends AccumulatorV2[String,mutable.Map[String,Long]]{

  private var map: mutable.Map[String, Long] = mutable.Map[String,Long]()//使用Map存储统计的结果

  override def isZero: Boolean = {
    map.isEmpty
  }//判断容器是否为空，因为每次使用这个累加器之前，需要确保累加器中容器是空的

  override def copy(): AccumulatorV2[String, mutable.Map[String, Long]] = {
    new Count
  }//得到累加器对象

  override def reset(): Unit = {
    map.clear()
  }//重置累加器，即清空累加器容器中的残留数据

  override def add(v: String): Unit = {
   map(v) = map.getOrElse(v,0L) + 1L
  }//一个容器内数据的聚合操作

  override def merge(other: AccumulatorV2[String, mutable.Map[String, Long]]): Unit = {
    var map1 = map
    var map2 = other.value

    map = map1.foldLeft(map2)(
      (x,y)=>{
        x(y._1) = x.getOrElse(y._1,0L) + y._2
        x
      }
    )
  }//多个容器之间数据的聚合操作，回到driver端合并

  override def value: mutable.Map[String, Long] = {
    map
  }//返回装满数据的容器
}

使用自定义累加器：
val sparkConf = new SparkConf().setMaster("local[*]").setAppName("my_count")
    val sc = new SparkContext(sparkConf)

    val count = new Count//得到累加器对象
    sc.register(count,"count")//注册累加器对象，使用自带的不用注册

    val rdd = sc.makeRDD(
      List("hello spark", "hello scala")
    )
    rdd.foreach(
      s=>{
        s.split(" ").foreach(
          s=>{
            count.add(s)//向累加器中添加数据，添加时调用累加器中的逻辑
          }
        )
      }
    )
    println(count.value)//得到累加器中的装满数据的容器

    sc.stop()
```

#### 广播变量

```
分布式，共享，只读变量

向所有节点发送一份大对象，在多个并行操作中使用同一个变量，但是 Spark会为每个任务分别发送
val conf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("wordcount04")

    val sc = new SparkContext(conf)

    val list = List(("a",4), ("c", 5), ("c", 6), ("d", 7))

    val rdd: RDD[(String, Int)] = sc.makeRDD(List( ("a",1), ("b", 2), ("c", 3), ("d", 4) ),4)

    val br: Broadcast[List[(String, Int)]] = sc.broadcast(list)

    val rdd1: RDD[(String, Int)] = rdd.map {
      case (k, v) => {
        var num = v
        for ((x, y) <- br.value) {
          if (k == x) {
            num = num + y
          }
        }
        (k, num)
      }
    }
    rdd1.collect().foreach(println)

    sc.stop()

说明：有一个list1里是去重的，另一个list2是未去重的，取出每一个list1中的元素和list2中元素进行去重，得到最终的聚合值
如果两个都是未去重的，可以先将一个去重再用上面的方法做聚合
```

#### Threadlocal

```
说明：线程内共享，线程间隔离，最常见的ThreadLocal使用场景为 用来解决数据库连接、Session管理

原理：变量是存储在当前线程的ThreadLocalMap中的，key为Threadlocal变量本身，value为任意，每个线程中可有多个threadLocal变量，就像上面代码中的longLocal和stringLocal，get之前，必须先set，否则会报空指针异常

简单使用，将sparkContext做成Threadlocal
val conf : ThreadLocal[SparkContext] = new ThreadLocal[SparkContext]

    def getEnv() ={
      var sc: SparkContext = conf.get()
      if(sc == null){
        val env = new SparkConf().setMaster("local[*]").setAppName("env")
        sc = new SparkContext(env)
      }
      sc
    }
    def clear() = {
      getEnv.stop()
      conf.remove()
    }
```

### 11、任务调度

```
1、job,stage,task介绍
job是以action为界，遇到一个action方法会生成一个job
stage是job的子集，以RDD的宽依赖为界，遇到shuffle做一次划分
task是stage的子集，以并行度（分区数）为界，分区是多少，则有多少个task

```

### 12、转换算子

```
map
mapPartitions
mapPartitionsWithIndex
flatMap
groupBy
filter
sample
distinct
coalesce
repartition
sortBy
union
subtract
zip
partionBy
reduceByKey
groupByKey
foldByKey
combineByKey
join
sortByKey

```

### 13、行动算子

```
reduce
print
foreach
collect
first
take
takeOrdered
countByKey
saveAsTextFile
```

### 14、业务经验

```
1、sparkstreaming对接kafka时，可以和kafka进行分区直连，分区越多，消费kafka的能力越大

2、偏移量维护问题：如果采用自动的方式，由于spark提交偏移量和处理数据时异步的，所以可能会造成数据丢失，生产中都是手动递交偏移量

3、偏移量的保存：MySQL中，redis中，es中均可

4、spark缺点：不是基于事件的，所以没有办法处理过期数据，而flink采用waterMark和窗口的方式可以处理过期数据

5、生产消息到kafka的三种方式：
	1、手写java代码，生产数据到kafka中
	2、在MySQL中的数据，通过binlog的方式，使用canal(不支持断点续传，搭建HA),或者使用maxwell(支持断点续传)监控MySQL中数据变化，写入kafka中，
	3、数据落盘到文件中，通过flume采集文件中的数据
	
6、消息写入kafka之后，如何进行数据分析：
	使用sparkstreaming进行数据处理（聚合），将最终的数据更新到MySQL，Hbase中进行查询

7、访问高可用的集群的方法：
拿namenode的高可用来说，可以在代码中指明一个active的nn,但是不能确定这个节点一直是active的，所以需要使用命名空间，指定命名空间的方法也十分简单粗暴，直接在集群管理界面下载hive中的配置文件core，hdfs,yarn，放置在工程的resources下即可使用命名空间

8、sparkstreaming处理数据有两个重要的参数：
spark.streaming.kafka.maxRatePatition=100,每个分区每秒从kafka中消费数据的个数，10个分区就是1000条，调整这个可以增大kafka的消费能力
spark.streaming.backpressure.enabled=true,背压机制，防止spark内部的消息积压，当因为处理数据不及时，超过分批时间时，会减少每秒拉取数据的个数

9、如何增大kafka的消费能力：
	1、增大分区并同步增加消费者个数，如sparkstreaming中的分区数，同时不能超过cpu核数
	2、增加消费者每次拉取数据的条数，如sparkstreaming中消费数据个数的参数
	3、增加CPU核数

10、kafka中参数介绍：
"auto.offset.reset" -> "earliest"，偏移量消费从最开始开始，适用于测试集群反复消费，还有就是手动维护偏移量的场合
"enable.auto.commit" -> (false: lang.Boolean) 手动维护偏移量参数，

11、sparkstreaming中checkpoint的使用问题：
	1、路径一般在hdfs上，
	2、会产生大量的小文件，因为他会为每一个分区维护一个状态
	3、中间stream.cache()会减少血缘依赖，提高性能
	4、一般不用，因为有小文件问题

12、替代checkpoint的方案：
将状态保存在MySQL等数据库中，每3秒进行一次查询更新

13、怎么解决sparkstreaming与MySQL的直连问题：
	1、使用德鲁伊连接池
	2、使用mapPartition,获取连接数等于分区数

14、正确获取连接的方式：
	stream.foreachPartition{
		partitions=>{
			建立连接
			partitions.foreach{
				使用连接
			}
		}
	}

15、错误获取连接的两种方式：
	1、stream.foreach{
		建立连接
		使用连接
	}
	2、stream.foreach{
		建立连接
		rdd=>{
			使用连接
		}
		
	}
	
16、多线程写MySQL时问题解决：
因为相同的key可能分布在不同的分区中，在多线程操作时，可能有两个分区的线程都要操作一条记录，这时就会发生线程安全问题，可以使用MySQL的事务，但是效率太低，我们的想法是对相同的key进行提前进行groupBy，这样相同的key就会到同一个分区内，避免了多线程的同步问题

17、jdk1.8的日期转换函数
DateTimeForMatter.ofPatten("yyyy-MM-dd").format(LocalDateTime.now())
```

