# Ranger

### 1、简介

```
Apache Ranger是一个用来在Hadoop平台上进行监控，启用服务，以及全方位数据安全访问管理的安全框架。
Ranger的愿景是在Apache Hadoop生态系统中提供全面的安全管理。随着企业业务的拓展，企业可能在多用户环境中运行多个工作任务，这就要求Hadoop内的数据安全性需要扩展为同时支持多种不同的需求进行数据访问，同时还需要提供一个可以对安全策略进行集中管理，配置和监控用户访问的框架。Ranger由此产生！
Ranger的官网：https://ranger.apache.org/
```

### 2、功能

```
	允许用户使用UI或REST API对所有和安全相关的任务进行集中化的管理
	允许用户使用一个管理工具对操作Hadoop体系中的组件和工具的行为进行细粒度的授权
	支持Hadoop体系中各个组件的授权认证标准
	增强了对不同业务场景需求的授权方法支持，例如基于角色的授权或基于属性的授权
	支持对Hadoop组件所有涉及安全的审计行为的集中化管理
```

### 3、支持的框架

```
	Apache Hadoop
	Apache Hive
	Apache HBase
	Apache Storm
	Apache Knox
	Apache Solr
	Apache Kafka
	YARN
	NIFI
```

### 4、框架

![](D:\data\学习笔记\笔记\picture\snipaste_20210114_233931.png)

### 5、工作原理

```
Ranager的核心是web应用程序，也成为RangerAdmin模块，此模块由管理策略，审计日志和报告等三部分组成。
	管理员角色的用户可以通过RangerAdmin提供的web界面或REST APIS来定制安全策略。这些策略会由Ranger提供的轻量级的针对不同Hadoop体系中组件的插件来执行。插件会在Hadoop的不同组件的核心进程启动后，启动对应的插件进程来进行安全管理！
```

### 6、Ranger的安装

#### 环境：

```
Ranger2.0要求对应的Hadoop为3.x以上，Hive为3.x以上版本，JDK为1.8以上版本！
```

#### 安装RangerAdmin

```
1、在Mysql数据库中创建Ranger存储数据的数据库:
mysql> create database ranger;

2、创建用户:
mysql> grant all privileges on ranger.* to ranger@'%'  identified by 'ranger';

3、加压安装包：
[username@hadoop103 target]tar -zxvf ranger-2.0.0-admin.tar.gz -C /opt/module/ranger

4、进行配置：
[username@hadoop103 ranger-2.0.0-admin]$ vim install.properties

#mysql驱动
SQL_CONNECTOR_JAR=/opt/software/mysql-connector-java-5.1.27-bin.jar
#mysql的主机名和root用户的用户名密码
db_root_user=root
db_root_password=000000
db_host=hadoop103
#ranger需要的数据库名和用户信息，和第二步创建的信息要一一对应
db_name=ranger
db_user=ranger
db_password=ranger
#其他ranger admin需要的用户密码
rangerAdmin_password=username123
rangerTagsync_password=username123
rangerUsersync_password=username123
keyadmin_password=username123
#ranger存储审计日志的路径，默认为solr，这里为了方便暂不设置
audit_store=
#策略管理器的url,rangeradmin安装在哪台机器，主机名就为对应的主机名
policymgr_external_url=http://hadoop103:6080
#启动ranger admin进程的linux用户信息
unix_user=username
unix_user_pwd=username
unix_group=username
#hadoop的配置文件目录
hadoop_conf=/opt/module/hadoop-3.1.3/etc/hadoop

5、切换到root用户，执行安装
[root@hadoop103 ranger-2.0.0-admin]# ./setup.sh

6、安装成功反馈：
Ranger all admins default password change request processed successfully..
Installation of Ranger PolicyManager Web Application is completed.

7、创建ranger的配置文件软连接到web应用下：
[root@hadoop103 ranger-2.0.0-admin]# ./set_globals.sh 
usermod：无改变
[2020/04/30 13:58:47]:  [I] Soft linking /etc/ranger/admin/conf to ews/webapp/WEB-INF/classes/conf

8、配置RangerAdmin web应用的配置信息
[root@hadoop103 ranger-2.0.0-admin]# cd /etc/ranger/admin/conf/
[root@hadoop103 conf]# vim ranger-admin-site.xml
<property>
      <name>ranger.jpa.jdbc.password</name>
      <value>ranger</value>
      <description />
</property>
<property>
       <name>ranger.service.host</name>
       <value>hadoop103</value>
</property>

9、启动：
[root@hadoop103 conf]# ranger-admin start
Starting Apache Ranger Admin Service
Apache Ranger Admin Service with pid 7058 has started

10、查看启动后的进程：
[root@hadoop103 ranger-2.0.0-usersync]# jps
7058 EmbeddedServer
8132 Jps

11、停止：
[root@hadoop103 conf]# ranger-admin stop

12、web访问：
http://hadoop103:6080

13、登录用户名和密码：username123 username123 
```

#### 安装RangerUsersync

```
1、简介：
RangerUsersync作为Ranger提供的一个管理模块，可以将Linux机器上的用户和组信息同步到RangerAdmin的数据库中进行管理！

2、解压软件：
[root@hadoop103 conf]# tar -zxvf /opt/software/apache-ranger-2.0.0/target/ranger-2.0.0-usersync.tar.gz -C /opt/module/ranger/

3、配置：
[root@hadoop103 ranger-2.0.0-usersync]# vim install.properties

#rangeradmin的url
POLICY_MGR_URL =http://hadoop103:6080
#同步间隔时间，单位(分钟)
SYNC_INTERVAL = 1
#运行此进程的linux用户
unix_user=root
unix_group=root
#rangerUserSync的用户密码，参考rangeradmin中install.properties的配置
rangerUsersync_password=username123
#hadoop的配置文件目录
hadoop_conf=/opt/module/hadoop-3.1.3/etc/hadoop

4、安装：
[root@hadoop103 ranger-2.0.0-usersync]# ./setup.sh

5、安装成功提示：
ranger.usersync.policymgr.password has been successfully created.
Provider jceks://file/etc/ranger/usersync/conf/rangerusersync.jceks was updated.
[I] Successfully updated password of rangerusersync user

6、启动：
[root@hadoop103 ranger-2.0.0-usersync]# ranger-usersync start
Starting Apache Ranger Usersync Service
Apache Ranger Usersync Service with pid 7510 has started.

7、web查看linux用户已同步
http://hadoop103:6080
```

#### 安装hive插件：

```
1、解压软件：
[root@hadoop103 ranger-2.0.0-usersync]# tar -zxvf /opt/software/apache-ranger-2.0.0/target/ranger-2.0.0-hive-plugin.tar.gz -C /opt/module/ranger/

2、配置：
[root@hadoop103 ranger-2.0.0-hive-plugin]# vim install.properties

#策略管理器的url地址
POLICY_MGR_URL=http://hadoop103:6080
#组件名称可以自定义
REPOSITORY_NAME=hive
#hive的安装目录
COMPONENT_INSTALL_DIR_NAME=/opt/module/hive
#hive组件的启动用户
CUSTOM_USER=root
#hive组件启动用户所属组
CUSTOM_GROUP=root

3、将hive的配置文件作为软连接安装到Ranger Hive-plugin目录下
[root@hadoop103 ranger-2.0.0-hive-plugin]# ln -s /opt/module/hive/conf/ conf

4、使用root用户启用Ranger Hive-plugin
[root@hadoop103 ranger-2.0.0-hive-plugin]# ./enable-hive-plugin.sh

5、重启hive生效

6、在web界面添加hive服务
```

### 7、权限控制

tom用户配置default库emp和dept表的所有列的读权限，为jack用户配置default库emp和dept表的所有列的读写权限

```
1、点击Add New Policy按钮

2、填写策略名称，以及此策略设计的库、表、列等信息

3、填写设计此策略的允许的用户权限

4、之后点击Add添加按钮，发现在面板上已经添加完成

5、结果：
发现tom和jack用户已经可以进行查询，但是只能查询自己有权限查询的表信息
tom用户只有读权限，没有写权限
jack用户有读权限和写权限
```

指定tom用户在查询emp表时，对手机号部分脱敏！

```
1、首先需要保证用户对指定的列有访问权限（参考上个步骤）

2、点击Masing标签，再点击Add New Policy

3、填写策略名称，以及此策略设计的库、表、列等信息

4、指定用户和脱敏操作
```

tom用户只允许查询emp表中job类型为SALESMAN的用户信息

```
1、行级别过滤也要求用户对指定表有access权限

2、选择Row Level Filter标签，点击Add New Policy:

3、选择对应的库和表

4、添加过滤规则和用户
```

### 8、官网

https://cwiki.apache.org/confluence/display/RANGER/Row-level+filtering+and+column-masking+using+Apache+Ranger+policies+in+Apache+Hive