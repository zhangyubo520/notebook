# ClickHouse

### 1、概述

```
1、俄国人使用c++编写
2、列式存储
3、开源
4、在线分析处理查询(OLAP)
```

### 2、原理概述

```
1、列式存储的好处：
对于列的聚合，计数更优
同一列的数据类型相同，压缩性能更好，更节省磁盘空间

2、写操作：
采用类似LSM tree 的结构，数据先顺序写入临时数据区，之后异步合并到主数据区，写入性能优异，单不支持删改行数据，也不支持事务

3、读操作：
clickhouse是单语句多线程，即一条查询语句可能是由多个线程完成的，而MySQL中一条查询语句只有一个线程，多个语句才有可能起多个线程执行，一条语句起多个线程，处理单条语句很强，但是不适合处理多条语句的情景

4、采用稀疏索引：
优点是可以快速定位一大片数据
缺点是不适合做点对点查询
Hbase和redis是支持hash索引的，可以点对点查询，Mysql使用B+数索引，即可以点对点，也可以范围查找

5、性能分析结果：适合单个大宽表查询
```

### 3、单机安装

```
1、修改虚拟机参数：
vim /etc/security/limits.conf
* soft nofile 65536 
* hard nofile 65536 
* soft nproc 131072 
* hard nproc 131072

vim /etc/selinux/config
SELINUX=disabled

2、安装依赖：yum install -y libtool  yum install -y *unixODBC*

3、下载下面的rpm文件，放到hadoop102虚拟机的/opt/software/目录下：
clickhouse-client-20.4.4.18-2.noarch.rpm
clickhouse-common-static-20.4.4.18-2.x86_64.rpm
clickhouse-common-static-dbg-20.4.4.18-2.x86_64.rpm
clickhouse-server-20.4.4.18-2.noarch.rpm

4、yum安装：
sudo rpm -ivh /opt/software/clickhouse*

5、安装后的安装位置如下：
/etc/systemd/system/multi-user.target.wants/clickhouse-server.service to 
/etc/systemd/system/clickhouse-server.service
/etc/clickhouse-server/config.xml: /var/lib/clickhouse/

6、修改配置文件：sudo vim /etc/clickhouse-server/config.xml
<listen_host>::</listen_host> 的注解打开

7、启动服务：sudo systemctl start clickhouse-server

8、启动客户端：clickhouse-client -m

9、关闭开机自启动：sudo systemctl disable clickhouse-server

10、官方文档介绍下载等：
https://clickhouse.tech/docs/en/
官网：https://clickhouse.yandex/
下载地址：http://repo.red-soft.biz/repos/clickhouse/stable/el6/

```

### 4、数据类型

```
1、有符号整型：
Int8 - [-128 : 127]
Int16 - [-32768 : 32767]
Int32 - [-2147483648 : 2147483647]
Int64 - [-9223372036854775808 : 9223372036854775807]

2、无符号整型：
UInt8 - [0 : 255]
UInt16 - [0 : 65535]
UInt32 - [0 : 4294967295]
UInt64 - [0 : 18446744073709551615]

推荐无符号整形

3、浮点型：
Float32 - float
Float64 – double

基本不用

4、布尔型：
没有，使用UInt8中的0代表false，1代表true

5、Decimal型：
Decimal32(s)，相当于Decimal(9-s,s)
Decimal64(s)，相当于Decimal(18-s,s)
Decimal128(s)，相当于Decimal(38-s,s)

有保留精度的场合使用，除法采用去尾法而不是四舍五入

6、字符串：
String：可变化的，使用较多
FixedString:不可变化的，使用较少

7、枚举类型：
包括 Enum8 和 Enum16 类型。Enum 保存 'string'= integer 的对应关系。
Enum8 用 'String'= Int8 对描述。
Enum16 用 'String'= Int16 对描述。

创建一个带有一个枚举 Enum8('hello' = 1, 'world' = 2) 类型的列：
CREATE TABLE t_enum
(
    x Enum8('hello' = 1, 'world' = 2)
)
ENGINE= TinyLog

插入数据：INSERT INTO t_enum VALUES ('hello'), ('world'), ('hello')

8、时间类型：
Date 接受 年-月-日 的字符串比如 ‘2019-12-16’
Datetime 接受 年-月-日 时:分:秒 的字符串比如 ‘2019-12-16 20:50:10’
Datetime64 接受 年-月-日 时:分:秒.亚秒 的字符串比如 ‘2019-12-16 20:50:10.66’

9、数组：array(T)
例如：select array(1,2,3) as n

```

### 5、表引擎

```
表引擎的作用：
1）数据的存储方式和位置 
2）并发数据访问。
3）索引的使用。
4）是否可以执行多线程请求。
5）数据如何拷贝副本。

1、TinyLog:
以列文件的形式保存在磁盘上，不支持索引，没有并发控制。一般保存少量数据的小表，生产环境上作用有限。可以用于平时练习测试用。

2、Memory：
性能高，但重启会消失

3、MergeTree:
可指定分区，指定主键，指定排序字段
create table t_order_mt(
    id UInt32,
    sku_id String,
    total_amount Decimal(16,2),
    create_time  Datetime
 ) engine =MergeTree
 partition by toYYYYMMDD(create_time)
   primary key (id)
   order by (id,sku_id)

partition by用来指定分区，没有指定的话只有一个，任何一个批次的数据写入都会产生一个临时分区，不会纳入任何一个已有的分区写入后的某个时刻（大概10-15分钟后），clickhouse会自动执行合并操作，手动合并使用命令：optimize table t_order_mt final

primary key用来指定主键可选，可重复，order by用来指定排序字段，必选
没有定义主键怎么办，默认使用order by里的字段
主键必须是order by中字段的前缀字段，比如order by 字段是 (id,sku_id)  那么主键必须是id 或者(id,sku_id)

4、二级索引
所以使用二级索引前需要增加设置：set allow_experimental_data_skipping_indices=1;
搜索条件中涉及了total_amount就可以建立一个二级索引，索引主要作用是过滤，主要作用于where过滤条件中：
INDEX a total_amount TYPE minmax GRANULARITY 5

5、数据声明周期
列级TTL:
  create table t_order_mt3(
    id UInt32,
    sku_id String,
    total_amount Decimal(16,2)  TTL create_time+interval 10 SECOND,
    create_time  Datetime 
 ) engine =MergeTree
 partition by toYYYYMMDD(create_time)
   primary key (id)
   order by (id, sku_id)
在建表语句某个字段后面增加 TTL create_time+interval 10 SECOND 创建时间10s后失效
插入数据查询列数据存在
10s后执行：optimize table t_order_mt final 列数据失效

表级TTL:
alter table t_order_mt3 MODIFY TTL create_time + INTERVAL 10 SECOND
10s后执行：optimize table t_order_mt final 表失效

时间周期：
- SECOND
- MINUTE
- HOUR
- DAY
- WEEK
- MONTH
- QUARTER
- YEAR

6、ReplacingMergeTree
可实现去重；
注意：
	实际上是使用order by 字段作为唯一键。
	去重不能跨分区。
	只有合并分区才会进行去重。
	认定重复的数据保留，版本字段值最大的。
	如果版本字段相同则保留最后一笔。

7、SummingMergeTree
提供预聚合：engine =SummingMergeTree(total_amount)

```

### 6、维度分析

```
1、select id , sku_id,sum(total_amount) from  t_order_mt group by id,sku_id with rollup;
with rollup : 从右至左去掉维度进行小计。

2、select id , sku_id,sum(total_amount) from  t_order_mt group by id,sku_id with cube;
with cube : 从右至左去掉维度进行小计，再从左至右去掉维度进行小计。

3、select id , sku_id,sum(total_amount) from  t_order_mt group by id,sku_id with totals;
with totals: 只计算合计。
```

### 7、导出数据

```
clickhouse-client  --query    "select * from test.t_order_mt" --format CSVWithNames> ~/rs1.csv

然后使用sz命令将~/rs1.csv文件导出到window桌面查看

```

### 8、副本

```
1、启动zookeeper集群 和另外一台clickhouse 服务器

2、修改两台服务器的配置文件：
mkdir /etc/clickhouse-server/config.d
vim /etc/clickhouse-server/config.d/metrika.xml

<?xml version="1.0"?>
<yandex>
  <zookeeper-servers>
     <node index="1">
	     <host>hadoop102</host>
		 <port>2181</port>
     </node>
	 <node index="2">
	     <host>hadoop102</host>
		 <port>2181</port>
     </node>
<node index="3">
	     <host>hadoop102</host>
		 <port>2181</port>
     </node>
  </zookeeper-servers>
</yandex>

vim /etc/clickhouse-server/config.xml
<zookeeper incl="zookeeper-servers" optional="true" />
<include_from>/etc/clickhouse-server/config.d/metrika.xml</include_from>

3、两台机器分别建表：
create table rep_t_order_mt_0105 (
    id UInt32,
    sku_id String,
    total_amount Decimal(16,2),
    create_time  Datetime
 ) engine =ReplicatedMergeTree('/clickhouse/tables/01/rep_t_order_mt_0105','rep_hdp1')
 partition by toYYYYMMDD(create_time)
   primary key (id)
   order by (id,sku_id);

create table rep_t_order_mt_0105 (
    id UInt32,
    sku_id String,
    total_amount Decimal(16,2),
    create_time  Datetime
 ) engine =ReplicatedMergeTree('/clickhouse/tables/01/rep_t_order_mt_0105','rep_hdp1')
 partition by toYYYYMMDD(create_time)
   primary key (id)
   order by (id,sku_id);

4、使用即可
```

### 9、分片集群

```
1、配置文件：vim /etc/clickhouse-server/config.d/metrika.xml
<yandex>
<clickhouse_remote_servers>
<gmall_cluster> <!-- 集群名称--> 
  <shard>         <!--集群的第一个分片-->
<internal_replication>true</internal_replication>
     <replica>    <!—该分片的第一个副本-->
          <host>hadoop102</host>
          <port>9000</port>
     </replica>
     <replica>    <!—该分片的第二个副本-->
          <host>hadoop103</host>
          <port>9000</port>
     </replica>
  </shard>

  <shard>  <!--集群的第二个分片-->
     <internal_replication>true</internal_replication>
     <replica>    <!—该分片的第一个副本-->
          <host>hadoop104</host>
          <port>9000</port>
     </replica>
</shard>



</gmall_cluster>

</clickhouse_remote_servers>



<zookeeper-servers>
  <node index="1">
    <host>hadoop102</host>
    <port>2181</port>
  </node>

  <node index="2">
    <host>hadoop103</host>
    <port>2181</port>
  </node>
  <node index="3">
    <host>hadoop104</host>
    <port>2181</port>
  </node>
</zookeeper-servers>

<macros>
<shard>01</shard>   <!—不同机器放的分片数不一样-->
<replica>rep_1_1</replica>  <!—不同机器放的副本数不一样-->

</macros>

</yandex>

2、分发配置(3台)
配置修改：
hadoop102		
<macros>
<shard>01</shard> 
<replica>rep_1_1</replica>
</macros>	

hadoop103
<macros>
<shard>01</shard> 
<replica>rep_1_2</replica>
</macros>	

hadoop104
<macros>
<shard>02</shard> 
<replica>rep_2_1</replica>
</macros>


3、使用：
    create table st_order_mt_0105 on cluster gmall_cluster (
    id UInt32,
    sku_id String,
    total_amount Decimal(16,2),
    create_time  Datetime
 ) engine =ReplicatedMergeTree('/clickhouse/tables/{shard}/st_order_mt_0105','{replica}')
 partition by toYYYYMMDD(create_time)
   primary key (id)
   order by (id,sku_id);


4、分布式表：
create table st_order_mt_0105_all on cluster gmall_cluster
(
    id UInt32,
    sku_id String,
    total_amount Decimal(16,2),
    create_time  Datetime
)engine = Distributed(gmall_cluster,test0105, st_order_mt_0105,hiveHash(sku_id))

Distributed( 集群名称，库名，本地表名，分片键)，分片键必须是整型数字
```

### 10、编写代码

```
1、引入依赖
<dependency>
    <groupId>ru.yandex.clickhouse</groupId>
    <artifactId>clickhouse-jdbc</artifactId>
    <version>0.1.55</version>
</dependency>

2、连接：jdbc:clickhouse://hadoop102:8123/test
驱动：ru.yandex.clickhouse.ClickHouseDriver

查询：
    val address = "jdbc:clickhouse://hadoop102:8123/test"//获取连接

    Class.forName("ru.yandex.clickhouse.ClickHouseDriver")//加载驱动

    val connection: Connection = DriverManager.getConnection(address)//获取连接

    val sql = "select * from t_order_mt"//sql语句

    val resultSet: ResultSet = connection.createStatement().executeQuery(sql)//将查询结果放入结果集

    println("id" + "\t" + "sku_id" + "\t" + "total_amount" + "\t" + "create_time")//打印表头
    while (resultSet.next()){//依次获取每条语句
      val id: String = resultSet.getString("id")
      val sku_id: String = resultSet.getString("sku_id")
      val total_amount: String = resultSet.getString("total_amount")
      val create_time = resultSet.getString("create_time")

      println(id + "\t" + sku_id + "\t" + total_amount + "\t" + create_time)//打印
    }

    connection.close()//关闭连接
    
```

