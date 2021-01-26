# HDFS

### 一、概念

#### 1、HDFS简介

```
1、是一个分布式的文件管理系统，通过目录树定位文件
2、适合一次写入，多次读出的场景，不支持文件的修改，适合做数据分析

优点：高容错，大容量，分布式

缺点：不支持并发写入和随机修改，不适合低延时数据访问，对小文件低效
```

#### 2、HDFS架构

```
1、namenode：控制端
 处理客户端读写请求，
 配置副本策略，
 管理数据块映射信息，
 管理HDFS名称空间

2、datanode：执行端
 存储实际的数据块，
 执行数据块的读写操作
 
3、client：客户端
文件切分，
与namenode交互，获取文件位置信息，
与datanode交互，读写数据，
提供一些命令管理HDFS，如格式化namenode操作，
提供一些命令访问HDFS，如对HDFS增删改查操作

4、secondarynamenode：秘书
协助namenode维护元数据：定期合并Fsimage和Edits,推送给namenode,紧急情况下，可部分恢复namenode
```

#### 3、文件块

```
HDFS中文件时分块存储的，块的大小通过配置参数dfs.blocksize来设定，默认是128m,与磁盘读写速度相关，速度越高此数值越大，
```

#### 4、HDFS中shell操作

```
1、启动Hadoop集群
start-dfs.sh
start-yarn.sh

2、-help：输出这个命令参数
hadoop fs -help rm 
查到参数有：（同时提供格式）
-rm [-f] [-r|-R] [-skipTrash] [-safely] <src> ... :
下面是解释：
  Delete all files that match the specified file pattern. Equivalent to the Unix
  command "rm <src>"
                                                                                 
  -f          If the file does not exist, do not display a diagnostic message or 
              modify the exit status to reflect an error.                        
  -[rR]       Recursively deletes directories.                                   
  -skipTrash  option bypasses trash, if enabled, and immediately deletes <src>.  
  -safely     option requires safety confirmation, if enabled, requires          
              confirmation before deleting large directory with more than        
              <hadoop.shell.delete.limit.num.files> files. Delay is expected when
              walking over large directory recursively to count the number of    
              files to be deleted before the confirmation. 

3、-ls: 显示目录信息
例如显示根目录信息：
hadoop fs -ls /
显示如下效果：
drwxr-xr-x   - atguigu supergroup          0 2020-04-11 16:39 /0213
-rw-r--r--   2 atguigu supergroup          0 2020-04-13 10:47 /banzhang.txt

4、-mkdir：在HDFS上创建目录
例如创建chensiqi.sh目录在0213目录下
hadoop fs -mkdir /0213/chensiqi.sh

5、-moveFromLocal：将本地系统磁盘文件剪切到HDFS中
例如将本地根目录下的abc.txt剪切至HDFS中的0213下
hadoop fs -moveFromLocal /abc.txt /0213

6、-copyFromLocal：从本地文件系统中拷贝文件到HDFS路径去
例如将本地根目录下的abc.txt文件上传至HDFS中一份
hadoop fs -copyFromLocal /abc.txt /0213

7、-copyToLocal：从HDFS拷贝到本地
hadoop fs -copyToLocal /0213/abc.txt /

8、-appendToFile：向一个存在的文件的末尾追加内容
hadoop fs -appendToFile ./temp.txt /banzhang.txt

9、-cat:显示文件内容
hadoop fs -cat /banzhang.txt

10、-chgrp 、-chmod、-chown：Linux文件系统中的用法一样，修改文件所属权限
hadoop fs  -chmod  666  /sanguo/shuguo/kongming.txt
hadoop fs  -chown  atguigu:atguigu   /sanguo/shuguo/kongming.txt

11、-cp ：从HDFS的一个路径拷贝到HDFS的另一个路径
hadoop fs -cp /banzhang.txt /0213

12、-mv：在HDFS目录中移动文件
hadoop fs -mv /0213/ab.txt /

13、-get：等同于copyToLocal，就是从HDFS下载文件到本地
hadoop fs -get /ab.txt /

14、-getmerge：合并下载多个文件，比如HDFS的目录 /user/atguigu/test下有多个文件:log.1, log.2,log.3,...
例如将/0213下文件内容都覆盖写入到本地abc.txt文件中
hadoop fs -getmerge /0213 ./abc.txt

15、-put：等同于copyFromLocal
hadoop fs -put ./abc.txt /

16、-tail：显示一个文件的末尾
hadoop fs -tail /ab.txt

17、-rm：删除文件或文件夹
hadoop fs -rm /ab.txt

18、-rmdir：删除空目录
hadoop fs -rmdir /test

19、-du统计文件夹的大小信息
共计：hadoop fs -du -s -h /0213
分类：hadoop fs -du -h /0213

20、-setrep：设置HDFS中文件的副本数量
hadoop fs -setrep 5 /0213
```

#### 5、web端的权限

```
方法1、在core-site.xml中修改http访问的静态用户为atguigu
<property>
        <name>hadoop.http.staticuser.user</name>
        <value>atguigu</value>
</property>

方法2、在hdfs-site.xml中关闭权限检查
<property>
        <name>dfs.permissions.enabled</name>
        <value>false</value>
</property>
```

#### 6、HDFS客户端操作

（在window系统中使用idea操作HDFS)

```
1、安装：将hadoop-3.1.0解压至指定文件夹
2、path环境变量设置：在window系统中配置上面文件夹下bin目录的环境变量
3、cmd检查：hadoop version

4、创建maven工程：
5、设置pom文件：
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

6、在项目的src/main /resources目录下，新建一个文件，命名为“log4j2.xml”，在文件中填入
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
7、建包和类
public class HdfsClient{	
   @Test
    public void testHdfsClient() throws IOException, InterruptedException {
        //1. 创建HDFS客户端对象,实际上是一种流
        FileSystem fileSystem = FileSystem.get(URI.create("hdfs://hadoop102:9820"), new Configuration(), "atguigu");
        //2. 操作集群(参数为new的Path对象)
        fileSystem.mkdirs(new Path("/testHDFS"));
        //3. 关闭资源
        fileSystem.close();
    }
}
8、执行
```

#### 7、参数优先级

```
参数优先级排序：
（1）客户端代码中设置的值 大于（configuration.set("dfs.replication", "2");
（2）ClassPath下的用户自定义配置文件 大于（可以在项目的resources中新建hdfs-site.xml文件，写入<property>
		<name>dfs.replication</name>
        <value>1</value>
</property>）
（3）然后是服务器的自定义配置(xxx-site.xml) 大于（在linux中自定义集群的配置）
（4）服务器的默认配置(xxx-default.xml)（在集群中默认的配置）

代码 > resources中的配置文件 > 自定义配置hsfs-site.xml > 服务器默认配置hdfs-site.xml
```

#### 8、文件上传

```
//上传
    @Test
    public void test3() throws URISyntaxException, IOException, InterruptedException {
        //获取流资源
        FileSystem fs = FileSystem.get(new URI("hdfs://hadoop102:9820"),new Configuration(),"atguigu");
        //使用流资源
        fs.copyFromLocalFile(new Path("f:/hdfs"),new Path("/"));
        //关闭流资源
        fs.close();
    }
```

#### 9、文件下载

```
//下载
    @Test
    public void test4() throws URISyntaxException, IOException, InterruptedException {
        FileSystem fs = FileSystem.get(new URI("hdfs://hadoop102:9820"),new Configuration(),"atguigu");

        fs.copyToLocalFile(new Path("/hdfs"),new Path("f:/"));

        fs.close();
    }
```

#### 10、文件删除

```
//删除
    @Test
    public void test5() throws URISyntaxException, IOException, InterruptedException {
        FileSystem fs = FileSystem.get(new URI("hdfs://hadoop102:9820"),new Configuration(),"atguigu");

        fs.delete(new Path("/hdfs"),true);

        fs.close();
    }
```

#### 11、文件改名或移动

```
//改名
    @Test
    public void test6() throws URISyntaxException, IOException, InterruptedException {
        FileSystem fs = FileSystem.get(new URI("hdfs://hadoop102:9820"),new Configuration(),"atguigu");

        fs.rename(new Path("/0213"),new Path("/0233"));

        fs.close();
    }
    //移动
    @Test
    public void test7() throws URISyntaxException, IOException, InterruptedException {
        FileSystem fs = FileSystem.get(new URI("hdfs://hadoop102:9820"),new Configuration(),"atguigu");

        fs.rename(new Path("/0233/abc.txt"),new Path("/"));

        fs.close();
    }
```

#### 12、文件详情查看

```
//文件详情查看
    @Test
    public void test8() throws URISyntaxException, IOException, InterruptedException {
        FileSystem fs = FileSystem.get(new URI("hdfs://hadoop102:9820"),new Configuration(),"atguigu");

        RemoteIterator<LocatedFileStatus> remoteIterator = fs.listFiles(new Path("/"),true);

        while(remoteIterator.hasNext()){
            LocatedFileStatus status = remoteIterator.next();

            System.out.println(status.getPath().getName());

            System.out.println(status.getGroup());

            System.out.println(status.getBlockLocations());

            System.out.println(status.getPermission());

            System.out.println(status.getLen());
            
        }
    }
```

#### 13、文件目录和文档判断

```
//判断是文档还是目录
    @Test
    public void test9() throws URISyntaxException, IOException, InterruptedException {
        FileSystem fs = FileSystem.get(new URI("hdfs://hadoop102:9820"),new Configuration(),"atguigu");

        FileStatus[] fsArr = fs.listStatus(new Path("/"));

        for(FileStatus status:fsArr){
            System.out.println(status.isFile());
            System.out.println(status.isDirectory());
            System.out.println("**************");
        }
    }
```

#### 14、网络拓扑-节点距离计算

```
节点距离：两个节点到达最近的共同祖先的距离的和。
```

#### 15、HDFS数据流（文件写）

```
1、客户端通过Distributed FileSystem模块向NameNode请求上传文件，NameNode检查目标文件是否已存在，父目录是否存在。
2、namenode返回是否可以上传
3、客户端切分出第一个block，并向namenode请求第一个block放入哪一个datanode服务器
4、namenode返回3个namenode节点，dn1,dn2,dn3
5、客户端通过FSDataOutputStream模块请求dn1上传数据，dn1收到请求会继续调用dn2，然后dn2调用dn3，将这个通信管道建立完成。
6、dn1、dn2、dn3逐级应答客户端。
7、客户端开始往dn1上传第一个Block（先从磁盘读取数据放到一个本地内存缓存），以Packet为单位，dn1收到一个Packet就会传给dn2，dn2传给dn3；dn1每传一个packet会放入一个应答队列等待应答。
8、当一个Block传输完成之后，客户端再次请求NameNode上传第二个Block的服务器。（重复执行3-7步）。

```

#### 16、机架感知

| 集群  |       |       |
| ----- | ----- | ----- |
| 机架1 | 机架2 | 机架3 |
| n-0   | n-0   | n-0   |
| n-1   | n-1   | n-1   |
| n-2   | n-2   | b-2   |

传输局的方式：

```
1、如果客户端在集群内，则在客户端所在机架某个节点存第一份，如果客户端不在集群内，则随机选择一个机架的某个节点存第一份

2、选择不同于第一份所在机架的机架，随机选择一个节点，存第二份

3、选择上一步选择好的机架，再随机选择一个节点，存第三份

说明：三份的话，有一个在一个机架，另外两份在另外一个机架
```

#### 17、HDFS数据流（文件读）

```
1、客户端通过Distributed FileSystem向NameNode请求下载文件，NameNode通过查询元数据，找到文件块所在的DataNode地址。
2、挑选一台DataNode（就近原则，然后随机）服务器，请求读取数据。
3、DataNode开始传输数据给客户端（从磁盘里面读取数据输入流，以Packet为单位来做校验）。
4、客户端以Packet为单位接收，先在本地缓存，然后写入目标文件。
```

#### 18、namenode工作机制

```
维护元数据（镜像文件）和日志文件
1、第一次启动namenode格式化后，创建Fsimage和Edits文件，后续启动时加载Fsimage和Edits到内存中；
2、客户端发来增删改请求；
3、namenode记录操作日志，并更新滚动日志；
4、namenode对内存中的数据进行增删改

```

#### 19、2namenode工作机制

```
帮助namenode将镜像文件和日志合并
1、Secondarynamenode询问namenode是否需要checkpoint,带回请求结果，
2、checkpoint请求执行checkpoint
3、namenode滚动正在写的日志，
4、将滚动前的编辑日志和镜像文件拷贝到secondarynamenode
5、secondarynamenode将编辑日志和镜文件加载到内存，合并
6、将上步处理好的文件命名为fsimage.chkpoint并拷贝给namenode
7、namenode将拷贝过来的fsimage.chkpoint命名为fsimage覆盖老文件
```

#### 20、datanode工作机制

```
1、datanode存储的内容包括：真是block数据,元数据，数据块长度，数据块校验和，时间戳

2、DataNode启动后向NameNode注册，通过后，周期性（1小时）的向NameNode上报所有的块信息。

3、心跳是每3秒一次，心跳返回结果带有NameNode给该DataNode的命令如复制块数据到另一台机器，或删除某个数据块。如果超过10分钟没有收到某个DataNode的心跳，则认为该节点不可用。

4、集群运行中可以安全加入和退出一些机器。
```

#### 21、namenode多目录设置

```
其实没啥用
<property>
    <name>dfs.namenode.name.dir</name>
<value>file:///${hadoop.tmp.dir}/name1,file:///${hadoop.tmp.dir}/name2</value>
</property>
```

#### 22、oiv查看Fsimage文件

```
镜像文件就在设置的data/name里

hdfs oiv -p 文件类型 -i镜像文件 -o 转换后文件输出路径

hdfs oiv -p XML -i fsimage_0000000000000000025 -o /opt/module/hadoop-3.1.3/fsimage.xml

```

#### 23、oev查看Edits文件

```
hdfs oev -p 文件类型 -i编辑日志 -o 转换后文件输出路径

hdfs oev -p XML -i edits_0000000000000000012-0000000000000000013 -o /opt/module/hadoop-3.1.3/edits.xml
```

#### 24、CheckPoint时间设置

```
1、默认一个小时执行一次合并
2、每一分钟还会检查操作次数，超过一百万次时也会执行合并


设置：hdfs-site.xml
<property>
  <name>dfs.namenode.checkpoint.period</name>
  <value>3600</value>
</property>
<property>
  <name>dfs.namenode.checkpoint.txns</name>
  <value>1000000</value>
<description>操作动作次数</description>
</property>

<property>
  <name>dfs.namenode.checkpoint.check.period</name>
  <value>60</value>
<description> 1分钟检查一次操作次数</description>
</property >

```

#### 25、NameNode故障处理

方法一：将SecondaryNameNode中数据拷贝到NameNode存储数据的目录

```
1. kill -9 NameNode进程
2. 删除NameNode存储的数据：rm -rf /opt/module/hadoop-3.1.3/data/tmp/dfs/name/*
3、拷贝SecondaryNameNode中数据到原NameNode存储数据目录：scp -r atguigu@hadoop104:/opt/module/hadoop-3.1.3/data/tmp/dfs/namesecondary/* ./name/
4、重新启动NameNode：hdfs --daemon start namenode
```

方法二：使用-importCheckpoint选项启动NameNode守护进程，从而将SecondaryNameNode中数据拷贝到NameNode目录中。

```
1、修改hdfs-site.xml中的
<property>
  <name>dfs.namenode.checkpoint.period</name>
  <value>120</value>
</property>

<property>
  <name>dfs.namenode.name.dir</name>
  <value>/opt/module/hadoop-3.1.3/data/tmp/dfs/name</value>
</property>

2、kill -9 NameNode进程
3、删除NameNode存储的数据
4、如果SecondaryNameNode不和NameNode在一个主机节点上，需要将SecondaryNameNode存储数据的目录拷贝到NameNode存储数据的平级目录（即和name目录平级），并删除in_use.lock文件（在拷贝过来的namesecondary里）
5、导入检查点数据：bin/hdfs namenode -importCheckpoint
6、等待120秒
7、启动namenode:hdfs --daemon start namenode
```

#### 26、安全模式

```

进入：
namenode启动时，创建新的镜像文件和日志文件，并等待datanode汇报，此期间，namenode处于安全模式，客户端只能读
数据块的信息以块列表的形式存储在DataNode中，datanode启动时，会向namenode发送最新的块列表信息
退出：
满足最小副本条件时（整个文件系统中，99.9%的块满足最小副本级别），30秒后自动退出

手动操作：
（1）bin/hdfs dfsadmin -safemode get		（功能描述：查看安全模式状态）
（2）bin/hdfs dfsadmin -safemode enter  	（功能描述：进入安全模式状态）
（3）bin/hdfs dfsadmin -safemode leave	（功能描述：离开安全模式状态）
（4）bin/hdfs dfsadmin -safemode wait	（功能描述：等待安全模式状态）
```

#### 27、多目录设置

```
namenode多目录：
1、配置hdfs-site.sh
<property>
    <name>dfs.namenode.name.dir</name>
	<value>file:///${hadoop.tmp.dir}/name1,file:///${hadoop.tmp.dir}/name2</value>
</property>

2、重新格式化集群（所以这种操作一般是规划好了的，一次搞定）

datanode多目录：
1、配置hdfs-site.sh
<property>
        <name>dfs.datanode.data.dir</name>
		<value>file:///${hadoop.tmp.dir}/data1,file:///${hadoop.tmp.dir}/data2</value>
</property>

2、重新格式化集群（所以这种操作一般都是提前规划好的，一次搞定）
```

#### 28、曾删新的datanode

```
增：
1、在hadoop104主机上再克隆一台hadoop105主机
2、修改IP地址和主机名称
3、删除原来HDFS文件系统留存的文件（/opt/module/hadoop-3.1.3/data和logs,很重要，需要检查一下到底有没有，有的话必须删除）
4、source一下配置文件：source /etc/profile
5、直接启动DataNode，即可关联到集群：在105上，hdfs --daemon start datanode
6、web端查看：http://hadoop102:9870/
7、平衡数据：	sbin/start-balancer.sh

添加黑白名单：
1、在NameNode的/opt/module/hadoop-3.1.3/etc/hadoop目录下分别创建whitelist 和blacklist文件
2、在whitelist中添加如下主机名称,假如集群正常工作的节点为102 103 104 105
hadoop102
hadoop103
hadoop104
hadoop105
3、编辑blacklist文件，添加105：vim blacklist
4、在NameNode的hdfs-site.xml配置文件中增加dfs.hosts 和 dfs.hosts.exclude配置
<property>
	<name>dfs.hosts</name>
	<value>/opt/module/hadoop-3.1.3/etc/hadoop/whitelist</value>
</property>

<property>
	<name>dfs.hosts.exclude</name>
	<value>/opt/module/hadoop-3.1.3/etc/hadoop/blacklist</value>
</property>
5、分发配置文件
6、重新启动集群

黑名单退役hadoop105：
1、vim blacklist
hadoop105
2、hdfs dfsadmin -refreshNodes
3、退役成功
说明：
添加到白名单的主机节点，都允许访问NameNode，不在白名单的主机节点，都会被直接退出。
添加到黑名单的主机节点，不允许访问NameNode，会在数据迁移后退出。
白名单用于确定允许访问NameNode的DataNode节点，内容配置一般与workers文件内容一致。 黑名单用于在集群运行过程中退役DataNode节点

```

#### 29、小文件

```
1、启动yarn:start-yarn.sh

2、合并小文件：
例如将input下的所有文件合并成input.har的归档文件，存储到output下
bin/hadoop archive -archiveName input.har –p  /user/atguigu/input   /user/atguigu/output

3、查看
hadoop fs -lsr har:///user/atguigu/output/input.har

4、还原
hadoop fs -cp har:/// user/atguigu/output/input.har/*    /user/atguigu
```

#### 30、web端查看无权限的问题

```
1、配置core-site.xml
<property>
        <name>hadoop.http.staticuser.user</name>
        <value>atguigu</value>
</property>
2、分发配置
3、重新停止集群：stop-dfs.sh
4、重新启动集群：start-dfs.sh
5、web端操作
```

31、常用端口号

| Daemon                  | App                           | Hadoop2     | Hadoop3 |
| ----------------------- | ----------------------------- | ----------- | ------- |
| NameNode Port           | Hadoop HDFS NameNode          | 8020 / 9000 | 9820    |
|                         | Hadoop HDFS NameNode HTTP UI  | 50070       | 9870    |
|                         | Hadoop HDFS NameNode HTTPS UI | 50470       | 9871    |
| Secondary NameNode Port | Secondary NameNode HTTP       | 50091       | 9869    |
|                         | Secondary NameNode HTTP UI    | 50090       | 9868    |
| DataNode Port           | Hadoop HDFS DataNode IPC      | 50020       | 9867    |
|                         | Hadoop HDFS DataNode          | 50010       | 9866    |
|                         | Hadoop HDFS DataNode HTTP UI  | 50075       | 9864    |
|                         | Hadoop HDFS DataNode HTTPS UI | 50475       | 9865    |

#### 32、DataNode工作机制

```
1）一个数据块在DataNode上以文件形式存储在磁盘上，包括两个文件，一个是数据本身，一个是元数据包括数据块的长度，块数据的校验和，以及时间戳。

2）DataNode启动后向NameNode注册，通过后，周期性（1小时）的向NameNode上报所有的块信息。

3）心跳是每3秒一次，心跳返回结果带有NameNode给该DataNode的命令如复制块数据到另一台机器，或删除某个数据块。如果超过10分钟没有收到某个DataNode的心跳，则认为该节点不可用。

4）集群运行中可以安全加入和退出一些机器。

```

#### 33、坏块的清除和修复

```
bin/hdfs fsck /

bin/hdfs fsck / -delete

hdfs debug recoverLease -path /hbase/.tmp/hbase-hbck.lock -retries 5

hdfs debug recoverLease -path /spark-history/application_1594643877917_0004_1  
hdfs debug recoverLease -path /tmp/logs/atguigu/logs-tfile/application_1594643877917_0004/hadoop102_38448 
hdfs debug recoverLease -path /tmp/logs/atguigu/logs-tfile/application_1594643877917_0004/hadoop103_45345
hdfs debug recoverLease -path /hbase/oldWALs/hadoop103%2C16020%2C1594728404627.meta.1594739213184.meta
hdfs debug recoverLease -path /hbase/oldWALs/hadoop103%2C16020%2C1594728404627.1594739213206
hdfs debug recoverLease -path /hbase/oldWALs/hadoop104%2C16020%2C1594728404699.1594739213220
hdfs debug recoverLease -path /hbase/oldWALs/hadoop102%2C16020%2C1594728406401.1594739214045
hdfs debug recoverLease -path /hbase/MasterProcWALs/pv2-00000000000000000058.log
hdfs debug recoverLease -path /hbase/.tmp/hbase-hbck.lock

hdfs fsck /spark-history/application_1594643877917_0004_1 -delete
hdfs fsck /tmp/logs/atguigu/logs-tfile/application_1594643877917_0004/hadoop102_38448 -delete
hdfs fsck /tmp/logs/atguigu/logs-tfile/application_1594643877917_0004/hadoop103_45345 -delete
hdfs fsck /hbase/oldWALs/hadoop103%2C16020%2C1594728404627.meta.1594739213184.meta -delete
hdfs fsck /hbase/oldWALs/hadoop103%2C16020%2C1594728404627.1594739213206 -delete
hdfs fsck /hbase/oldWALs/hadoop104%2C16020%2C1594728404699.1594739213220 -delete
hdfs fsck /hbase/oldWALs/hadoop102%2C16020%2C1594728406401.1594739214045 -delete
hdfs fsck /hbase/MasterProcWALs/pv2-00000000000000000058.log -delete
hdfs fsck /hbase/.tmp/hbase-hbck.lock -delete



```

