# Hadoop

### 一、概念

#### 1、关于Hadoop

```
1、大数据：IT行业术语，是指无法在一定时间范围内用常规软件工具进行捕捉、管理和处理的数据集合，是需要新处理模式才能具有更强的决策力、洞察发现力和流程优化能力的海量、高增长率和多样化的信息资产。

2、大数据特点：大量、高速、多样、低价值密度

3、hadoop:Hadoop是Apache基金会开发的分布式系统基础架构，用来解决数据的存储和海量数据的分析计算问题

4、Hadoop三大发行版本：
Apache版本最原始（最基础）的版本，对于入门学习最好。
官网地址：http://hadoop.apache.org/releases.html
下载地址：https://archive.apache.org/dist/hadoop/common/

Cloudera内部集成了很多大数据框架。对应产品CDH。
官网地址：https://www.cloudera.com/downloads/cdh/5-10-0.html
下载地址：http://archive-primary.cloudera.com/cdh5/cdh/5/

Hortonworks文档较好。对应产品HDP。
官网地址：https://hortonworks.com/products/data-center/hdp/
下载地址：https://hortonworks.com/downloads/#data-platform

5、Hadoop优点：高可靠性	、高扩展性、高效性、高容错性

6、Hadoop1.x组成：HDFS、MapReduce、Common

7、Hadoop2.x组成：HDFS、MapReduce、Yarn、Common

8、HDFS组成：NameNode和DataNode、SecondaryNameNode

9、Yarn组成：ResourceManager、NodeManager、ApplicationMaster、Container

10、MapReduce组成：Map、Reduce

11、ResourceManager的作用：处理客户端请求
```

#### 2、ResourceManager

```
作用：
1、处理客户端请求
2、监控NodeManager
3、启动或监控ApplicationMaster
4、资源的分配和调度
```

#### 3、NodeManager

```
作用：
1、管理单个节点上的资源
2、处理来自ResourceManager的命令
3、处理来自ApplicationManager的命令
```

#### 4、ApplicationManager

```
作用：
1、负责数据的切分
2、为应用程序申请资源并分配内部的任务
3、任务的监控和容错
```

#### 5、container

```
作用：
封装了YARN中的资源抽象，封装了某个节点的多个资源：内存、CPU、磁盘、网络等
```

#### 6、大数据技术生态体系

```
1）Sqoop：Sqoop是一款开源的工具，主要用于在Hadoop、Hive与传统的数据库（MySql）间进行数据的传递，可以将一个关系型数据库（例如 ：MySQL，Oracle 等）中的数据导进到Hadoop的HDFS中，也可以将HDFS的数据导进到关系型数据库中。
2）Flume：Flume是一个高可用的，高可靠的，分布式的海量日志采集、聚合和传输的系统，Flume支持在日志系统中定制各类数据发送方，用于收集数据； 
3）Kafka：Kafka是一种高吞吐量的分布式发布订阅消息系统； 
4）Storm：Storm用于“连续计算”，对数据流做连续查询，在计算时就将结果以流的形式输出给用户。
5）Spark：Spark是当前最流行的开源大数据内存计算框架。可以基于Hadoop上存储的大数据进行计算。
6）Flink：Flink是当前最流行的开源大数据内存计算框架。用于实时计算的场景较多。
7）Oozie：Oozie是一个管理Hdoop作业（job）的工作流程调度管理系统。
8）Hbase：HBase是一个分布式的、面向列的开源数据库。HBase不同于一般的关系数据库，它是一个适合于非结构化数据存储的数据库。
9）Hive：Hive是基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张数据库表，并提供简单的SQL查询功能，可以将SQL语句转换为MapReduce任务进行运行。 其优点是学习成本低，可以通过类SQL语句快速实现简单的MapReduce统计，不必开发专门的MapReduce应用，十分适合数据仓库的统计分析。
10）ZooKeeper：它是一个针对大型分布式系统的可靠协调系统，提供的功能包括：配置维护、名字服务、分布式同步、组服务等。
```

#### 7、环境准备jdk+Hadoop

```
1、克隆
2、修改克隆机ip：vim /etc/sysconfig/network-scripts/ifcfg-ens33
3、修改主机名：vim /etc/hostname
4、配置主机名和ip映射：vim /etc/hosts
5、配置window系统中的主机名和ip映射：vim C:\Windows\System32\drivers\etc\hosts
6、关闭防火墙：sudo systemctl disable firewalld
7、创建atguigu用户：sudo useradd atguigu    		sudo passwd atguigu
8、提升atguigu用户权限：vim /etc/sudoers 添加atguigu   ALL=(ALL)  NOPASSWD:ALL
9、在/opt目录下创建两个文件夹sudo mkdir module      sudo mkdir software
10、修改上面两个文件夹为atguigu所有：sudo chown atguigu:atguigu /opt/module /opt/software
11、安装jdk和Hadoop
```

#### 8、安装jdk

```
1、上传jdk安装包至/opt/software文件夹下

2、解压：tar -zxvf jdk-8u212-linux-x64.tar.gz -C /opt/module/

3、配置环境变量：在/etc/profile.d目录新建my_env.sh,vim /etc/profile/my_env.sh,添加如下内容：
#JAVA_HOME
export JAVA_HOME=/opt/module/jdk1.8.0_212
export PATH=$PATH:$JAVA_HOME/bin

4、source /etc/profile

5、检查是否安装成功：java -version

说明：需要先卸载原有的jdk
rpm -qa | grep jdk 查找
sudo rpm -e --nodeps + 版本号 卸载
其次创建文件夹
sudo mkdir /opt/module /opt/software
sudo chown atguigu:atguigu /opt/module /opt/software
```

#### 9、安装Hadoop

```
1、将Hadoop安装包上传至/opt/software下

2、解压：tar -zxvf hadoop-3.1.3.tar.gz -C /opt/module

3、配置环境变量：vim /etc/profile.d/my_env.sh 添加如下内容
#Hadoop
export HADOOP_HOME=/opt/module/hadoop-3.1.3
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

4、source /etc/profile

5、检查是否成功：Hadoop version

说明：两个环境变量可以合起来写如下：
#JAVA_HOME
export JAVA_HOME=/opt/module/jdk1.8.0_212
#Hadoop
export HADOOP_HOME=/opt/module/hadoop-3.1.3
export PATH=$PATH:$HADOOP_HOME/bin:$JAVA_HOME/bin:$HADOOP_HOME/sbin
```

#### 10、Hadoop目录结构

```
bin:存放的是对Hadoop相关服务(如HDFS/YARN)进行操作的脚本

etc:配置文件

lib:Hadoop的本地库(对数据进行压缩解压缩的功能)

sbin:启动和停止Hadoop相关服务的脚本

share:存放Hadoop依赖的jar包，文档、和官方案例
```

#### 11、本地模式

```
1、创建input目录：mkdir /opt/module/hadoop-3.1.3/input

2、准备一个文件放入input：touch /opt/module/hadoop-3.1.3/input/exercise.txt

3、执行share目录下的MapReduce命令：

bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar grep input output 'dfs[a-z.]+'

4、查看运行结果
cat output/*

1、创建wcinput目录

2、创建文件wc.input,有内容的

3、运行：
bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar wordcount wcinput wcoutput

4、查看结果：cat wcoutput/part-r-00000
```

#### 10、伪分布

```
1、环境配置：
进入/opt/module/hadoop-3.1.3/etc
a、配置hadoop-env.sh,  添加 export JAVA_HOME=/opt/module/jdk1.8.0_212

b、配置core-site.xml,  添加 

<!-- 指定HDFS中NameNode的地址 -->
<property>
<name>fs.defaultFS</name>
    <value>hdfs://hadoop101:9820</value>
</property>

<!-- 指定Hadoop运行时产生文件的存储目录 -->
<property>
	<name>hadoop.tmp.dir</name>
	<value>/opt/module/hadoop-3.1.3/data/tmp</value>
</property>

c、配置hdfs-site.xml,  添加

<!-- 指定HDFS副本的数量 -->
<property>
	<name>dfs.replication</name>
	<value>1</value>
</property>

d、配置yarn-site.xml,  添加

<!-- Reducer获取数据的方式 -->
<property>
 	<name>yarn.nodemanager.aux-services</name>
 	<value>mapreduce_shuffle</value>
</property>

<!-- 指定YARN的ResourceManager的地址 -->
<property>
	<name>yarn.resourcemanager.hostname</name>
	<value>hadoop101</value>
</property>

<property>
	<name>yarn.nodemanager.env-whitelist</name>        				<value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
</property>

e、配置mapred-site.xml,  添加
<!-- 指定MR运行在YARN上 -->
<property>
	<name>mapreduce.framework.name</name>
	<value>yarn</value>
</property>

2、格式化：bin/hdfs namenode -format(注意先先删除原先的data目录和logs目录)

3、启动namenode，datanode，ResourceManager,NodeManager：
bin/hdfs --daemon start namenode
bin/hdfs --daemon start datanode
bin/yarn --daemon start resourcemanager
bin/yarn --daemon start nodemanager

4、检查是否成功：jps(jdk中的命令)
42945 NameNode
47138 ResourceManager
44517 DataNode
47898 NodeManager
48351 Jps

5、web端查看：
查看mapreduce：http://hadoop101:9870
查看yarn:http://hadoop101:8088
```

#### 11、完全分布

```
一、多台机器，规划：
hadoop102 :  NameNode           DataNode  NodeManager

hadoop103 :  ResourceManager    DataNode  NodeManger
 
hadoop104 :  SecondaryNameNode  DataNode  NodeManager

二、配置文件：
1、hadoop-env.sh（指定使用的jdk路径）
添加：export JAVA_HOME=/opt/module/jdk1.8.0_212(配置过的就不用了)

2、core-site.sh（指定namenode的地址，指定数据存储的地址）
添加：
<!-- 指定NameNode的地址 -->
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://hadoop102:9820</value>
	</property>
	
<!-- 指定hadoop数据的存储目录   
      官方配置文件中的配置项是hadoop.tmp.dir ,用来指定hadoop数据的存储目录,此次配置用的hadoop.data.dir是自己定义的变量， 因为在hdfs-site.xml中会使用此配置的值来具体指定namenode 和 datanode存储数据的目录-->
    <property>
        <name>hadoop.data.dir</name>
        <value>/opt/module/hadoop-3.1.3/data</value>
	</property>

<!-- 下面是兼容性配置，先跳过 -->
<!-- 配置该atguigu(superUser)允许通过代理访问的主机节点 -->
    <property>
        <name>hadoop.proxyuser.atguigu.hosts</name>
        <value>*</value>
	</property>
<!-- 配置该atguigu(superuser)允许代理的用户所属组 -->
    <property>
        <name>hadoop.proxyuser.atguigu.groups</name>
        <value>*</value>
	</property>
<!-- 配置该atguigu(superuser)允许代理的用户-->
    <property>
        <name>hadoop.proxyuser.atguigu.users</name>
        <value>*</value>
    </property>
<!-- 使web端有权限增删文件-->
    <property>
        <name>hadoop.http.staticuser.user</name>
        <value>atguigu</value>
	</property>
    
3、hdfs-site.sh（）
添加：
<!-- 指定NameNode数据的存储目录 -->
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file://${hadoop.data.dir}/name</value>
  </property>
  
<!-- 指定Datanode数据的存储目录 -->
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file://${hadoop.data.dir}/data</value>
  </property>
    
<!-- 指定SecondaryNameNode数据的存储目录 -->
  <property>
    <name>dfs.namenode.checkpoint.dir</name>
    <value>file://${hadoop.data.dir}/namesecondary</value>
  </property>
   
   <!-- 兼容配置，先跳过 -->
  <property>
    <name>dfs.client.datanode-restart.timeout</name>
    <value>30s</value>
  </property>

<!-- nn web端访问地址-->
<property>
  <name>dfs.namenode.http-address</name>
  <value>hadoop102:9870</value>
</property>
<!-- 2nn web端访问地址-->
  <property>
    <name>dfs.namenode.secondary.http-address</name>
    <value>hadoop104:9868</value>
  </property>

4、yarn-site.sh
添加：
<property>
     <name>yarn.nodemanager.aux-services</name>
     <value>mapreduce_shuffle</value>
</property>

<!-- 指定ResourceManager的地址-->
<property>
     <name>yarn.resourcemanager.hostname</name>
     <value>hadoop103</value>
</property>

<!-- 环境变量的继承 -->
<property>
     <name>yarn.nodemanager.env-whitelist</name><value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>
    
5、mapred-site.sh
添加：
<!--指定MapReduce程序运行在Yarn上 -->
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>

三、单点启动，单点停止
1、格式化：hdfs namenode -format（没有格式化需要先格式化，如果再次格式化请删除data和logs）
2、启动namenode：在102上，hdfs --daemon start namenode
3、启动secondarynamenode：在104上，hdfs --daemon start namenode
4、启动resourcemanager：在103上，yarn --daemon start resourcemanager
5、启动datanode和nodemanager：在三台机器上分别，hdfs --daemon start datanode 
yarn --daemon start nodemanager
6、jps查看启动是否成功
停止把上面的start换成stop即可

四、群起群停
1、格式化：hdfs namenode -format（没有格式化需要先格式化，如果再次格式化请删除data和logs）

2、设置免密登录

3、设置works
进入：vim /opt/module/hadoop-3.1.3/etc/hadoop/workers
添加：
hadoop102
hadoop103
hadoop104
说明：其他东西删除，无空行，无空格，每个主机都得配置（其实所有配置文件三台机器保持一致）

4、启动
102上：start-dfs.sh
103上：start-yarn.sh
说明：停止使用stop

5、web访问：http://hadoop102:9870
```

#### 12、免密登录

```
查看：进入~/.ssh目录查看

以102免密登录102、103和104为例：
在102上：
1、生成公私钥
ssh-keygen -t rsa
说明：敲（三个回车），就会生成两个文件id_rsa（私钥）、id_rsa.pub（公钥）

2、分发拷贝(在102上)
ssh-copy-id hadoop102
ssh-copy-id hadoop103
ssh-copy-id hadoop104

在103上：
1、生成公私钥
ssh-keygen -t rsa
说明：敲（三个回车），就会生成两个文件id_rsa（私钥）、id_rsa.pub（公钥）

2、分发拷贝（在103上）
ssh-copy-id hadoop102
ssh-copy-id hadoop103
ssh-copy-id hadoop104

在104上：
1、生成公私钥
ssh-keygen -t rsa
说明：敲（三个回车），就会生成两个文件id_rsa（私钥）、id_rsa.pub（公钥）

2、分发拷贝（在104上）
ssh-copy-id hadoop102
ssh-copy-id hadoop103
ssh-copy-id hadoop104
```

#### 13、历史服务器

```
1、配置：
vi mapred-site.xml
<!-- 历史服务器端地址 -->
<property>
    <name>mapreduce.jobhistory.address</name>
    <value>hadoop102:10020</value>
</property>

<!-- 历史服务器web端地址 -->
<property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>hadoop102:19888</value>
</property>
2、分发配置
3、在102上启动：mapred --daemon start historyserver
4、查看jps
```

#### 14、日志聚集

```
1、配置
vim yarn-site.xml
添加：
<!--增加日志聚集功能 -->
<property>
        <name>yarn.log-aggregation-enable</name>
        <value>true</value>
    </property>
    <property>  
        <name>yarn.log.server.url</name>  
        <value>http://hadoop102:19888/jobhistory/logs</value>  
    </property>
    <property>
        <name>yarn.log-aggregation.retain-seconds</name>
        <value>604800</value>
</property>
2、分发配置
3、关闭启动NodeManager 、ResourceManager和HistoryServer
4、删除输出文件
5、查看：http://hadoop102:19888/jobhistory
```

#### 15、课堂笔记

```
远程同步数据脚本:
scp  rsync  都可以实现从 server1 到server2数据的拷贝

脚本实现的效果:
在hadoop102执行脚本，可以将指定要拷贝的数据，拷贝到103  和 104 

脚本使用 :

xsync  文件/目录

脚本存放的位置:

我们希望脚本可以在很方便的在任意位置运行，因此脚本需要放到一个PATH环境变量所能指向的一个目录中.

当前系统PATH变量如下:
/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/atguigu/.local/bin:/home/atguigu/bin

因为我们使用的是atguigu用户， 因此可以把脚本放到 /home/atguigu/bin目录下。




完全分布式集群规划: (按照3个副本来说)
1个NameNode  3个DataNode  1个SecondaryNameNode   1个ResourceManager  3个NodeManager

理论情况: 需要有6个机器

实际情况: 3台机器

因为NameNode 、SecondaryNameNode 、 ResourceManager运行中需要的资源比较多，因此分布到不同的节点中. 

hadoop102 :  NameNode           DataNode  NodeManager

hadoop103 :  ResourceManager    DataNode  NodeManger
 
hadoop104 :  SecondaryNameNode  DataNode  NodeManager


完全分布式启动以后的情况:
[atguigu@hadoop102 hadoop-3.1.3]$ jps
16728 DataNode
24907 NodeManager
15231 NameNode

[atguigu@hadoop103 hadoop]$ jps

71969 ResourceManager
68487 DataNode
72975 NodeManager

[atguigu@hadoop104 hadoop]$ jps
72936 NodeManager
69160 SecondaryNameNode
68478 DataNode



单个节点启动集群的解决方案:   [群起]




集群格式化问题:

1. 集群是否需要每次都格式化？ 
   不需要. 正常情况下， 一个新配置好的集群需要格式化，后续就不要再进行格式化操作。
   除非整个集群的数据都不要了，集群遇到严重的问题，需要重新搭建，等搭建好后需要格式化.

2. 如果要重新格式化集群需要注意什么问题?

   如果要重新格式化需要删除  data目录  和  logs目录 。

   如果不删除 , 重新格式化会生成新的集群id, 而DN记录的还是之前的集群id
   ,当DN启动以后找不到NN,然后DN直接下线.

   DN启动起来以后，会自动找NN进行注册. 




作业: 

1. 完全分布式最少搭2遍
   a. 准备3台机器(修改主机名 修改ip等。 具体看克隆虚拟机操作)
   b. 在3台机器上安装jdk和hadoop ,可按照课堂操作，直接从101 拷贝到 102 103 104 
   c. 编写分发脚本 
   d. 参照文档在102 配置 core-site.xml   hdfs-site.xml  yarn-site.xml  mapred-site.xml
   e. 将配置好的 xxx-site.xml 分发到103  104  ， 保证集群中 102 103 104 的配置文件内容同步
   f. 配置环境变量， 可单独在102 103 104 重新手动配置 ，在熟悉一下环境变量的配置
      也可以按照课堂操作， 在102 配置好， 通过scp的方式复制到103  104 (需要注意使用root)
   g. 启动集群 
      hdfs --daemon  start/stop  xxx    
      yarn --daemon  start/stop  xxx

   h. 启动成功后jps查看， web端查看.

   注意: 在搭建过程中一定要仔细，特别是配置文件不能配错， 该做的操作都要做，不要丢三落四.
         遇到问题注意思考，看日志

```

#### 16、Hadoop 3.x 和2.x主要区别

了解：

```
1)最低Java版本从7升级到8
2)引入纠删码(Erasure Coding)
主要解决数据量大到一定程度磁盘空间存储能力不足的问题.
HDFS中的默认3副本方案在存储空间中具有200%的额外开销。但是，对于I/O活动相对较少冷数据集，在正常操作期间很少访问其他块副本，但仍然会消耗与第一个副本相同的资源量。 
纠删码能勾在不到50%数据冗余的情况下提供和3副本相同的容错能力，因此，使用纠删码作为副本机制的改进是自然而然，也是未来的趋势.
3)重写了Shell脚本
重写了Shell脚本，修改了之前版本长期存在的一些错误，并提供了一些新功能,在尽可能保证兼容性的前提下，一些新变化仍然可能导致之前的安装出现问题。
例如: 
所有Hadoop Shell脚本子系统现在都会执行hadoop-env.sh这个脚本，它允许所有环节变量位于一个位置；
守护进程已通过*-daemon.sh选项从*-daemon.sh移动到了bin命令中，在Hadoop3中，我们可以简单的使用守护进程来启动、停止对应的Hadoop系统进程；
4)引入了新的API依赖
之前Hadoop客户端操作的Maven依赖为hadoop-client，这个依赖直接暴露了Hadoop的下级依赖，当用户和Hadoop使用相同依赖的不同版本时，可能造成冲突。
Hadoop3.0引入了提供了hadoop-client-api 和hadoop-client-runtime依赖将下级依赖隐藏起来，一定程度上来解决依赖冲突的问题 

5)MapReduce任务的本地化优化
MapReduce引入了一个NativeMapOutputCollector的本地化(C/C++)实现，对于shuffle密集的任务，可能提高30%或者更高的性能 
6)支持超过两个NN
HDFS NameNode高可用性的初始实现为单个Active NameNode  和 单个 Standby NameNode, 将edits复制到三个JournalNode。 该体系结构能够容忍系统中一个NN或者一个JN故障.但是，某些部署需要更高程序的容错能力，Hadoop3.x允许用户运行一个Active NameNode 和 多个 Standby NameNode。 
7)许多服务的默认端口改变了
Hadoop3.x之前，多个Hadoop服务的默认端口位于Linux临时端口范围(63768~61000). 这意味着在启动时，由于与另一个应用程序冲突，服务有时无法绑定到端口.
在Hadoop3.x中，这些可能冲突的端口已移出临时范围，受影响的有NameNode ,
SecondaryNamenode , DataNode 和 KMS 
8)添加对Microsoft Azure Data Lake  和 阿里云对象存储系统的支持
9)DataNode内部实现Balancer
一个DN管理多个磁盘，当正常写入时，多个磁盘是平均分配的。然而当添加新磁盘时，这种机制会造成DN内部严重的倾斜。
之前的DataNode Balancer只能实现DN之间的数据平衡，Hadoop3.x实现了内部的数据平衡。 
10)重做的后台和任务堆内存管理
已实现根据服务器自动配置堆内存，HADOOP_HEAPSIZE变量失效。简化MapTask 和ReduceTask的堆内存配置，现已不必同时在配置中和Java启动选项中指定堆内存大小，旧有配置不会受到影响。 
11)HDFS实现服务器级别的Federation分流
对于HDFS Federation， 添加了一个对统一命名空间的RPC路由层 。 和原来的HDFS Federation没有变化，只是目前挂在管理不必在客户端完成，而是放在的服务器，从而简化了HDFS Federation访问。
12)Yarn的时间线服务升级到V2
 Yarn的时间线服务是MRJobHistory的升级版，提供了在Yarn上运行第三方程序的历史支持，该服务在Hadoop3.0升级为第二版

13)容量调度器实现API级别的配置
现在容量调度器可以实现通过REST API来改变配置，从而让管理员可以实现调度器自动配置。
14)Yarn实现更多种资源类型的管理
Yarn调度器现已可以通过配置实现用户自定义的资源管理。现在Yarn可以根据CPU和内存意外的资源管理其任务队列
```

#### 17、时间同步

```
在102上：
1、关闭ntp服务并设置非自启动（三台）
sudo systemctl stop ntpd
sudo systemctl disable ntpd
2、配置
vim /etc/ntp.conf
restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap去掉注释
server 0.centos.pool.ntp.org iburst    注释掉
server 1.centos.pool.ntp.org iburst    注释掉
server 2.centos.pool.ntp.org iburst    注释掉
server 3.centos.pool.ntp.org iburst    注释掉
server 127.127.1.0					   添加
fudge 127.127.1.0 stratum 10           添加

vim /etc/sysconfig/ntpd
SYNC_HWCLOCK=yes                       添加

3、重启ntp服务并设置自启动
systemctl start ntpd
systemctl enable ntpd

其他机器上：
编写定时任务
crontab -e
*/10 * * * * /usr/sbin/ntpdate hadoop102
```

#### 18、服务器之间拷贝配置文件

```
一、使用scp 待拷贝的文件 目的地
1、例如:拷贝本地文件至103下home文件夹下
scp ~/abc.txt atguigu@hadoop103:~/

2、例如:拷贝103上的文件至104中
scp atguigu@hadoop103:~/abc.txt atguigu@hadoop104:~/

3、例如：拷贝104上的文件至本地
scp atguigu@hadoop104:~/abc.txt ~/
说明：如果是文件夹，使用-r参数递归

二、使用rsync 
1、本地文件拷贝至103上
rsync -av ~/bin atguigu@hadoop103:~/
rsync和scp区别：用rsync做文件的复制要比scp的速度快，rsync只对差异文件做更新。scp是把所有文件都复制过去。

三、群发脚本使用xsync
首先应该在~目录下mkdir bin，然后在~\bin目录下创建文档如下，提升权限，然后执行
#!/bin/bash
if [ $# -lt 1 ];then
echo "请输入源文件的绝对路径"
exit;
fi
for host in hadoop103 hadoop104
do
echo 开始向$host传输文件
for file in $@
do
if [ -e $file ];then
parentdir=$(dirname "$file")
rsync -av $file atguigu@$host:$parentdir
else
echo "文件不存在"
fi
done
done


#!/bin/bash
#1. 判断参数个数
if [ $# -lt 1 ]
then
  echo Not Enough Arguement!
  exit;
fi
#2. 遍历集群所有机器
for host in hadoop103 hadoop104
do
  echo ====================  $host  ====================
  #3. 遍历所有目录，挨个发送
  for file in $@
  do
    #4 判断文件是否存在
    if [ -e $file ]
    then
      #5. 获取父目录
      pdir=$(cd -P $(dirname $file); pwd)
      #6. 获取当前文件的名称
      fname=$(basename $file)
      ssh $host "mkdir -p $pdir"
      rsync -av $pdir/$fname $host:$pdir
    else
      echo $file does not exists!
    fi
  done
done



```

#### 19、压缩

MR支持的压缩格式

| 压缩格式 | hadoop自带？ | 算法    | 文件扩展名 | 是否可切分 | 换成压缩格式后，原来的程序是否需要修改 |
| -------- | ------------ | ------- | ---------- | ---------- | -------------------------------------- |
| DEFLATE  | 是，直接使用 | DEFLATE | .deflate   | 否         | 和文本处理一样，不需要修改             |
| Gzip     | 是，直接使用 | DEFLATE | .gz        | 否         | 和文本处理一样，不需要修改             |
| bzip2    | 是，直接使用 | bzip2   | .bz2       | 是         | 和文本处理一样，不需要修改             |
| LZO      | 否，需要安装 | LZO     | .lzo       | 是         | 需要建索引，还需要指定输入格式         |
| Snappy   | 是，直接使用 | Snappy  | .snappy    | 否         | 和文本处理一样，不需要修改             |

| 压缩格式 | 对应的编码/解码器                          |
| -------- | ------------------------------------------ |
| DEFLATE  | org.apache.hadoop.io.compress.DefaultCodec |
| gzip     | org.apache.hadoop.io.compress.GzipCodec    |
| bzip2    | org.apache.hadoop.io.compress.BZip2Codec   |
| LZO      | com.hadoop.compression.lzo.LzopCodec       |
| Snappy   | org.apache.hadoop.io.compress.SnappyCodec  |

| 压缩算法 | 原始文件大小 | 压缩文件大小 | 压缩速度 | 解压速度 |
| -------- | ------------ | ------------ | -------- | -------- |
| gzip     | 8.3GB        | 1.8GB        | 17.5MB/s | 58MB/s   |
| bzip2    | 8.3GB        | 1.1GB        | 2.4MB/s  | 9.5MB/s  |
| LZO      | 8.3GB        | 2.9GB        | 49.3MB/s | 74.6MB/s |

Gzip

```
1、压缩比高，速度快，Hadoop本身支持，大部分Linux也支持Gzip命令，使用方便
2、不支持split
3、压缩后在130M以内都可以使用（一个块大小内）Gzip
```

Bzip2

```
1、高压缩比，支持split,Hadoop本身支持，使用方便
2、压缩和解压缩速度慢
3、对速度没什么要求的
```

LZO

```
1、速度快，压缩比合理，支持split,可在Linux中下载
2、压缩率比Gzip低，需要安装，需要对文件特殊处理（为了建索引，需要在指定InputFormat格式为LZO
3、文件压缩后大于200M,越大优势越明显
```

Snappy

```
1、速度快，压缩率合理，低于Gzip,需要安装，不支持split
2、使用在Mapper和Reducer之间，或者Mapper和Mapper之间
```

使用压缩的位置

```
1、输入端压缩：不用指定压缩形式，自动检测扩展名，使用恰当的进行压缩和解压缩

2、Mapper输出压缩：LZO或者Snappy

3、Reducer输出压缩：
```

参数配置

| 参数                                                         | 默认值                                      | 阶段        | 建议                                          |
| ------------------------------------------------------------ | ------------------------------------------- | ----------- | --------------------------------------------- |
| io.compression.codecs   （在core-site.xml中配置）            |                                             |             | Hadoop使用文件扩展名判断是否支持某种编解码器  |
| mapreduce.map.output.compress（在mapred-site.xml中配置）     | false                                       | mapper输出  | 这个参数设为true启用压缩                      |
| mapreduce.map.output.compress.codec（在mapred-site.xml中配置） | org.apache.hadoop.io.compress.DefaultCodec  | mapper输出  | 企业多使用LZO或Snappy编解码器在此阶段压缩数据 |
| mapreduce.output.fileoutputformat.compress（在mapred-site.xml中配置） | false                                       | reducer输出 | 这个参数设为true启用压缩                      |
| mapreduce.output.fileoutputformat.compress.codec（在mapred-site.xml中配置） | org.apache.hadoop.io.compress. DefaultCodec | reducer输出 | 使用标准工具或者编解码器，如gzip和bzip2       |
| mapreduce.output.fileoutputformat.compress.type（在mapred-site.xml中配置） | RECORD                                      | reducer输出 | SequenceFile输出使用的压缩类型：NONE和BLOCK   |

数据流的压缩和解压缩

```
CompressionCodec有两个方法可以用于轻松地压缩或解压缩数据：

要想对正在被写入一个输出流的数据进行压缩，我们可以使用
createOutputStream(OutputStreamout)方法创建一个CompressionOutputStream, 将
其以压缩格式写入底层的流。

相反，要想对从输入流读取而来的数据进行解压缩，则调用
createInputStream(InputStreamin函数,从而获得一个CompressionInputStream, 从
而从底层的流读取未压缩的数据。
```



#### 20、优化

优化重点：

```
1、计算机性能
2、i/o操作：包括数据倾斜，mapTask和ReduceTask数量设置，小文件过多等
```

输入时

```
合并小文件，使用CombineTextInputFormat作为输入
```

map阶段

```
1、减小溢写次数（spill),通过调整io.sort.mb和sort.spill.percent参数值，来调大触发spill的内存上限，减小i/o

2、增大merge（合并）次数，通过调整io.sort.factor参数，增大merge的文件数目，来减小合并次数，减小i/o

3、不影响业务逻辑的情况下使用combine，进行combine处理，减小i/o
```

reduce阶段

```
1、合理设置reduceTask和MapperTask个数，避免一个处理太长时间

2、设置map和reduce共存，调整slowstart.completedmaps参数，使map运行一定程度后，reduce开始运行，减少reduce等待时间

3、规避使用reduce，因为reduce在连接数据集的时候将花费大量时间，

4、设置buffer，mapReduce.reduce.input.buffer.percent,默认是0，设置大于0时，可以让reduce直接从buffer中读取数据

```

i/o

```
1、采用压缩

2、使用SequenceFile二进制文件
```

减小数据倾斜

```
1、对元数据进行抽样，确定合理的分区范围，

2、使用自定义分区

3、使用Combine

4、使用map join,避免使用reduce join
```

#### 21、参数调优

资源类

以下参数是在用户自己的MR应用程序中配置就可以生效（mapred-default.xml）

| 配置参数                                      | 参数说明                                                     |
| --------------------------------------------- | ------------------------------------------------------------ |
| mapreduce.map.memory.mb                       | 一个MapTask可使用的资源上限（单位:MB），默认为1024。如果MapTask实际使用的资源量超过该值，则会被强制杀死。 |
| mapreduce.reduce.memory.mb                    | 一个ReduceTask可使用的资源上限（单位:MB），默认为1024。如果ReduceTask实际使用的资源量超过该值，则会被强制杀死。 |
| mapreduce.map.cpu.vcores                      | 每个MapTask可使用的最多cpu core数目，默认值: 1               |
| mapreduce.reduce.cpu.vcores                   | 每个ReduceTask可使用的最多cpu core数目，默认值: 1            |
| mapreduce.reduce.shuffle.parallelcopies       | 每个Reduce去Map中取数据的并行数。默认值是5                   |
| mapreduce.reduce.shuffle.merge.percent        | Buffer中的数据达到多少比例开始写入磁盘。默认值0.66           |
| mapreduce.reduce.shuffle.input.buffer.percent | Buffer大小占Reduce可用内存的比例。默认值0.7                  |
| mapreduce.reduce.input.buffer.percent         | 指定多少比例的内存用来存放Buffer中的数据，默认值是0.0        |

应该在YARN启动之前就配置在服务器的配置文件中才能生效（yarn-default.xml）

| 配置参数                                 | 参数说明                                        |
| ---------------------------------------- | ----------------------------------------------- |
| yarn.scheduler.minimum-allocation-mb     | 给应用程序Container分配的最小内存，默认值：1024 |
| yarn.scheduler.maximum-allocation-mb     | 给应用程序Container分配的最大内存，默认值：8192 |
| yarn.scheduler.minimum-allocation-vcores | 每个Container申请的最小CPU核数，默认值：1       |
| yarn.scheduler.maximum-allocation-vcores | 每个Container申请的最大CPU核数，默认值：4       |
| yarn.nodemanager.resource.memory-mb      | 给Containers分配的最大物理内存，默认值：8192    |

Shuffle性能优化的关键参数，应在YARN启动之前就配置好（mapred-default.xml）

| 配置参数                         | 参数说明                          |
| -------------------------------- | --------------------------------- |
| mapreduce.task.io.sort.mb        | Shuffle的环形缓冲区大小，默认100m |
| mapreduce.map.sort.spill.percent | 环形缓冲区溢出的阈值，默认80%     |

容错类

| 配置参数                     | 参数说明                                                     |
| ---------------------------- | ------------------------------------------------------------ |
| mapreduce.map.maxattempts    | 每个Map Task最大重试次数，一旦重试参数超过该值，则认为Map Task运行失败，默认值：4。 |
| mapreduce.reduce.maxattempts | 每个Reduce Task最大重试次数，一旦重试参数超过该值，则认为Map Task运行失败，默认值：4。 |
| mapreduce.task.timeout       | Task超时时间，经常需要设置的一个参数，该参数表达的意思为：如果一个Task在一定时间内没有任何进入，即不会读取新的数据，也没有输出数据，则认为该Task处于Block状态，可能是卡住了，也许永远会卡住，为了防止因为用户程序永远Block住不退出，则强制设置了一个该超时时间（单位毫秒），默认是600000。如果你的程序对每条输入数据的处理时间过长（比如会访问数据库，通过网络拉取数据等），建议将该参数调大，该参数过小常出现的错误提示是“AttemptID:attempt_14267829456721_123456_m_000224_0 Timed out after 300 secsContainer killed by the ApplicationMaster.”。 |

22、小文件处理

```
1)小文件优化的方向：
（1）在数据采集的时候，就将小文件或小批数据合成大文件再上传HDFS。
（2）在业务处理之前，在HDFS上使用MapReduce程序对小文件进行合并。
（3）在MapReduce处理时，可采用CombineTextInputFormat提高效率。
（4）开启uber模式，实现jvm重用
 
2)Hadoop Archive
是一个高效的将小文件放入HDFS块中的文件存档工具，能够将多个小文件打包成一个HAR文件，从而达到减少NameNode的内存使用

3)SequenceFile
SequenceFile是由一系列的二进制k/v组成，如果为key为文件名，value为文件内容，可将大批小文件合并成一个大文件

4)CombineTextInputFormat
CombineTextInputFormat用于将多个小文件在切片过程中生成一个单独的切片或者少量的切片。

5)开启uber模式，实现jvm重用。默认情况下，每个Task任务都需要启动一个jvm来运行，如果Task任务计算的数据量很小，我们可以让同一个Job的多个Task运行在一个Jvm中，不必为每个Task都开启一个Jvm. 

开启uber模式，在mapred-site.xml中添加如下配置
<!--  开启uber模式 -->
<property>
  <name>mapreduce.job.ubertask.enable</name>
  <value>true</value>
</property>

<!-- uber模式中最大的mapTask数量,可向下修改  --> 
<property>
  <name>mapreduce.job.ubertask.maxmaps</name>
  <value>9</value>
</property>

<!-- uber模式中最大的reduce数量，可向下修改 -->
<property>
  <name>mapreduce.job.ubertask.maxreduces</name>
  <value>1</value>
</property>

<!-- uber模式中最大的输入数据量，如果不配置，则使用dfs.blocksize 的值，可向下修改 -->
<property>
  <name>mapreduce.job.ubertask.maxbytes</name>
  <value></value>
</property>

6)配置mapreduce.job.jvm.numtasks 参数实现在一个Jvm中运行多个Task . 如果设置为
-1 ,这没有数量限制。 一般设置在 10-20之间.
```

#### 22、jps脚本

```
#!/bin/bash
for host in hadoop102 hadoop103 hadoop104
do
  echo ====================  $host  ====================
      ssh $host jps
done
```

#### 23、配置支持lzo压缩

编译

```
Hadoop支持LZO

0. 环境准备
maven（下载安装，配置环境变量，修改sitting.xml加阿里云镜像）
gcc-c++
zlib-devel
autoconf
automake
libtool
通过yum安装即可，yum -y install gcc-c++ lzo-devel zlib-devel autoconf automake libtool

1. 下载、安装并编译LZO

wget http://www.oberhumer.com/opensource/lzo/download/lzo-2.10.tar.gz

tar -zxvf lzo-2.10.tar.gz

cd lzo-2.10

./configure -prefix=/usr/local/hadoop/lzo/

make

make install

2. 编译hadoop-lzo源码

2.1 下载hadoop-lzo的源码，下载地址：https://github.com/twitter/hadoop-lzo/archive/master.zip
2.2 解压之后，修改pom.xml
    <hadoop.current.version>3.1.3</hadoop.current.version>
2.3 声明两个临时环境变量
     export C_INCLUDE_PATH=/usr/local/hadoop/lzo/include
     export LIBRARY_PATH=/usr/local/hadoop/lzo/lib 
2.4 编译
    进入hadoop-lzo-master，执行maven编译命令
    mvn package -Dmaven.test.skip=true
2.5 进入target，hadoop-lzo-0.4.21-SNAPSHOT.jar 即编译成功的hadoop-lzo组件
```

将编译好的文件放入hadoop的common目录下

```
cd /opt/module/hadoop-3.1.3/share/hadoop/common

ls
hadoop-lzo-0.4.20.jar

分发

配置文件：core-site.xml
<configuration>
    <property>
        <name>io.compression.codecs</name>
        <value>
            org.apache.hadoop.io.compress.GzipCodec,
            org.apache.hadoop.io.compress.DefaultCodec,
            org.apache.hadoop.io.compress.BZip2Codec,
            org.apache.hadoop.io.compress.SnappyCodec,
            com.hadoop.compression.lzo.LzoCodec,
            com.hadoop.compression.lzo.LzopCodec
        </value>
    </property>

    <property>
        <name>io.compression.codec.lzo.class</name>
        <value>com.hadoop.compression.lzo.LzoCodec</value>
    </property>
</configuration>

分发配置文件
重启集群

手动创建索引
hadoop jar /opt/module/hadoop-3.1.3/share/hadoop/common/hadoop-lzo-0.4.20.jar  com.hadoop.compression.lzo.DistributedLzoIndexer /input/bigtable.lzo

其中bigtable.lzo是压缩后的文件
```



### 二、代码

#### 1、测试数据流压缩

```

```

#### 2、测试Mapper输出压缩

```
// 开启map端输出压缩
	configuration.setBoolean("mapreduce.map.output.compress", true);
	
// 设置map端输出压缩方式
	configuration.setClass("mapreduce.map.output.compress.codec", BZip2Codec.class, CompressionCodec.class);
```

#### 3、测试Reducer输出压缩

```
// 设置reduce端输出压缩开启
		FileOutputFormat.setCompressOutput(job, true);
		
		// 设置压缩的方式
	    FileOutputFormat.setOutputCompressorClass(job, BZip2Codec.class); 
//	    FileOutputFormat.setOutputCompressorClass(job, GzipCodec.class); 
//	    FileOutputFormat.setOutputCompressorClass(job, DefaultCodec.class);
```

