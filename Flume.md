# Flume

### 1、flume定义

```
1、Flume是Cloudera提供的一个高可用的，高可靠的，分布式的海量日志采集、聚合和传输的系统。（分布式日志采集系统）
2、Flume基于流式架构，灵活简单。

主要作用是可以实时读取本地磁盘数据，写入hdfs
```

### 2、flume组成

```
1、一个jvm进程agent
2、包含三个线程：source、channel、sink
3、传输数据的单元是event
```

agent

```
jvm进程：将数据以事件的形式从源头传向目的地
```

source

```
接受数据，类型包括：avro、thrift、exec、jms、spooling directory、netcat、sequence generator、syslog、http、legacy。

exec : 适合监听一个实时追加的文件，不能实现断点续传
spooling:适合监听同步一个目录下的新文件，
taildir:适合监听一个目录下多个实时追加文件，支持断点续传
```

taildir source

```
（1）断点续传、多目录
（2）哪个flume版本产生的？Apache1.7、CDH1.6
（3）没有断点续传功能时怎么做的？ 自定义
（4）taildir挂了怎么办？
     不会丢数：断点续传
     重复数据：
（5）怎么处理重复数据？
     不处理：生产环境通常不处理，因为会影响传输效率
     处理
         1）自身：在taildirsource里面增加自定义事务
         2）找兄弟：下一级处理（hive dwd sparkstreaming flink布隆）、去重手段（groupby、开窗取窗口第一条、redis）
（6）taildir source 是否支持递归遍历文件夹读取文件？
     不支持。  自定义  递归遍历文件夹 +读取文件
```

channel

```
缓冲区:允许source 和 sink运行在不同的速率上，是线程安全的
自带的两个：memory channel(基于内存，速度快) 和 file channel(基于磁盘，速度慢)，前者可能会丢失数据，后者不会


file channel 磁盘，可靠，速度慢，默认100万条event，可以dataDirs指向多个硬盘，增加吞吐量
memory channel 内存，不可靠，速度快，默认100条event
kafka channel 磁盘，数据在kafka中，省去了sink阶段，比memory还快，1.6产生，bug是自动增加数据头，需要清洗数据，1.7解决了


下一级是kafka，优先选用kafka channel
金融等数据，file channel
日志等数据，memory channel
```

sink

```
轮询channel中事件，并批量移除，然后批量写入其他存储或索引系统，类型包括：hdfs、logger、avro、thrift、ipc、file、HBase、solr、自定义
```

event

```
数据传输单元：包含header 和 body两部分，前者存放数据属性k-v键值对，后者存放数据的字节数组
```

### 3、flume安装

```
1、上传安装包apache-flume-1.9.0-bin.tar.gz至linux系统中/opt/software文件夹下

2、解压安装包至/opt/module文件夹下，改名为flume

3、删除lib文件夹下的guava-11.0.2.jar来兼容hadoop3.1.3
rm /opt/module/flume/lib/guava-11.0.2.jar

4、检查jdk和hadoop环境变量配置正确
#JAVA_HOME
export JAVA_HOME=/opt/module/jdk1.8.0_212
export PATH=$PATH:$JAVA_HOME/bin
#HADOOP_HOME
export HADOOP_HOME=/opt/module/hadoop-3.1.3
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
```

### 4、flume简单案例

#### 案例1、监听端口输入

sources.type = netcat

```
1、安装netcat:		sudo yum install -y nc

2、检查端口占用情况：		sudo netstat -tunlp | grep 44444

3、编写配置文件flume-netcat-logger.conf：	vim /opt/module/flume/job/flume-netcat-logger.conf
添加内容如下：
# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = netcat
a1.sources.r1.bind = localhost
a1.sources.r1.port = 44444

# Describe the sink
a1.sinks.k1.type = logger

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1

4、启动agent进程进行监听：bin/flume-ng agent -c conf/ -n a1 -f job/flume-netcat-logger.conf -Dflume.root.logger=INFO,console

5、测试新开窗口向本机端口44444发送数据：		nc localhost 44444


说明：将端口接受到的数据及时打印至控制台
```

#### 案例2、监控单个文件

sources.type = exec 不能断点续传

```
1、编写配置文件：vim /opt/module/flume/job/flume-file-hdfs.conf
# Name the components on this agent
a2.sources = r2
a2.sinks = k2
a2.channels = c2

# Describe/configure the source
a2.sources.r2.type = exec
a2.sources.r2.command = tail -F /home/atguigu/upload/abc.txt
a2.sources.r2.shell = /bin/bash -c

# Describe the sink
a2.sinks.k2.type = hdfs
a2.sinks.k2.hdfs.path = hdfs://hadoop102:9820/flume/%Y%m%d/%H
#上传文件的前缀
a2.sinks.k2.hdfs.filePrefix = logs-
#是否按照时间滚动文件夹
a2.sinks.k2.hdfs.round = true
#多少时间单位创建一个新的文件夹
a2.sinks.k2.hdfs.roundValue = 1
#重新定义时间单位
a2.sinks.k2.hdfs.roundUnit = hour
#是否使用本地时间戳
a2.sinks.k2.hdfs.useLocalTimeStamp = true
#积攒多少个Event才flush到HDFS一次
a2.sinks.k2.hdfs.batchSize = 100
#设置文件类型，可支持压缩
a2.sinks.k2.hdfs.fileType = DataStream
#多久生成一个新的文件
a2.sinks.k2.hdfs.rollInterval = 60
#设置每个文件的滚动大小
a2.sinks.k2.hdfs.rollSize = 134217700
#文件的滚动与Event数量无关
a2.sinks.k2.hdfs.rollCount = 0

# Use a channel which buffers events in memory
a2.channels.c2.type = memory
a2.channels.c2.capacity = 1000
a2.channels.c2.transactionCapacity = 100

# Bind the source and sink to the channel
a2.sources.r2.channels = c2
a2.sinks.k2.channel = c2

2、启动agent进程：bin/flume-ng agent --conf conf/ --name a2 --conf-file job/flume-file-hdfs.conf -Dflume.root.logger=INFO,console

3、启动hdfs:start-dfs.sh

4、在hadoop集群上查看：hadoop102:9870

说明：将文件的新内容及时传入集群
```

#### 案例3、监控目录

sources.type = spooldir 同步一个目录下的新文件，不适合对追加文件进行监听

```
1、创建配置文件：vim /opt/module/flume/job/flume-dir-hdfs.conf
a3.sources = r3
a3.sinks = k3
a3.channels = c3

# Describe/configure the source
a3.sources.r3.type = spooldir
a3.sources.r3.spoolDir = /home/atguigu/upload
a3.sources.r3.fileSuffix = .COMPLETED
#忽略所有以.tmp结尾的文件，不上传
a3.sources.r3.ignorePattern = ([^ ]*\.tmp)

# Describe the sink
a3.sinks.k3.type = hdfs
a3.sinks.k3.hdfs.path = hdfs://hadoop102:9820/flume/upload/%Y%m%d/%H
#上传文件的前缀
a3.sinks.k3.hdfs.filePrefix = upload-
#是否按照时间滚动文件夹
a3.sinks.k3.hdfs.round = true
#多少时间单位创建一个新的文件夹
a3.sinks.k3.hdfs.roundValue = 1
#重新定义时间单位
a3.sinks.k3.hdfs.roundUnit = hour
#是否使用本地时间戳
a3.sinks.k3.hdfs.useLocalTimeStamp = true
#积攒多少个Event才flush到HDFS一次
a3.sinks.k3.hdfs.batchSize = 100
#设置文件类型，可支持压缩
a3.sinks.k3.hdfs.fileType = DataStream
#多久生成一个新的文件
a3.sinks.k3.hdfs.rollInterval = 60
#设置每个文件的滚动大小大概是128M
a3.sinks.k3.hdfs.rollSize = 134217700
#文件的滚动与Event数量无关
a3.sinks.k3.hdfs.rollCount = 0

# Use a channel which buffers events in memory
a3.channels.c3.type = memory
a3.channels.c3.capacity = 1000
a3.channels.c3.transactionCapacity = 100

# Bind the source and sink to the channel
a3.sources.r3.channels = c3
a3.sinks.k3.channel = c3

2、启动agent线程：bin/flume-ng agent --conf conf/ --name a3 --conf-file job/flume-dir-hdfs.conf -Dflume.root.logger=INFO,console

3、测试，在目录/home/atguigu/upload下创建三个文件：
touch atguigu.txt
touch atguigu.tmp
touch atguigu.log

4、查看hadoop集群：hadoop102:9870

说明：将目录下的新文件及时传入集群
```

#### 多个文件断点续传

sources.type = taildir 适合监听多个实时追加的文件，且支持断点续传

```
1、编写配置文件：vim /opt/module/flume/job/flume-taildir-hdfs.conf
a3.sources = r3
a3.sinks = k3
a3.channels = c3

# Describe/configure the source
a3.sources.r3.type = TAILDIR
#指明断点续传文件读取位置所存储的文件位置
a3.sources.r3.positionFile = /opt/module/flume/tail_dir.json

a3.sources.r3.filegroups = f1 f2
a3.sources.r3.filegroups.f1 = /opt/module/flume/files/.*file.*
a3.sources.r3.filegroups.f2 = /opt/module/flume/files/.*log.*

# Describe the sink
a3.sinks.k3.type = hdfs
a3.sinks.k3.hdfs.path = hdfs://hadoop102:9820/flume/upload2/%Y%m%d/%H
#上传文件的前缀
a3.sinks.k3.hdfs.filePrefix = upload-
#是否按照时间滚动文件夹
a3.sinks.k3.hdfs.round = true
#多少时间单位创建一个新的文件夹
a3.sinks.k3.hdfs.roundValue = 1
#重新定义时间单位
a3.sinks.k3.hdfs.roundUnit = hour
#是否使用本地时间戳
a3.sinks.k3.hdfs.useLocalTimeStamp = true
#积攒多少个Event才flush到HDFS一次
a3.sinks.k3.hdfs.batchSize = 100
#设置文件类型，可支持压缩
a3.sinks.k3.hdfs.fileType = DataStream
#多久生成一个新的文件
a3.sinks.k3.hdfs.rollInterval = 60
#设置每个文件的滚动大小大概是128M
a3.sinks.k3.hdfs.rollSize = 134217700
#文件的滚动与Event数量无关
a3.sinks.k3.hdfs.rollCount = 0

# Use a channel which buffers events in memory
a3.channels.c3.type = memory
a3.channels.c3.capacity = 1000
a3.channels.c3.transactionCapacity = 100

# Bind the source and sink to the channel
a3.sources.r3.channels = c3
a3.sinks.k3.channel = c3

2、启动agent进程：bin/flume-ng agent --conf conf/ --name a3 --conf-file job/flume-taildir-hdfs.conf -Dflume.root.logger=INFO,console

3、测试，创建目录files：mkdir /home/atguigu/flume/files
新建文件，echo hello >> file1.txt，echo atguigu >> file2.txt
echo hello >> f1.file，echo atguigu >> f2.log

4、集群查看：hadoop102:9870

说明：监控文件并实现断点续传
```

### 5、flume事务

#### put事务

```
doput拉数据进入临时缓冲区putlist
docommit先去检查channel中有没有空位置，有的话就推送数据进入channel，没有的话等dorollback回滚数据
dorollback:channals内存不足，回滚数据
```

#### take事务

```
dotake从channel拉数据进入临时缓冲区takelist
docommit检查数据是否发送成功（万一hdfs挂了没有成功），如果成功推送数据进入其他系统，就清空takelist,否则将数据重新放回channel
dorollback:发送数据出现异常，将数据重新放入channals
```

#### agent原理

```
1、source接受数据
2、channel阶段，经过拦截器后放回channel，经过选择器后放回channel，
3、进入sinkprocessor
4、进入sink
```

#### 拦截器

```
自带的：
Timestamp Interceptor(时间戳拦截器)
Host Interceptor(主机拦截器)
静态拦截器(Static Interceptor)
正则过滤拦截器(Regex Filtering Interceptor)

自定义：
1、实现interceptor
2、重写四个方法：initialize 初始化，intercept(Event event)方法 处理单个event，intercept(List<Event> events)方法 处理多个方法，close方法
3、静态内部类：Builder


说明：下一级处理，如在hive的dwd层，sparkSteaming里处理，影响性能，不适合做实时性比较高的场合
```

#### 选择器

```
1、replicating(复制)：发送到所有的channel

2、multiplexing(多路复用):根据不同原则发往不同的channel
```



### 6、flume agent进程原理

```
1.source接收数据发给channel processor

2.拦截器加header（这里会有一个拦截器组）

3.channel selector 根据header分拣数据发送给channel Processor（Replicating（复制）和Multiplexing（多路复用），前者会发送到所有的channal,后者才会选择性发送）

4.channel Process根据分拣器分配结果分配数据进入不同channel

5.进入sinkprocessor

6.一个sink对应一个channel


1）ChannelSelector
ChannelSelector的作用就是选出Event将要被发往哪个Channel。其共有两种类型，分别是Replicating（复制）和Multiplexing（多路复用）。
ReplicatingSelector会将同一个Event发往所有的Channel，Multiplexing会根据相应的原则，将不同的Event发往不同的Channel。

2）SinkProcessor
SinkProcessor共有三种类型，分别是DefaultSinkProcessor、LoadBalancingSinkProcessor和FailoverSinkProcessor
DefaultSinkProcessor对应的是单个的Sink，LoadBalancingSinkProcessor和FailoverSinkProcessor对应的是Sink Group，LoadBalancingSinkProcessor可以实现负载均衡的功能，FailoverSinkProcessor可以错误恢复的功能。
```



### 7、Flume拓扑结构及案例

#### 串联

```
source-channel-sink-source-channel-sink-系统

说明：优点缓冲区多，缺点稳定性差
```

#### 并联

```
source-channel1-sink-系统3
	  -channel2-sink-系统2
	  -channel3-sink-系统1
```

#### 负载均衡和故障转移

```
source-channel-sink-source-channel1-sink-系统1
			  -sink-source-channel2-sink-系统1
			  -sink-source-channel3-sink-系统1
```

#### 聚合

```
source1-channel1-sink-source-channel-系统1
source2-channel2-sink-source-channel-系统1
source3-channel3-sink-source-channel-系统1

说明：常用
```

#### 案例1：并联

使用Flume-1监控文件变动，Flume-1将变动内容传递给Flume-2，Flume-2负责存储到HDFS。同时Flume-1将变动内容传递给Flume-3，Flume-3负责输出到Local FileSystem。

​					->f2->hdfs

file->f1

​					->f3->local file

```
准备工作：
1、创建配置文件的目录g1:mkdir /opt/module/flume/job/g1
2、创建输出到本地文件系统的文件位置：mkdir /opt/module/datas/flume
3、创建要监控的文件：touch ~/abc.txt

配置文件：
1、输入段：vim flume-file-flume.conf		直接从监控文件读取数据，并分发给所有channel
# Name the components on this agent
a1.sources = r1
a1.sinks = k1 k2
a1.channels = c1 c2
# 将数据流复制给所有channel
a1.sources.r1.selector.type = replicating

# Describe/configure the source
a1.sources.r1.type = exec
a1.sources.r1.command = tail -F /opt/module/flume/abc.txt
a1.sources.r1.shell = /bin/bash -c

# Describe the sink
# sink端的avro是一个数据发送者
a1.sinks.k1.type = avro
a1.sinks.k1.hostname = hadoop102 
a1.sinks.k1.port = 4141

a1.sinks.k2.type = avro
a1.sinks.k2.hostname = hadoop102
a1.sinks.k2.port = 4142

# Describe the channel
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

a1.channels.c2.type = memory
a1.channels.c2.capacity = 1000
a1.channels.c2.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1 c2
a1.sinks.k1.channel = c1
a1.sinks.k2.channel = c2

2、输出段1：vim flume-flume-hdfs.conf	输出到hdfs
# Name the components on this agent
a2.sources = r1
a2.sinks = k1
a2.channels = c1

# Describe/configure the source
# source端的avro是一个数据接收服务
a2.sources.r1.type = avro
a2.sources.r1.bind = hadoop102
a2.sources.r1.port = 4141

# Describe the sink
a2.sinks.k1.type = hdfs
a2.sinks.k1.hdfs.path = hdfs://hadoop102:9820/flume/%Y%m%d/%H
#上传文件的前缀
a2.sinks.k1.hdfs.filePrefix = flume-
#是否按照时间滚动文件夹
a2.sinks.k1.hdfs.round = true
#多少时间单位创建一个新的文件夹
a2.sinks.k1.hdfs.roundValue = 1
#重新定义时间单位
a2.sinks.k1.hdfs.roundUnit = hour
#是否使用本地时间戳
a2.sinks.k1.hdfs.useLocalTimeStamp = true
#积攒多少个Event才flush到HDFS一次
a2.sinks.k1.hdfs.batchSize = 100
#设置文件类型，可支持压缩
a2.sinks.k1.hdfs.fileType = DataStream
#多久生成一个新的文件
a2.sinks.k1.hdfs.rollInterval = 600
#设置每个文件的滚动大小大概是128M
a2.sinks.k1.hdfs.rollSize = 134217700
#文件的滚动与Event数量无关
a2.sinks.k1.hdfs.rollCount = 0

# Describe the channel
a2.channels.c1.type = memory
a2.channels.c1.capacity = 1000
a2.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a2.sources.r1.channels = c1
a2.sinks.k1.channel = c1

3、输出段2：vim flume-flume-dir.conf	 	输出到本地
# Name the components on this agent
a3.sources = r1
a3.sinks = k1
a3.channels = c2

# Describe/configure the source
a3.sources.r1.type = avro
a3.sources.r1.bind = hadoop102
a3.sources.r1.port = 4142

# Describe the sink
a3.sinks.k1.type = file_roll
a3.sinks.k1.sink.directory = /opt/module/flume/datas

# Describe the channel
a3.channels.c2.type = memory
a3.channels.c2.capacity = 1000
a3.channels.c2.transactionCapacity = 100

# Bind the source and sink to the channel
a3.sources.r1.channels = c2
a3.sinks.k1.channel = c2

测试：在hadoop102上依次启动flume-flume-dir，flume-flume-hdfs，flume-file-flume进程
bin/flume-ng agent --conf conf/ --name a3 --conf-file job/g1/flume-flume-dir.conf -Dflume.root.logger=INFO,console

bin/flume-ng agent --conf conf/ --name a2 --conf-file job/g1/flume-flume-hdfs.conf -Dflume.root.logger=INFO,console

bin/flume-ng agent --conf conf/ --name a1 --conf-file job/g1/flume-file-flume.conf -Dflume.root.logger=INFO,console

查看：echo hello >> abc.txt 后在集群和本地查看
```

#### 案例2：聚合

hadoop102上

```
source是exec监控文件，sink是zvro序列化

配置文件如下
  1 # Name the components on this agent
  2 a1.sources = r1
  3 a1.sinks = k1
  4 a1.channels = c1
  5 
  6 # Describe/configure the source
  7 a1.sources.r1.type = exec
  8 a1.sources.r1.command = tail -F /opt/module/flume/abc.txt
  9 a1.sources.r1.shell = /bin/bash -c
 10 
 11 # Describe the sink
 12 a1.sinks.k1.type = avro
 13 a1.sinks.k1.hostname = hadoop104
 14 a1.sinks.k1.port = 4141
 15 
 16 # Describe the channel
 17 a1.channels.c1.type = memory
 18 a1.channels.c1.capacity = 1000
 19 a1.channels.c1.transactionCapacity = 100
 20 
 21 # Bind the source and sink to the channel
 22 a1.sources.r1.channels = c1
 23 a1.sinks.k1.channel = c1

```

hadoop103上

```
source是netcat，sink是zvro

配置文件如下
# Name the components on this agent
a2.sources = r1
a2.sinks = k1
a2.channels = c1

# Describe/configure the source
a2.sources.r1.type = netcat
a2.sources.r1.bind = hadoop103
a2.sources.r1.port = 44444
# Describe the sink
a2.sinks.k1.type = avro
a2.sinks.k1.hostname = hadoop104
a2.sinks.k1.port = 4141

# Use a channel which buffers events in memory
a2.channels.c1.type = memory
a2.channels.c1.capacity = 1000
a2.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a2.sources.r1.channels = c1
a2.sinks.k1.channel = c1

```

hadoop104上

```
source是avro,sink是logger

配置文件如下
#Name the components on this agent
a3.sources = r1
a3.sinks = k1
a3.channels = c1

# Describe/configure the source
a3.sources.r1.type = avro
a3.sources.r1.bind = hadoop104
a3.sources.r1.port = 4141

# Describe the sink
# Describe the sink
a3.sinks.k1.type = logger

# Describe the channel
a3.channels.c1.type = memory
a3.channels.c1.capacity = 1000
a3.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a3.sources.r1.channels = c1
a3.sinks.k1.channel = c1

```

```
说明：先启动下游hadoop104上的agent，再启动hadoop102上agent进行测试，再启动hadoop103上的agent进行测试
```



### 8、flume监控：ganglia

安装

```
1、三台机器安装：sudo yum install -y epel-release

2、三台机器安装：sudo yum -y install ganglia-gmond

3、hadoop102安装：sudo yum -y install ganglia-gmetad
				 sudo yum -y install ganglia-web


说明：
1、gmond（Ganglia Monitoring Daemon）是一种轻量级服务，安装在每台需要收集指标数据的节点主机上。使用gmond，你可以很容易收集很多系统指标数据，如CPU、内存、磁盘、网络和活跃进程的数据等。
2、gmetad（Ganglia Meta Daemon）整合所有信息，并将其以RRD格式存储至磁盘的服务。
3、gweb（Ganglia Web）Ganglia可视化工具，gweb是一种利用浏览器显示gmetad所存储数据的PHP前端。在Web界面中以图表方式展现集群的运行状态下收集的多种不同指标数据。
```

部署即编写配置文件

```
hadoop102上的配置：
1、sudo vim /etc/httpd/conf.d/ganglia.conf
<Location /ganglia>
  #Require local 将下面的修改为自己主机在vmnet8网络中的ip地址
  Require ip 192.168.5.1
  # Require ip 10.1.2.3
  # Require host example.org
</Location>

2、sudo vim /etc/ganglia/gmetad.conf
data_source "hadoop102" hadoop102

3、sudo vim /etc/ganglia/gmond.conf
cluster {
#修改name
  name = "hadoop102"
  owner = "unspecified"
  latlong = "unspecified"
  url = "unspecified"
}
udp_send_channel {
  #bind_hostname = yes # Highly recommended, soon to be default.
                       # This option tells gmond to use a source address
                       # that resolves to the machine's hostname.  Without
                       # this, the metrics may appear to come from any
                       # interface and the DNS names associated with
                       # those IPs will be used to create the RRDs.
  # mcast_join = 239.2.11.71
#增加host=hadoop102
  host = hadoop102
  port = 8649
  ttl = 1
}
udp_recv_channel {
  # mcast_join = 239.2.11.71
  port = 8649
#更改bind为0.0.0.0
  bind = 0.0.0.0
  retry_bind = true
  # Size of the UDP buffer. If you are handling lots of metrics you really
  # should bump it up to e.g. 10MB or even higher.
  # buffer = 10485760
}
4、sudo vim /etc/selinux/config		说明：必须重启生效
#修改为disabled
SELINUX=disabled

临时生效则执行：sudo setenforce 0

hadoop103 和hadoop104上配置sudo vim /etc/ganglia/gmond.conf（同hadoop102）


```

启动

```
[atguigu@hadoop102 flume]$ sudo systemctl start httpd
[atguigu@hadoop102 flume]$ sudo systemctl start gmetad
[atguigu@hadoop102 flume]$ sudo systemctl start gmond
[atguigu@hadoop103 flume]$ sudo systemctl start gmond
[atguigu@hadoop104 flume]$ sudo systemctl start gmond

[atguigu@hadoop102 flume]$ bin/flume-ng agent \
--conf conf/ \
--name a1 \
--conf-file job/flume-netcat-logger.conf \
-Dflume.root.logger==INFO,console \
-Dflume.monitoring.type=ganglia \
-Dflume.monitoring.hosts=Hadoop102:8649

nc localhost 44444
```

查看

```
http://hadoop102/ganglia
```

说明

| 字段（图表名称）      | 字段含义                            |
| --------------------- | ----------------------------------- |
| EventPutAttemptCount  | source尝试写入channel的事件总数量   |
| EventPutSuccessCount  | 成功写入channel且提交的事件总数量   |
| EventTakeAttemptCount | sink尝试从channel拉取事件的总数量。 |
| EventTakeSuccessCount | sink成功读取的事件的总数量          |
| StartTime             | channel启动的时间（毫秒）           |
| StopTime              | channel停止的时间（毫秒）           |
| ChannelSize           | 目前channel中事件的总数量           |
| ChannelFillPercentage | channel占用百分比                   |
| ChannelCapacity       | channel的容量                       |

### 9、编写source、selector、sink

source

```

```

selector

```

```

sink

```

```

### 10、面试题

#### 4.1 你是如何实现Flume数据传输的监控的

使用第三方框架Ganglia实时监控Flume。

#### 4.2 Flume的Source，Sink，Channel的作用？你们Source是什么类型？

**1****）作用**

（1）Source组件是专门用来收集数据的，可以处理各种类型、各种格式的日志数据，包括avro、thrift、exec、jms、spooling directory、netcat、sequence generator、syslog、http、legacy

（2）Channel组件对采集到的数据进行缓存，可以存放在Memory或File中。

（3）Sink组件是用于把数据发送到目的地的组件，目的地包括Hdfs、Logger、avro、thrift、ipc、file、Hbase、solr、自定义。

**2****）我公司采用的Source类型为：**

（1）监控后台日志：exec

（2）监控后台产生日志的端口：netcat

Exec spooldir

#### 4.3 Flume的Channel Selectors

![img](file:///C:/Users/o/AppData/Local/Temp/msohtmlclip1/01/clip_image002.png)

#### 4.4 Flume参数调优

**1****）Source**

增加Source个（使用Tair Dir Source时可增加FileGroups个数）可以增大Source的读取数据的能力。例如：当某一个目录产生的文件过多时需要将这个文件目录拆分成多个文件目录，同时配置好多个Source 以保证Source有足够的能力获取到新产生的数据。

batchSize参数决定Source一次批量运输到Channel的event条数，适当调大这个参数可以提高Source搬运Event到Channel时的性能。

**2****）Channel** 

type 选择memory时Channel的性能最好，但是如果Flume进程意外挂掉可能会丢失数据。type选择file时Channel的容错性更好，但是性能上会比memory channel差。

使用file Channel时dataDirs配置多个不同盘下的目录可以提高性能。

Capacity 参数决定Channel可容纳最大的event条数。transactionCapacity 参数决定每次Source往channel里面写的最大event条数和每次Sink从channel里面读的最大event条数。**transactionCapacity需要大于Source和Sink的batchSize参数。**

**3****）Sink** 

增加Sink的个数可以增加Sink消费event的能力。Sink也不是越多越好够用就行，过多的Sink会占用系统资源，造成系统资源不必要的浪费。

batchSize参数决定Sink一次批量从Channel读取的event条数，适当调大这个参数可以提高Sink从Channel搬出event的性能。

#### 4.5 Flume的事务机制

Flume的事务机制（类似数据库的事务机制）：Flume使用两个独立的事务分别负责从Soucrce到Channel，以及从Channel到Sink的事件传递。比如spooling directory source 为文件的每一行创建一个事件，一旦事务中所有的事件全部传递到Channel且提交成功，那么Soucrce就将该文件标记为完成。同理，事务以类似的方式处理从Channel到Sink的传递过程，如果因为某种原因使得事件无法记录，那么事务将会回滚。且所有的事件都会保持到Channel中，等待重新传递。

#### 4.6 Flume采集数据会丢失吗?

根据Flume的架构原理，Flume是不可能丢失数据的，其内部有完善的事务机制，Source到Channel是事务性的，Channel到Sink是事务性的，因此这两个环节不会出现数据的丢失，唯一可能丢失数据的情况是Channel采用memoryChannel，agent宕机导致数据丢失，或者Channel存储数据已满，导致Source不再写入，未写入的数据丢失。

Flume不会丢失数据，但是有可能造成数据的重复，例如数据已经成功由Sink发出，但是没有接收到响应，Sink会再次发送数据，此时可能会导致数据的重复。