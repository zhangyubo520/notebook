### azkaban

### 1、azkaban概述

```
工作流调度系统
```

### 2、常见的任务调度

```
1、简单的任务调度：直接使用linux的crontab来定义

2、复杂的任务调度：开发调度平台或使用现成的开源调度系统，比如ooize、azkaban、Cascading、Hamake

```

### 3、ooize和azkaban对比

```
1、功能上：
两者均可以调度mapreduce，pig，java，脚本工作流任务
两者均可以定时执行工作流任务

2、传参数：
Azkaban支持直接传参，例如${input}
Oozie支持参数和EL表达式，例如${fs:dirSize(myInputDir)}

3、定时执行
Azkaban的定时执行任务是基于时间的
Oozie的定时执行任务基于时间和输入数据

```

4、azkaban特点

```
1）Web用户界面
2）方便上传工作流
3）方便设置任务之间的关系
4）调度工作流
5）认证/授权(权限的工作)
6）能够杀死并重新启动工作流
7）模块化和可插拔的插件机制
8）项目工作区
9）工作流和任务的日志记录和审计
```

### 5、集群搭建

```
1、上传包：azkaban-db-3.84.4.tar.gz，azkaban-exec-server-3.84.4.tar.gz，azkaban-web-server-3.84.4.tar.gz

2、解压：先创建文件夹mkdir /opt/module/azkaban，然后解压ls /opt/software/azkaban*.tar.gz | xargs -n1 tar zxC /opt/module/azkaban -f

3、初始化mysql:
登录：mysql -uroot -p000000
mysql> CREATE DATABASE azkaban;
mysql> CREATE USER 'azkaban'@'%' IDENTIFIED BY '000000';
mysql> GRANT SELECT,INSERT,UPDATE,DELETE ON azkaban.* to 'azkaban'@'%' WITH GRANT OPTION;
mysql> use azkaban;
mysql> source /opt/module/azkaban/azkaban-db-3.84.4/create-all-sql-3.84.4.sql
mysql> quit;

4、修改mysql配置
sudo vim /etc/my.cnf
max_allowed_packet=1024M

5、重启mysql
sudo systemctl restart mysqld

6、设置Executor Server
vim /opt/module/azkaban/azkaban-exec-server-3.84.4/conf/azkaban.properties
...
default.timezone.id=Asia/Shanghai
...
azkaban.webserver.url=http://hadoop102:8081
...
database.type=mysql
mysql.port=3306
mysql.host=hadoop102
mysql.database=azkaban
mysql.user=root
mysql.password=123456
mysql.numconnections=100
executor.metric.reports=true
executor.metric.milisecinterval.default=60000

7、设置Web Server
vim /opt/module/azkaban/azkaban-web-server-3.84.4/conf/azkaban.properties
...
default.timezone.id=Asia/Shanghai
...
database.type=mysql
mysql.port=3306
mysql.host=hadoop102
mysql.database=azkaban
mysql.user=root
mysql.password=123456
mysql.numconnections=100
...
azkaban.executorselector.filters=StaticRemainingFlowSize,CpuStatus

8、修改azkaban-users.xml文件（在web文件下）
vim /opt/module/azkaban/azkaban-web-server-3.84.4/conf/azkaban-users.xml
<azkaban-users>
  <user groups="azkaban" password="azkaban" roles="admin" username="azkaban"/>
  <user password="metrics" roles="metrics" username="metrics"/>
  <user password="123456" roles="metrics,admin" username="atguigu"/>

  <role name="admin" permissions="ADMIN"/>
  <role name="metrics" permissions="METRICS"/>
</azkaban-users>

9、同步azkaban至另外两个节点

10、激活executor server（三台）：
cd /opt/module/azkaban/azkaban-exec-server-3.84.4
bin/start-exec.sh
目录下出现executor.port文件，说明启动成功
激活命令：
curl -G "hadoop102:$(<./executor.port)/executor?action=activate" && echo
curl -G "hadoop103:$(<./executor.port)/executor?action=activate" && echo
curl -G "hadoop104:$(<./executor.port)/executor?action=activate" && echo
三台机器均会出现{"status":"success"}提示

11、在102启动web server
cd /opt/module/azkaban/azkaban-web-server-3.84.4
bin/start-web.sh

12、访问http://hadoop102:8081,并用atguigu，123456用户登陆
```

### 6、范例

#### 1、入门案例

```
1、新建文件：first.project，写入：
azkaban-flow-version: 2.0

2、新建文件：basic.flow，写入：
nodes:
  - name: jobA
    type: command
    config:
      command: echo "This is an echoed text."

3、将两个文件打包到一起（必须zip）

4、在web页面新建工程并上传zip文件


5、执行
```

#### 2、有依赖的

```
nodes:
  - name: jobC
    type: command
    # jobC 依赖 JobA和JobB
    dependsOn:
      - jobA
      - jobB
    config:
      command: echo "I’m JobC"

  - name: jobA
    type: command
    config:
      command: echo "I’m JobA"

  - name: jobB
    type: command
    config:
      command: echo "I’m JobB"
```

#### 3、全局变量

```
config:
  words.to.print: "This is for test!"

nodes:
  - name: jobA
    type: command
    config:
      command: echo ${words.to.print}
```

#### 4、子工作流

```
nodes:
  - name: jobC
    type: command
    # jobC 依赖embedded_flow
    dependsOn:
      - embedded_flow
    config:
      command: echo "I’m JobC"

  - name: embedded_flow
    type: flow
    config:
      prop: value
    nodes:
      - name: jobB
        type: noop
        dependsOn:
          - jobA

      - name: jobA
        type: command
        config:
          command: pwd
```

#### 5、java代码

```
1、新建java程序
package com.atguigu;

public class AzTest {
    public static void main(String[] args) {
        System.out.println("This is for testing!");
    }
}

2、打包成jar包

3、testJava.flow
nodes:
  - name: test_java
    type: javaprocess
    config:
      Xms: 96M
      Xmx: 200M
      java.class: com.atguigu.AzTest

4、将Jar包、flow文件和project文件打包成zip，上传到集群并执行
```

#### 6、条件工作流

```
nodes:
 - name: JobA
   type: command
   config:
     command: sh /opt/module/write_to_props.sh

 - name: JobB
   type: command
   dependsOn:
     - JobA
   config:
     command: echo "This is JobB."
   condition: ${JobA:param1} == "AAA"

 - name: JobC
   type: command
   dependsOn:
     - JobA
   config:
     command: echo "This is JobC."
   condition: ${JobA:param1} == "BBB"
```

#### 7、根据执行是否失败来判断下一个任务执行

```
nodes:
  - name: JobA
    type: command
    config:
      command: sh /opt/module/write_to_props.sh

  - name: JobB
    type: command
    dependsOn:
      - JobA
    config:
      command: echo "This is JobB."
    condition: ${JobA:param1} == "AAA"

  - name: JobC
    type: command
    dependsOn:
      - JobA
    config:
      command: echo "This is JobC."
    condition: ${JobA:param1} == "BBB"

  - name: JobD
    type: command
    dependsOn:
      - JobB
      - JobC
    config:
      command: echo "This is JobD."
    condition: one_success

  - name: JobE
    type: command
    dependsOn:
      - JobB
      - JobC
    config:
      command: echo "This is JobE."
    condition: all_success

  - name: JobF
    type: command
    dependsOn:
      - JobB
      - JobC
      - JobD
      - JobE
    config:
      command: echo "This is JobF."
condition: all_done


说明：
1)	all_success: 全部成功(默认)
2)	all_done：全部完成
3)	all_failed：全部失败
4)	one_success：至少一个成功
5)	one_failed：至少一个失败

```

### 7、邮件报警

```
1、web端配置文件
vim /opt/module/azkaban/azkaban-web-server-3.84.4/conf/azkaban.properties

#这里设置邮件发送服务器，需要 申请邮箱，切开通stmp服务，以下只是例子
mail.sender=atguigu@126.com
mail.host=smtp.126.com
mail.user=atguigu@126.com
mail.passwd=password

#这里设置工作流成功或者失败默认向哪里发送服务
job.failure.email=atguigu@126.com
job.success.email=atguigu@126.com
2、重启web服务

3、修改工作流文件flow
nodes:
  - name: jobA
    type: command
    config:
      command: echo "This is an echoed text."
      failure.emails: atguigu@126.com
      success.emails: atguigu@126.com
      notify.emails: atguigu@126.com 

```

### 8、电话报警

```

```

### 9、YAML语法