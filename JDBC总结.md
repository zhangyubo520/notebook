# JDBC总结

### 一、概念

#### 1：JDBC的理解

```
sun公司定义的一套ApI,实现对数据库的管理操作（包括获取连接和CURD等操作），
说明：面向应用的API，Java API，抽象接口，供应用程序开发人员使用（连接数据库，执行SQL语句，获得结果）；
面向数据库的API：Java Driver API，供开发商开发数据库驱动程序用
```

#### 2、获取连接的4要素

```
1、驱动：com.mysql.jdbc.Driver
2、URL: jdbc:mysql://localhost:3306/test
3、用户名和密码：userName,password
```

#### 3、连接方式

##### 方式1：

```
//获取要素			
			String url = "jdbc:mysql://localhost:3306/test";//URL
            String user = "root";//用户名
            String password = "abc123";//密码
            String driverName = "com.mysql.jdbc.Driver";//驱动
//加载驱动
			Class.forName(driverName);
//获取连接
			Connection connection = DriverManager.getConnection(url,user,password);
```

##### 方式2：

```
//获取要素
			*将下面内容写入Properties*
			url = "jdbc:mysql://localhost:3306/test";//URL
            user = "root";//用户名
            password = "abc123";//密码
            driverName = "com.mysql.jdbc.Driver";//驱动
            *读取文件*
            Properties p = new Properties();
            InputStream is = ClassLoder.getSystemClassLoder().getResourceAsStream("文件路径");
            p.load(is);
            url = p.getProperty();//URL
            user = p.getProperty();//用户名
            password = p.getProperty();//密码
            driverName = p.getProperty();//驱动
//加载驱动
			Class.forName(driverName);
//获取连接
			Connection connection = DriverManager.getConnection(url,user,password);
```

##### 方式3：

```
//获取要素
			配置文件为
			username=root
			password=123456
			driverClassName=com.mysql.jdbc.Driver

			initialSize=10
			maxActive=20
			maxWait=1000
			filters=wall
			读取文件：
			Properties p = new Properties()
			p.load(ClassLoader.getSystemClassLoader().getResouceAsStream(配置文件路径))
//创建数据源
			DataSource ds = druidDataSourceFactory.createDataSource(p);
//获取连接
			Connection connection = ds.getConnection();
```

#### 4、数据类型的对应关系

| Java类型           | SQL类型                  |
| ------------------ | ------------------------ |
| boolean            | BIT                      |
| byte               | TINYINT                  |
| short              | SMALLINT                 |
| int                | INTEGER                  |
| long               | BIGINT                   |
| String             | CHAR,VARCHAR,LONGVARCHAR |
| byte   array       | BINARY  ,    VAR BINARY  |
| java.sql.Date      | DATE                     |
| java.sql.Time      | TIME                     |
| java.sql.Timestamp | TIMESTAMP                |

#### 5、事务的理解

```
1、概念：事务是使数据从一种状态到另一种状态的过程，
2、属性：原子性、一致性、隔离性、持久性，
3、脏读：事务a读取了事务b更新但没有提交的数据，若事务b回滚，则事务a读取的数据就是临时且无效的，
4、不可重复读：事务a读取了事务b更新提交前的数据，当事务b更新提交后，事务a读取的就不是原来的值了，
5、幻读：对于两个事务T1, T2, T1 从一个表中读取了一个字段, 然后 T2 在该表中插入了一些新的行。之后, 如果 T1 再次读取同一个表, 就会多出几行。
```



### 二、代码

#### 1、创建连接工具类

```
public class JDBCUtils {
    private static DataSource source;//声明静态属性连接池

    static {
        Properties p = new Properties();
        InputStream is = ClassLoader.getSystemClassLoader().getResourceAsStream("druid.properties");//获取配置文件
        try {
            p.load(is);
            source = DruidDataSourceFactory.createDataSource(p);//创建针对配置文件的连接池
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static Connection getConnection() throws Exception {
        Connection connection = source.getConnection();//从连接池中获取连接
        return connection;
    }

    public static void close(Connection connection) {//归还连接
        try {
            if (connection != null)
                connection.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
说明：连接池的属性和要连接的数据库写在了配置文件里，每次更新配置文件就可以了
```

#### 2、增删改查

##### 1、增

```
	QueryRunner runner = new QueryRunner();//创建QueryRunner对象
	
	Connection connection = JDBCUtils.getConnection();//使用工具类创建连接
	
	String sql = "insert into customers(name,email,birth)values(?,?,?)";//填写sql语句
	
	int count = runner.update(connection, sql, "何成飞", "he@qq.com", "1992-09-08");//调用update()方法进行数据库操作
		
	JDBCUtils.close(connection);//关闭连接
	
说明:
1、update()方法返回值是影响的数据条数
2、update()方法的参数列表分别是：连接，sql语句，形参列表
```

##### 2、删

	QueryRunner runner = new QueryRunner();//创建QueryRunner对象
	
	Connection connection = JDBCUtils.getConnection();//使用工具类创建连接
	
	String sql = "delete from customers where id = ?";//填写sql语句
	
	int count = runner.update(connection,sql,17);//调用update()方法进行数据库操作
		
	JDBCUtils.close(connection);//关闭连接
##### 3、改

```
QueryRunner runner = new QueryRunner();//创建QueryRunner对象

Connection connection = JDBCUtils.getConnection();//使用工具类创建连接

String sql = "update customers set name = ? where id = ?";//填写sql语句

int count = runner.update(connection,sql,"陈思齐"，14);//调用update()方法进行数据库操作
	
JDBCUtils.close(connection);//关闭连接
```

##### 4、查一条数据

```
QueryRunner runner = new QueryRunner();//创建QueryRunner对象

Connection connection = JDBCUtils.getConnection();//使用工具类创建连接

String sql = "select id,name,salary from customers where id = ?";//填写sql语句

BeanHandler<Customer> handler = new BeanHandler<>(Customer.class);//获取Beanhandler对象

Customer customer = runner.query(connection,sql,handler,14);//调用query()方法进行数据库操作
	
JDBCUtils.close(connection);//关闭连接

说明：
1、BeanHandler用来接收一条数据，使用前需要创建表数据对应的对象即Customer类的javabean
2、使用query方法处理查询语句，参数多了BeanHandler对象
```

##### 5、查多条数据

```
QueryRunner runner = new QueryRunner();//创建QueryRunner对象

Connection connection = JDBCUtils.getConnection();//使用工具类创建连接

String sql = "select id,name,salary from customers where id < ?";//填写sql语句

BeanListHandler<Customer> handler = new BeanListHandler<>(Customer.class);//获取Beanhandler对象构成的list

List<Customer> customers = runner.query(connection,sql,handler,14);//调用query()方法进行数据库操作
	
JDBCUtils.close(connection);//关闭连接
```

##### 6、查一条记录，使用Map

```
QueryRunner runner = new QueryRunner();//创建QueryRunner对象

Connection connection = JDBCUtils.getConnection();//使用工具类创建连接

String sql = "select id,name,salary from customers where id = ?";//填写sql语句

MapHandler handler = new MapHandler();//获取Maphandler对象构成的map

Map<String,Object> map = runner.query(connection,sql,handler,14);//调用query()方法进行数据库操作
	
JDBCUtils.close(connection);//关闭连接

说明：String为列名，对应的数据为Object类型，表现形式是name="Tom"
```

##### 7、查一条记录，最大值

```
QueryRunner runner = new QueryRunner();//创建QueryRunner对象

Connection connection = JDBCUtils.getConnection();//使用工具类创建连接

String sql = "select max(salary) from customers";//填写sql语句

ScalarHandler handler = new ScalarHandler();//获取Scalarhandler对象构成的map

long maxSalary = runner.query(connection,sql,handler);//调用query()方法进行数据库操作
	
JDBCUtils.close(connection);//关闭连接

说明：
1、ArrayHandler：用数组呈现表中的一行数据，

2、ArrayListHandler：用List呈现多行数据，List中的元素为数组，数组中为表中的一行数据,

3、BeanHandler：用JavaBean实例呈现表中一行数据，前提是造一个和表对应的javaBean,

4、BeanListHandler：用List呈现多行数据，List中的元素为javaBean实例，javaBean实例中为表中一行数据，

5、ColumnListHandler：将某一列的数据存放到List中，

6、KeyedHandler(name)：将结果的每一行数据都封装到一个Map里，再把这些map再存到一个map里，其key为指定的key，比如说name,遍历map时我们就能用name,便利到所有人的信息，

7、MapHandler：将结果的第一行数据封装到一个Map里，key是列名，value就是对应的值，

8、MapListHandler：将结果的每一行数据都封装到一个Map里，然后再存放到List，

9、ScalarHandler：查询单个值对象（对应单行函数），
```

#### 3、设置隔离

```
1、显示级别：select @@tx_isolation
2、设置级别：set transaction isolation level read committed (当前数据库)
		   set global transaction isolation level read committed (全局)
3、新建用户：create user `Tom` identified by '123abc'
4、授予权限：

#授予通过网络方式登录的tom用户，对所有库所有表的全部权限，密码设为abc123.
grant all privileges on *.* to tom@'%'  identified by 'abc123'; 

 #给tom用户使用本地命令行方式，授予atguigudb这个库下的所有表的插删改查的权限。
grant select,insert,delete,update on atguigudb.* to tom@localhost identified by 'abc123'; 
```

#### 4、PreparedStatement

```
//1、注册驱动
		Class.forName("com.mysql.jdbc.Driver");
		
		//2、获取连接
		Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/day04_test01_bookstore", "root", "1234");
		
		//3、编写sql
		String sql = "DELETE FROM orders WHERE id = '15275760194821'";
		
		//4、创建PreparedStatement
		PreparedStatement pst = conn.prepareStatement(sql);//此时的sql带?的
		
		//5、执行sql
		int len = pst.executeUpdate();
		System.out.println(len>=0 ? "删除成功" : "删除失败");
		//6、关闭
		pst.close();
		conn.close();
```

