# MapReduce

### 一、概念

#### 1、MapReduce简介

```
1、分布式计算程序框架
2、核心功能：将用户编码和自带组件组合成程序，运行在hdfs集群上

优点：高容错（一个节点的任务失败了，会自动将任务转移到其他节点），易于编程，大容量（pb级），易于扩展（通过增加机器来增加计算能力）
缺点：非实时，非有向图计算，非流式计算
```

#### 2、MapReduce核心工作原理

假设待处理文件大小是300m,统计文件的单词数

```
Map阶段：
1、切分出三个切片，切分出3个部分，开三个MapTask处理线程
2、每一个MapTask线程，读取第一行，以空格为切分，读出单词1，单词2，单词3...依次封装成<单词1，1>...的键值对，
3、将键值对的键首字母是a-p的放到一块磁盘区域，p-z的放到另一块磁盘区域

排序阶段shuffle

Reduce阶段：
1、将Map阶段生成的数据拷贝到内存中，开两个ReduceTask线程，一个处理a-p的数据，一个处理p-z的数据
2、每个ReduceTask线程，读取相同单词生成的迭代器，统计次数，组成新的键值对<单词1，sum1>...
3、输出结果到磁盘

三个实例进程：
MrAppMaster:过程调度和状态协调
MapTask:负责Map阶段整个数据处理过程
ReduceTask:负责Reduce阶段整个数据处理过程
```

#### 3、常用数据序列化类型

| **Java类型** | **Hadoop Writable类型** |
| ------------ | ----------------------- |
| Boolean      | BooleanWritable         |
| Byte         | ByteWritable            |
| Int          | IntWritable             |
| Float        | FloatWritable           |
| Long         | LongWritable            |
| Double       | DoubleWritable          |
| String       | Text                    |
| Map          | MapWritable             |
| Array        | ArrayWritable           |

#### 4、Map编程规范

```
1.编写Mapper类

重写map方法，把送来的一行数据进行进行处理，得出<K,V>对，使用context.write(k,v)写入磁盘

说明：
a.继承于系统Mapper类
b.输入的数据是<k,v>的形式，自己可以定义其类型
c.输出的数据是<k,v>的形式，自己可以定义其类型
d.在map方法中写自己的核心逻辑，即怎么处理送来的<k,v>得到想要的<k,v>
c.map()方法会对每一个<k,v>调用一次（案例中是7行）

2.编写Reducer类

重写reduce方法，把送来的相同k值的<k,v>对组成的迭代器进行处理，得出新的<k,v>对，使用context.write(k,v)写入磁盘

说明：
a.继承于系统Reducer类
b.输入的数据是<k,v>形式，mapper的输出类型
c.输出的数据形式是<k,v>，自定义其类型
d.在reduce中编写核心逻辑，即怎么处理送过来的<k,v>得到想要的<k,v>
e.reduce()方法会对每一个不同的k调用一次(案例中是8个不同的单词)

3.编写Driver类

新建job任务:
1、配置信息：Configuration conf = new Configuration();

2、实例化：Job job = Job.getInstance(conf);

3、通过反射设置job的信息：
job.setJarByClass(自己编写的Driver类名.class)
job.setMapperClass(自己编写的Mapper类名.class)
job.setReducerClass(自己编写的Mapper类名.class)

4、通过反射设置job的输出结果的信息：
job.setOutputKeyClass(Text.class)
job.setOutputValueClass(FlowcountBean.class)

5、创建输入输出流：
FileInputFormat.setInputPaths(job,new Path("文件输入路径"))
FileOutputFormat.setOutputPath(job,new Path("文件输出路径"))

6、提交任务并退出
boolean result = job.waitForCompletion(true)
System.exit(result ? 0 : 1)
```

#### 5、MapReduce测试

方法一：window系统下，即本地测试

```
1、在window中配置环境变量:指向hadoop安装路径中bin目录
2、创建maven工程
3、编写pom.xml文件
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
    </dependency>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-slf4j-impl</artifactId>
        <version>2.12.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-client</artifactId>
        <version>3.1.3</version>
    </dependency>
</dependencies>
4、编写src/main/resources/log4j2.xml文件
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="error" strict="true" name="XMLConfig">
    <Appenders>
        <!-- 类型名为Console，名称为必须属性 -->
        <Appender type="Console" name="STDOUT">
            <!-- 布局为PatternLayout的方式，
            输出样式为[INFO] [2018-01-22 17:34:01][org.test.Console]I'm here -->
            <Layout type="PatternLayout"
                    pattern="[%p] [%d{yyyy-MM-dd HH:mm:ss}][%c{10}]%m%n" />
        </Appender>

    </Appenders>

    <Loggers>
        <!-- 可加性为false -->
        <Logger name="test" level="info" additivity="false">
            <AppenderRef ref="STDOUT" />
        </Logger>

        <!-- root loggerConfig设置 -->
        <Root level="info">
            <AppenderRef ref="STDOUT" />
        </Root>
    </Loggers>

</Configuration>
5、运行main方法
```

方法二：集群运行

```
1、集群已经配置过hadoop,配置hadoop可略过

2、配置pom.xml,增加打包信息，指明Driver类
<build>
		<plugins>
			<plugin>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>2.3.2</version>
				<configuration>
					<source>1.8</source>
					<target>1.8</target>
				</configuration>
			</plugin>
			<plugin>
				<artifactId>maven-assembly-plugin </artifactId>
				<configuration>
					<descriptorRefs>
						<descriptorRef>jar-with-dependencies</descriptorRef>
					</descriptorRefs>
					<archive>
						<manifest>
						
							<mainClass>com.atguigu.mr.WordcountDriver</mainClass>
							
						</manifest>
					</archive>
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
	
3、打包：Maven下找到自己要打包的module，点击lifecycle中的install即可
4、启动集群
5、在linux里执行：hadoop jar MapReduceDemo-1.0-SNAPSHOT.jar at.zhangyubo.mdex.WordcountDriver ~/Hello.txt ~/output
```

方法三：在Windows上向集群提交任务

```
1、添加必要配置信息
// 1 获取配置信息以及封装任务
		Configuration configuration = new Configuration();
	    //设置HDFS NameNode的地址
       configuration.set("fs.defaultFS", "hdfs://hadoop102:9820");
       // 指定MapReduce运行在Yarn上
       configuration.set("mapreduce.framework.name","yarn");
       // 指定mapreduce可以在远程集群运行
		configuration.set("mapreduce.app-submission.cross-platform","true");
    
	//指定Yarn resourcemanager的位置
	configuration.set("yarn.resourcemanager.hostname","hadoop103");
// 2 设置jar加载路径
	job.setJar("D:\\input\\MapReduce-1.0-SNAPSHOT.jar");
2、编辑任务配置
VM options -HADOOP_USER_NAME=atguigu
Program arguments hdfs://hadoop102:9820/home/atguigu/Hello.txt hdfs://hadoop102:9820/home/atguigu/output

3、启动集群，提交任务，并在集群中查看
```

#### 6、序列化

```

序列化:把内存中的对象，转化成字节序列，方便存储在磁盘或者进行网络传输
反序列化：把磁盘上或者网络上的字节序列，转化成内存中的对象使用

java中的序列化框架：Serializable
hadoop中的序列化框架：Writable


在hadoop中对象实现序列化：
1、空参构造器:
public FlowBean(){}
2、重写write()方法
public void write(DataOutput out){
    out.writeLong(upFlow);
	out.writeLong(downFlow);
	out.writeLong(sumFlow);
}

3、重写readFields()方法
public void readFields(DataInput in){
    upFlow = in.readLong();
	downFlow = in.readLong();
	sumFlow = in.readLong();
}
```

#### 7、切片机制

```

1、默认与分块大小一致128M，
2、对每一个文件进行分别切分
3、切片数量决定了map阶段的并行度
4、每一个切片分配一个mapTask

```

#### 8、Job提交流程

```
1、为什么会产生yarn？
2、Configuration类的作用是什么？
3、GenericOptionParser的作用是什么？
4、如何将命令行中的参数配置传递给变量conf？
5、哪个方法获得传入的参数？
6、如何在命令行指定reduce的数量？
7、如何在控制台打印job(mapreduce)的进度？
8、默认情况下map、reduce的个数是几个？
9、setJarByClass的作用是什么？
10、配置了哪个参数，在提交job的时候，会创建一个YARNRunner对象进行任务提交？
11、哪个类实现了读取yarn-site.xml、core-site.xml、mapred-site.xml等配置文件中的配置属性？
12、JobSubmitter类中哪个方法实现了吧job提交到集群？
13、DistributedChche在mapredecce中发挥什么作用？

```

#### 9、FileInputFormat切片原理

```
过程：
1、找到文件存储目录

2、遍历文件目录下的每一个文件

3、对每一个文件单独切分：
a、计算文件大小
b、计算切片大小：computeSplitSize(Math.max(minSize,Math.min(maxSize,blocksize)))= =bolocksize=128M
c、默认切片大小是128M
d、开始切第一个切片，0-128M，计算剩下的是否大于块的1.1倍，不大于就把剩下的当成另一个切片，继续这个过程，直至切分完成
e、将切片信息写入切片规划文件
f、整个切片过程写在getSplit()方法中
g、切片信息记录了元数据的信息，比如其实位置，长度，所在节点列表等

4、提交切片信息至yarn上，yarn上的MrAppMaster根据切片信息计算开启MapTask个数

设置切片大小：
computerSplitSize(Math.Max(minSize,Math.Min(maxSize,blocksize)))
一般blocksize是128M，minSize是1B，MaxSize默认是Long.MAXValue
minSize:用来设置切片较大值，比如设置256M,
maxSize:用来设置切片较小值，比如设置64M,
```

#### 10、FileInputFormat实现类

```
TextInputFormat
介绍：
1、默认的实现类，默认情况下，为每一个输入文件按照切片原则切片
2、读取每一行的记录，不包括终止符("\t"或者"\n")
3、键key为该行数据在整个文件中的偏移量，LongWritable类型
4、值value为该行内容

KeyValueTextInputFormat
介绍：
1、也是一行记录，key值和value值包含在这一行数据中，
2、通过命令conf.set(KeyValueLineRecordReader.KEY_VALUE_SEPERATOR,"\t")设置，默认是tab
3、将一行数据通过分割符切分，前一部分是键，后部分是值，值也可以重新设置
4、也就是说，可以通过这个直接获取想要的信息作为key,
5、但是获取一行数据后进行字符串切分也能做到，除非分隔符后面刚好是我们想要的key


NlineInputFormat
介绍:
1、改变了切片规则，
2、切片数量=输入文件总行数/N,如果不能整除，后面加1，
3、数据形式和TextInputFormat一样，键为该行数据的偏移量，值为该行数据

CombineTextInputFormat
介绍：
1、改变了切片规则，
2、使用CombineTextInputFormat.setMaxInputSplitSize(job, 4194304);// 4M设置切片最大值
3、切片规则：
	1、文件大小为size,若size小于4M,分为一块，若大于，则判断是否小于2*4M,若小于则平分文件为两块，大小均为size/2,若大于2*4M,则先分出一个4M的块，剩下的按照上面的流程往下分，直至分完该文件，通过上述过程，分完目录下的所有文件，
	2、将所有的切分后的文件依次和4M比较，如果大于就直接是一个片，否则就和下一个合并成一个片，
	
自定义InputFormat

```

#### 11、MapReduce工作流程

##### 1.综述

```
			 InputFormat
map阶段：Input------------>Mapper


shuffle阶段：

				  OutputFormat
reduce阶段：Reducer----------->Reducer

```

##### 2.分区

方式1：使用默认分区

```
1、在JavaBean类中重写hashCode()方法，
2、定义reduceTask个数：job.setNumReduceTask(2);
3、查看：有两个输出文件


总结：
(1)如ReduceTask的数量大于getPartition的结果数，则会多产生几个空的输和出文件
(2)如果1< ReduceTask数量小于getPartition的结果数，则有一部分分区数据无处安放,会exception
(3)如果RedreduceTask的数量=1,，则不管MapTask端输出多少个分区文件,最终结果都交给这一一个
ReduceTask,最终也就只会产生一个结果文件
(4)分区号必须从零开始，逐一累加。
```

方式2：自定义分区类

```
1、重写getPartition()方法
public WordcountPartitioner extends Partitioner<SortJavaBean,LongWritable>{
    @Override
    public int getPartition(SortJavaBean sortJavaBean, LongWritable longWritable, int i) {
            char c = sortJavaBean.getName().charAt(0);
            int parttioner = 0;

            if(c >= 'a' && c < 'q'){
                parttioner = 0;
            }else if(c >= 'q' && c <= 'z'){
                parttioner = 1;
            }else{
                parttioner = 2;
            }
            return parttioner;
    }
}

2、设置分区个数：job.setNumReduceTask(3);
3、查看：有三个输出文件
```

##### 3.全排序

```
说明：整个数据进行排序，reduce排序压力巨大，一般不用

1、默认行为，按照字典排序，使用的是快速排序，排的是键的索引，设置的reduceTask数量为1

mapTask阶段：写入缓冲区，到达阈值（80%)后进行快排，写入磁盘，全部处理完毕后，对磁盘中的文件进行一次归并排序

reduceTask阶段:拷贝数据，读入内存，超过阈值则溢写到磁盘，磁盘文件到达阈值，进行一次归并排序生成一个大文件，所有数据拷贝完成后，对内存和磁盘中的进行一次归并排序
```

##### 4.分区排序

```
说明:增加reduce数量，减轻reduce排序负担，设置的reduceTask数量为分区数量

设置：job.setNumReduceTask(3)

结果：三个输出文件内部有序
```

##### 5.combiner组件

```
说明:在map阶段就进行多次排序汇总，减轻reduce排序压力
```

##### 6.分组排序（groupingComparable)辅助排序

```
说明：先按照一定规则分组，分组内的排序

设置：job.setGroupingComparatorClass(OrderGroupingComparator.class);

结果：先按照分组类中的规则进行分组，符合规则的进入一个reduce，通过迭代器获取，分组内是排好序的

分组类：
package com.atguigu.mapreduce.order;
import org.apache.hadoop.io.WritableComparable;
import org.apache.hadoop.io.WritableComparator;

public class OrderGroupingComparator extends WritableComparator {

	protected OrderGroupingComparator() {
		super(OrderBean.class, true);
	}//将比较对象类传入构造器

	@Override
	public int compare(WritableComparable a, WritableComparable b) {

		OrderBean aBean = (OrderBean) a;
		OrderBean bBean = (OrderBean) b;

		int result;
		if (aBean.getOrder_id() > bBean.getOrder_id()) {
			result = 1;
		} else if (aBean.getOrder_id() < bBean.getOrder_id()) {
			result = -1;
		} else {
			result = 0;
		}

		return result;
	}
}
```

##### 7、预排序（combine）

```
说明：为了减轻reduce压力，在每个mapTask的节点上设置combine进行预排序，减小reduceTask的压力

设置：job.setCombinerClass(WordcountReducer.class);
这里面可以用reducer类的逻辑，如上设置的
也可以再定义一个combine类继承Reducer类，重新写一个逻辑：
job.setCombinerClass(WordcountCombineReducer.class);

结果：进入reduceTask的数据量减少

```

##### 8、默认分区

```
默认分区是根据key的hashcode对reducetask的个数取模，用户没有办法指定特定数据进入特定分区
```





#### 12、mapTask工作机制

```
（1）Read阶段：MapTask通过用户编写的RecordReader，从输入InputSplit中解析出一个个key/value。

（2）Map阶段：该节点主要是将解析出的key/value交给用户编写map()函数处理，并产生一系列新的key/value。

（3）Collect收集阶段：在用户编写map()函数中，当数据处理完成后，一般会调用OutputCollector.collect()输出结果。在该函数内部，它会将生成的key/value分区（调用Partitioner），并写入一个环形内存缓冲区中。

（4）Spill阶段：即“溢写”，当环形缓冲区满后，MapReduce会将数据写到本地磁盘上，生成一个临时文件。需要注意的是，将数据写入本地磁盘之前，先要对数据进行一次本地排序，并在必要时对数据进行合并、压缩等操作。


溢写阶段详情：

步骤1：利用快速排序算法对缓存区内的数据进行排序，排序方式是，先按照分区编号Partition进行排序，然后按照key进行排序。这样，经过排序后，数据以分区为单位聚集在一起，且同一分区内所有数据按照key有序。

步骤2：按照分区编号由小到大依次将每个分区中的数据写入任务工作目录下的临时文件output/spillN.out（N表示当前溢写次数）中。如果用户设置了Combiner，则写入文件之前，对每个分区中的数据进行一次聚集操作。

步骤3：将分区数据的元信息写到内存索引数据结构SpillRecord中，其中每个分区的元信息包括在临时文件中的偏移量、压缩前数据大小和压缩后数据大小。如果当前内存索引大小超过1MB，则将内存索引写到文件output/spillN.out.index中。

Combine阶段：当所有数据处理完成后，MapTask对所有临时文件进行一次合并，以确保最终只会生成一个数据文件。

当所有数据处理完后，MapTask会将所有临时文件合并成一个大文件，并保存到文件output/file.out中，同时生成相应的索引文件output/file.out.index。
	在进行文件合并过程中，MapTask以分区为单位进行合并。对于某个分区，它将采用多轮递归合并的方式。每轮合并io.sort.factor（默认10）个文件，并将产生的文件重新加入待合并列表中，对文件排序后，重复以上过程，直到最终得到一个大文件。
	让每个MapTask最终只生成一个数据文件，可避免同时打开大量文件和同时读取大量小文件产生的随机读取带来的开销。
```

#### 13、shuffle工作机制

```
即排序
```

#### 14、ReduceTask工作机制

```
（1）Copy阶段：ReduceTask从各个MapTask上远程拷贝一片数据，并针对某一片数据，如果其大小超过一定阈值，则写到磁盘上，否则直接放到内存中。

（2）Merge阶段：在远程拷贝数据的同时，ReduceTask启动了两个后台线程对内存和磁盘上的文件进行合并，以防止内存使用过多或磁盘上文件过多。

（3）Sort阶段：按照MapReduce语义，用户编写reduce()函数输入数据是按key进行聚集的一组数据。为了将key相同的数据聚在一起，Hadoop采用了基于排序的策略。由于各个MapTask已经实现对自己的处理结果进行了局部排序，因此，ReduceTask只需对所有数据进行一次归并排序即可。

（4）Reduce阶段：reduce()函数将计算结果写到HDFS上。

说明：
1、reducetask=0，表示没有reduce阶段
2、reducetask默认值是1，输出文件数为1
3、reducetask数量不是任意设置的，需要结合集群性能和业务需要，比如如果需要全局结果，reducetask数量必须设置成1
4、reducetask等于1时，分区数大于1也不会执行分区过程，因为执行分区的前提是判断reducenum的值是否是大于1的
```

#### 15、OutputFormat实现类

1、TextOutputFormat

```
1、默认的输出实现类
2、把键值对写为文本行
3、调用的是键值对中类中的toString()方法
```

2、SequenceFileOutputFormat

```
1、后续mapreduce的输入
```

3、用户自定义

```
1、自定义类继承FileOutputFormat
2、改写RecordWriter,具体改写数据的write()方法

1、
package com.atguigu.mapreduce.outputformat;
import java.io.IOException;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.RecordWriter;
import org.apache.hadoop.mapreduce.TaskAttemptContext;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
}
public class FilterOutputFormat extends FileOutputFormat<Text, NullWritable>{

	@Override
	public RecordWriter<Text, NullWritable> getRecordWriter(TaskAttemptContext job)			throws IOException, InterruptedException {

		// 创建一个RecordWriter
		return new FilterRecordWriter(job);
	}
}

2、
package com.atguigu.mapreduce.outputformat;
import java.io.IOException;
import org.apache.hadoop.fs.FSDataOutputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.RecordWriter;
import org.apache.hadoop.mapreduce.TaskAttemptContext;

public class FilterRecordWriter extends RecordWriter<Text, NullWritable> {

	FSDataOutputStream atguiguOut = null;
	FSDataOutputStream otherOut = null;

	public FilterRecordWriter(TaskAttemptContext job) {

		// 1 获取文件系统
		FileSystem fs;

		try {
			fs = FileSystem.get(job.getConfiguration());

			// 2 创建输出文件路径
			Path atguiguPath = new Path("e:/atguigu.log");
			Path otherPath = new Path("e:/other.log");

			// 3 创建输出流
			atguiguOut = fs.create(atguiguPath);
			otherOut = fs.create(otherPath);
		} catch (IOException e) {
			e.printStackTrace();
		}
	}

	@Override
	public void write(Text key, NullWritable value) throws IOException, InterruptedException {

		// 判断是否包含“atguigu”输出到不同文件
		if (key.toString().contains("atguigu")) {
			atguiguOut.write(key.toString().getBytes());
		} else {
			otherOut.write(key.toString().getBytes());
		}
	}

	@Override
	public void close(TaskAttemptContext context) throws IOException, InterruptedException {

		// 关闭资源
IOUtils.closeStream(atguiguOut);
		IOUtils.closeStream(otherOut);	
		}
}
```

#### 16、Reduce Join简介

```
说明：key的属性中增加标记的属性，已标记来源不同的数据，reduceTask的任务巨大

Map阶段：对两个表或两个文件的数据打上标记，标记封装在key中，

reduce阶段：对不同标记的数据进行不同的处理，提取有用的信息放入容器中（对象，集合，数组），然后提取不同容器中的信息来填充k-v对，填好的信息写出，
```

#### 17、Map Join简介

```
说明:Map Join适用于一张表十分小、一张表很大的场景。

设置：	
// 6 加载缓存数据
job.addCacheFile(new URI("file:///e:/input/inputcache/pd.txt"));
		
// 7 Map端Join的逻辑不需要Reduce阶段，设置reduceTask数量为0
job.setNumReduceTasks(0);


```

需要定义的mapper类

```
package com.atguigu.mapjoin;

import org.apache.commons.lang.StringUtils;
import org.apache.hadoop.fs.FSDataInputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IOUtils;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.net.URI;
import java.util.HashMap;
import java.util.Map;

public class MjMapper extends Mapper<LongWritable, Text, Text, NullWritable> {

    //pd表在内存中的缓存
    private Map<String, String> pMap = new HashMap<>();

    private Text line = new Text();

    //任务开始前将pd数据缓存进PMap
    @Override
    protected void setup(Context context) throws IOException, InterruptedException {
        
        //从缓存文件中找到pd.txt
        URI[] cacheFiles = context.getCacheFiles();
        Path path = new Path(cacheFiles[0]);

        //获取文件系统并开流
        FileSystem fileSystem = FileSystem.get(context.getConfiguration());
        FSDataInputStream fsDataInputStream = fileSystem.open(path);

        //通过包装流转换为reader
        BufferedReader bufferedReader = new BufferedReader(
                new InputStreamReader(fsDataInputStream, "utf-8"));

        //逐行读取，按行处理
        String line;
        while (StringUtils.isNotEmpty(line = bufferedReader.readLine())) {
            String[] fields = line.split("\t");
            pMap.put(fields[0], fields[1]);
        }

        //关流
        IOUtils.closeStream(bufferedReader);

    }

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String[] fields = value.toString().split("\t");

        String pname = pMap.get(fields[1]);

        line.set(fields[0] + "\t" + pname + "\t" + fields[2]);

        context.write(line, NullWritable.get());

    }
}
```



#### 18、计数器

方法1

```
定义计数项：
1、使用枚举类定义
enum Mycounter{A,B,C...}
2、context.getCounter(Mycounter.A).increment(1)

```

方法2

```
直接使用名字来统计
context.getCounter("第一项统计",第二项统计...).increment(1)
```



#### 19、数据清洗

```
对数据进行初步删选，去掉不符合的信息，一般只需要mapper
```

#### 20、小文件处理

```
1、设置小文件合并的参数配置：
1、可以单独在hive中配置：
    set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat;

    set hive.merge.mapFiles=true;

    set hive.merge.mapredFiles=true;

    set hive.merge.size.per.task=256000000;//每个Mapper要处理的数据，就把上面的5M10M……合并成为一个
    
        set mapred.max.split.size=256000000 // mapred切分的大小

    set mapred.min.split.size.per.node=128000000//低于128M就算小文件，数据在一个节点会合并，在多个不同的节点会把数据抓过来进行合并。
    
2、也可以单独在hadoop中配置：可以通过控制文件的数量控制mapper数量

    mapreduce.input.fileinputformat.split.minsize（default：0），小于这个值会合并

    mapreduce.input.fileinputformat.split.maxsize 大于这个值会切分
```

#### 21、分区规则

```
1、默认的分区规则：
将数据key的hash值和reducetask数量取模得到对应的分区号

2、保证全局有序时的分区：方法一：设置一个reducetask，并发性太差，方法二：自定义分区，数据最大值除以reducetask数量的商作为分界线，一倍商，两倍商，三倍商。。。去往不同的分区，这样全局就有序，还能利用并发性，但是可能会有数据倾斜

3、数据倾斜时的分区：自定义分区，散列化数据，但是必须保证同样的key进入同一个key

4、多字段排序时的分区：自定义分区，排序字段有多个，分区字段有一个
```





### 二、代码

#### 1、统计文件中单词出现次数

1、Mapper类

```
package at.zhangyubo.mdex;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

/**
 * 对传来的数据进行处理
 * @author zhangyubo
 * @create 2020-04-14 15:29
 */
public class WordcountMapper extends Mapper<LongWritable, Text, Text, IntWritable> {
//声明输出的kv对
    IntWritable outV = new IntWritable(1);

    Text outK = new Text();

    /**
     *
     * @param key 读取数据位置，一般是一行数据的首单词坐标
     * @param value 一行数据
     * @param context 调度Mapper运行
     * @throws IOException
     * @throws InterruptedException
     */
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        System.out.println("开始调用map方法");//记录调用了几次方法

        String line = value.toString();

        String[] spilts = line.split(" ");//把传过来的一行数据按空格切分，放入数组中

        for (String word: spilts) {
            outK.set(word);//编辑k,v对
            context.write(outK,outV);//通过context写出到磁盘
        }
    }
}
```

2、Reducer类

```
package at.zhangyubo.mdex;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

/**
 * 把map阶段传过来的数据进行筛选，输出需要数据，必要时进行封装
 * @author zhangyubo
 * @create 2020-04-14 15:29
 */
public class WordcountReducer extends Reducer<Text, IntWritable,Text,IntWritable> {
    //声明输出的k,v对
    IntWritable V = new IntWritable();
    /**
     *
     * @param key 一个单词
     * @param values 同一个单词对应的value组成的迭代器（这个实例中都是1）
     * @param context 调度
     * @throws IOException
     * @throws InterruptedException
     */
    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {

        int sum = 0;

        for(IntWritable value:values){
            sum += value.get();
        }//统计每个单词出现的次数
        //编辑新的v
        V.set(sum);

        context.write(key,V);//把新的kv对写出到磁盘
    }
}

```

3、Driver类

```

1、创建Configuration实例，用于配置参数；

2、编辑配置参数，包括运行使用的机器，用户等参数；

3、创建Job实例，用于编辑任务详情；

4、编辑任务详情，包括指定运行的jar包，Mapper类，Reducer类，Driver类，输出键值类，输出的键类等其他信息；

5、编辑输入输出流，使用FileInputFormat等

6、使用waitForCompletion()方法提交任务

package at.zhangyubo.mdex;

import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import org.apache.hadoop.conf.Configuration;
import java.io.IOException;

/**
 * @author zhangyubo
 * @create 2020-04-14 15:29
 */
public class WordcountDriver {
    public static void main(String[] args) throws InterruptedException, IOException, ClassNotFoundException {

        // 1 获取配置信息以及封装任务
        Configuration configuration = new Configuration();

        //设置HDFS NameNode的地址
        configuration.set("fs.defaultFS", "hdfs://hadoop102:9820");
        // 指定MapReduce运行在Yarn上
        configuration.set("mapreduce.framework.name","yarn");
        // 指定mapreduce可以在远程集群运行
        configuration.set("mapreduce.app-submission.cross-platform","true");

        //指定Yarn resourcemanager的位置
        configuration.set("yarn.resourcemanager.hostname","hadoop103");
		//参数设置完成后创建job实例
        Job job = Job.getInstance(configuration);

        // 2 设置jar加载路径
        //方法1、指定主运行类
        job.setJarByClass(WordcountDriver.class);

        // 2 设置jar加载路径
        //方法2、指定maven打包地址
        job.setJar("E:\\Maven\\LocalRepository\\at\\zhangyubo\\com\\MapReduceDemo\\1.0-SNAPSHOT\\MapReduceDemo-1.0-SNAPSHOT.jar");

        // 3 设置map和reduce类
        job.setMapperClass(WordcountMapper.class);
        job.setReducerClass(WordcountReducer.class);

        // 4 设置map输出
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);



        // 5 设置最终输出kv类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        // 6 设置输入和输出路径
        //通过传递参数的方式，参数可以在main方法中的参数设置中设置，也可以在运行在集群上时指定
        FileInputFormat.setInputPaths(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
        //方法2、也可以直接指定路径
        FileInputFormat.setInputPaths(job, new Path("d:/input"));
        FileOutputFormat.setOutputPath(job, new Path("d:/output"));   

        // 7 提交
        boolean result = job.waitForCompletion(true);

        System.exit(result ? 0 : 1);
    }
}
注意：在集群上运行时，输入输出路径都得是集群上的
```

4、pom.xml文件

```
    <?xml version="1.0" encoding="UTF-8"?>
    <Configuration status="error" strict="true" name="XMLConfig">
        <Appenders>
            <!-- 类型名为Console，名称为必须属性 -->
            <Appender type="Console" name="STDOUT">
                <!-- 布局为PatternLayout的方式，
                输出样式为[INFO] [2018-01-22 17:34:01][org.test.Console]I'm here -->
                <Layout type="PatternLayout"
                        pattern="[%p] [%d{yyyy-MM-dd HH:mm:ss}][%c{10}]%m%n" />
            </Appender>

        </Appenders>

        <Loggers>
            <!-- 可加性为false -->
            <Logger name="test" level="info" additivity="false">
                <AppenderRef ref="STDOUT" />
            </Logger>

            <!-- root loggerConfig设置 -->
            <Root level="info">
                <AppenderRef ref="STDOUT" />
            </Root>
        </Loggers>

    </Configuration>
```



5、log4j2.xml文件

```
    <?xml version="1.0" encoding="UTF-8"?>
    <Configuration status="error" strict="true" name="XMLConfig">
        <Appenders>
            <!-- 类型名为Console，名称为必须属性 -->
            <Appender type="Console" name="STDOUT">
                <!-- 布局为PatternLayout的方式，
                输出样式为[INFO] [2018-01-22 17:34:01][org.test.Console]I'm here -->
                <Layout type="PatternLayout"
                        pattern="[%p] [%d{yyyy-MM-dd HH:mm:ss}][%c{10}]%m%n" />
            </Appender>

        </Appenders>

        <Loggers>
            <!-- 可加性为false -->
            <Logger name="test" level="info" additivity="false">
                <AppenderRef ref="STDOUT" />
            </Logger>

            <!-- root loggerConfig设置 -->
            <Root level="info">
                <AppenderRef ref="STDOUT" />
            </Root>
        </Loggers>

    </Configuration>
```



#### 2、统计手机的上下行流量

1、Mapper类

```
package at.zhangyubo.phcount;

        import org.apache.hadoop.io.LongWritable;
        import org.apache.hadoop.io.Text;
        import org.apache.hadoop.mapreduce.Mapper;

        import java.io.IOException;

/**
 * 处理传来的数据
 * @author zhangyubo
 * @create 2020-04-15 10:24
 */
public class FlowcountMapper extends Mapper<LongWritable, Text,Text,FlowcountBean> {

//    声明输入输出的kv
    FlowcountBean v = new FlowcountBean();
    Text k = new Text();

    /**
     *
     * @param key 读取数据的游标
     * @param value 一行数据
     * @param context
     * @throws IOException
     * @throws InterruptedException
     */
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {

        String line = value.toString();

        String[] splits = line.split("\t");
        String phoneNum = splits[1];

        long upFlow = Long.parseLong(splits[splits.length - 3]);

        long downFlow = Long.parseLong(splits[splits.length - 2]);//从传来的数据中删选想要的信息，封装成kv对

        k.set(phoneNum);
        v.setUpFlow(upFlow);
        v.setDownFlow(downFlow);
        v.setSumFlow(upFlow,downFlow);//编辑kv对

        context.write(k,v);//kv对写入磁盘
    }
}
```

2、Reducer类

```
package at.zhangyubo.phcount;

import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

/**
 *对map阶段的数据再加工，归类汇总输出
 * @author zhangyubo
 * @create 2020-04-15 10:25
 */
public class FlowcountReducer extends Reducer<Text, FlowcountBean, Text, FlowcountBean> {

//    声明输出对象，k不用声明
    FlowcountBean flowcountBean = new FlowcountBean();

    /**
     *
     * @param key 电话号码Text类型
     * @param values 封装的是同一号码的不同流量信息，等待汇总，流量信息和号码被封装在FlowcountBean中
     * @param context 调度
     * @throws IOException
     * @throws InterruptedException
     */
    @Override
    protected void reduce(Text key, Iterable<FlowcountBean> values, Context context) throws IOException, InterruptedException {

        //定义输出v及其属性
        long sum_up = 0;
        long sum_down = 0;
        FlowcountBean flowcountBean = new FlowcountBean();

        //上行加上行，下行加下行
        for (FlowcountBean f : values) {
            sum_up += f.getUpFlow();
            sum_down += f.getDownFlow();

        }//合并上行流量和下行流量

        flowcountBean.setUpFlow(sum_up);
        flowcountBean.setDownFlow(sum_down);
        flowcountBean.setSumFlow(sum_up,sum_down);//编辑kv

        context.write(key, flowcountBean);//kv写入磁盘

    }
}

```

3、Driver类

```
package at.zhangyubo.phcount;


import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;

/**
 * @author zhangyubo
 * @create 2020-04-15 10:25
 */
public class FlowcountDriver {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        Configuration conf = new Configuration();

        Job job = Job.getInstance(conf);

        job.setJarByClass(FlowcountDriver.class);

        job.setMapperClass(FlowcountMapper.class);

        job.setReducerClass(FlowcountReducer.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(FlowcountBean.class);



        FileInputFormat.setInputPaths(job,new Path("phone_data.txt"));
        FileOutputFormat.setOutputPath(job,new Path("f:/output2"));

        boolean result = job.waitForCompletion(true);

        System.exit(result ? 0 : 1 );

    }



}

```

#### 21、优化

```
1）Map阶段
（1）增大环形缓冲区大小。由100m扩大到200m
（2）增大环形缓冲区溢写的比例。由80%扩大到90%
（3）减少对溢写文件的merge次数。（10个文件，一次20个merge）
（4）不影响实际业务的前提下，采用Combiner提前合并，减少 I/O。
2）Reduce阶段
（1）合理设置Map和Reduce数：两个都不能设置太少，也不能设置太多。太少，会导致Task等待，延长处理时间；太多，会导致 Map、Reduce任务间竞争资源，造成处理超时等错误。
（2）设置Map、Reduce共存：调整slowstart.completedmaps参数，使Map运行到一定程度后，Reduce也开始运行，减少Reduce的等待时间。
（3）规避使用Reduce，因为Reduce在用于连接数据集的时候将会产生大量的网络消耗。
（4）增加每个Reduce去Map中拿数据的并行数
（5）集群性能可以的前提下，增大Reduce端存储数据内存的大小。 
3）IO传输
采用数据压缩的方式，减少网络IO的的时间。安装Snappy和LZOP压缩编码器。
压缩：
（1）map输入端主要考虑数据量大小和切片，支持切片的有Bzip2、LZO。注意：LZO要想支持切片必须创建索引；
（2）map输出端主要考虑速度，速度快的snappy、LZO；
（3）reduce输出端主要看具体需求，例如作为下一个mr输入需要考虑切片，永久保存考虑压缩率比较大的gzip。
4）整体
（1）NodeManager默认内存8G，需要根据服务器实际配置灵活调整，例如128G内存，配置为100G内存左右，yarn.nodemanager.resource.memory-mb。
（2）单任务默认内存8G，需要根据该任务的数据量灵活调整，例如128m数据，配置1G内存，yarn.scheduler.maximum-allocation-mb。
（3）mapreduce.map.memory.mb ：控制分配给MapTask内存上限，如果超过会kill掉进程（报：Container is running beyond physical memory limits. Current usage:565MB of512MB physical memory used；Killing Container）。默认内存大小为1G，如果数据量是128m，正常不需要调整内存；如果数据量大于128m，可以增加MapTask内存，最大可以增加到4-5g。
（4）mapreduce.reduce.memory.mb：控制分配给ReduceTask内存上限。默认内存大小为1G，如果数据量是128m，正常不需要调整内存；如果数据量大于128m，可以增加ReduceTask内存大小为4-5g。
（5）mapreduce.map.java.opts：控制MapTask堆内存大小。（如果内存不够，报：java.lang.OutOfMemoryError）
（6）mapreduce.reduce.java.opts：控制ReduceTask堆内存大小。（如果内存不够，报：java.lang.OutOfMemoryError）
（7）可以增加MapTask的CPU核数，增加ReduceTask的CPU核数
（8）增加每个Container的CPU核数和内存大小
（9）在hdfs-site.xml文件中配置多目录（多磁盘）
（10）NameNode有一个工作线程池，用来处理不同DataNode的并发心跳以及客户端并发的元数据操作。dfs.namenode.handler.count= ，比如集群规模为8台时，此参数设置为41。可通过简单的python代码计算该值，代码如下。



1、fair-scheduler.xml中的weight的作用是什么？怎么样能测出来weight的作用？
2、使用fair调度策略，会不会出现负载不均衡的现象？网上说：默认批处理会出现负载不均衡，但每次都是均衡的， 涉及到yarn.scheduler.fair.max.assign和yarn.scheduler.assignmultiple 两个配置项。这块到底是什么样的？


(1) weight主要用在资源共享之时，weight越大，拿到的资源越多。比如一个pool中有20GB内存用不了，这时候可以共享给其他pool，其他每个pool拿多少，就是由权重决定的
(2)fair也会出现，均衡不均衡是相对的，只不多这两个参数可以缓解。
```

