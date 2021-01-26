# redis

### 1、概述

```
开源的k-v存储系统，高频，数据量较小，临时（remote dictionary server），单线程


acid:原子性，一致性，隔离性，持久性
cap:强一致性，可用性，分区容错性

ca:传统数据库
cp:
ap:

base:基本可用，软状态，最终一致


内存存储和持久化
取最新n个数据
```



### 2、安装

```
1、解压：tar -zxvf redis-3.2.5.tar.gz -C /opt/module/

2、安装gcc:yum install gcc-c++

3、在redis-3.2.5目录下执行make

4、在redis-3.2.5目录下执行make install

5、默认安装目录为/usr/local/bin

6、复制配置文件：cp /opt/module/redis-3.2.5/redis.conf /opt/module/redis-3.2.5/myredis.conf

7、修改配置文件：vim /opt/module/redis-3.2.5/myredis.conf
daemonize no 改成 yes
protected-mode yes 改成 no

8、启动服务：redis-server /opt/module/redis-3.2.5/myredis.conf

9、启动客户端：redis-cli

10、测试 ping 

11、退出客户端：exit

12、退出redis服务的两个操作：
redis-cli shutdown 
redis-cli -p 6379 shutdown
在客户端内执行shutdown

说明：
1、端口号是6379
2、默认16个数据库，类似数组下标从0开始，初始默认使用0号库

配置说明：
ip:默认bind=127.0.0.1只接受本机的请求，不写的情况下，任何ip均可访问
```

### 3、客户端操作

```
查看所有键：keys *
查看键是否有：exists k1   返回1代表
删除：del k1
设置键的时间（单位是秒）：expire k1 s
查看键的剩余时间：ttl k1    -1代表永不过期，-2代表已经过期
查看当前数据库key的数量：dbsize
清空库：flushdb
会触发存盘的清空库：flushall
剪切当前库的内容到其他库比如2号库：       move k1 2
总共16个库，默认使用0号库，如果切换其他库例如8号库：   select 8
修改key的名字：rename key newkey 不存在时会报错
修改key的名字：renamenx key newkey 存在时才修改
type key：查看key 的类型 不存在返回none
```

### 4、数据类型

http://redisdoc.com/

#### string

```
普通操作：

添加：		set k1 v1
查询：		get k1
不存在时才添加: 		setnx k1 v1
添加带过期时间的：		setex k1 90 v1

进阶操作：
如果key是数字时：
key值自增1：		incr k1
key值自减1：		decr k1
key值自减n:		decrby k1 n
key值自增n:		incrby k1 n


同时添加多个：		mset k1 v1 k2 v2
同时查看多个：		mget k1 k2
```

#### set

```
添加：sadd k1 v1 v2 v3
查找：smembers k1
判断是否有某值：sismembers k1 v1
删除：srem k1 v1 v2
交集：sinter k1 k2
合集：sunion k1 k2
差集：sdiff k1 k2

底层实现：hashtable
```

#### list

```
底层是双向链表，所以分左右，l代表左，r代表右

插入一个或多个：		lpush/rpush k1 v1 v2 v3
删除一个：				lpop/rpop k1
从k1表右面删除一个插入到k2表左面：		rpoplpush k1 k2
获取列表长度：			llen
删除n个v1 :           lrem k1 n v1
截取子list :          ltrim n1 n2
查看：                 lrange k1 0 -1
```

#### hash

```
添加：hset k1 f1 v1
查看：hget k1 f1
批量添加：hmset k1 f1 v1 f2 v2 f3 v3
查看是否存在：hexists k1 f1
查看所有值：hgetall k1
键值对中键自增n：hincrby k1 f1 n

hkeys key 查看key包含的所有键
hvals key 查看key包含的所有值
hgetall key 查看key包含的所有键和值

```

#### zset

```
添加：zadd k1 n1 v1 n2 v2
查看：zrange k1 n1 n2 withscores 带n，从n1到n2
逆序：zrevrange k1 n1 n2 withscores 带n,从n2到n1
为n加增量：zincrby k1 m1 v1
删除：zrem k1 v1
统计个数：zcount k1 min max
返回排名：zrank k1 v1
```

### 5、idea操作

```
1、创建maven工程

2、pom文件：
    <dependencies>
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>3.2.0</version>
        </dependency>
    </dependencies>
    
3、建包，建类，main方法
    public static void main(String[] args) {
    
        Jedis hadoop102 = new Jedis("hadoop102", 6379);
        System.out.println(hadoop102.get("k1"));
        
    }
    
4、使用的数据类型和方法同数据类型

说明：1、需要禁用linux的防火墙，2、需要注释掉bind 127.0.0.1 3、需要protect-mode no
```

### 6、雪崩和击穿

```
雪崩：大面积缓存失效，导致大量请求落在数据库，导致数据库崩溃
解决：缓存过期时间采用随机值，防止同一时间大量数据同时过期

击穿：某个特殊请求在缓存和数据库中均没有，但有不断的请求，有理由相信这是恶意攻击
解决：把null值存在缓存中，请求不在范围内的就用null值顶替，避免数据库被骚扰
```

### 7、单线程和io多路复用

```
Redis 是单线程+多路IO复用技术
多路复用：使用一个线程来检查多个文件描述符的就绪状态
　　　　　如果有一个文件描述符就绪，则返回
　　　　　否则阻塞直到超时
　　　　　得到就绪状态后进行真正的操作可以在同一个线程里执行，也可以启动线程执行（线程池）
　　　　　本质上是没有阻塞的
阻塞lO（串行）：给女神发一条短信, 说我来找你了,然后就默默的一直等着 女神下楼,这个期间除了等待你不会做其他事情,属于备胎做法.
非阻塞IO：给女神发短信，如果不回,接着再发,一直 发到女神下楼,这个期间你除了发短信等待不会做其他事情,属于专-做法.
IO多路复用：是找一个宿管大妈来帮你监视下楼的女生,这个期间你可以些其他的事情.例如可以顺便看看其他妹子,玩玩王者荣耀,上个厕所等等.
IO复用又包括select, poll, epoll模式那么它们的区别是什么?
select：
　　一个女生下楼, select大妈都不知道这个是不是你的女神，她需要一个一 个 询问，并且select大妈能力还有限，最多一次帮 你监视1024个妹子
poll：
　　poll大妈不限制盯着女生的数量，只要是经过宿舍楼门口的女生，都会帮你去问是不是你女神
epoll:
　　epoll大妈不限制盯着女生的数量，并且也不需要一个- 个去问.那么如何做呢? epol1大妈会为每个进宿舍楼的女生脸上贴上一一个大字条,上面写上女生自己的名字,只要女姓下楼了, 　　epoll大妈就知道这个是不是你女神了,然后大妈再通知你.
```

### 8、持久化

RDB

```
默认开启的，内存全量快照；会fork一个子线程，

触发存盘的操作：
save    会阻塞主进程，
shutdown
kill
flushall

不触发的操作：
kill -9
意外宕机
flushdb

自动存盘操作触发条件(时间要素和数据变化量机制)：
#   after 900 sec (15 min) if at least 1 key changed
#   after 300 sec (5 min) if at least 10 keys changed
#   after 60 sec if at least 10000 keys changed

优点：省磁盘空间
恢复速度快

缺点：
全量备份----->重操作------->要有一定的周期间隔-------->两次间隔时间内宕机，未保存的数据会丢失
```

AOF

```
默认不开启，AOF和RDB同时开启的时候，默认取AOF的数据

AOF文件的保存路径，同RDB的路径一致，启动命令的路径下

# If unsure, use "everysec".
# appendfsync always
appendfsync everysec
# appendfsync no

重写机制：隔一段时间后，Redis就会启动AOF文件的内容压缩，只保留可以恢复数据的最小指令集.可以使用命令bgrewriteaof
AOF文件持续增长而过大时，会fork出一条新进程来将文件重写(也是先写临时文件最后再rename)，遍历新进程的内存中数据，每条记录有一条的Set语句。重写aof文件的操作，并没有读取旧的aof文件，而是将整个内存中的数据库内容用命令的方式重写了一个新的aof文件，这点和快照有点类似。

备份机制稳健，丢失数据的概率更低
可以处理误操作

占用更多磁盘空间
备份速度慢
如果每次读写都同步的话，性能有影响
存在个别bug,造成恢复不能

建议两个备份机制同时开
如果对数据不敏感，可以单独使用rdb

```

### 9、主从复制

```
1、配置文件
daemonize yes
port 6379
dir ./
dbfilename dump6379.rdb
pidfile /var/run/
protected-mode no
slaveof hadoop102 6379


从机指定主机：slaveof hadoop102 6379
查看：info replication

说明：
主机宕机后，从机会一直死等主机，并不会上位，主机回来后，从机还能正常复制
其中一台从机宕机后，如果slaveof写在命令行，从机回来后不会认主机，但是把命令写入配置文件宕机回来后就可以跟上大部队

一主二仆：
一个Master，两个Slave，Slave只能读不能写；当Slave与Master断开后需要重新slave of连接才可建立之
前的主从关系；除非配置到配置文件中，Master挂掉后，Master关系依然存在，Master重启即可恢复。

薪火相传：
上一个Slave可以是下一个Slave的Master，Slave同样可以接收其他slaves的连接和同步请求，那么该slave作为了
链条中下一个slave的Master，如此可以有效减轻Master的写压力。如果slave中途变更转向，会清除之前的数据，重新建立最新的。

反客为主：当Master挂掉后，Slave可键入命令 slaveof no one使当前redis停止与其他Master redis数据同步，转成Master redis。

复制原理：
       1、Slave启动成功连接到master后会发送一个sync命令；

       2、Master接到命令启动后的存盘进程，同时收集所有接收到的用于修改数据集命令，在后台进程执行完毕之后，master

            将传送整个数据文件到slave，以完成一次完全同步；

       3、全量复制：而slave服务在数据库文件数据后，将其存盘并加载到内存中；

       4、增量复制：Master继续将新的所有收集到的修改命令依次传给slave，完成同步；

       5、但是只要是重新连接master，一次完全同步（全量复制）将被自动执行。
```

哨兵模式：

```
反客为主的自动版
配置：
1、在Master对应redis.conf同目录下新建sentinel.conf文件，名字绝对不能错；

2、配置哨兵，在sentinel.conf文件中填入内容：
sentinel monitor mymaster 192.168.248.102 6379 1
说明：上面最后一个数字1，表示主机挂掉后slave投票看让谁接替成为主机，得票数多少后成为主机。

3、启动哨兵模式：redis-sentinel  /myredis/sentinel.conf

哨兵模式：会监控主机是否健康，周期型的pingpong主机
主机下线：某台哨兵认为主机不行了
客观下线：和其他哨兵一起判断，进行投票，达到票数阈值，进行主从切换
对于外部客户端来说，主机时常切换，一般java访问集群时，首先访问的是哨兵，哨兵将请求传递给主机
问题：建立连接比较慢，所以会使用连接池，JedisSentinelPool

集群：目的是横向扩展，负载均衡
1、 客户端方案，早期方案通过JedisShardInfo来实现分片
      问题：1 分片规则耦合在客户端
                2 需要自己实现很多功能
                3 不提供高可用
2、 第三方代理中间件模式：twemproxy、 codis
     问题：1 成为瓶颈和风险点  2 版本基本上不再更新了
     
3、redis3.0以后出的官方redis-cluster方案
      问题：有高可用，但没有读写分离  hash槽


```

### 10、集群

配置

```
1、在hadoop102上在安装目录下（/opt/module/redis-3.2.5/） mkdir conf cd conf

2、vim redis6379.conf 填写如下内容：
cluster-enabled yes
cluster-config-file nodes-6379.conf
cluster-node-timeout 15000
port 6379
daemonize yes
dir /opt/module/redis-3.2.5/data dbfilename dump6379.rdb
pidfile /var/run/redis_6379.pid
appendonly no
protected-mode no
save 300 10

3、复制五份，并更改配置文件redis6380.conf、 redis6381.conf、 redis6389.conf、redis6390.conf、 redis6391.conf
批量替换命令 :%s/6379/6380/g

4、启动六个服务：
redis-server /opt/module/redis-3.2.5/conf/redis6379.conf
redis-server /opt/module/redis-3.2.5/conf/redis6380.conf
redis-server /opt/module/redis-3.2.5/conf/redis6381.conf
redis-server /opt/module/redis-3.2.5/conf/redis6382.conf
redis-server /opt/module/redis-3.2.5/conf/redis6383.conf
redis-server /opt/module/redis-3.2.5/conf/redis6384.conf



5、确定启动后

6、合体
/opt/module/redis-3.2.5/src/redis-trib.rb create --replicas 1 192.168.248.102:6379 192.168.248.102:6380 192.168.248.102:6381 192.168.248.102:6382 192.168.248.102:6383 192.168.248.102:6384 

7、添加节点：/opt/module/redis-3.2.5/src/redis-trib.rb add-node 192.168.248.102:7010 192.168.248.102:7000
```

启动

```
1、进入客户端操作：redis-cli -c -p 6379
不进入客户端操作：/opt/module/redis-3.2.5/src/redis-cli -c -h 192.168.248.102 -p 6379 keys "*"

2、查看集群cluster nodes 

3、关闭：ps -ef | grep redis | awk '{print $2}' | xargs kill

```

11、配置文件

```
1. Redis默认不是以守护进程的方式运行，可以通过该配置项修改，使用yes启用守护进程
    daemonize no
    
2. 当Redis以守护进程方式运行时，Redis默认会把pid写入/var/run/redis.pid文件，可以通过pidfile指定
# If a pid file is specified, Redis writes it where specified at startup
# and removes it at exit.
#
# When the server runs non daemonized, no pid file is created if none is
# specified in the configuration. When the server is daemonized, the pid file
# is used even if not specified, defaulting to "/var/run/redis.pid".
#
# Creating a pid file is best effort: if Redis is not able to create it
# nothing bad happens, the server will start and run normally.

    pidfile /var/run/redis.pid
3. 指定Redis监听端口，默认端口为6379，作者在自己的一篇博文中解释了为什么选用6379作为默认端口，因为6379在手机按键上MERZ对应的号码，而MERZ取自意大利歌女Alessia Merz的名字
    port 6379
4. 绑定的主机地址
    bind 127.0.0.1
5.当 客户端闲置多长时间后关闭连接，如果指定为0，表示关闭该功能
    timeout 300
6. 指定日志记录级别，Redis总共支持四个级别：debug、verbose、notice、warning，默认为verbose
    loglevel verbose
7. 日志记录方式，默认为标准输出，如果配置Redis为守护进程方式运行，而这里又配置为日志记录方式为标准输出，则日志将会发送给/dev/null
    logfile stdout
8. 设置数据库的数量，默认数据库为0，可以使用SELECT <dbid>命令在连接上指定数据库id
    databases 16
9. 指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合
    save <seconds> <changes>
    Redis默认配置文件中提供了三个条件：
    save 900 1
    save 300 10
    save 60 10000
    分别表示900秒（15分钟）内有1个更改，300秒（5分钟）内有10个更改以及60秒内有10000个更改。
 
10. 指定存储至本地数据库时是否压缩数据，默认为yes，Redis采用LZF压缩，如果为了节省CPU时间，可以关闭该选项，但会导致数据库文件变的巨大
    rdbcompression yes
11. 指定本地数据库文件名，默认值为dump.rdb
    dbfilename dump.rdb
12. 指定本地数据库存放目录
    dir ./
13. 设置当本机为slav服务时，设置master服务的IP地址及端口，在Redis启动时，它会自动从master进行数据同步
    slaveof <masterip> <masterport>
14. 当master服务设置了密码保护时，slav服务连接master的密码
    masterauth <master-password>
15. 设置Redis连接密码，如果配置了连接密码，客户端在连接Redis时需要通过AUTH <password>命令提供密码，默认关闭
    requirepass foobared
16. 设置同一时间最大客户端连接数，默认无限制，Redis可以同时打开的客户端连接数为Redis进程可以打开的最大文件描述符数，如果设置 maxclients 0，表示不作限制。当客户端连接数到达限制时，Redis会关闭新的连接并向客户端返回max number of clients reached错误信息
    maxclients 128
17. 指定Redis最大内存限制，Redis在启动时会把数据加载到内存中，达到最大内存后，Redis会先尝试清除已到期或即将到期的Key，当此方法处理 后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。Redis新的vm机制，会把Key存放内存，Value会存放在swap区
    maxmemory <bytes>
18. 指定是否在每次更新操作后进行日志记录，Redis在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为 redis本身同步数据文件是按上面save条件来同步的，所以有的数据会在一段时间内只存在于内存中。默认为no
    appendonly no
19. 指定更新日志文件名，默认为appendonly.aof
     appendfilename appendonly.aof
20. 指定更新日志条件，共有3个可选值： 
    no：表示等操作系统进行数据缓存同步到磁盘（快） 
    always：表示每次更新操作后手动调用fsync()将数据写到磁盘（慢，安全） 
    everysec：表示每秒同步一次（折衷，默认值）
    appendfsync everysec
 
21. 指定是否启用虚拟内存机制，默认值为no，简单的介绍一下，VM机制将数据分页存放，由Redis将访问量较少的页即冷数据swap到磁盘上，访问多的页面由磁盘自动换出到内存中（在后面的文章我会仔细分析Redis的VM机制）
     vm-enabled no
22. 虚拟内存文件路径，默认值为/tmp/redis.swap，不可多个Redis实例共享
     vm-swap-file /tmp/redis.swap
23. 将所有大于vm-max-memory的数据存入虚拟内存,无论vm-max-memory设置多小,所有索引数据都是内存存储的(Redis的索引数据 就是keys),也就是说,当vm-max-memory设置为0的时候,其实是所有value都存在于磁盘。默认值为0
     vm-max-memory 0
24. Redis swap文件分成了很多的page，一个对象可以保存在多个page上面，但一个page上不能被多个对象共享，vm-page-size是要根据存储的 数据大小来设定的，作者建议如果存储很多小对象，page大小最好设置为32或者64bytes；如果存储很大大对象，则可以使用更大的page，如果不 确定，就使用默认值
     vm-page-size 32
25. 设置swap文件中的page数量，由于页表（一种表示页面空闲或使用的bitmap）是在放在内存中的，，在磁盘上每8个pages将消耗1byte的内存。
     vm-pages 134217728
26. 设置访问swap文件的线程数,最好不要超过机器的核数,如果设置为0,那么所有对swap文件的操作都是串行的，可能会造成比较长时间的延迟。默认值为4
     vm-max-threads 4
27. 设置在向客户端应答时，是否把较小的包合并为一个包发送，默认为开启
    glueoutputbuf yes
28. 指定在超过一定的数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法
    hash-max-zipmap-entries 64
    hash-max-zipmap-value 512
29. 指定是否激活重置哈希，默认为开启（后面在介绍Redis的哈希算法时具体介绍）
    activerehashing yes
30. 指定包含其它的配置文件，可以在同一主机上多个Redis实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件
    include /path/to/local.conf
```

### 11、布隆过滤器

```
原理：布隆过滤器是用于判断一个元素是否在集合中。通过一个位数组和N个hash函数实现。

优点：
1、空间效率高，所占空间小。
2、查询时间短。

缺点：
1、元素添加到集合中后，不能被删除。
2、有一定的误判率
```

