# Canal

### 1、概念

```
canal是阿里巴巴旗下的一款开源项目，纯Java开发。基于数据库增量日志解析，提供增量数据订阅&消费，目前主要支持了MySQL
```

### 2、原理

```
mysql的主从复制：
1、master将改变记录写入二进制日志中（被称为二进制事件）
2、slave将master的二进制事件拷贝到中继日志中，
3、slave重做中继日志中的事件

canal的原理：
1、canal模拟mysql slave的交互协议，伪装自己为slave，向master发送dump协议
2、master收到dump请求，开始推送binary log给canal
3、canal解析binary log 对象
```

### 3、使用场景

```
1、更新缓存
2、抓取业务数变化，用来制作拉链表
3、抓取业务数据变化，用来制作实时数据
```



### 4、安装

```
1、MySQL配置:
vim /etc/my.cnf
[mysqld]
server-id= 1
log-bin=mysql-bin
binlog_format=row
binlog-do-db=gmall2020

2、进入mysql:mysql -uroot -p123456

3、执行：GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'canal'@'%' IDENTIFIED BY 'canal' ;

5、重启MySQL

6、下载canal：https://github.com/alibaba/canal/releases  （canal.deployer-1.1.4.tar.gz）

7、解压：tar -zxvf canal.deployer-1.1.4.tar.gz /opt/module/canal/

8、修改配置：vim /opt/module/canal/conf/canal.properties
canal.zkServers = hadoop102:2181, hadoop103:2181, hadoop104:2181         zk地址
canal.serverMode = kafka
canal.mq.servers = hadoop102:9092, hadoop103:9092, hadoop104:9092        kafka地址

9、修改实例配置：vim /opt/module/canal/conf/example/instance.properties
canal.instance.master.address=hadoop102:3306
canal.instance.dbUsername=canal
canal.instance.dbPassword=canal
canal.instance.connectionCharset = UTF-8
# enable druid Decrypt database password
canal.instance.enableDruid=false

10、启动zk和kafka

11、启动canal
/opt/module/canal/bin/startup.sh
```

### 5、HA

```
1、vim /opt/module/canal/conf/canal.properties
注释掉canal.instance.global.spring.xml = classpath:spring/file-instance.xml，
打开canal.instance.global.spring.xml = classpath:spring/default-instance.xml

2、分发

3、重启

```

### 6、演示使用

```
1、打开kafka消费者：
kafka-console-consumer.sh --bootstrap-server  hadoop1:9092 --topic GMALL2020_DB

2、改变MySQL的数据（手动改变任意数据）

3、可以看到kafka消费到了数据

4、cd /opt/module/canal/conf/example  ll
-rw-rw-r-- 1 atguigu atguigu 307200 7月  24 16:43 h2.mv.db
-rwxrwxr-x 1 atguigu atguigu   2076 7月  24 14:46 instance.properties
-rw-rw-r-- 1 atguigu atguigu    114 7月  24 16:43 meta.dat
```

7、