# Maxwell

### 1、概念

```
maxwell 是由美国zendesk开源，用java编写的Mysql实时抓取软件。 其抓取的原理也是基于binlog
```

### 2、安装

```
1、tar -zxvf maxwell-1.25.0.tar.gz /opt/module/maxwell

2、在MySQL中创建元数据库：CREATE DATABASE maxwell ;

3、在MySQL中分配账户可以操作这个数据库：GRANT ALL   ON maxwell.* TO 'maxwell'@'%' IDENTIFIED BY '123456';

4、分配账户可以监控其他数据库的权限：GRANT  SELECT ,REPLICATION SLAVE , REPLICATION CLIENT  ON *.* TO maxwell@'%'

5、创建配置文件：vim /opt/module/maxwell/conf/maxwell.properties
producer=kafka
kafka.bootstrap.servers=hadoop102:9092,hadoop103:9092,hadoop104:9092
kafka_topic=ODS_DB_GMALL2020_M

host=hadoop102
user=maxwell
password=123456
producer_partition_by=primary_key

client_id=maxwell_1

6、启动:nohup /opt/module/maxwell-1.25.0/bin/maxwell --config  /opt/module/maxwell/conf/maxwell.properties >/dev/null 2>&1 &

7、mysql中执行测试语句：INSERT INTO z_user_info VALUES(30,'zhang3','13810001010'),(31,'li4','1389999999');

8、kafka消费查看：创建分区，并消费
```

