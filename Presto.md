# Presto

### 1、概念

```
Presto:开源的，分布式，SQL查询引擎，数据量支持GB到PB的数据，用来处理秒级查询的场景

虽然可以解析SQL,但是并不是标准的数据库，并不能替代MySql,Oracle,也不能处理oltp在线事务

优点：
1、基于内存，速度快
2、同时连接多个数据源

缺点：
1、有一些操作是边读数据边计算，再清内存，消耗内存不高，但是连表查询会产生大量临时数据，速度会变慢
```

### 2、安装Presto Server

```
1、tar -zxvf presto-server-0.196.tar.gz -C /opt/module/

2、mv presto-server-0.196/ presto

3、cd /opt/module/presto mkdir data mkdir etc

4、cd /opt/module/presto/etc vim jvm.config
-server
-Xmx16G
-XX:+UseG1GC
-XX:G1HeapRegionSize=32M
-XX:+UseGCOverheadLimit
-XX:+ExplicitGCInvokesConcurrent
-XX:+HeapDumpOnOutOfMemoryError
-XX:+ExitOnOutOfMemoryError

5、配置hive数据源：cd /opt/module/presto/etc mkdir catalog

6、配置hive配置文件：cd /opt/module/presto/etc/catalog vim hive.properties
connector.name=hive-hadoop2
hive.metastore.uri=thrift://hadoop102:9083

7、配置node:cd /opt/module/presto/etc  vim node.properties
node.environment=production
node.id=ffffffff-ffff-ffff-ffff-ffffffffffff
node.data-dir=/opt/module/presto/data

8、分发xsync presto

9、修改node.properties  使用不同的node.id

10、hadoop102上配置coordinator节点：cd /opt/module/presto/etc vim config.properties
coordinator=true
node-scheduler.include-coordinator=false
http-server.http.port=8881
query.max-memory=50GB
discovery-server.enabled=true
discovery.uri=http://hadoop102:8881

11、另外两天配置worker节点：cd /opt/module/presto/etc vim config.properties
coordinator=false
http-server.http.port=8881
query.max-memory=50GB
discovery.uri=http://hadoop102:8881

12、前台启动：/opt/module/presto/bin/launcher run (三台)

13、后台启动：/opt/module/presto/bin/launcher start (三台)

14、日志查看：/opt/module/presto/data/var/log
```

### 3、安装Presto 命令行cli

```
1、将presto-cli-0.196-executable.jar上传到hadoop102的/opt/module/presto文件夹下

2、mv presto-cli-0.196-executable.jar  prestocli

3、给执行权限：chmod +x /opt/module/presto/prestocli

4、启动：/opt/module/presto/prestocli --server hadoop102:8881 --catalog hive --schema default

5、测试：select * from schema.table limit 100
说明：查询必须加上schema
```

### 4、安装Presto 可视化cli

```
1、解压：cd /opt/module/ unzip yanagishima-18.0.zip

2、配置：cd /opt/module/yanagishima-18.0/conf vim yanagishima.properties
jetty.port=7080
presto.datasources=atguigu-presto
presto.coordinator.server.atguigu-presto=http://hadoop102:8881
catalog.atguigu-presto=hive
schema.atguigu-presto=default
sql.query.engines=presto

3、启动：nohup /opt/module/yanagishima-18.0/bin/yanagishima-start.sh >y.log 2>&1 &

4、web查看：http://hadoop102:7080
```

