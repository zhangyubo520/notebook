# yarn

### 一、概念

#### 1、yarn概述

```

1、yarn是一个资源调度平台，负责为运算程序提供服务器运算资源，相当于一个分布式的操作系统平台，

2、而MapReduce等运算程序则相当于运行于操作系统之上的应用程序。

3、YARN主要由ResourceManager、NodeManager、ApplicationMaster和Container等组件构成

```

#### 2、yarn运行机制

```
1、客户端提交任务：job.waitForCompletion()

2、客户端向ResourceManager申请Application

3、ResourceManager返回资源提交路径和application_id

4、客户端提交job运行所需资源：包括job.split,job.xml,主程序类.jar

5、客户端提交资源完毕，向ResourceManager申请MrAppMaster

6、将客户端请求初始化成一个Task放入任务队列中

7、NodeManager领取任务，创建容器Container:包含cpu,ram,MrAppMaster

8、下载job资源到本地：包括job.split,job.xml,主程序类.jar

9、MrAppmaster向ResourceManager申请启动mapTask,

10、有两个NodeManager领取任务，创建了两个Container,

11、MrAppMaster向两个NodeManager发送启动脚本，

12、NodeManager各自运行自己的mapTask,MrAppMaster监控他们运行完毕后，向ResourceManager申请运行ReduceTask的容器，

13、创建ReduceTask容器，拷贝MapTask时产生某个分区的数据，运行

14、MrAppManager监控程序运行完毕后，向ResourceManager注销自己


说明：
1、进度查看：
YARN中的任务将其进度和状态(包括counter)返回给应用管理器, 客户端每秒(通过mapreduce.client.progressmonitor.pollinterval设置)向应用管理器请求进度更新, 展示给用户。

2、作业完成：
除了向应用管理器请求作业进度外, 客户端每5秒都会通过调用waitForCompletion()来检查作业是否完成。时间间隔可以通过mapreduce.client.completion.pollinterval来设置。作业完成之后, 应用管理器和Container会清理工作状态。作业的信息会被作业历史服务器存储以备之后用户核查。
```

#### 3、yarn资源调度器

1、FIFO

```
1、单队列，先进先出，基本不用
2、先进先出
3、大任务可能会浪费资源，
```

2、Capacity Schedule

```
1、apache默认使用，消耗资源少
2、多个队列，以队列为单位分配资源
3、每个队列有最低保证资源，最大限度资源，可设置
4、一个队列的剩余资源可以共享给其他队列，需求来时又可以让其他队列释放借走的资源（优先自己用）
5、单个队列内部是FIFO原则，
6、可动态更新配置文件来动态管理集群
```

3、Fair Scheduler

```
1、多队列多作业，每个队列可以单独配置
2、同一个队列按照优先级分享整个队列的资源，并行运行
3、每个作业可设置最小资源量，
4、cdh默认使用，消耗资源多

说明：
1、Fair Scheduler 可选择按照FIFO、Fair或DRF策略为应用程序分配资源，
2、Fair是默认的，按照最小最大算法实现资源的多路复用方式
3、DRF策略可以考虑CPU和网络等情况，不仅仅是内存
```

4、设置资源调度器

```
默认是Capacity Schedule：
通过配置文件yarn-default.xml配置：

<property>
    
    <description>The class to use as the resource scheduler.</description>
    
    <name>yarn.resourcemanager.scheduler.class</name>
	
	<value>
	org.apache.hadoop.yarn.server.resourcemanager.scheduler.capacity.CapacityScheduler
	</value>
	
</property>
```



#### 4、配置多队列的容量调度器

默认Yarn的配置下，容量调度器只有一条Default队列。在capacity-scheduler.xml中可以配置多条队列，并降低default队列资源占比

默认的任务提交都是提交到default队列

```
<!-- 指定多队列,hive是新增的一条队列的名称 -->
<property>
    <name>yarn.scheduler.capacity.root.queues</name>
    <value>default,hive</value>
</property>

<!-- 指定default队列的额定容量 -->
<property>
    <name>yarn.scheduler.capacity.root.default.capacity</name>
    <value>40</value>
</property>

<!-- 指定hive队列的额定容量 -->
<property>
    <name>yarn.scheduler.capacity.root.hive.capacity</name>
    <value>60</value>
</property>

<!-- 指定default队列允许单用户占用的资源占比 -->
<property>
    <name>yarn.scheduler.capacity.root.default.user-limit-factor</name>
    <value>1</value>
</property>

<!-- 指定hive队列允许单用户占用的资源占比 -->
<property>
    <name>yarn.scheduler.capacity.root.hive.user-limit-factor</name>
    <value>1</value>
</property>

<!-- 指定default队列的最大容量-->
<property>
    <name>yarn.scheduler.capacity.root.default.maximum-capacity</name>
    <value>60</value>
</property>

<!-- 指定hive队列的最大容量-->
<property>
    <name>yarn.scheduler.capacity.root.hive.maximum-capacity</name>
    <value>80</value>
</property>

<!-- 指定default队列的状态 -->
<property>
    <name>yarn.scheduler.capacity.root.default.state</name>
    <value>RUNNING</value>
</property>

<!-- 指定hive队列的状态 -->
<property>
    <name>yarn.scheduler.capacity.root.hive.state</name>
    <value>RUNNING</value>
</property>

<!-- 指定default队列允许哪些用户提交job-->
<property>
    <name>yarn.scheduler.capacity.root.default.acl_submit_applications</name>
    <value>*</value>
</property>

<!-- 指定hive队列允许哪些用户提交job-->
<property>
    <name>yarn.scheduler.capacity.root.hive.acl_submit_applications</name>
    <value>*</value>
</property>

<!-- 指定default队列允许哪些用户进行管理-->
<property>
    <name>yarn.scheduler.capacity.root.default.acl_administer_queue</name>
    <value>*</value>
</property>

<!-- 指定hive队列允许哪些用户进行管理-->
<property>
    <name>yarn.scheduler.capacity.root.hive.acl_administer_queue</name>
    <value>*</value>
</property>

<!-- 指定default队列允许哪些用户提交配置优先级的job-->
<property>       				
	<name>
		yarn.scheduler.capacity.root.default.acl_application_max_priority
	</name>	
    <value>*</value>
</property>

<!--指定hive队列允许哪些用户提交配置优先级的job -->
<property>
  	<name>yarn.scheduler.capacity.root.hive.acl_application_max_priority</name>
  	<value>*</value>
</property>

<!-- 指定default队列允许job运行的最大时间-->
<property>
     <name>
     	yarn.scheduler.capacity.root.default.maximum-application-lifetime
     </name>
     <value>-1</value>
 </property>
 
 <!-- 指定hive队列允许job运行的最大时间-->
 <property>
     <name>
     	yarn.scheduler.capacity.root.hive.maximum-application-lifetime
     </name>
     <value>-1</value>
 </property>
 
<!-- 指定default队列允许job运行的默认时间-->
<property>
     <name>yarn.scheduler.capacity.root.default.default-application-lifetime
     </name>
     <value>-1</value>  
</property>

<!-- 指定hive队列允许job运行的默认时间-->
<property>
     <name>yarn.scheduler.capacity.root.hive.default-application-lifetime
     </name>
     <value>-1</value>
</property>
```

重启yarn完成设置

#### 5、向指定队列比如Hive提交任务

默认的任务提交都是提交到default队列的。如果希望向其他队列提交任务，需要在Driver中声明

```
configuration.set("mapred.job.queue.name", "hive");
```

#### 6、推测执行

条件：

```

（1）每个Task只能有一个备份任务
（2）当前Job已完成的Task必须>=0.05（5%）

```

设置：默认是打开的，mapred-site.xml文件中

```
<property>
  	<name>mapreduce.map.speculative</name>
  	<value>true</value>
  	<description>If true, then multiple instances of some map tasks may be executed in parallel.</description>
</property>

<property>
  	<name>mapreduce.reduce.speculative</name>
  	<value>true</value>
  	<description>If true, then multiple instances of some reduce tasks may be executed in parallel.</description>
</property>
```

不建议使用的情况：

```
（1）任务间存在严重的负载倾斜；
（2）特殊任务，比如任务向数据库中写数据。
```

#### 7、任务队列配置

```
按框架分队列：spark，flink,hive
按业务分队列：登录模块，购物车模块，物流，业务部门1，业务部门2，可配置优先级
```

#### 8、报错处理

```
1、1.2 GB of 1 GB physical memory used; 2.2 GB of 2.1 GB virtual memory used. Killing contain

<property>
     <name>yarn.nodemanager.vmem-check-enabled</name>
     <value>false</value>
</property>
```

