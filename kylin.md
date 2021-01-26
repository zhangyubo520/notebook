# kylin

### 1、概念

```
Apache Kylin是一个开源的分布式分析引擎，提供Hadoop/Spark之上的SQL查询接口及多维分析（OLAP）能力以支持超大规模数据，最初由eBay Inc开发并贡献至开源社区。它能在亚秒内查询巨大的Hive表

Kylin的主要特点包括支持SQL接口、支持超大规模数据集、亚秒级响应、可伸缩性、高吞吐率、BI工具集成

```

### 2、kylin安装

```
1、安装前环境准备：Hadoop、Hive、Zookeeper、HBase需要提前准备好，环境变量配置好

2、解压：tar -zxvf apache-kylin-3.0.2-bin.tar.gz -C /opt/module/

3、改名：mv /opt/module/apache-kylin-3.0.2-bin /opt/module/kylin

4、启动准备：需先启动Hadoop（hdfs，yarn，jobhistoryserver）、Zookeeper、Hbase

5、启动Kylin: /opt/module/kylin/bin/kylin.sh start

6、web查看http://hadoop102:7070/kylin

7、登录：用户名为：ADMIN，密码为：KYLIN

8、关闭kylin: /opt/module/kylin/bin/kylin.sh stop
```

### 3、kylin的简单实用

```
略
```

### 4、原理理解

```
略
```

### 5、java程序

```
1、依赖：
    <dependencies>
        <dependency>
            <groupId>org.apache.kylin</groupId>
            <artifactId>kylin-jdbc</artifactId>
            <version>2.5.1</version>
        </dependency>
    </dependencies>

2、程序编写：
public class TestKylin {

    public static void main(String[] args) throws Exception {

        //Kylin_JDBC 驱动
        String KYLIN_DRIVER = "org.apache.kylin.jdbc.Driver";

        //Kylin_URL
        String KYLIN_URL = "jdbc:kylin://hadoop102:7070/FirstProject";

        //Kylin的用户名
        String KYLIN_USER = "ADMIN";

        //Kylin的密码
        String KYLIN_PASSWD = "KYLIN";

        //添加驱动信息
        Class.forName(KYLIN_DRIVER);

        //获取连接
        Connection connection = DriverManager.getConnection(KYLIN_URL, KYLIN_USER, KYLIN_PASSWD);

        //预编译SQL
        PreparedStatement ps = connection.prepareStatement("SELECT sum(sal) FROM emp group by deptno");

        //执行查询
        ResultSet resultSet = ps.executeQuery();

        //遍历打印
        while (resultSet.next()) {
            System.out.println(resultSet.getInt(1));
        }
    }
}

```

