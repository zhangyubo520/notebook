# 	ZooKeeper

### 一、概念

#### 1、介绍

```
1、开源分布式

2、观察者模式

3、等于文件系统+通知机制

4、特点：
有一个leader和多个follower，
半数机制（集群节点为奇数个最后）
每个节点数据相同
更新原子性，要么成功，要么失败
实时性，实时更新数据

5、数据结构：类似Linux的文件系统

6、应用：统一命名服务，统一配置管理，统一集群管理、服务器动态上下线、软负载均衡
```

#### 2、安装

```
1、需要jdk
2、拷贝安装包至/opt/software下
3、解压至/opt/module下
4、配置修改：
文件改名字：mv zoo_sample.cfg zoo.cfg
修改配置：vim zoo.cfg修改dataDir=/opt/module/zookeeper-3.5.7/zkData
mkdir zkData

5、启动：bin/zkServer.sh start

6、查看：jps回车后显示：81093 QuorumPeerMain

7、停止：bin/zkServer.sh stop

8、查看状态：bin/zkServer.sh status

9、启动客户端：bin/zkCli.sh

10、退出客户端：bin/zkCli.sh

```

#### 3、分布式安装

```
1、三台（最小集群）安装jdk

2、在一台上安装zookeeper至/opt/module下（名字为zookeeper）

3、在conf文件夹下配置zoo.cfg,（没有的话创建）
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
 # do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=/opt/module/zookeeper/zkData
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
4lw.commands.whitelist=*
server.2=hadoop102:2888:3888
server.3=hadoop103:2888:3888
server.4=hadoop104:2888:3888
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.

4、创建文件夹/opt/module/zookeeper/zkData，（之前有的删除）

5、创建文件myid:touch /opt/module/zookeeper/zkData/myid,(文件内容为2)

6、分发zookeeper-3.5.7：sh myditru.sh /opt/module/zookeeper

7、编写群起群停脚本：vim ~/bin/myzookeeper(内容如下：) 
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
echo "开始启动 $i 的zookeeper"
ssh $i /opt/module/zookeeper/bin/zkServer.sh start
;;
"status")
echo "$i 的zookeeper状态如下："
ssh $i /opt/module/zookeeper/bin/zkServer.sh status
;;
"stop")
echo "开始关闭 $i 的zookeeper"
ssh $i /opt/module/zookeeper/bin/zkServer.sh stop
;;
*)
echo "参数错误"
;;
esac
done
8、提升脚本运行权限：chmod 744 myzookeeper

9、启动：sh myzookeeper start

10、状态查看：sh myzookeeper status

11、停止：sh myzookeeper stop
```

#### 4、客户端命令

```
1、进入某个客户端：bin/zkCli.sh

2、查看所有命令：help

3、查看节点下子节点：ls /

4、查看节点下子节点及自身信息 ls -s /

5、创建普通节点：create /chensiqi "xiaotiantian"

6、创建带序列节点：create -s /zhangxiaofu "男子汉"

7、创建临时节点：create -e /yongqiang "自库伦"

8、创建带序列的临时节点：create -s -e /yongxin "yongganjianqing"

9、删除空节点 delete /zhangyubo

10、删除非空节点： deleteall /zhangyubo

11、节点的值：get /zhangyubo

12、节点详细的值：get -s /zhangyubo

13、监听节点变化：get -w /zhangyubo(仅能监听一次修改)

14、设置节点的值：set "zhangyubo"

15、节点状态：stat /zhangyubo
```

#### 5、节点介绍

```
4种：
持久普通节点：关闭集群再重启不会消失，节点名后没有序号

持久序号节点：~不会消失，~有序号

短暂普通节点：~会消失，~无序号

短暂序号节点：~会消失，~有序号

节点默认最多可以存储1M数据
```

#### 6、选举机制

集群为新开，没有数据的情况下

```
情况1、如果集群半数以上同时启动，每个机器给自己投一票，紧接着互相交换选票，机器中序号最大的即为leader,其他（包括后起的）均为follower

情况2、半数及一下同时启动，每个机器给自己投一票，紧接着互相先换选票，序号最大的得到全部选票，之后每开启一个机器就和序号最大的交换选票，序号大的得到全部选票，判断是否达到半数以上，若达到，得到全部选票的即为leader,其他（包括后起的）均为follower
```

集群不是新开，存在数据的情况下，

```
情况1：数据是同步的，每个机器自己给自己投一票，紧接着交换选票，机器中序号最大的即为leader,其他均为follower

情况2：有一个存在最新数据：即为leader

情况3：有最新数据（版本相同，无法直接确定leader)：按照情况1的原则在他们中选择一个为leader

```



#### 7、在idea中使用

```
1、创建maven module

2、设置pom.xml
<dependencies>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>RELEASE</version>
		</dependency>
		<dependency>
			<groupId>org.apache.logging.log4j</groupId>
			<artifactId>log4j-core</artifactId>
			<version>2.8.2</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/org.apache.zookeeper/zookeeper -->
		<dependency>
			<groupId>org.apache.zookeeper</groupId>
			<artifactId>zookeeper</artifactId>
			<version>3.5.7</version>
		</dependency>
</dependencies>

3、src/main/resources下设置log4j.properties
log4j.rootLogger=INFO, stdout  
log4j.appender.stdout=org.apache.log4j.ConsoleAppender  
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout  
log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m%n  
log4j.appender.logfile=org.apache.log4j.FileAppender  
log4j.appender.logfile.File=target/spring.log  
log4j.appender.logfile.layout=org.apache.log4j.PatternLayout  
log4j.appender.logfile.layout.ConversionPattern=%d %p [%c] - %m%n

4、创建包和类、测试

```

#### 8、常用API测试

1、创建客户端

```
private String connection = "hadoop102:2181,hadoop103:2181,hadoop104:2181";//zookeeper运行节点

    private int sessionTime = 10000;//节点通信的相应时间

    private ZooKeeper zooKeeper = null;//客户端对象

    @Before
    public void init() throws IOException {
        zooKeeper = new ZooKeeper(connection, sessionTime, new Watcher() {
            public void process(WatchedEvent watchedEvent) {
                System.out.println(watchedEvent.getType() + "-----" + watchedEvent.getPath());
            }
        });
    }

    @After
    public void close() throws InterruptedException {
        zooKeeper.close();
    }
```

测试创建节点

```
@Test
    public void testCreate() throws KeeperException, InterruptedException {
        String nodeName = zooKeeper.create("/zhangyubo",//节点名
                "紫面昆仑侠".getBytes(),//节点内容
                ZooDefs.Ids.OPEN_ACL_UNSAFE, //节点访问权限
                CreateMode.CONTAINER);//节点是否是临时的

        System.out.println(nodeName);
    }
```

测试获取子节点并监控

```
@Test
    public void testGetChildren() throws KeeperException, InterruptedException {

        List<String> children = zooKeeper.getChildren("/zhangyubo", true);

        for (String child : children) {
            System.out.println(child);
        }

        Thread.sleep(Long.MAX_VALUE);
    }
```

测试节点是否存在

```
@Test
    public void testExist() throws KeeperException, InterruptedException {

        Stat stat = zooKeeper.exists("/zhangyubo/chensiqi", false);

        if(stat == null){
            System.out.println("不存在");
        }else{
            System.out.println(String.valueOf(stat));
        }
    }
```

测试删除节点

```
    @Test
    public void testDelete() throws KeeperException, InterruptedException {

        String path = "/zhangyubo/chensiqi";

        Stat stat = zooKeeper.exists(path,false);

        if(stat != null){
            zooKeeper.delete(path,stat.getVersion());
        }
    }
```



#### 9、写数据流程

```
1、客户端写请求到某节点

2、如果Server1不是Leader，那么Server1会把接受到的请求进一步转发给
Leader，因为每个ZooKeeper的Server里面有一个是Leader。这个Leader 会将写
请求广播给各个Server,比如Server 1和Server2，各个Server会将该写请 求加入待
写队列，并向Leader发送成功信息。

3、当Leader收到半数以上Server 的成功信息（包含它自己），说明该写操作可以执行。
Leader会向各个Server发送提交信息，各个Server收到信息后会落实队列里的写
请求，此时写成功。

4、leader通知客户端已经写入成功
```

#### 10、Paxos算法

```

```

#### 11、面试题

```
1、选举机制：

2、ZooKeeper的监听原理：

3、ZooKeeper的部署方式有哪几种？集群中的角色有哪些？集群最少需要几台机器？

（1）部署方式单机模式、集群模式
（2）角色：Leader和Follower
（3）集群最少需要机器数：3

4、ZooKeeper的常用命令

ls create get delete set…
```

#### 12、监听原理

```
1、一个main()线程
2、创建zookeeper客户端对象，这时会启动两个线程，一个用来监听，一个用来通信
3、通过通信线程将监听事件发送给zookeeper
4、zookeeper将监听事件加入自己的监听事件表中
5、zookeeper监听到数据或路径变化，将这个消息发送给监听线程
6、监听线程调用process()方法记录消息，采取相应的措施，例如更新服务器列表等



常见的监听：
1、监听节点数据变化
get -w path

2、监控节点数量变化
ls -w path
```

#### 13、配置文件参数介绍

```

```

#### 14、回调函数

```

```

#### 15、安装台数

```
台数多：可靠性高，但通讯延迟高

10台服务器，3个zk
20台服务器，5个zk
50台服务器，7个zk
100台服务器，11个zk
```

#### 16、应用场景

```
1、hadoop高可用集群中的namenode的脑裂问题解决（保证同一时刻只有一个namenode）

2、HBase，使用ZK的事件处理确保整个集群只有一个HMaster，觉察HRegionServer联机和宕机，存储访问控制列表

3、Kafka中使用zk管理broker

4、命名管理、分布式锁、集群管理
```

