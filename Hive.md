



# hive

### 一、概念

#### 1、hive概述

```
Hive：由Facebook开源用于解决海量结构化日志的数据统计工具。

Hive是基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张表，并提供类SQL查询功能。

本质是：将HQL转化成MapReduce程序

说明：
1、hive数据存储在hdfs上
2、hive底层实现是MapReduce
3、hive程序运行在yarn上

优点：操作方便，学习方便，处理海量数据，支持自定义函数

缺点：延迟高，迭代算法无法表达，数据挖掘方面不擅长，自动生成的MapReduce通常不够智能，调优困难，
```

#### 2、hive架构

```
1）用户接口：Client
CLI（command-line interface）、JDBC/ODBC(jdbc访问hive)、WEBUI（浏览器访问hive）

2）元数据：Metastore
元数据包括：表名、表所属的数据库（默认是default）、表的拥有者、列/分区字段、表的类型（是否是外部表）、表的数据所在目录等；
默认存储在自带的derby数据库中，推荐使用MySQL存储Metastore

3）Hadoop
使用HDFS进行存储，使用MapReduce进行计算。

4）驱动器：Driver
（1）解析器（SQL Parser）：将SQL字符串转换成抽象语法树AST，这一步一般都用第三方工具库完成，比如antlr；对AST进行语法分析，比如表是否存在、字段是否存在、SQL语义是否有误。
（2）编译器（Physical Plan）：将AST编译生成逻辑执行计划。
（3）优化器（Query Optimizer）：对逻辑执行计划进行优化。
（4）执行器（Execution）：把逻辑执行计划转换成可以运行的物理计划。对于Hive来说，就是MR/Spark。

总结：
Hive通过给用户提供的一系列交互接口，接收到用户的指令(SQL)，使用自己的Driver，结合元数据(MetaStore)，将这些指令翻译成MapReduce，提交到Hadoop中执行，最后，将执行返回的结果输出到用户交互接口。
```

#### 3、执行延迟的原因

```
1、Hive 在查询数据的时候，由于没有索引，需要扫描整个表，因此延迟较高

2、另外一个导致 Hive 执行延迟高的因素是 MapReduce框架。由于MapReduce 本身具有较高的延迟，因此在利用MapReduce 执行Hive查询时，也会有较高的延迟
```

#### 4、hive和数据库比较

```
1.4.1 查询语言

由于SQL被广泛的应用在数据仓库中，因此，专门针对Hive的特性设计了类SQL的查询语言HQL。熟悉SQL开发的开发者可以很方便的使用Hive进行开发。

1.4.2 数据更新

由于Hive是针对数据仓库应用设计的，而数据仓库的内容是读多写少的。因此，Hive中不建议对数据的改写，所有的数据都是在加载的时候确定好的。而数据库中的数据通常是需要经常进行修改的，因此可以使用 INSERT INTO …  VALUES 添加数据，使用 UPDATE … SET修改数据。

1.4.3 执行延迟

Hive 在查询数据的时候，由于没有索引，需要扫描整个表，因此延迟较高。另外一个导致 Hive 执行延迟高的因素是 MapReduce框架。由于MapReduce 本身具有较高的延迟，因此在利用MapReduce 执行Hive查询时，也会有较高的延迟。相对的，数据库的执行延迟较低。当然，这个低是有条件的，即数据规模较小，当数据规模大到超过数据库的处理能力的时候，Hive的并行计算显然能体现出优势。

1.4.4 数据规模

由于Hive建立在集群上并可以利用MapReduce进行并行计算，因此可以支持很大规模的数据；对应的，数据库可以支持的数据规模较小。
```

#### 5、hive下载地址

```
1）Hive官网地址
http://hive.apache.org/

2）文档查看地址
https://cwiki.apache.org/confluence/display/Hive/GettingStarted

3）下载地址
http://archive.apache.org/dist/hive/

4）github地址
https://github.com/apache/hive
```

#### 6、hive安装步骤

##### 第一大步：安装MySQL

```

1、检查当前系统是否安装过Mysql:
运行：[atguigu@hadoop102 ~]$ rpm -qa|grep mariadb
结果：mariadb-libs-5.5.56-2.el7.x86_64 //如果存在通过第二步卸载

2、卸载原有mysql:
运行：[atguigu @hadoop102 ~]$ sudo rpm -e --nodeps mariadb-libs-5.5.56-2.el7.x86_64

3、上传安装包至/opt/software下
运行：[atguigu @hadoop102 software]# ll
结果：mysql-5.7.28-1.el7.x86_64.rpm-bundle.tar

4、解压mysql至/opt/software下:
运行：[atguigu @hadoop102 software]# tar -xf mysql-5.7.28-1.el7.x86_64.rpm-bundle.tar
结果：
-rw-r--r--. 1 9月  30 2019 mysql-community-client-5.7.28-1.el7.x86_64.rpm
-rw-r--r--. 1 9月  30 2019 mysql-community-common-5.7.28-1.el7.x86_64.rpm
-rw-r--r--. 1 9月  30 2019 mysql-community-devel-5.7.28-1.el7.x86_64.rpm
-rw-r--r--. 1 9月  30 2019 mysql-community-embedded-5.7.28-1.el7.x86_64.rpm
-rw-r--r--. 1 9月  30 2019 mysql-community-embedded-compat-5.7.28-1.el7.x86_64.rpm
-rw-r--r--. 1 9月  30 2019 mysql-community-embedded-devel-5.7.28-1.el7.x86_64.rpm
-rw-r--r--. 1 9月  30 2019 mysql-community-libs-5.7.28-1.el7.x86_64.rpm
-rw-r--r--. 1 9月  30 2019 mysql-community-libs-compat-5.7.28-1.el7.x86_64.rpm
-rw-r--r--. 1 9月  30 2019 mysql-community-server-5.7.28-1.el7.x86_64.rpm
-rw-r--r--. 1 9月  30 2019 mysql-community-test-5.7.28-1.el7.x86_64.rpm

5、安装mysql至/opt/module下
依次运行：
[atguigu @hadoop102 software]$ sudo rpm -ivh mysql-community-common-5.7.28-1.el7.x86_64.rpm
[atguigu @hadoop102 software]$ sudo rpm -ivh mysql-community-libs-5.7.28-1.el7.x86_64.rpm
[atguigu @hadoop102 software]$ sudo rpm -ivh mysql-community-libs-compat-5.7.28-1.el7.x86_64.rpm
[atguigu @hadoop102 software]$ sudo rpm -ivh mysql-community-client-5.7.28-1.el7.x86_64.rpm
[atguigu @hadoop102 software]$ sudo rpm -ivh mysql-community-server-5.7.28-1.el7.x86_64.rpm

结果：
atguigu@hadoop102: /opt/module/hive 21:43:07
$ rpm -qa | grep mysql
mysql-community-libs-compat-5.7.28-1.el7.x86_64
mysql-community-common-5.7.28-1.el7.x86_64
mysql-community-server-5.7.28-1.el7.x86_64
mysql-community-libs-5.7.28-1.el7.x86_64
mysql-community-client-5.7.28-1.el7.x86_64

错误1：依赖检测失败：libaio.so.1()(64bit) 被 mysql-community-server-5.7.28-1.el7.x86_64 需要
解决：[atguigu@hadoop102 software] yum install -y libaio

6、查看/etc/my.cnf文件
运行：vim /etc/my.conf
结果：找到这条信息：datadir=/var/lib/mysql

7、删除/var/lib/mysql文件夹下所有
运行：[atguigu @hadoop102 mysql]# rm -rf *

8、初始化mysql
运行：[atguigu @hadoop102 opt]$ sudo mysqld --initialize --user=mysql

9、查看初始化生成的默认密码
运行：[atguigu @hadoop102 opt]$ sudo cat /var/log/mysqld.log
结果：2020-04-24T07:23:59.791236Z 1 [Note] A temporary password is generated for root@localhost: qc6sYPFvFv/+

10、启动mysql
运行：[atguigu @hadoop102 opt]$ sudo systemctl start mysqld

11、初次登陆mysql（这里输入密码不是显示的，复制粘贴第九步的密码
运行：[atguigu @hadoop102 opt]$ mysql -uroot -p
回车输入初始默认密码 Enter password:qc6sYPFvFv/+

12、修改密码（注意分号结束命令，不然不生效）
运行：mysql> set password = password("123456");

13、退出（注意分号结束命令，不然不生效）
运行：mysql> quit;

14、再次登陆
运行：
atguigu@hadoop102: /var/log 22:03:03
$ mysql -uroot -p123456
结果：可以登陆

15、修改mysql库下的user表中的root用户允许任意ip连接
运行：
mysql> update mysql.user set host='%' where user='root';
mysql> flush privileges;

```

##### 第二大步：安装hive

```
1、上传安装包apache-hive-3.1.2-bin.tar.gz至linux中的/opt/software下

2、解压
运行：
[atguigu@hadoop102 software]$ tar -zxvf /opt/software/apache-hive-3.1.2-bin.tar.gz -C /opt/module/

3、改名（容易记，方便操作）
mv /opt/module/apache-hive-3.1.2-bin/ /opt/module/hive

4、配置环境变量
运行：sudo vim /etc/profile.d/my_env.sh
结果：增加了
#HIVE_HOME
export HIVE_HOME=/opt/module/hive
export PATH=$PATH:$HIVE_HOME/bin

5、解决日志Jar包冲突
运行：
[atguigu@hadoop102 software]$ mv $HIVE_HOME/lib/log4j-slf4j-impl-2.10.0.jar $HIVE_HOME/lib/log4j-slf4j-impl-2.10.0.bak

6、安装mysql驱动
上传mysql-connector-java-5.1.48.jar至/opt/module/hive/lib下

7、配置文件编辑
运行：[atguigu@hadoop102 software]$ vim $HIVE_HOME/conf/hive-site.xml
添加：（注意用户名和用户密码的配置，不然后面hive初始化连不上mysql）
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>

    <!-- jdbc连接的URL -->
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://hadoop102:3306/metastore?useSSL=false</value>
	</property>

    <!-- jdbc连接的Driver-->
    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.jdbc.Driver</value>
	</property>

	<!-- jdbc连接的username-->
    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>root</value>
    </property>

    <!-- jdbc连接的password -->
    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>123456</value>
    </property>
    
    <!-- Hive默认在HDFS的工作目录 -->
    <property>
        <name>hive.metastore.warehouse.dir</name>
        <value>/user/hive/warehouse</value>
    </property>
    
   <!-- Hive元数据存储版本的验证 -->
    <property>
        <name>hive.metastore.schema.verification</name>
        <value>false</value>
    </property>
    
    <!-- 指定存储元数据要连接的地址 -->
    <property>
        <name>hive.metastore.uris</name>
        <value>thrift://hadoop102:9083</value>
    </property>
    
    <!-- 指定hiveserver2连接的端口号 -->
    <property>
    <name>hive.server2.thrift.port</name>
    <value>10000</value>
    </property>
    
   <!-- 指定hiveserver2连接的host -->
    <property>
        <name>hive.server2.thrift.bind.host</name>
        <value>hadoop102</value>
    </property>
    
    <!-- 元数据存储授权  -->
    <property>
        <name>hive.metastore.event.db.notification.api.auth</name>
        <value>false</value>
    </property>
    
    <!-- 修改Hive的计算引擎  -->
    <property>
        <name>hive.execution.engine</name>
        <value>tez</value>
    </property>
    <property>
        <name>hive.tez.container.size</name>
        <value>2048</value>
    </property>

</configuration>
```

##### 第三大步：安装Tez引擎

```
1、上传安装包：tez-0.10.1-SNAPSHOT-minimal.tar.gz
			 tez-0.10.1-SNAPSHOT.tar.gz
			 
2、在/opt/module下创建文件夹tez
运行：[atguigu@hadoop102 software]$ mkdir /opt/module/tez

3、解压：tar -zxvf /opt/software/tez-0.10.1-SNAPSHOT-minimal.tar.gz -C /opt/module/tez

4、上传依赖到hdfs：
运行：
[atguigu@hadoop102 software]$ hadoop fs -mkdir /tez
[atguigu@hadoop102 software]$ hadoop fs -put /opt/software/tez-0.10.1-SNAPSHOT.tar.gz /tez

5、配置环境变量
运行： [atguigu@hadoop102 software]$ vim $HADOOP_HOME/etc/hadoop/shellprofile.d/tez.sh
添加：
hadoop_add_profile tez
function _tez_hadoop_classpath
{
    hadoop_add_classpath "$HADOOP_HOME/etc/hadoop" after
    hadoop_add_classpath "/opt/module/tez/*" after
    hadoop_add_classpath "/opt/module/tez/lib/*" after
}

6、解决日志冲突：
运行：[atguigu@hadoop102 software]$ rm /opt/module/tez/lib/slf4j-log4j12-1.7.10.jar

7、编写配置文件
运行：[atguigu@hadoop102 software]$ vim $HADOOP_HOME/etc/hadoop/tez-site.xml
添加：
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
<property>
	<name>tez.lib.uris</name>
    <value>${fs.defaultFS}/tez/tez-0.10.1-SNAPSHOT.tar.gz</value>
</property>

<property>
     <name>tez.use.cluster.hadoop-libs</name>
     <value>true</value>
</property>

<property>
     <name>tez.am.resource.memory.mb</name>
     <value>1024</value>
</property>

<property>
     <name>tez.am.resource.cpu.vcores</name>
     <value>1</value>
</property>

<property>
     <name>tez.container.max.java.heap.fraction</name>
     <value>0.4</value>
</property>

<property>
     <name>tez.task.resource.memory.mb</name>
     <value>1024</value>
</property>

<property>
     <name>tez.task.resource.cpu.vcores</name>
     <value>1</value>
</property>

</configuration>
```

##### Hive on Spark配置

```
1、tar -zxvf /opt/software/spark-2.4.5-bin-without-hive.tgz -C /opt/module

2、mv /opt/module/spark-2.4.5-bin-without-hive /opt/module/spark

3、sudo vim /etc/profile.d/my_env.sh 添加：
export SPARK_HOME=/opt/module/spark
export PATH=$PATH:$SPARK_HOME/bin

4、source /etc/profile.d/my_env.sh

5、配置文件：mv /opt/module/spark/conf/spark-env.sh.template /opt/module/spark/conf/spark-env.sh
vim /opt/module/spark/conf/spark-env.sh  添加：
export SPARK_DIST_CLASSPATH=$(hadoop classpath)

6、vim /opt/module/hive/conf/spark-defaults.conf 是新建的，添加：
spark.master                               yarn
spark.eventLog.enabled                   true
spark.eventLog.dir                        hdfs://hadoop102:9820/spark-history
spark.executor.memory                    1g
spark.driver.memory					   1g

7、在hdfs上创建文件夹spark-history 和 spark-jars
hadoop fs -mkdir /spark-history
hadoop fs -mkdir /spark-jars

8、上传hadoop fs -put /opt/module/spark/jars/* /spark-jars

9、修改hive的配置文件：vim hive-site.xml
<!--Spark依赖位置-->
<property>
    <name>spark.yarn.jars</name>
    <value>hdfs://hadoop102:9820/spark-jars/*</value>
</property>
  
<!--Hive执行引擎-->
<property>
    <name>hive.execution.engine</name>
    <value>spark</value>
</property>

<!--Hive和spark连接超时时间-->
<property>
    <name>hive.spark.client.connect.timeout</name>
    <value>10000ms</value>
</property>

10、测试：
create external table student(id int, name string) location '/student';
hive (default)> insert into table student values(2,'bc');
```

##### 第四大步：初始化hive

```
1、登陆mysql
[atguigu@hadoop102 software]$ mysql -uroot -p123456

2、新建Hive元数据库（注意分号）
mysql> create database metastore;

3、退出mysql
mysql> quit;

4、初始化hive
运行：[atguigu@hadoop102 software]$ schematool -initSchema -dbType mysql -verbose
结果：schematool completed

```

##### 第五大步：启动

```
1、编写启动脚本
运行：[atguigu@hadoop102 hive]$ vim $HIVE_HOME/bin/hiveservices.sh
添加：
#!/bin/bash
HIVE_LOG_DIR=$HIVE_HOME/logs
if [ ! -d $HIVE_LOG_DIR ]
then
	mkdir -p $HIVE_LOG_DIR
fi
#检查进程是否运行正常，参数1为进程名，参数2为进程端口
function check_process()
{
    pid=$(ps -ef 2>/dev/null | grep -v grep | grep -i $1 | awk '{print $2}')
    ppid=$(netstat -nltp 2>/dev/null | grep $2 | awk '{print $7}' | cut -d '/' -f 1)
    echo $pid
    [[ "$pid" =~ "$ppid" ]] && [ "$ppid" ] && return 0 || return 1
}

function hive_start()
{
    metapid=$(check_process HiveMetastore 9083)
    cmd="nohup hive --service metastore >$HIVE_LOG_DIR/metastore.log 2>&1 &"
    cmd=$cmd" sleep 4; hdfs dfsadmin -safemode wait >/dev/null 2>&1"
    [ -z "$metapid" ] && eval $cmd || echo "Metastroe服务已启动"
    server2pid=$(check_process HiveServer2 10000)
    cmd="nohup hive --service hiveserver2 >$HIVE_LOG_DIR/hiveServer2.log 2>&1 &"
    [ -z "$server2pid" ] && eval $cmd || echo "HiveServer2服务已启动"
}

function hive_stop()
{
    metapid=$(check_process HiveMetastore 9083)
    [ "$metapid" ] && kill $metapid || echo "Metastore服务未启动"
    server2pid=$(check_process HiveServer2 10000)
    [ "$server2pid" ] && kill $server2pid || echo "HiveServer2服务未启动"
}

case $1 in
"start")
    hive_start
    ;;
"stop")
    hive_stop
    ;;
"restart")
    hive_stop
    sleep 2
    hive_start
    ;;
"status")
    check_process HiveMetastore 9083 >/dev/null && echo "Metastore服务运行正常" || echo "Metastore服务运行异常"
    check_process HiveServer2 10000 >/dev/null && echo "HiveServer2服务运行正常" || echo "HiveServer2服务运行异常"
    ;;
*)
    echo Invalid Args!
    echo 'Usage: '$(basename $0)' start|stop|restart|status'
    ;;
esac

2、给执行权限
[atguigu@hadoop102 hive]$ chmod +x $HIVE_HOME/bin/hiveservices.sh

3、启动
[atguigu@hadoop102 hive]$ hiveservices.sh start
```

#### 7、访问hive客户端

方式一：HiveJDBC访问

```
1、启动beeline客户端
运行：[atguigu@hadoop102 hive]$ bin/beeline -u jdbc:hive2://hadoop102:10000 -n atguigu
结果：
Connecting to jdbc:hive2://hadoop102:10000
Connected to: Apache Hive (version 3.1.2)
Driver: Hive JDBC (version 3.1.2)
Transaction isolation: TRANSACTION_REPEATABLE_READ
Beeline version 3.1.2 by Apache Hive
0: jdbc:hive2://hadoop102:10000>

```

方式二：hive访问

```
1、启动hive客户端
运行：[atguigu@hadoop102 hive]$ bin/hive
结果：
which: no hbase in (/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/opt/module/jdk1.8.0_212/bin:/opt/module/hadoop-3.1.3/bin:/opt/module/hadoop-3.1.3/sbin:/opt/module/hive/bin:/home/atguigu/.local/bin:/home/atguigu/bin)
Hive Session ID = 36f90830-2d91-469d-8823-9ee62b6d0c26

Logging initialized using configuration in jar:file:/opt/module/hive/lib/hive-common-3.1.2.jar!/hive-log4j2.properties Async: true
Hive Session ID = 14f96e4e-7009-4926-bb62-035be9178b02
hive>
```

#### 8、hive客户端常用命令

概览

```
[atguigu@hadoop102 hive]$ bin/hive -help
usage: hive
 -d,--define <key=value>          Variable subsitution to apply to hive
                                  commands. e.g. -d A=B or --define A=B
    --database <databasename>     Specify the database to use
 -e <quoted-query-string>         SQL from command line
 -f <filename>                    SQL from files
 -H,--help                        Print help information
    --hiveconf <property=value>   Use value for given property
    --hivevar <key=value>         Variable subsitution to apply to hive
                                  commands. e.g. --hivevar A=B
 -i <filename>                    Initialization SQL file
 -S,--silent                      Silent mode in interactive shell
 -v,--verbose                     Verbose mode (echo executed SQL to the console)
```

常用

```
1、“-e”不进入hive的交互窗口执行sql语句
[atguigu@hadoop102 hive]$ bin/hive -e "select id from student;"

2、“-f”执行脚本中sql语句
在/opt/module/hive/下创建datas目录并在datas目录下创建hivef.sql文件
[atguigu@hadoop102 datas]$ touch hivef.sql
文件中写入正确的sql语句
select *from student;
运行：[atguigu@hadoop102 hive]$ bin/hive -f /opt/module/hive/datas/hivef.sql
运行并把结果写入文件：
[atguigu@hadoop102 hive]$ bin/hive -f /opt/module/hive/datas/hivef.sql  > /opt/module/datas/hive_result.txt

3、退出
hive(default)>quit;

4、在hive cli命令窗口中如何查看hdfs文件系统
hive(default)>dfs -ls /;

5、查看在hive中输入的所有历史命令
（1）进入到当前用户的根目录/root或/home/atguigu
（2）查看. hivehistory文件
[atguigu@hadoop102 ~]$ cat .hivehistory

6、启动元数据服务：hive --service metastore &
nohup hive --service metastore 2>&1 &
nohup bin/hive --service metastore >/dev/null 2>&1 &

7、显示当前操作的数据库：set hive.cli.print.current.db=true;

8、设置队列：set mapreduce.job.queuename=hive;

9、设置简单任务不走mr:set hive.fetch.task.conversion=more

10、任何任务都走mr:set hive.fetch.task.conversion=minimal

11、yarn application -list查看yarn的内部运行进程

12、显示表头 set hive.cli.print.header=true  

<property>
<name>hive.cli.print.current.db</name>
<value>true</value>
</property>
<property>
<name>hive.cli.print.header</name>
<value>true</value>
</property>

13、select unix_timestamp('2019-08-02 15:48:18','yyyy-MM-dd HH:mm:ss')

14、set hive.server2.logging.operation.level=NONE
hive.server2.logging.operation.level=EXECUTION

conda deactivate

vsmdcpbustkgeaib
```



#### 9、hive修改日志文件位置

```
说明：Hive的log默认存放在/tmp/atguigu/hive.log目录下（当前用户名下）

1、修改配置文件名
运行：[atguigu@hadoop102 conf]$ mv hive-log4j2.properties.template hive-log4j.properties

2、在hive-log4j.properties文件中修改log存放位置
修改：hive.log.dir=/opt/module/hive/logs

```

#### 10、参数配置说明

```
1、查看所有配置：hive>set;

2、配置设置方法一：配置文件方式
默认配置文件：hive-default.xml 
用户自定义配置文件：hive-site.xml

3、命令行配置（仅对本次有效）
[atguigu@hadoop103 hive]$ bin/hive -hiveconf mapred.reduce.tasks=10;
查看：hive (default)> set mapred.reduce.tasks;

4、内部参数配置（仅对本次有效）
hive (default)> set mapred.reduce.tasks=100;
查看：hive (default)> set mapred.reduce.tasks;

说明：三种方式优先级依次递增
```

#### 11、hive数据类型

基本数据类型

| Hive数据类型 | Java数据类型 | 长度                                                 | 例子                                 |
| ------------ | ------------ | ---------------------------------------------------- | ------------------------------------ |
| TINYINT      | byte         | 1byte有符号整数                                      | 20                                   |
| SMALINT      | short        | 2byte有符号整数                                      | 20                                   |
| INT          | int          | 4byte有符号整数                                      | 20                                   |
| BIGINT       | long         | 8byte有符号整数                                      | 20                                   |
| BOOLEAN      | boolean      | 布尔类型，true或者false                              | TRUE  FALSE                          |
| FLOAT        | float        | 单精度浮点数                                         | 3.14159                              |
| DOUBLE       | double       | 双精度浮点数                                         | 3.14159                              |
| STRING       | string       | 字符系列。可以指定字符集。可以使用单引号或者双引号。 | ‘now is the time’ “for all good men” |
| TIMESTAMP    |              | 时间类型                                             |                                      |
| BINARY       |              | 字节数组                                             |                                      |

集合数据类型

| 数据类型 | 描述                                                         | 语法示例                                       |
| -------- | ------------------------------------------------------------ | ---------------------------------------------- |
| STRUCT   | 和c语言中的struct类似，都可以通过“点”符号访问元素内容。例如，如果某个列的数据类型是STRUCT{first STRING, last STRING},那么第1个元素可以通过字段.first来引用。 | struct()例如struct<street:string, city:string> |
| MAP      | MAP是一组键-值对元组集合，使用数组表示法可以访问数据。例如，如果某个列的数据类型是MAP，其中键->值对是’first’->’John’和’last’->’Doe’，那么可以通过字段名[‘last’]获取最后一个元素 | map()例如map<string, int>                      |
| ARRAY    | 数组是一组具有相同类型和名称的变量的集合。这些变量称为数组的元素，每个数组元素都有一个编号，编号从零开始。例如，数组值为[‘John’, ‘Doe’]，那么第2个元素可以通过数组名[1]进行引用。 | Array()例如array<string>                       |

类型转化

```
1、隐式转换：
TINYINT可以转换成INT，INT可以转换成BIGINT
所有整数类型、FLOAT和STRING类型都可以隐式地转换成DOUBLE
TINYINT、SMALLINT、INT都可以转换为FLOAT
BOOLEAN类型不可以转换为任何其它的类型

2、强行转换：
select cast('1' as int); 
说明：转换失败会返回null


如何在声明结构时指明分割字段：

row format delimited fields terminated by ' '  列分隔符

collection items terminated by '_' map struct array 之间的分隔符

map keys terminated by ':' map中k-v之间的分割符

lines terminated by '\n' 行分隔符
```

#### 12、ddl操作

```
1、创建库 
create database if not exists homeworks
comment "homework of zhangyubo";
说明：默认存储在/user/hive/warehouse下，(这是在配置文件中自己设置的)
2、删除库 
drop database if exists homeworks; 	删除空的数据库
drop database if exists homeworks cascade; 	删除非空的数据库
说明：若库不为空，则使用cascade

3、修改库 
alter database db_hive set dbproperties('createtime'='20170830');
说明：很少用，可修改的只有属性，而属性应用太少

4、查询库 
show databases; 	显示所有数据库
show databases like 'home*';	模糊查询后显示
desc database mydb; 	显示数据库的信息
desc database extended mydb; 	显示数据库的详细信息

5、切换库
use homeworks;


1、创建表
create table if not exists t1(id int,name string) 	指明属性
row format delimited fields terminated by '\t'		指明分割符
stored as textfile;									指明使用文件作为表的真实数据

create table if not exists t2 as select id,name from t1;	根据查询结果

create table if not exists t3 like t1;				根据其他表结构
说明：有管理表和外部表之分，管理表hive认为自己有权限，删除后真实数据删除，外部表hive认为自己没有权限，删除后真实数据还在，前面创建的均是管理表，要想创建外部表，需要在create后增加external,系统默认创建的表就是管理表
2、删除表
drop table t1;

3、修改表
alter table t1 set tblproperties('EXTERNAL'='TRUE');	修改表为外部表
alter table t1 set tblproperties('EXTERNAL'='FALSE');	修改表为内部表
alter table t1 rename to t4;							重命名
alter table t1 add columns(nition string);				增加列
alter table t1 change column id ID string;				更新列
alter table t1 replace columns(name string);			替换列，类型依次一致


4、查询表
show tables;
desc t1;
desc formatted t1;

5、清空表数据：Truncate只能删除管理表，不能删除外部表中数据
truncate table student;

建表说明：
--如果建表时，没有location具体指定，则表对放到当前所使用的库目录下,比如默认建库的路径是create database test; /user/hive/warehouse/test.db
如果use test;create table first(id string);则first表的路径默认是：/user/hive/warehouse/test.db/first
--如果建表时，通过location具体指定，则表就对应location指定的目录
--如果建表时，使用的是default库，则表直接放到/user/hive/warehouse下
-- 管理表(内部表):管理表会控制数据的生命周期，删除管理表会将数据从HDFS一并删除
-- 外部表:        外部表不会控制数据的生命周期， 删除外部表不会将数据从HDFS删除
```

#### 13、dml操作

上传(导入)

```
1、load:
load data local inpath '/home/atguigu/student.txt' into table t1;	本地
load data inpath '/user/hive/warehouse/homeworks.db/t1/student.txt' into table t2;	hdfs
以上均为追加操作，如果想实现覆盖操作需要在into前增加overwrite


2、insert
insert into table t2 values(1016,'wujunru');			追加
insert overwrite table t2 values(1016,'wangzhaojun'); 	覆盖
insert into table t1 select id,name from t2 where id=1015; 追加
from student
              insert overwrite table student partition(month='201707')
              select id, name where month='201709'
              insert overwrite table student partition(month='201706')
              select id, name where month='201709';		多表多分区

3、import from
import table student2  from '/user/hive/warehouse/export/student'; 	只能导入由export导出的


```

下载(导出)

```
insert：

insert overwrite local directory 
'/opt/module/hive/datas/export/student'				本地路径
row format delimited fields terminated by '\t'		带格式
select * from student;								查询结果

hadoop 
dfs -get /user/a.txt /~/st

hive shell
bin/hive -e 'select * from t1' > /~/st

export to
export table t1 to '/user/hive/warehouse/st';   export和import主要用于两个Hadoop平台集群之间Hive表迁移

sqoop
```

#### 14、查询

```
select * from t1;
```

limit

```
select * from t1 limit 1,5;		找出第1,2,3,4,5行的数据
说明：第一个参数指明起始行，有0行，后一个参数指明需要几条数据
```

where 

```
select * from t2 where id > 1010;
```

<=>

```
select null<=>null;			true
select null<=>'a';			false
select 1<=>null; 			false
```

in

```
select * from t2 where id in(1010,1013);				枚举显示
```

like

```
%代表任意个字符
_代表一个字符


select * from t2 where name like 'z%';			以z开头
select * from t2 where name like '%z';			以z结尾
select * from t2 where name like '_z%;			第二个是z
select * from t2 where name like '%a%';			包含a
```

rlike

```
RLIKE : 通过正则进行匹配.  
常用正则符号:
\ :  转义
^ :  从头匹配
$ :  匹配结尾
* :  0~n
[]:  表示范围  [a-z]  [0-9]  [a-z,0-9]
```

group by

```
select name from t3 group by name;				查询都有什么人，连null值也会显示
select name,count(*) from t3 group by name;		每个组的数据条数
说明：前后应该有相同字段
```

hiving

```
说明：
1、where后面不能写分组函数，而having后面可以使用分组函数。
2、having只用于group by分组统计语句。（mysql中可以，但是效率低不常用，hive中是直接不让用）

select name，avg(cost) avg_cost from t3 group by name having avg_cost >= 40;	平均消费大于40

```

join on

```
说明：Hive支持通常的SQL JOIN语句，但是只支持等值连接，不支持非等值连接。

select e.empno, e.ename, d.deptno, d.dname
from emp e 
join dept d
on e.deptno = d.deptno;
```

left join on

```
select e.empno, e.ename, d.deptno, d.dname
from emp e 
left join dept d
on e.deptno = d.deptno;
说明：左面表中数据全部显示，包括null
```

right join on

```
select e.empno, e.ename, d.deptno, d.dname
from emp e 
right join dept d
on e.deptno = d.deptno;
说明：右面表中数据全部显示，包括null
```

full join on

```
select e.empno, e.ename, d.deptno, d.dname
from emp e 
full join dept d
on e.deptno = d.deptno;
说明：两个表中所有数据均显示，包括null
```

#### 15、自定义函数

UDF（User-Defined-Function）一进一出

```
步骤：
1、创建一个Maven工程Hive 
2、导入依赖
<dependencies>
		<dependency>
			<groupId>org.apache.hive</groupId>
			<artifactId>hive-exec</artifactId>
			<version>3.1.2</version>
		</dependency>
</dependencies>
3、创建一个类封装函数逻辑（继承udf,重写evaluate)

4、打包上传/opt/module/hive/datas/myudf.jar

5、将jar包添加到hive的classpath：add jar /opt/module/hive/datas/myudf.jar;

6、创建临时函数与开发好的java class关联：

create temporary function mylower as "com.atguigu.hive.Lower";	注意是全类名

7、使用：select ename, mylower(ename) lowername from emp;
```

UDAF（User-Defined Aggregation Function）多进一出 	类似count()

```
步骤同上
```

UDTF（User-Defined Table-Generating Functions）一进多出 	类似later view explode()

```
步骤同上
```



#### 16、排序

order by	全局排序

```
select * from t3 order by cost;			整个表排序，只有一个reduce
select * from t3 order by name,cost;

说明：asc 升序，desc 降序（默认升序）；order by在整个句子的结尾,一般不用

```

sort by 	排序

```
说明：为每个reducer产生一个排好序的文件，Reducer内部是有序的，全局不做排序

设置reduce数量：set mapreduce.job.reduces=3;
查看reduce数量：set mapreduce.job.reduces;

select * from t3 sort by cost;		分批次排序（先设置reduces=3,不然没效果）

```

distribute by	分区

```
select * from t3 distribute by name sort by cost;   

说明：distribute by 的分区规则是分区字段的hash值与分区数取模后，余数相同的分到一个区，hive要求distribute by 一定要配合 sort by 使用
```

cluster by

```
说明：当distribute by和sorts by字段相同时，可以使用cluster by方式,但是不能倒序，默认是升序

select * from t3 cluster by name;	

```

#### 17、内置函数

```
显示：show functions;

查看：desc function upper;

查看详情：desc function extended upper;

```

nvl

```
nvl给null赋值：select * nvl(name,"zhangyubo") from t3; 如果name有null值，用"zhangyubo"代替
```

case when else

```
case when else 一次要统计多个情况时：
select id,sum(case sex when '男' then 1 else 0 end)man,
sum(case sex when '女' then 1 else 0 end)woman from t4
group by id;				每个部门的男生有几个，女生有几个

id man woman
A  1   2
B  2   1
```

concat

```
concat连接：select concat('a',',','b',',','c');		a,b,c在中间声明分割符
```

concat_ws

```
concat_ws连接：select concat_ws('a','b','c','d','e');	bacadae 首位声明分割符
说明：它是一个特殊形式的 CONCAT()。第一个参数声明参数间的分隔符。分隔符可以是与后面参数一样的字符串。如果分隔符是 NULL，返回值也将为 NULL。这个函数会跳过分隔符参数后的任何 NULL 和空字符串。分隔符将被加到被连接的字符串之间

select t1.year,t2.amount from(
	select year,concat_ws(',',m1,m2,m3,m4) str from A
)t1 
lateral view explode(split(t1.str,','))t2 as amount

select id,sub,score,rk from(
select id,sub,score,row_number() over(partition by sub order by score desc) rk from score
)t1
where rk=1;
select id,sub,score,row_number() over(partition by sub order by score desc) rk from score;
```

collect_set

```
collect_set基本数据类型去重：select collect_set(name);   '"jack","tony","mart","neil"'
```

lateral view explode as

```
lateral view explode as炸裂：
select t5.name,t6.type from t5 lateral view explode(split(type,','))t6 as type;

select t.id,t6.type from
(select id,concat_ws(',',collect_set(sub)) as rk from score group by id)t
lateral view explode(split(rk,','))t6 as type
 select id,concat_ws(',',collect_set(sub)) as rk from score group by id;
```

其他函数

```
截取字符串：substr(‘abcde’,3) 返回字符串从3位置(c)到结尾的字符串 返回cde
截取字符串：substr(‘abcde’,3,2) 返回cd

大小写转换：
upper("abD") 转大写
lower("abD") 转小写

去除空格：
trim(" abc ") 去除字符串

分割字符串得到字符串数组：
split(string str, string pat) 作用：按照pat字符串分割str，返回分割后的字符串数组

补足字符串：
lpad(string str, int len, string pad) 作用：将str进行用pad进行左补足到len位
rpad(string str, int len, string pad) 作用：将str进行用pad进行右补足到len位
```



#### 18、窗口函数

说明1：行数不变，列数加一

```
作用：增加一列统计信息；不会对原来的查询数据有影响，即行数不变，列数加一

例子：
jack,2017-01-01,10
tony,2017-01-02,15
jack,2017-02-03,23
tony,2017-01-04,29
jack,2017-01-05,46
jack,2017-04-06,42
tony,2017-01-07,50
jack,2017-01-08,55
mart,2017-04-08,62
mart,2017-04-09,68
neil,2017-05-10,12
mart,2017-04-11,75
neil,2017-06-12,80
mart,2017-04-13,94

过滤
select name from business where substring(orderdate,1,7) = '2017-04';
jack
mart
mart
mart
mart

分组聚合
select name from business where substring(orderdate,1,7) = '2017-04' group by name;
mart
jack

增加一列信息：总人数
select name,count(1) over() from business where substring(orderdate,1,7) = '2017-04' group by name;
mart	2
jack	2
相当于把上一个查询结果当成一个窗口，整个聚合

select name,count(1) over(partition by name) from business where substring(orderdate,1,7) = '2017-04' group by name;
mart	1
jack	1
相当于把上一个查询结果中相同name当成一个窗口聚合

select id from(
select id,sum(if(sub = '语文',1,0)) + sum(if(sub = '英语',1,0)) + sum(if(sub = '数学',1,0))c from score group by id
)t where c=3;

select id from(
select id,sum(if(sub = '语文',1,0)) + sum(if(sub = '英语',1,0))c1,sum(if(sub = '数学',1,0))c2 from score group by id
)t where c1=2 and c2=0;

```

说明2：窗口大小partition 和 order 

```
1、sum(cost) over(partition by name):
相同名字的当成一个窗口，然后聚合这个窗口内的参数（每个顾客）

2、sum(cost) over(partition by name,month(orderdate))
同一个名字，相同月份当成一个窗口，然后聚合这个窗口内的参数（每个顾客每月）

3、sum(cost) over(order by month(orderdate))
先按月份排序，从开头月份到当前月份形成一个窗口，聚合窗口内的参数（按月累计）

select unix_timestamp('1970-01-01 00:00:18','yyyy-MM-dd HH:mm:ss');
```

说明3：窗口控制rows

```
CURRENT ROW 当前行 current row
n PRECEDING 往前n行 n preceding
n FOLLOWING 往后n行 n following
UNBOUNDED：起点，UNBOUNDED PRECEDING 表示从前面的起点， UNBOUNDED FOLLOWING表示到后面的终点 unbounded
LAG(col,n,default_val)：往前第n行数据 lag
LEAD(col,n, default_val)：往后第n行数据 lead
NTILE(n)：把有序窗口的行分发到指定数据的组中，各个组有编号，编号从1开始，对于每一行，NTILE返回此行所属的组的编号。注意：n必须为int类型。 ntile

select name,orderdate,cost,
sum(cost) over(),
sum(cost) over(partition by name),
sum(cost) over(partition by name order by orderdate),
sum(cost) over(partition by name order by orderdate rows between unbounded preceding and current row),
sum(cost) over(partition by name order by orderdate rows between 1 preceding and current row),
sum(cost) over(partition by name order by orderdate rows between 1 preceding and 1 following),
sum(cost) over(partition by name order by orderdate rows between current row and unbounded following)
from business;

jack	2017-01-01	10	661	176	10	10	10	56	176
jack	2017-01-05	46	661	176	56	56	56	111	166
jack	2017-01-08	55	661	176	111	111	101	124	120
jack	2017-02-03	23	661	176	134	134	78	120	65
jack	2017-04-06	42	661	176	176	176	65	65	42
mart	2017-04-08	62	661	299	62	62	62	130	299
mart	2017-04-09	68	661	299	130	130	130	205	237
mart	2017-04-11	75	661	299	205	205	143	237	169
mart	2017-04-13	94	661	299	299	299	169	169	94
neil	2017-05-10	12	661	92	12	12	12	92	92
neil	2017-06-12	80	661	92	92	92	92	92	80
tony	2017-01-02	15	661	94	15	15	15	44	94
tony	2017-01-04	29	661	94	44	44	44	94	79
tony	2017-01-07	50	661	94	94	94	79	79	50

解释：名字	日期	当天消费	总计	每人总计	每人累计	每人累计	每人两天总计	每人3天总计	每人累计倒序

```

over

```
over开窗：
select name,count(*) from t3 group by name;		使用group by统计有关组的信息
select name,count(*) over(partition by name) from t3; 统计组中每个数据的信息

```

current row	n preceding	n following	unbounded preceding	unbounded following

lag(name,n,'zhangyubo')	lead(name,n,'zhangyubo')	ntile(n)

```
select *,
sum(cost) 
over(partition by name order by cost rows between 1 following and current row)cost1
from t3;		当前行和前一行
```

```
select *,
sum(cost) 
over(partition by name order by cost rows between current row and 1 following)cost1
from t3;		当前行和下一行
```

```
select *,
sum(cost) 
over(partition by name order by cost rows between 1 preceding and 1 following)cost1
from t3;		前一行、当前行、下一行
```

```
select *,
sum(cost) 
over(partition by name order by cost rows between unbounded preceding and current row)cost1
from t3;		起点到当前行
```

```
select *,
sum(cost) 
over(partition by name order by cost rows between current row and unbounded following)cost1
from t3;		当前行到末尾
```

```
select *,
lag(cost,1,0) 
over(partition by name order by cost)cost1
from t3;		前一行数据，没有就用0填充
```

```
select *,
lead(cost,1,0) 
over(partition by name order by cost)cost1
from t3;		后一行数据，没有就用0填充
```

```
select * from(
select *,ntile(5) over(order by cost) num from t3)t
where t.num = 1;		
对数据分组，分成5组，可以用来求前20%数据的问题
```

##### rank	dense_rank 	row_number

```
select name,sex,id,rank() over(partition by sex order by id)num from t4;

说明：会有并列第一等情况，下一名次会有跳跃

| 凤姐    | 女    | A   | 1              |
| 婷姐    | 女    | B   | 2              |
| 婷婷    | 女    | B   | 2              |
| 悟空    | 男    | A   | 1              |
| 大海    | 男    | A   | 1              |
| 宋宋    | 男    | B   | 3              |
```

```
select name,sex,id,dense_rank() over(partition by sex order by id)num from t4;

说明：会有并列第一等情况，下一名次不会跳跃

| 凤姐    | 女    | A   | 1    |
| 婷姐    | 女    | B   | 2    |
| 婷婷    | 女    | B   | 2    |
| 悟空    | 男    | A   | 1    |
| 大海    | 男    | A   | 1    |
| 宋宋    | 男    | B   | 2    |

```

```
select name,sex,id,row_number() over(partition by sex order by id)num from t4;

说明：不会有并列，名次顺延

| 凤姐    | 女    | A   | 1    |
| 婷姐    | 女    | B   | 2    |
| 婷婷    | 女    | B   | 3    |
| 悟空    | 男    | A   | 1    |
| 大海    | 男    | A   | 2    |
| 宋宋    | 男    | B   | 3    |

```

##### 日期相关

```
current_date 当前日期
date_add(current_date,90)  当前日期的前90天
date_sub(current_date,90)  当前日期的后90天
datediff(current_date,'2020-06-27') 天数差

时间转时间戳ms：
select unix_timestamp(); --获得当前时区的UNIX时间戳

select unix_timestamp(‘2017-09-15 14:23:00’); 

select unix_timestamp(‘2017-09-15 14:23:00’,‘yyyy-MM-dd HH:mm:ss’);

select unix_timestamp(‘20170915 14:23:00’,‘yyyyMMdd HH:mm:ss’); 

时间戳ms转时间
select from_unixtime(1505456567); 

select from_unixtime(1505456567,‘yyyyMMdd’); 

select from_unixtime(1505456567,‘yyyy-MM-dd HH:mm:ss’); 

select from_unixtime(unix_timestamp(),‘yyyy-MM-dd HH:mm:ss’); --获取系统当前时间

获取当前日期：
select current_date from dual 2017-09-15

时间转日期：
select to_date(‘2017-09-15 11:12:00’) from dual; 2017-09-15

获取日期中的年/月/日/时/分/秒/周
with dtime as(select from_unixtime(unix_timestamp(),‘yyyy-MM-dd HH:mm:ss’) as dt)

select year(dt),month(dt),day(dt),hour(dt),minute(dt),second(dt),weekofyear(dt) from dtime

计算两个日期之间的天数：
select datediff(‘2017-09-15’,‘2017-09-01’) from dual; 

日期增加和减少:
hive> select date_add(‘2017-09-15’,1) from dual;    
2017-09-16

hive> select date_sub(‘2017-09-15’,1) from dual;    
2017-09-14
```



#### 19、文件压缩和存储

map输出

```
开启hive中间传输数据压缩功能：
hive (default)>set hive.exec.compress.intermediate=true;

开启mapreduce中map输出压缩功能
hive (default)>set mapreduce.map.output.compress=true;

设置mapreduce中map输出数据的压缩方式
hive (default)>set mapreduce.map.output.compress.codec=
 org.apache.hadoop.io.compress.SnappyCodec;

```

reduce输出

```
开启hive最终输出数据压缩功能
hive (default)>set hive.exec.compress.output=true;

开启mapreduce最终输出数据压缩
hive (default)>set mapreduce.output.fileoutputformat.compress=true;

设置mapreduce最终数据输出压缩方式
hive (default)> set mapreduce.output.fileoutputformat.compress.codec =org.apache.hadoop.io.compress.SnappyCodec;
 
设置mapreduce最终数据输出压缩为块压缩
hive (default)> set mapreduce.output.fileoutputformat.compress.type=BLOCK;

```

存储说明

```
1、TEXTFILE和SEQUENCEFILE的存储格式都是基于行存储的；

2、ORC和PARQUET是基于列式存储的。
```

Textfile

```
默认格式，数据不做压缩，磁盘开销大，数据解析开销大。可结合Gzip、Bzip2使用，但使用Gzip这种方式，hive不会对数据进行切分，从而无法对数据进行并行操作。
```

orc

```
每个Orc文件由1个或多个stripe组成，每个stripe一般为HDFS的块大小，每一个stripe包含多条记录，这些记录按照列进行独立存储，对应到Parquet中的row group的概念。每个Stripe里有三部分组成，分别是Index Data，Row Data，Stripe Footer：

1）Index Data：一个轻量级的index，默认是每隔1W行做一个索引。这里做的索引应该只是记录某行的各字段在Row Data中的offset。
2）Row Data：存的是具体的数据，先取部分行，然后对这些行按列进行存储。对每个列进行了编码，分成多个Stream来存储。
3）Stripe Footer：存的是各个Stream的类型，长度等信息。
每个文件有一个File Footer，这里面存的是每个Stripe的行数，每个Column的数据类型信息等；每个文件的尾部是一个PostScript，这里面记录了整个文件的压缩类型以及FileFooter的长度信息等。在读取文件时，会seek到文件尾部读PostScript，从里面解析到File Footer长度，再读FileFooter，从里面解析到各个Stripe信息，再读各个Stripe，即从后往前读。
```

parquet

```
Parquet文件是以二进制方式存储的，所以是不可以直接读取的，文件中包括该文件的数据和元数据，因此Parquet格式文件是自解析的。

（1）行组(Row Group)：每一个行组包含一定的行数，在一个HDFS文件中至少存储一个行组，类似于orc的stripe的概念。
（2）列块(Column Chunk)：在一个行组中每一列保存在一个列块中，行组中的所有列连续的存储在这个行组文件中。一个列块中的值都是相同类型的，不同的列块可能使用不同的算法进行压缩。
（3）页(Page)：每一个列块划分为多个页，一个页是最小的编码的单位，在同一个列块的不同页可能使用不同的编码方式。
通常情况下，在存储Parquet数据的时候会按照Block大小设置行组的大小，由于一般情况下每一个Mapper任务处理数据的最小单位是一个Block，这样可以把每一个行组由一个Mapper任务处理，增大任务执行并行度。Parquet文件的格式。

一个文件中可以存储多个行组，文件的首位都是该文件的Magic Code，用于校验它是否是一个Parquet文件，Footer length记录了文件元数据的大小，通过该值和文件长度可以计算出元数据的偏移量，文件的元数据中包括每一个行组的元数据信息和该文件存储数据的Schema信息。除了文件中每一个行组的元数据，每一页的开始都会存储该页的元数据，在Parquet中，有三种类型的页：数据页、字典页和索引页。数据页用于存储当前行组中该列的值，字典页存储该列值的编码字典，每一个列块中最多包含一个字典页，索引页用来存储当前行组下该列的索引，目前Parquet中还不支持索引页。
```

#### 20、分区表和分桶表

创建分区表：

```
create table dept_partition(
deptno int, dname string, loc string
)
partitioned by (day string)
row format delimited fields terminated by '\t';
```

上传数据指明分区

```
load data local inpath '/opt/module/hive/datas/dept_20200401.log' into table dept_partition partition(day='20200401');

分区表加载数据时，必须指定分区
```

增加分区

```
alter table dept_partition add partition(day='20200405') partition(day='20200406');
```

删除分区

```
alter table dept_partition drop partition (day='20200404'), partition(day='20200405');
```

查看分区

```
show partitions t3;

desc formatted t3;

select id,sum(if(sex='男',1,0)),sum(if(sex='女',1,0)) from sex group by id;

B	1	2
A	2	1

```

二级分区

```
create table dept_partition2(
               deptno int, dname string, loc string
               )
               partitioned by (day string, hour string)
               row format delimited fields terminated by '\t';
               
load data local inpath '/opt/module/hive/datas/dept_20200401.log' into table
dept_partition2 partition(day='20200401', hour='12');
```

分区与数据的联系

```
说明：增加分区会增加表文件夹下的文件夹数目（符合分区的数据和分区挂钩）


上传数据后手动对应
创建文件夹
hive (default)> dfs -mkdir -p
 /user/hive/warehouse/mydb.db/dept_partition2/day=20200401/hour=13;
放数据
hive (default)> dfs -put /opt/module/datas/dept_20200401.log  /user/hive/warehouse/mydb.db/dept_partition2/day=20200401/hour=13;
手动对应
hive (default)> msck repair table dept_partition2;


上传数据后添加分区
创建文件夹并上传数据
hive (default)> dfs -mkdir -p
 /user/hive/warehouse/mydb.db/dept_partition2/day=20200401/hour=14;
hive (default)> dfs -put /opt/module/hive/datas/dept_20200401.log  /user/hive/warehouse/mydb.db/dept_partition2/day=20200401/hour=14;
创建分区
hive (default)> alter table dept_partition2 add partition(day='201709',hour='14');

直接上传至分区
创建目录
dfs -mkdir -p
/user/hive/warehouse/mydb.db/dept_partition2/day=20200401/hour=15;

上传数据至分区
hive (default)> load data local inpath '/opt/module/hive/datas/dept_20200401.log' into table
 dept_partition2 partition(day='20200401',hour='15');

```

动态分区

```
（1）开启动态分区功能（默认true，开启）
hive.exec.dynamic.partition=true
（2）设置为非严格模式（动态分区的模式，默认strict，表示必须指定至少一个分区为静态分区，nonstrict模式表示允许所有的分区字段都可以使用动态分区。）
hive.exec.dynamic.partition.mode=nonstrict
（3）在所有执行MR的节点上，最大一共可以创建多少个动态分区。默认1000
hive.exec.max.dynamic.partitions=1000
（4）在每个执行MR的节点上，最大可以创建多少个动态分区。该参数需要根据实际的数据来设定。比如：源数据中包含了一年的数据，即day字段有365个值，那么该参数就需要设置成大于365，如果使用默认值100，则会报错。
hive.exec.max.dynamic.partitions.pernode=100
（5）整个MR Job中，最大可以创建多少个HDFS文件。默认100000
hive.exec.max.created.files=100000
（6）当有空分区生成时，是否抛出异常。一般不需要设置。默认false
hive.error.on.empty.partition=false


说明：创建时指定根据表中的某个字段进行分区
```

创建分桶表

```
create table stu_buck(id int, name string)
clustered by(id) 
into 4 buckets
row format delimited fields terminated by '\t';
```

传数据至分桶表

```
load data local inpath   '/opt/module/hive/datas/student.txt' into table stu_buck;
```

规则

```
Hive的分桶采用对分桶字段的值进行哈希，然后除以桶的个数求余的方式决定该条记录存放在哪个桶当中
```

#### 21、抽样查询

```
hive (default)> select * from stu_buck tablesample(bucket 1 out of 4 on id);

说明：数据随机分成4分，要第一份数据

hive (default)> select * from stu_buck tablesample(bucket 2 out of 4 on id);

说明：数据随机分成4分，要第二份数据

注意：前一个值小于后一个值
```

#### 22、调优

#### 23、复杂数据类型

```
array<bigint>
map<string,bigint>
struct<id:int,name:string,age:int>
```

#### 24、hive配置多队列

```
1、默认是容量调度器，只有一个队列default
vim /opt/module/hadoop-3.1.3/etc/hadoop/capacity-scheduler.xml

<property>
    <name>yarn.scheduler.capacity.root.queues</name>
    <value>default,hive</value>
    <description>
      The queues at the this level (root is the root queue).
    </description>
</property>
<property>
    <name>yarn.scheduler.capacity.root.default.capacity</name>
<value>50</value>
    <description>
      default队列的容量为50%
    </description>
</property>
<property>
    <name>yarn.scheduler.capacity.root.hive.capacity</name>
<value>50</value>
    <description>
      hive队列的容量为50%
    </description>
</property>

<property>
    <name>yarn.scheduler.capacity.root.hive.user-limit-factor</name>
<value>1</value>
    <description>
      一个用户最多能够获取该队列资源容量的比例
    </description>
</property>

<property>
    <name>yarn.scheduler.capacity.root.hive.maximum-capacity</name>
<value>80</value>
    <description>
      hive队列的最大容量
    </description>
</property>

<property>
    <name>yarn.scheduler.capacity.root.hive.state</name>
    <value>RUNNING</value>
</property>

<property>
    <name>yarn.scheduler.capacity.root.hive.acl_submit_applications</name>
<value>*</value>
    <description>
      访问控制，控制谁可以将任务提交到该队列
    </description>
</property>

<property>
    <name>yarn.scheduler.capacity.root.hive.acl_administer_queue</name>
<value>*</value>
    <description>
      访问控制，控制谁可以管理(包括提交和取消)该队列的任务
    </description>
</property>

<property>
    <name>yarn.scheduler.capacity.root.hive.acl_application_max_priority</name>
<value>*</value>
<description>
      访问控制，控制用户可以提交到该队列的任务的最大优先级
    </description>
</property>

<property>
    <name>yarn.scheduler.capacity.root.hive.maximum-application-lifetime</name>
<value>-1</value>
    <description>
      hive队列中任务的最大生命时长
</description>
</property>
<property>
    <name>yarn.scheduler.capacity.root.hive.default-application-lifetime</name>
<value>-1</value>
    <description>
      hive队列中任务的默认生命时长
</description>
</property>

2、分发：xsync /opt/module/hadoop-3.1.3/etc/hadoop/capacity-scheduler.xml

3、重新启动集群

4、设置任务提交的队列：mapreduce.job.queue.name=hive
```

#### 25、全量表和增量表

```
全量表：没有分区，全量表不能记录历史记录，每次往全量表中插入数据都会覆盖之前的数据
例如，23号的数据，24号数据来了之后会覆盖23号的数据，依次

快照表：有历史记录，累计型，有分区
例如，23号的数据是23号当天的数据，24号数据来是23、24号的数据，25号数据是23、24、25的数据，依次增加

增量表：记录的是每天新增的数据

拉链表：记录的是每天新增及变化的表


```

#### 26、让分区表和数据产生关联的三种方式

```
1、上传数据后修复
dfs -mkdir -p /user/hive/warehouse/dept_partition2/month=201709/day=12;
dfs -put /opt/module/datas/dept.txt  /user/hive/warehouse/dept_partition2/month=201709/day=12;
msck repair table dept_partition2;

2、上传后添加分区
dfs -mkdir -p /user/hive/warehouse/dept_partition2/month=201709/day=12;
dfs -put /opt/module/datas/dept.txt  /user/hive/warehouse/dept_partition2/month=201709/day=12;
alter table dept_partition2 add partition(month='201709',day='11');

3、手动创建分区文件夹然后上传数据
dfs -mkdir -p /user/hive/warehouse/dept_partition2/month=201709/day=10;
load data local inpath '/opt/module/datas/dept.txt' into table dept_partition2 partition(month='201709',day='10');

```

#### 27、动态分区

```
1、开启动态分区 hive.exec.dynamic.partition=true 默认是开启的

2、设置非严格模式 hive.exec.dynamic.partition.mode=nonstrict 默认是strict
```

#### 28、优化

```
1、小表join大表：新版的hive已经对小表JOIN大表和大表JOIN小表进行了优化。小表放在左边和右边已经没有明显区别。

2、如果不指定MapJoin或者不符合MapJoin的条件，那么Hive解析器会将Join操作转换成Common Join，即：在Reduce阶段完成join。容易发生数据倾斜。可以用MapJoin把小表全部加载到内存在map端进行join，避免reducer处理。

3、行列过滤
列处理：在SELECT中，只拿需要的列，如果有，尽量使用分区过滤，少用SELECT *。
行处理：在分区剪裁中，当使用外关联时，如果将副表的过滤条件写在Where后面，那么就会先全表关联，之后再过滤

4、列式存储

5、采用分区技术

6、合理设置map个数
mapred.min.split.size: 指的是数据的最小分割单元大小；min的默认值是1B
mapred.max.split.size: 指的是数据的最大分割单元大小；max的默认值是256MB
通过调整max可以起到调整map数的作用，减小max可以增加map数，增大max可以减少map数。
需要提醒的是，直接调整mapred.map.tasks这个参数是没有效果的。

7、合理设置reduce个数：
Reduce个数并不是越多越好
（1）过多的启动和初始化Reduce也会消耗时间和资源；
（2）另外，有多少个Reduce，就会有多少个输出文件，如果生成了很多个小文件，那么如果这些小文件作为下一个任务的输入，则也会出现小文件过多的问题；
在设置Reduce个数的时候也需要考虑这两个原则：处理大数据量利用合适的Reduce数；使单个Reduce任务处理数据量大小要合适；

8、小文件处理
(1)实用CombineHiveInputFormat可以对小文件进行合并
(2)合并小文件
SET hive.merge.mapfiles = true; -- 默认true，在map-only任务结束时合并小文件
SET hive.merge.mapredfiles = true; -- 默认false，在map-reduce任务结束时合并小文件
SET hive.merge.size.per.task = 268435456; -- 默认256M
SET hive.merge.smallfiles.avgsize = 16777216; -- 当输出文件的平均大小小于16m该值时，启动一个独立的map-reduce任务进行文件merge
(3)jvm重用set mapreduce.job.jvm.numtasks=10

9、开启Combiner(不影响最终业务的情况下)set hive.map.aggr=true；

10、启用压缩
set hive.exec.compress.intermediate=true --启用中间数据压缩
set mapreduce.map.output.compress=true --启用最终数据压缩
set mapreduce.map.outout.compress.codec=…; --设置压缩方式

11、实用tez或者spark引擎
```

### 30、数据倾斜

```
产生的可能原因：
1、不同数据类型导致
2、空值过多

解决办法：
1、group by

2、mapjoin

3、开启负载均衡
set hive.groupby.skewindata=true;
```

31、MR\Tez\Spark的优缺点

```
Mr/tez/spark区别：
Mr引擎：多job串联，基于磁盘，落盘的地方比较多。虽然慢，但一定能跑出结果。一般处理，周、月、年指标。

Spark引擎：虽然在Shuffle过程中也落盘，但是并不是所有算子都需要Shuffle，尤其是多算子过程，中间过程不落盘  DAG有向无环图。 兼顾了可靠性和效率。一般处理天指标。

Tez引擎：完全基于内存。  注意：如果数据量特别大，慎重使用。容易OOM。一般用于快速出结果，数据量比较小的场景。
```



#### 1、顾客

```
（1）查询在2017年4月份购买过的顾客及总人数
select name,
count(*) over()
from t3
where substring(buy_date,7,1)='4'
group by name;


（2）查询顾客的购买明细及月购买总额
select name,buy_date,cost,
sum(cost) over(partition by month(buy_date))
from t3;



（3）上述的场景, 将每个顾客的cost按照日期进行累加
select name,buy_date,cost,
sum(cost) over(order by buy_date)xin
from t3;




（4）查询每个顾客上次的购买时间
select name,buy_date,cost,
lag(buy_date,1,'2017-01-01') over(order by buy_date)time1 from t3;



select name,buy_date,cost,sum(cost)
over(partition by name order by cost rows between UNBOUNDED PRECEDING and current row) from t3

（5）查询前20%时间的订单信息
```

显示数据库：set hive.cli.print.current.db=true;

```
select get_json_object('{"name":"大郎","sex":"男","age":"25"}','$.name');  单条json
select get_json_object('[{"name":"chensi","age":"20"},{"name":"lijing","age":"19"}]','$[0].name'); 多条json
```

#### 2、电影

| 字段        | 备注                        | 详细描述               |
| ----------- | --------------------------- | ---------------------- |
| video id    | 视频唯一id（String）        | 11位字符串             |
| uploader    | 视频上传者（String）        | 上传视频的用户名String |
| age         | 视频年龄（int）             | 视频在平台上的整数天   |
| category    | 视频类别（Array<String>）   | 上传视频指定的视频分类 |
| length      | 视频长度（Int）             | 整形数字标识的视频长度 |
| views       | 观看次数（Int）             | 视频被浏览的次数       |
| rate        | 视频评分（Double）          | 满分5分                |
| Ratings     | 流量（Int）                 | 视频的流量，整型数字   |
| conments    | 评论数（Int）               | 一个视频的整数评论数   |
| related ids | 相关视频id（Array<String>） | 相关视频的id，最多20个 |

videoId string, 

uploader string, 

age int, 

 category array<string>, 

 length int, 

 views int, 

 rate float, 

 ratings int, 

 comments int,

 relatedId array<string>)

```
1、观看top10
select videoId,views from video_orc order by views desc limit 10;

2、类别top10
select distinct
concat(tmp.t,'-',sum(tmp.tv) over(partition by tmp.t))
from(
select e.type t,v.views tv from video_orc v lateral view explode(category) e as type
) tmp

3、类别包含的top10个数
select type,count(type) from
(select videoId,category,views,relatedId from video_orc order by views desc limit 20) tmp
lateral view explode(category) e as type
group by type

4、
select type2,count(type2) from
(select type,category from
(select videoId,category,views,relatedId from video_orc order by views desc limit 50) t1
lateral view explode(relatedId) e as type) t2
lateral view explode(category) f as type2
group by type2

select t2.type,c.category from
((select type from
(select videoId,category,views,relatedId from video_orc order by views desc limit 50) t1
lateral view explode(relatedId) e as type)t2
join video_orc c
on c.videoId=t2.type)


select type,videoId,views
from video_orc
lateral view explode(category) e as type
where e.type='Music'
order by views desc
limit 10;
```

```
select * from order_user
union
select id,name,first_date,nvl(last_date,'9999-09-09') from order_user_tmp 
```

