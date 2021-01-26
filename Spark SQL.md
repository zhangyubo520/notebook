# Spark SQL

### 1、概念

```
是spark用于处理结构化数据的spark模块
```

### 2、两个数据抽象

#### DataFrame

```
介绍：以RDD为基础的分布式数据集，类似二维表格，具有更多结构信息，懒执行，性能比RDD更好

创建：
1、spark.read.format("xxx").load("xxx")
2、通过RDD转换：rdd.map(s=>User(s._1,s._2)).toDF
3、
```

#### DataSet

```
介绍：分布式数据集合，强类型的，使用样例类定义数据的结构信息，DataFrame是特殊的DataSet，相当于DataSet[Row]

创建：
1、样例类转换：Seq(Person("zhangsan",2)).toDS()
2、使用基本类型序列：Seq(1,2,3,4,5).toDS
3、RDD转换：rdd.map(s=>User(s._1,s._2)).toDS
```

### 3、两种语法风格

#### SQL语法

```
使用该语法需要先创建临时视图或者全局视图

	val con: SparkConf = new SparkConf().setMaster("local[*]").setAppName("test01")

    val ssc: SparkSession = new spark.sql.SparkSession.Builder().config(con).getOrCreate()//创建上下文环境

    val df: DataFrame = ssc.read.format("json").load("input/User.json")//读取文件生成DataFrame

    df.createOrReplaceTempView("user")//创建临时视图

    val df1: DataFrame = ssc.sql("select * from user")//使用临时视图
    df1.show()//显示
    
全局表的说明：
df.createGlobalTempView("people")
ssc.sql("SELECT * FROM global_temp.people").show()//必须使用global_temp.的方式使用全局视图
```

#### DSL语法

```
 	val con: SparkConf = new SparkConf().setMaster("local[*]").setAppName("test01")

    val ssc: SparkSession = new spark.sql.SparkSession.Builder().config(con).getOrCreate()//创建上下文环境

    val df: DataFrame = ssc.read.format("json").load("input/User.json")//读取文件生成DataFrame

    df.printSchema()
    df.select("name").show()//直接使用select等
```

### 4、RDD、DS和DF的转换

```
首先需要引入import ssc.implicits._ //引入上下文环境的隐式转换

1、RDD转换成DF:
普通转换：rdd.toDF("id","name")
一般使用样例类转换：rdd.map(s=>User(s._1,s._2)).toDF.show

2、RDD转换成DS
rdd.map(s=>User(s._1,s._2)).toDS

3、DF和DS转换成RDD:ds.rdd 和 df.rdd

4、DF和DS互相转换：df.as[User] 和 ds.toDF
```

### 5、idea开发

```
1、添加依赖
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-sql_2.12</artifactId>
    <version>2.4.5</version>
</dependency>
2、创建环境

	val con: SparkConf = new SparkConf().setMaster("local[*]").setAppName("test01")

    val ssc: SparkSession = new spark.sql.SparkSession.Builder().config(con).getOrCreate()//创建上下文环境
3、使用

4、释放资源：ssc.stop()
```

### 6、使用自定义函数

#### UDF

对一行数据处理

```
	val con: SparkConf = new SparkConf().setMaster("local[*]").setAppName("test01")

    val ssc: SparkSession = new spark.sql.SparkSession.Builder().config(con).getOrCreate()//创建上下文环境

    val df: DataFrame = ssc.read.format("json").load("input/User.json")//读取文件生成DataFrame

    ssc.udf.register("fun",//函数名称
      (x : String,y : Long) => {
        (x + "已离职",y + 1)
    })//编写函数体，对表中一行数据进行处理

    df.createOrReplaceTempView("user")//创建临时视图

    ssc.sql("select fun(name,age) as newName from user").show()//生成新的一列进行显示
	/*
	|         newName|
	+----------------+
	|[lisi已离职, 19]|
	+----------------+
	 */
```

#### UDAF

多行聚合函数，自带的聚合函数有count()，countDistinct()，avg()，max()，min()等

```
class MyUDAF extends UserDefinedAggregateFunction{
    override def inputSchema: StructType = {
      StructType(Array(StructField("age",LongType)))
    }//输入的数据及类型

    override def bufferSchema: StructType = {
      StructType(Array(
        StructField("sum",LongType),
        StructField("count",LongType),
      ))
    }//缓冲区的数据及类型

    override def dataType: DataType = {
      DoubleType
    }//最终结果的类型

    override def deterministic: Boolean = {
      true
    }//相同的输入是否一直返回相同的输出

    override def initialize(buffer: MutableAggregationBuffer): Unit = {
      buffer(0) = 0L
      buffer(1) = 0L
    }//将缓冲区的两个数据均初始化

    override def update(buffer: MutableAggregationBuffer, input: Row): Unit = {
      buffer(0) = buffer.getLong(0) + input.getLong(0)
      buffer(1) = buffer.getLong(1) + 1
    }//更新缓冲区数据

    override def merge(buffer1: MutableAggregationBuffer, buffer2: Row): Unit = {
      buffer1(0) = buffer1.getLong(0) + buffer2.getLong(0)
      buffer1(1) = buffer1.getLong(1) + buffer2.getLong(1)
    }//缓冲区数据两两合并

    override def evaluate(buffer: Row): Any = {
      buffer.getLong(0).toDouble / buffer.getLong(1)
    }//最终结果怎么得到
    
    使用说明：
    创建：val myUDAF = new MyUDAF 
    注册：ssc.udf.register("age",myUDAF)
    使用：ssc.sql("select age(age) from user").show()
```

### 7、数据的读取和保存

#### 读取

```
ssc.read.
format option load 数据格式 参数 文件路径
csv	jdbc json orc   parquet   schema   table   text   textFile
```

#### 保存

```
ssc.write.
format option save mode 数据格式 参数 保存路径 保存类型(error 存在会报错 append 存在会追加 overwrite ignore)
csv  jdbc   json  orc   parquet textFile
```

#### 各种数据格式介绍

```
parquet：默认的，一种能够有效存储嵌套数据的列式存储格式，可以通过spark.sql.sources.default修改

json:自动推测文件结构，转换成DataSet[Row],每一行都应该是{"name":"li","age":15L}格式

csv:
ssc.read.format("csv").option("sep", ";").option("inferSchema", "true").option("header", "true").load("data/user.csv")//分隔符 使用类型推断 第一行作为类型

MySQL：
需要导入依赖：
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.27</version>
</dependency>

1、val df: DataFrame = ssc.read.format("jdbc")
      .option("url", "jdbc:mysql://hadoop102:3306/dbtable") //dbtable是数据库的名字
      .option("driver", "com.mysql.jdbc.Driver")
      .option("user", "root") //用户
      .option("password", "123456") //密码
      .option("dbtable", "user") //表
      .load()
2、val df: DataFrame = ssc.read.format("jdbc")
      .options(
        Map(
          "url" -> "jdbc:mysql://hadoop102:3306/dbtable?user=root&password=123456",
          "dbtable" -> "user",
          "driver" -> "com.mysql.jdbc.Driver")).load()
          
3、val props: Properties = new Properties()
props.setProperty("user", "root")
props.setProperty("password", "123123")
val df: DataFrame = spark.read.jdbc("jdbc:mysql://hadoop102:3306/dbtable", "user", props)

Hive:支持 Hive 表访问、UDF (用户自定义函数)以及 Hive 查询语言(HiveQL/HQL)
1、导入依赖
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-hive_2.12</artifactId>
    <version>2.4.5</version>
</dependency>

<dependency>
    <groupId>org.apache.hive</groupId>
    <artifactId>hive-exec</artifactId>
    <version>3.1.2</version>
</dependency>
2、hive-site.xml文件拷贝到项目的resources目录

3、增加System.setProperty("HADOOP_USER_NAME", "root")

4、使用
System.setProperty("HADOOP_USER_NAME", "atguigu")//hdfs访问权限
    val se : SparkSession = MyUtil.getSe("test03")//获取资源

    se.sql(
      """
        |CREATE TABLE `product_info`(
        |  `product_id` bigint,
        |  `product_name` string,
        |  `extend_info` string)
        |row format delimited fields terminated by '\t'
            """.stripMargin)

    se.sql(
      """
        |load data local inpath 'input/product_info.txt' into table product_info
            """.stripMargin)

    se.sql(
      """
        |CREATE TABLE `city_info`(
        |  `city_id` bigint,
        |  `city_name` string,
        |  `area` string)
        |row format delimited fields terminated by '\t'
            """.stripMargin)

    se.sql(
      """
        |load data local inpath 'input/city_info.txt' into table city_info
            """.stripMargin)
```

### 8、各区域热门商品 Top3

| **地区** | **商品名称** | **点击次数** | **城市备注**                    |
| -------- | ------------ | ------------ | ------------------------------- |
| **华北** | 商品A        | 100000       | 北京21.2%，天津13.2%，其他65.6% |
| **华北** | 商品P        | 80200        | 北京63.0%，太原10%，其他27.0%   |
| **华北** | 商品M        | 40000        | 北京63.0%，太原10%，其他27.0%   |
| **东北** | 商品J        | 92000        | 大连28%，辽宁17.0%，其他 55.0%  |

```
第一步，建表并插入数据
System.setProperty("HADOOP_USER_NAME", "atguigu")//hdfs访问权限
    val se : SparkSession = MyUtil.getSe("test03")//获取资源

    se.sql(
      """
        |CREATE TABLE `product_info`(
        |  `product_id` bigint,
        |  `product_name` string,
        |  `extend_info` string)
        |row format delimited fields terminated by '\t'
            """.stripMargin)

    se.sql(
      """
        |load data local inpath 'input/product_info.txt' into table product_info
            """.stripMargin)

    se.sql(
      """
        |CREATE TABLE `city_info`(
        |  `city_id` bigint,
        |  `city_name` string,
        |  `area` string)
        |row format delimited fields terminated by '\t'
            """.stripMargin)

    se.sql(
      """
        |load data local inpath 'input/city_info.txt' into table city_info
            """.stripMargin)

    se.stop()
```

```
第二步、分步查询获得最终结果
System.setProperty("HADOOP_USER_NAME", "atguigu")
    val se : SparkSession = MyUtil.getSe("test03")

    se.sql(
      """
        |select
        |   a.*,
        |   c.area,
        |   p.product_name,
        |   c.city_name
        |from user_visit_action a
        |join city_info c on c.city_id = a.city_id
        |join product_info p on p.product_id = a.click_product_id
        |where a.click_product_id > -1
            """.stripMargin).createOrReplaceTempView("t1")

//    se.sql("select * from t1").show(10)

    val remark = new CityRemark
    se.udf.register("cityRemark",remark)


    se.sql(
      """
        |select
        |    area,
        |    product_name,
        |    count(*) as clickCount,
        |    cityRemark(city_name)
        |from t1 group by area, product_name
            """.stripMargin).createOrReplaceTempView("t2")

    se.sql(
      """
        |select
        |    *,
        |    rank() over( partition by area order by clickCount desc ) as rank
        |from t2
            """.stripMargin).createOrReplaceTempView("t3")

    se.sql(
      """
        |select
        |   *
        |from t3
        |where rank <= 3
            """.stripMargin).show


    se.stop()
    
    //UDAF类
    class CityRemark extends UserDefinedAggregateFunction{
    override def inputSchema: StructType = {

      StructType(Array(StructField("cityName", StringType)))
    }

    override def bufferSchema: StructType = {

      StructType(Array(
        StructField("total",LongType),
        StructField("citymap",MapType(StringType,LongType))
      ))
    }

    override def dataType: DataType = StringType

    override def deterministic: Boolean = true

    override def initialize(buffer: MutableAggregationBuffer): Unit = {
      buffer(0) = 0L
      buffer(1) = Map[String,Long]()
    }

    override def update(buffer: MutableAggregationBuffer, input: Row): Unit = {
      val cityname = input.getString(0)
      buffer(0) = buffer.getLong(0) + 1
      val citymap = buffer.getAs[Map[String,Long]](1)

      val click = citymap.getOrElse(cityname,0L) + 1

      buffer(1) = citymap.updated(cityname,click)

    }

    override def merge(buffer1: MutableAggregationBuffer, buffer2: Row): Unit = {
      buffer1(0) = buffer1.getLong(0) + buffer2.getLong(0)

      val map1 = buffer1.getAs[Map[String,Long]](1)
      val map2 = buffer2.getAs[Map[String,Long]](1)

      buffer1(1) = map1.foldLeft(map2){
        case (map,(k,v)) => {
              map.updated(k, map.getOrElse(k,0L) + v)
        }
      }
    }

    override def evaluate(buffer: Row): Any = {
      val total = buffer.getLong(0)

      val citymap = buffer.getMap[String,Long](1)

      val cityList: List[(String, Long)] = citymap.toList.sortWith {
        case (left, right) => {
          left._2 > right._2
        }
      }.take(2)
      var rest = 0L
      val s = new StringBuilder
      cityList.foreach{
        case ( city, cnt ) => {
          val r = ( cnt * 100 / total )
          s.append(city + " " + r + "%,")
          rest = rest + r
        }
      }
      s.toString() + "其他：" + (100 - rest) + "%"


    }
```

