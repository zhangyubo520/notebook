# HA

### 一、概念

### 1、能自动故障转移的HDFS-HA集群

#### 1、cp -r /opt/module/hadoop-3.1.3 /opt/module/ha/

```
目的：可继承原来集群的配置文件，
结果：module下出现新的文件夹ha,文件夹下是复制过来的hadoop-3.1.3
```

#### 2、在hadoop102上配置core-site.xml	hdfs-site.xml	yarn-site.xml

```
目的：搭建新的集群，集群规划体现在配置文件中
结果：产生三个新集群的配置文件core-site.xml	hdfs-site.xml	yarn-site.xml
```

##### core-site.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->

<!-- Put site-specific property overrides in this file. -->

<configuration>
   <!-- 把多个NameNode的地址组装成一个集群mycluster -->
		<property>
			<name>fs.defaultFS</name>
        	<value>hdfs://mycluster</value>
		</property>
	
	<!-- 指定hadoop运行时产生文件的存储目录 -->
		<property>
			<name>hadoop.tmp.dir</name>
			<value>/opt/module/ha/hadoop-3.1.3/data/tmp</value>
		</property>
   <!-- 声明journalnode服务器存储目录-->
	<property>
		<name>dfs.journalnode.edits.dir</name>
		<value>file://${hadoop.tmp.dir}/jn</value>
	</property>

   <!-- 指定zk集群的位置-->
    <property>
	<name>ha.zookeeper.quorum</name>
	<value>hadoop102:2181,hadoop103:2181,hadoop104:2181</value>
   </property>

<!-- 兼容性配置，用于兼容后续的框架使用 -->
<!-- 配置该atguigu(superUser)允许通过代理访问的主机节点 -->
    <property>
        <name>hadoop.proxyuser.atguigu.hosts</name>
        <value>*</value>
</property>
<!-- 配置该atguigu(superuser)允许代理的用户所属组 -->
    <property>
        <name>hadoop.proxyuser.atguigu.groups</name>
        <value>*</value>
</property>
<!-- 配置该atguigu(superuser)允许代理的用户-->
    <property>
        <name>hadoop.proxyuser.atguigu.users</name>
        <value>*</value>
    </property>
<!-- 指定web端操作使用的用户 -->
<property>
        <name>hadoop.http.staticuser.user</name>
        <value>atguigu</value>
 </property>


</configuration>

```

##### hdfs-site.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->

<!-- Put site-specific property overrides in this file. -->

<configuration>
<!-- 副本数 --> 
<property>
  <name>dfs.replication</name>
  <value>3</value>
</property>
	<!-- 完全分布式集群名称 -->
	<property>
		<name>dfs.nameservices</name>
		<value>mycluster</value>
	</property>
  <!-- NameNode数据存储目录 -->
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file://${hadoop.tmp.dir}/name</value>
  </property>
 <!-- DataNode数据存储目录 -->
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file://${hadoop.tmp.dir}/data</value>
  </property>

	<!-- 集群中NameNode节点都有哪些 -->
	<property>
		<name>dfs.ha.namenodes.mycluster</name>
		<value>nn1,nn2,nn3</value>
	</property>

	<!-- nn1的RPC通信地址 -->
	<property>
		<name>dfs.namenode.rpc-address.mycluster.nn1</name>
		<value>hadoop102:9000</value>
	</property>

	<!-- nn2的RPC通信地址 -->
	<property>
		<name>dfs.namenode.rpc-address.mycluster.nn2</name>
		<value>hadoop103:9000</value>
	</property>
	<!-- nn3的RPC通信地址 -->
	<property>
		<name>dfs.namenode.rpc-address.mycluster.nn3</name>
		<value>hadoop104:9000</value>
	</property>


	<!-- nn1的http通信地址 -->
	<property>
		<name>dfs.namenode.http-address.mycluster.nn1</name>
		<value>hadoop102:9870</value>
	</property>

	<!-- nn2的http通信地址 -->
	<property>
		<name>dfs.namenode.http-address.mycluster.nn2</name>
		<value>hadoop103:9870</value>
	</property>
	<!-- nn3的http通信地址 -->
	<property>
		<name>dfs.namenode.http-address.mycluster.nn3</name>
		<value>hadoop104:9870</value>
	</property>

	<!-- 指定NameNode元数据在JournalNode上的存放位置 -->
	<property>
		<name>dfs.namenode.shared.edits.dir</name>
	<value>qjournal://hadoop102:8485;hadoop103:8485;hadoop104:8485/mycluster</value>
	</property>

	<!-- 配置隔离机制，即同一时刻只能有一台服务器对外响应 -->
	<property>
		<name>dfs.ha.fencing.methods</name>
		<value>sshfence</value>
	</property>

	<!-- 使用隔离机制时需要ssh无秘钥登录-->
	<property>
		<name>dfs.ha.fencing.ssh.private-key-files</name>
		<value>/home/atguigu/.ssh/id_rsa</value>
	</property>

	<!-- 访问代理类：client用于确定哪个NameNode为Active -->
	<property>		<name>dfs.client.failover.proxy.provider.mycluster</name>
	<value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
	</property>
        <!-- 开启自动故障转移-->
        <property>
	   <name>dfs.ha.automatic-failover.enabled</name>
	   <value>true</value>
        </property>


</configuration>

```

##### yarn-site.xml

```
<?xml version="1.0"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->
<configuration>
   <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>

    <!--启用resourcemanager ha-->
    <property>
        <name>yarn.resourcemanager.ha.enabled</name>
        <value>true</value>
    </property>
 
    <!--声明HA resourcemanager的地址-->
    <property>
        <name>yarn.resourcemanager.cluster-id</name>
        <value>cluster-yarn1</value>
    </property>
     <!-- 指定RM的逻辑列表 -->
    <property>
        <name>yarn.resourcemanager.ha.rm-ids</name>
        <value>rm1,rm2,rm3</value>
    </property>

   <!--  =========== rm1 配置============  --> 
<!-- 指定rm1 的主机名 -->
    <property>
        <name>yarn.resourcemanager.hostname.rm1</name>
        <value>hadoop102</value>
    </property>
    <!-- 指定rm1的web端地址 -->
<property>
        <name>yarn.resourcemanager.webapp.address.rm1</name>
        <value>hadoop102:8088</value>
</property>
   <!-- 指定rm1的内部通信地址 -->
    <property>
        <name>yarn.resourcemanager.address.rm1</name>
        <value>hadoop102:8032</value>
    </property>
  <!-- 指定AM向rm1申请资源的地址 -->
    <property>
        <name>yarn.resourcemanager.scheduler.address.rm1</name>  
        <value>hadoop102:8030</value>
    </property>
  <!-- 指定供NM连接的地址 -->  
<property>
        <name>yarn.resourcemanager.resource-tracker.address.rm1</name>
        <value>hadoop102:8031</value>
</property>

<!--  =========== rm2 配置============  --> 

    <property>
        <name>yarn.resourcemanager.hostname.rm2</name>
        <value>hadoop103</value>
</property>

<property>
        <name>yarn.resourcemanager.webapp.address.rm2</name>
        <value>hadoop103:8088</value>
</property>
    <property>
        <name>yarn.resourcemanager.address.rm2</name>
        <value>hadoop103:8032</value>
    </property>
    <property>
        <name>yarn.resourcemanager.scheduler.address.rm2</name>
        <value>hadoop103:8030</value>
    </property>

<property>
        <name>yarn.resourcemanager.resource-tracker.address.rm2</name>
        <value>hadoop103:8031</value>
</property>

<!--  =========== rm3 配置============  --> 

    <property>
        <name>yarn.resourcemanager.hostname.rm3</name>
        <value>hadoop104</value>
    </property>

<property>
        <name>yarn.resourcemanager.webapp.address.rm3</name>
        <value>hadoop104:8088</value>
</property>
    <property>
        <name>yarn.resourcemanager.address.rm3</name>
        <value>hadoop104:8032</value>
    </property>
    <property>
        <name>yarn.resourcemanager.scheduler.address.rm3</name>
        <value>hadoop104:8030</value>
    </property>

<property>
        <name>yarn.resourcemanager.resource-tracker.address.rm3</name>
        <value>hadoop104:8031</value>
</property>
 
    <!--指定zookeeper集群的地址--> 
    <property>
        <name>yarn.resourcemanager.zk-address</name>
        <value>hadoop102:2181,hadoop103:2181,hadoop104:2181</value>
    </property>

    <!--启用自动恢复--> 
    <property>
        <name>yarn.resourcemanager.recovery.enabled</name>
        <value>true</value>
    </property>
 
    <!--指定resourcemanager的状态信息存储在zookeeper集群--> 
    <property>
        <name>yarn.resourcemanager.store.class</name>     <value>org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore</value>
</property>
 
<!-- 环境变量的继承 -->
  <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>
  

<!-- 日志聚集 -->
   <property>
        <name>yarn.log-aggregation-enable</name>
        <value>true</value>
    </property>
    <property>  
        <name>yarn.log.server.url</name>  
        <value>http://hadoop102:19888/jobhistory/logs</value>  
    </property>
    <property>
        <name>yarn.log-aggregation.retain-seconds</name>
        <value>604800</value>
</property>
  <!-- 不检查虚拟内存使用 -->
    <property>
    <name>yarn.nodemanager.vmem-check-enabled</name>
    <value>false</value>
  </property>
</configuration>

```

#### 3、更改hadoop102中环境变量：vim /etc/profile.d/my_env.xml

```
目的：保证启动hadoop102上节点进程是新建集群的
结果：初始化集群时，hadoop102是新集群的一个节点
问题：第10步初始化失败
```

my_env.xml

```
#JAVA_HOME
JAVA_HOME=/opt/module/jdk1.8.0_212
#HADOOP_HOME
HADOOP_HOME=/opt/module/ha/hadoop-3.1.3
PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
export PATH JAVA_HOME HADOOP_HOME
```

#### 4、删除/opt/module/ha/hadoop-3.1.3下的data和logs

```
目的：保证是全新的集群
结果：hadoop-3.1.3像新安装的一样
问题：第10步初始化失败
```

#### 5、分发：ha文件夹（包含配置文件）

```
目的：在hadoop103、hadoop104上配置集群新节点
结果：在hadoop103和hadoop104上出现/opt/module/ha/hadoop-3.1.3（配置是新的，data也删除了）
问题：导致另外两个节点使用的还是老集群的配置
```

#### 6、分发：my_env.xml

```
目的：保证启动hadoop103，hadoop104上节点进程是新建集群的
结果：初始化集群时，hadoop102是新集群的一个节点
问题：导致另外两个节点使用的还是老集群的配置
```

#### 7、三个节点依次	source /etc/profile ,不行就重启

```
目的：保证第六步的配置生效
问题：导致第六步失效
```

#### 8、三个节点依次	rm -rf /tmp/*

```
目的：为第九步做准备
结果:每个节点上/tmp下为空
问题：个别没删干净，需要重启机器重新删
```

#### 9、三个节点依次	hdfs --daemon start journalnode

```
目的：初始化集群的准备
结果：jps时三台均启动了JournalNode
```

#### 10、初始化 	hdfs namenode -format

```
目的：读取集群配置文件，并初始化集群
结果：无提示表示启动成功，
初始化失败可能原因：
1、配置文件中某个节点不是最新的，没有分发或者文件中有错误（很难发现，建议全部删除，复制蓝本）
2、环境变量配置错误，或者某个节点配置错误，或者压根某个节点就没有配置
```

11、在hadoop102上启动nn   	hdfs --daemon start namenode

```
结果：jps时出现namenode线程
```

12、同步另外两个nn 	 hdfs namenode -bootstrapStandby

```
问题：未做的话导致第13步出错
```

13、在另外两台上启动nn 	hdfs --daemon start namenode

```
目的：验证第12步没有错
结果：jps时出现namenode线程
```

14、关闭集群 	stop-dfs.sh

15、配置zookeeper集群并依次启动	zkServer.sh start

16、初始化HA在zookeeper中的状态	hdfs zkfc -formatZK

17、start-dfs.sh

18、start-yarn.sh

### 2、高可用集群测试

```
1、将Active NameNode进程kill,实现自动故障转移			kill -9 namenode的进程id
结果，新的nn变成active

2、将nn1 转换成active 
hdfs haadmin -transitionToActive nn1

3、查看节点状态是否是active
hdfs haadmin -getServiceState nn1
```

