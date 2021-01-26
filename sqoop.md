# sqoop

### 1、概述

```
1、Sqoop是一款开源的工具，主要用于在Hadoop(Hive)与传统的数据库(mysql、postgresql...)间进行数据的传递
2、可以将一个关系型数据库（例如 ： MySQL ,Oracle ,Postgres等）中的数据导进到Hadoop的HDFS中
3、可以将HDFS的数据导进到关系型数据库中。
```

### 2、原理

```
将导入导出的命令翻译成mr程序，底层其实是MapReduce
只有Map阶段，没有Reduce阶段的任务。默认是4个MapTask
```

### 3、部署

```
1、下载安装包 rz sqoop-1.4.6.bin__hadoop-2.0.4-alpha.tar.gz

2、解压安装包 tar -zxvf sqoop-1.4.6.bin__hadoop-2.0.4-alpha.tar.gz -C /opt/module

3、向lib目录导入驱动 cp /opt/software/mysql-connector-java-5.1.27-bin.jar ./lib

4、配置文件
mv /opt/module/sqoop/conf/sqoop-env-template.sh sqoop-env.sh
vim sqoop-env.sh
export HADOOP_COMMON_HOME=/opt/module/hadoop-2.7.2
export HADOOP_MAPRED_HOME=/opt/module/hadoop-2.7.2
export HIVE_HOME=/opt/module/hive
export ZOOKEEPER_HOME=/opt/module/zookeeper-3.4.10
export ZOOCFGDIR=/opt/module/zookeeper-3.4.10/conf
export HBASE_HOME=/opt/module/hbase

5、测试
bin/sqoop help

bin/sqoop list-databases --connect jdbc:mysql://hadoop102:3306/ --username root --password 123456
```

### 4、导入到hdfs

全表导入

```
bin/sqoop import \
--connect jdbc:mysql://hadoop102:3306/mydb \
--username root \
--password 123456 \
--table mytable \
--target-dir /test \
--delete-target-dir \
--num-mappers 1 \
--fields-terminated-by '\t'
```

查询导入

```
bin/sqoop import \
--connect jdbc:mysql://hadoop102:3306/mydb \
--username root \
--password 000000 \
--query 'select id,name from mytable where $CONDITIONS' \
--target-dir /test \
--delete-target-dir \
--num-mappers 1 \
--fields-terminated-by '\t'
```

### 5、导入到hive

```
bin/sqoop import \
--connect jdbc:mysql://hadoop102:3306/mydb \
--username root \
--password 000000 \
--table mytable \
--num-mappers 1 \
--hive-import \
--fields-terminated-by '\t' \
--hive-overwrite \
--hive-table test_hive

说明：数据先导入到hdfs，然后再迁移到hive,默认的hdfs目录是/user/atguigu/表名
```

### 6、导入到Hbase

```
bin/sqoop import \
--connect jdbc:mysql://hadoop102:3306/company \
--username root \
--password 000000 \
--table company \
--columns "id,name,sex" \
--column-family "info" \
--hbase-create-table \
--hbase-row-key "id" \
--hbase-table "hbase_company" \
--num-mappers 1 \
--split-by id

说明：sqoop1.4.6只支持HBase1.0.1之前的版本的自动创建HBase表的功能，所以后续的版本需要Hbase手动创建
```

### 7、HIVE/HDFS导出到mysql

```
bin/sqoop export \
--connect jdbc:mysql://hadoop102:3306/company \
--username root \
--password 000000 \
--table staff \
--num-mappers 1 \
--export-dir /user/hive/warehouse/staff_hive \
--input-fields-terminated-by "\t"

说明：mysql中的表不存在，不会自动创建，所以需要提前创建好表
```

### 8、sqoop命令打包

```
1、编写脚本文件：vim sqoop.opt

2、编写如下内容
import \
--connect jdbc:mysql://hadoop102:3306/mydb \
--username root \
--password 123456 \
--table mytable \
--target-dir /test \
--delete-target-dir \
--num-mappers 1 \
--fields-terminated-by '\t'

3、chmod 744 sqoop.opt

4、运行脚本：bin/sqoop --options-file 脚本的绝对路径
```

### 9、Sqoop数据导出Parquet

```
Ads层数据用Sqoop往MySql中导入数据的时候，如果用了orc（Parquet）不能导入，需转化成text格式
（1）创建临时表，把Parquet中表数据导入到临时表，把临时表导出到目标表用于可视化
（2）Sqoop里面有参数，可以直接把Parquet转换为text
（3）ads层建表的时候就不要建Parquet表
```

