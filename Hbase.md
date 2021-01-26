# Hbase

### 1、概念

```
分布式，可扩展，支持海量数据存储的，noSql数据库；随机读写，海量数据，实时，数十亿行 * 数百万列的大表
```

### 2、逻辑结构

```
列族
region：
row key：行标识，有序，查询时候的依赖
列：列名存储的时候生成
store：存储单元

数据模型：
name space 有点像database，默认有两个default(用户默认使用的表)和hbase(内置的表)
table 表，定义时只需要指明列族即可，不需要指明具体的列
row 每一个行都由一个rowKey和多个column组成，查询数据按照rowKey查询
column 每一个列都由列族和列限定符限定，例如info:name info:age
time stamp 时间戳，每行数据写入时均会加上该字段，用来表示数据的版本，其值为数据的写入时间
cell {rowkey,info:name,time stamp}唯一确定的单元

```

### 3、物理结构

```
TimeStamp：通过时间戳标明数据操作的版本
Type：操作数据的类型：插入，删除，修改
StoreFile：存储在hdfs中
```

### 4、架构

```
Region Server : 对数据进行get,put,delete;对region进行切分，压缩region
```

```
Master： 管理所有Region Server，对表，create，dalete,alter；对Region Server，分配regions到Region Server,监控Region Server 的状态，负载均衡，故障转移
```

```
zookeeper：做master的高可用，Region Server 的监控，元数据的入口，集群配置的维护
```

```
hdfs:底层数据存储，为高可用提供支持
```

### 5、安装部署

```
1、安装并启动zookeeper（三台）执行脚本 zookeeper.sh start 启动

2、Hadoop集群启动：start-dfs.sh start-yarn.sh

3、解压压缩文件 tar -zxvf hbase-2.0.5-bin.tar.gz -C /opt/module/

4、修改名字 mv hbase-2.0.5 hbase

5、增加环境变量 sudo vim /etc/profile.d/my_env.sh
#HBASE_HOME
export HBASE_HOME=/opt/module/hbase
export PATH=$PATH:$HBASE_HOME/bin

6、检查环境变量 source /etc/profile echo $HBASE_HOME

7、配置文件
hbase-env.sh
export HBASE_MANAGES_ZK=false

hbase-site.xml
<configuration>
    <property>
        <name>hbase.rootdir</name>
        <value>hdfs://hadoop102:9820/hbase</value>
    </property>

    <property>
        <name>hbase.cluster.distributed</name>
        <value>true</value>
    </property>

    <property>
        <name>hbase.zookeeper.quorum</name>
        <value>hadoop102,hadoop103,hadoop104</value>
    </property>

    <property>
        <name>hbase.unsafe.stream.capability.enforce</name>
        <value>false</value>
    </property>
    
    <property>
        <name>hbase.wal.provider</name>
        <value>filesystem</value>
    </property>
    <property>
        <name>hbase.master.maxclockskew</name>
        <value>180000</value>
        <description>Time difference of regionserver from master</description>
	</property>
</configuration>

regionservers
hadoop102
hadoop103
hadoop104

8、分发
xsync hbase

9、start-hbase.sh stop-hbase.sh

10、查看http://hadoop102:16010

11、启动客户端：hbase shell

12、高可用配置：
touch conf/backup-masters
echo hadoop103 > conf/backup-masters

分发

启动

查看
```

### 6、底层架构

```
memStore
storeFile
wal
blockcache
```

region server

```
region:一个region server下管理着多个region,一个region下有多个store
store：每个store对应一个列族，包含memstore 和 storefile
storefile：实际的存储的物理文件，以hfile的形式存储在hdfs上，每个store会包含多个storefile，内部有序
memstore：写缓存，数据有序，所以会先写在memstore中，等刷写时机到时候，刷写到storefile中，每次刷写都会形成新的hfile
wal:写缓存的日志文件，防止memstore内存数据丢失，当系统故障时，可以使用wal恢复数据
blockcache：读缓存，每次查询的数据都会缓存在blockcache中，方便下次查询使用
```

写流程

```

```

memstore的刷写时机

```

```

读流程

```

```

storefile compaction

```
分为minor compaction 和 major compaction

minor compaction会把临近的小文件初次合并，清理掉部分过期和删除的数据

major compaction会将storefile下所有文件进行大合并，清理掉所有过期和删除的数据
```

region split

```

```



### 7、客户端操作

```
进入客户端：hbase shell

ddl：
最简单的：create 'student','info'//表名加列族
多个列族：create 'student',{NAME => 'info'},{NAME => 'city_info'}
查看表结构：desc 'student'
修改表：alter 'student',{NAME => 'msg'} //增加列族
删除列族：alter 'student','delete' => 'msg'
删除表：disable 'student' drop 'student' enable 'student'

dml:
添加数据：put 'student','1001','info:name','zhangsan'//表名，rowkey,列名，值
查询某个数据：get 'student','1001','info:name'//表名，rowkey,列名
查询某个范围的数据：scan 'student',{STARTROW => '1001',STOPROW => '1003!'}//左闭右开
修改某个值：put 'student','1001','info:name','lifang'//表名，rowkey,列名，新值
删除某个值：delete 'student','1001','info:age'
删除行数据：deleteall 'student','1001'
清空表：truncate 'student'
```

数据准备

```
put 'student','1001','info:name','Chen'
put 'student','1001','info:sex','male'
put 'student','1001','info:age','18'
put 'student','1002','info:name','Janna'
put 'student','1002','info:sex','female'
put 'student','1002','info:age','20'
put 'student','1003','info:name','Janna'
put 'student','1003','info:sex','female'
put 'student','1003','info:age','20'
put 'student','1004','info:name','Janna'
put 'student','1004','info:sex','female'
put 'student','1004','info:age','20'
```

### 8、写流程

```
1、客户端向zookeeper请求meta元数据（所有的region信息）地址
2、返回meta的位置（比如在Hadoop102）上
3、客户端向hadoop102请求元数据信息，hadoop102返回元数据信息，客户端会把元数据信息缓存，方便下次使用，如果元数据信息更新，会报错，客户端会重新向zookeeper请求元数据
4、通过元数据信息找到对应的regionServer,通信后将数据顺序追加到wal
5、将数据写入memstore,会进行排序，所以内部有序，完成后向客户端发送ack
6、等待时机将memstore数据刷写到hfile
```

### 9、flush刷写机制

```
以region为单位刷写（列族越少越好，最好是一个，最坏不要超过三个）（每次flush都会生成索引文件，和布隆过滤器）

刷写时机：

1.当某个memstroe的大小达到了hbase.hregion.memstore.flush.size（默认值128M），其所在region的所有memstore都会刷写。
当memstore的大小达到了
hbase.hregion.memstore.flush.size（默认值128M）
* hbase.hregion.memstore.block.multiplier（默认值4）//512会阻塞
时，会阻止继续往该memstore写数据。

2.当region server中memstore的总大小达到
java_heapsize//堆内存
*hbase.regionserver.global.memstore.size（默认值0.4）//写内存分配40%
*hbase.regionserver.global.memstore.size.lower.limit（默认值0.95）//比例95%
region会按照其所有memstore的大小顺序（由大到小）依次进行刷写。直到region server中所有memstore的总大小减小到上述值以下。
当region server中memstore的总大小达到//从大到小刷写，安全后就停止刷写防止小文件
java_heapsize
*hbase.regionserver.global.memstore.size（默认值0.4）//堆内存*0.4会阻塞
时，会阻止继续往所有的memstore写数据。

3. 到达自动刷写的时间，也会触发memstore flush。自动刷新的时间间隔由该属性进行配置hbase.regionserver.optionalcacheflushinterval（默认1小时）。//按时间flush

4.当WAL文件的数量超过hbase.regionserver.max.logs，region会按照时间顺序依次进行刷写，直到WAL文件数量减小到hbase.regionserver.max.log以下（该属性名已经废弃，现无需手动设置，最大值为32）

5、手动刷写 flush 'student'
```

### 10、读流程

```
1、客户端向zookeeper请求meta元数据（所有的region信息）地址
2、返回meta的位置（比如在Hadoop102）上
3、客户端向hadoop102请求元数据信息，hadoop102返回元数据信息，客户端会把元数据（包含所查表的元数据，并不是所有的meta）信息缓存，方便下次使用，如果元数据信息更新，会报错，客户端会重新向zookeeper请求元数据
4、通过元数据信息找到对应的regionServer，通信
5、向memstore和hfile中查询，将数据合并，包括所有版本，不同类型（put/delete）
6、将查询的数据以数据块(默认64kb)的方式缓存到block cache
7、将最终结果返回给客户端

优化读：扫描hfile时，通过时间过滤器，rowkey过滤器，布隆过滤器减少扫描的HFile数量
使用block cache
布隆过滤器：
blockcache存储的内容:
1、get、scan请求获取的数据、
2、缓存hbase:meta（可快速获取HFile的元数据）、
3、HFiles的索引，HBase以HDFS格式将其数据存储在HDFS中、
4、Bloom filters，如果使用Bloom过滤器，它们将存储在BlockCache中。

```

### 11、compaction合并

```
作用：减少HFile的个数，以及清理掉过期和删除的数据

小合并：Minor Compaction：Minor Compaction会将临近的若干个较小的HFile合并成一个较大的HFile，并清理掉部分过期和删除的数据
大合并：Major Compaction会将一个Store下的所有的HFile合并成一个大HFile，并且会清理掉所有过期和删除的数据。
小合并每次flush,大合并默认时7天，很耗费性能，大合并会关闭默认合并，手动执行
小合并删除部分数据，大合并会删除所有该删的数据
```

### 12、客户端命令

```
HBase管理命令:
Help ‘close_region’
close_region ‘t3..*' 通过web 页面或meta表查到region名字，关闭region
scan ‘t3’ 提示region 没在线
assign ‘t3../*’启用region
scan ‘t3’

Zookeeper 浏览
1. 进入Hbase安装目录
2. bin/hbase shell 显示hbase 下命令
3. bin/hbase zkcli 进入zookeeper命令行
4. help 查看帮助命令
5. ls / 查看根目录
6. ls /hbase 查看Hbase目录下内容
7. ls /hbase/table  查看table下内容
8. 打开新窗口，进入Hbase命令行
9. list 查看当前表
10. list_namespace_tables ‘hbase’ 查看命名空间里的表
11. get /hbase/master  查看master 信息
12. get /hbase/rs  查看regionserver 信息

Hbase 内部表
1. bin/hbase shell  进入Hbase命令行
2. help  查看帮助命令
3. list_namespace  查看命名空间
4. list_namespace_tables ‘hbase’  查看hbase里的表
5. Scan ‘hbase:namespace’  查看namespace表中数据
6. 打开新窗口，添加命名空间
7. create_namespace ‘ns1’  创建新的命名空间
8. list_namespace  再查看命名空间
9. Scan ‘hbase:namespace’  查看namespace表中数据是否添加一条
10. scan ‘hbase:meta’  查看hbase:meta表
11. create ‘t2’,’info’  创建一张表，查看meta表的变化
12. Scan ‘hbase:meta’  再次查看meta 表中信息，多了4条数据
13. ENCODE，Region在HDFS上的名字，通过web界面查看

Hbase hbck
1. put ‘t2’,’rs001’,’info:name’,’tom’ --添加一条数据
2. scan ‘t2’ --查看表中数据
3. deleteall ‘hbase:meta’,’t2..’--删除meta表中t2信息
4. scan ‘hbase:meta’--查看meta表中信息
5. scan ‘t2’ --查看表中数据
6. exit退出命令行，重新进入
7. scan ‘t2’ --查看表中数据,(报错，得示Unknow table t2)
8. 查看HDFS中文件是否存在，刷新web界面
9. bin/hbase hbck –help  查看hbck 的帮助命令
10. bin/hbase hbck –fix
11. scan ‘hbase:meta’
12. scan ‘t2’
```

### 13、API操作

#### 新建命名空间

```

```

#### 新建表

```

```

#### 插入一条数据

```

```

#### 获取一条数据

```

```

#### 获取多条数据

```

```

#### 删除数据

```

```

### 14、预分区

```

```

### 15、rowKey的设计

```
1、满足业务需求
2、防止热点
3、长度越小越好

方案1："时间戳反转".hash%5_时间戳反转_userid 时间戳根据业务取
```

### 16、底层存储

```
LSM Tree:
```

### 17、预分区和rowKey设计

```
create 'staff1','info',SPLITS => ['1000','2000','3000','4000'] //分区键，5个分区

create 'staff2','info',{NUMREGIONS => 15, SPLITALGO => 'HexStringSplit'} //15个分区

create 'staff3',SPLITS_FILE => 'splits.txt' //按照文件分区
文件如下：
aaaa
bbbb
cccc
dddd


rowKey的设计：
使用散列值，时间戳倒置，字符串拼接
```

### 18、优化

```
1.Zookeeper会话超时时间
hbase-site.xml
属性：zookeeper.session.timeout=60000
解释：默认值为90000毫秒（90s）。当某个RegionServer挂掉，90s之后Master才能察觉到。可适当减小此值，以加快Master响应，可调整至600000毫秒。

2.设置RPC监听数量
hbase-site.xml
属性：hbase.regionserver.handler.count=30
解释：默认值为30，用于指定RPC监听的数量，可以根据客户端的请求数进行调整，读写请求较多时，增加此值。cpu核数的2倍

3.手动控制Major Compaction
hbase-site.xml
属性：hbase.hregion.majorcompaction=0
解释：默认值：604800000秒（7天）， Major Compaction的周期，若关闭自动Major Compaction，可将其设为0
手动：major_compact ‘student’

4.优化HStore文件大小
hbase-site.xml
属性：hbase.hregion.max.filesize
解释：默认值10737418240（10GB），如果需要运行HBase的MR任务，可以减小此值，因为一个region对应一个map任务，如果单个region过大，会导致map任务执行时间过长。该值的意思就是，如果HFile的大小达到这个数值，则这个region会被切分为两个Hfile。

一个map对应一个region,map对应的数据过大，所以需要减小这个值
5.优化HBase客户端缓存
hbase-site.xml
属性：hbase.client.write.buffer
解释：默认值2097152bytes（2M）用于指定HBase客户端缓存，增大该值可以减少RPC调用次数，但是会消耗更多内存，反之则反之。一般我们需要设定一定的缓存大小，以达到减少RPC次数的目的。

2M发送一次写请求
6.指定scan.next扫描HBase所获取的行数
hbase-site.xml
属性：hbase.client.scanner.caching
解释：用于指定scan.next方法获取的默认行数，值越大，消耗内存越大。

一批一批的读，每次拉取多少行，
7.BlockCache占用RegionServer堆内存的比例
hbase-site.xml
属性：hfile.block.cache.size
解释：默认0.4，读请求比较多的情况下，可适当调大


8.MemStore占用RegionServer堆内存的比例
hbase-site.xml
属性：hbase.regionserver.global.memstore.size
解释：默认0.4，写请求较多的情况下，可适当调大

```

### 19、SQL皮肤凤凰

#### 部署安装：

```
1、解压改名
tar -zxvf apache-phoenix-5.0.0-HBase-2.0-bin.tar.gz -C /opt/module/
mv apache-phoenix-5.0.0-HBase-2.0-bin phoenix
2、复制jar包到hbase中cp /opt/module/phoenix/phoenix-5.0.0-HBase-2.0-server.jar /opt/module/hbase/lib/

3、分发传入的jar包：xsync /opt/module/hbase/lib/phoenix-5.0.0-HBase-2.0-server.jar

4、配置环境变量：
#phoenix
export PHOENIX_HOME=/opt/module/phoenix
export PHOENIX_CLASSPATH=$PHOENIX_HOME
export PATH=$PATH:$PHOENIX_HOME/bin

5、胖客户端：sqlline.py hadoop102,hadoop103,hadoop104:2181

6、瘦客户端：
先启动：queryserver.py start
然后启动：sqlline.py hadoop102,hadoop103,hadoop104:2181

7、默认情况下，直接在HBase中创建的表，通过Phoenix是查看不到的。如果要在Phoenix中操作直接在HBase中创建的表，则需要在Phoenix中进行表的映射。映射方式有两种：视图映射和表映射

8、视图映射和表映射
视图映射：Phoenix创建的视图是只读的，所以只能用来做查询，无法通过视图对源数据进行修改等操作。在phoenix中创建关联test表的视图
create view "test"(id varchar primary key,"info1"."name" varchar, "info2"."address" varchar);
删除：drop view "test";

表映射：
使用Apache Phoenix创建对HBase的表映射，有两种方法：
（1）HBase中不存在表时，可以直接使用create table指令创建需要的表,系统将会自动在Phoenix和HBase中创建person_infomation的表，并会根据指令内的参数对表结构进行初始化。
（2）当HBase中已经存在表时，可以以类似创建视图的方式创建关联表，只需要将create view改为create table即可。
create table "test"(id varchar primary key,"info1"."name" varchar, "info2"."address" varchar) column_encoded_bytes=0;
```

#### linux中操作

```
显示所有表!table
退出!quit
创建表：
CREATE TABLE IF NOT EXISTS student(
id VARCHAR primary key,
name VARCHAR,
addr VARCHAR);

CREATE TABLE IF NOT EXISTS us_population (
State CHAR(2) NOT NULL,
City VARCHAR NOT NULL,
Population BIGINT
CONSTRAINT my_pk PRIMARY KEY (state, city));

插入数据：upsert into student values('1001','zhangsan','beijing');

使用：select * from student where id='1001'

删除：delete from student where id='1001'

删除表：drop table student;
```

#### API操作

瘦客户端

```
1、启动服务：queryserver.py start
2、导入依赖
<dependencies>
    <!-- https://mvnrepository.com/artifact/org.apache.phoenix/phoenix-queryserver-client -->
    <dependency>
        <groupId>org.apache.phoenix</groupId>
        <artifactId>phoenix-queryserver-client</artifactId>
        <version>5.0.0-HBase-2.0</version>
    </dependency>
</dependencies>
3、编码
		String connectionUrl = ThinClientUtil.getConnectionUrl("hadoop102", 8765);
        System.out.println(connectionUrl);
        Connection connection = DriverManager.getConnection(connectionUrl);
        PreparedStatement preparedStatement = connection.prepareStatement("select * from student");

        ResultSet resultSet = preparedStatement.executeQuery();

        while (resultSet.next()) {
            System.out.println(resultSet.getString(1) + "\t" + resultSet.getString(2));
        }

        //关闭
        connection.close();
```

胖客户端

```
1、导入依赖
<dependencies>
    <dependency>
        <groupId>org.apache.phoenix</groupId>
        <artifactId>phoenix-core</artifactId>
        <version>5.0.0-HBase-2.0</version>
        <exclusions>
            <exclusion>
                <groupId>org.glassfish</groupId>
                <artifactId>javax.el</artifactId>
            </exclusion>
            <exclusion>
                <groupId>org.apache.hadoop</groupId>
                <artifactId>hadoop-common</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.glassfish</groupId>
        <artifactId>javax.el</artifactId>
        <version>3.0.1-b06</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-common</artifactId>
        <version>2.8.4</version>
    </dependency>
</dependencies>

2、代码
		String url = "jdbc:phoenix:hadoop102,hadoop103,hadoop104:2181";

        Properties properties = new Properties();

        properties.put("phoenix.schema.isNamespaceMappingEnabled","true");
        System.out.println(url);

        Connection connection = DriverManager.getConnection(url,properties);

        PreparedStatement pr = connection.prepareStatement("select * from \"test\"");

        ResultSet resultSet = pr.executeQuery();

        while (resultSet.next()) {
            System.out.println(resultSet.getString(1) + "\t" + resultSet.getString(2));
        }

        //关闭
        connection.close();

```



### 20、与hive集成

```
1、搭建：
在hive中配置文件hive-site.xml：
	<property>
        <name>hive.zookeeper.quorum</name>
        <value>hadoop102,hadoop103,hadoop104</value>
    </property>

    <property>
        <name>hive.zookeeper.client.port</name>
        <value>2181</value>
    </property>

```

使用方法1：在hive中创建表，同时关联 HBase表

```
1、创建表：
CREATE TABLE hive_hbase_emp_table(
empno int,
ename string,
job string,
mgr int,
hiredate string,
sal double,
comm double,
deptno int)
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,info:ename,info:job,info:mgr,info:hiredate,info:sal,info:comm,info:deptno")
TBLPROPERTIES ("hbase.table.name" = "hbase_emp_table");

2、插入数据：
insert into table hive_hbase_emp_table select * from emp;
```

使用方法2：针对HBase中的表，在hive中创建外部表关联进行分析

```
1、在HBase中存储了一张表hbase_emp_table

2、在hive中创建外部表
CREATE EXTERNAL TABLE relevance_hbase_emp(
empno int,
ename string,
job string,
mgr int,
hiredate string,
sal double,
comm double,
deptno int)
STORED BY 
'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
WITH SERDEPROPERTIES ("hbase.columns.mapping" = 
":key,info:ename,info:job,info:mgr,info:hiredate,info:sal,info:comm,info:deptno") 
TBLPROPERTIES ("hbase.table.name" = "hbase_emp_table");

3、使用：select * from relevance_hbase_emp;

```

    <property>
        <name>yarn.nodemanager.vmem-check-enabled</name>
        <value>false</value>
    </property>