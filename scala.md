# scala

## 一、基础

面向对象的函数式编程语言，官网https://www.scala-lang.org/

### 1、程序执行流程

```
类名.class调用 static main

static main调用类名$.class

类名$.class调用main

main调用println
```

### 2、访问权限问题

#### java中

```
private:私有化，即只能在本类中使用

protect：同一个包内可用(跨类) + 有条件的跨包(说明：当调用者和提供者是子父类关系时可以调用)

缺省：同一个包下可以用(跨类)

public：哪里都能用(跨包)

例子：
```

### 3、构造对象

```
1.new
val a = new A()

2.apply
val a = A()			需要在Object中声明apply方法单例构造对象

3.反射
val clazz : Class[A] = classof[A]
val a = clazz.newInstance()

4.克隆
a.clone()

5.反序列化
ObjectInputStream
```

### 4、将多个子类的方法抽到父类

```
将方法声明成protected的放在父类中即可，仅子父类能用（与java中的权限不同)
```

### 5、单例模式

```

```

### 6、数组

Array（不可变数组）

```
创建数组
val a = new Array[Int](6)
val b = Array(1,2,3,4,5,6)
val c = Array(6)

使用数组
for(i <- a){
    a(i) = a
}

println(a.mkString(","))

a(1) = 1

改变数组（数组不可变，每次改变数组长度的行为都会生成新的数组）
  val narr = b :+ "5" //末尾添加字符串5生成新的数组[Any]
  val narr1 = 5 +: b	//首位之前添加5生成新的数组[Int]
  val narr2 = b ++ narr 	//末尾添加整个数组生成新的数组[Any]

遍历数组
for(i <- Range(0,c.length)){
    println(a(i))
}

for(i <- a){}

```

ArrayBuffer（可变数组）

```
创建数组
val a = new ArrayBuffer[String]()
val b = ArrayBuffer(1,2,3,4)
val c = ArrayBuffer(5)

使用数组
a.insert(0, 2,3,4)	//从首位开始添加2,3,4（2会占据原来0号位置）
a.update(1, 2)	//把1号位置上的元素改变为2
a(1) = 2 	//把1号位置上的元素改变为2
a.remove(1,2)	//从原来的1号位置开始移除两个元素（原来1号位置上的元素会被移除）
a.append(1)		//从末尾添加元素	
改变数组
  val narr = b :+ "5" //末尾添加字符串5生成新的数组[Any]
  val narr1 = 5 +: b	//首位之前添加5生成新的数组[Int]
  val narr2 = b ++ narr 	//末尾添加整个数组生成新的数组[Any]
遍历数组
for(i <- Range(0,c.length)){
    println(a(i))
}

for(i <- a){}
```

### 7、头脑风暴

```
1、val arr : Array[Int] = Array(1,2,3,4)中val咋理解的？为啥叫不可变数组？
val 代表 arr指向一个地址值不可变，任何将arr重新指向新的地址值的操作都是不允许的

2、函数柯里化怎么理解？
def fun(i : Int)(j : Int) : Unit = {
  println(i + j)
}
val a = fun(5) _
a(4) //输出9，柯里化的理解，就是可以层层使用，类似嵌套函数

3、尾递归
尾递归本质就是每次调用递归函数都会算出一个结果存储到一个变量内存中，每次调用时访问这个变量内存就行，压栈太快还是会出现oom的，只是从逻辑上减少了压栈，没有本质解决问题

4、
```

### 8、包

声明

```
域名反写+项目名称+模块名+程序类型名称
```

作用

```
管理类 Util IO 等
区分类 Util.Date sql.Date等
访问权限
编译类，类的编译程序应该和所在的包名一致
```

scala中package

```
1、包名可以和源码路径不一致(scala会根据package的声明编译指定的class路径)

2、package可以多次声明
package com
package zhangyubo
package exer0526

3、层级结构
在package包名后增加花括号，设定作用域和层级关系，子包可以直接使用父包的内容
package com{
    package zhangyubo{
    	class A{}
    	package exer0526{
    		val a = new A()//有层级关系的时候不用使用import了
    	}
	}
}
4、包也是对象（包是包，包对象是包对象）
可以将包中所有共同性的抽到包对象中（新建的当前包的对象）

```

### 9、import

含义

```
1、导包其实是导类
2、静态导入（导入类的静态属性和静态方法，这样就不用用类名，可以直接使用静态的东西）
```

scala中

```
1、import可以声明在任意位置
2、java中默认导入的是java.lang包
3、scala中默认导入的是java.lang包和scala包和Predef(类似静态导入)
4、scala中import可以导包
5、scala中使用import java.util._导入包内的所有类
6、在一行导入多个类import java.util.{ArrayList,HashMap}
7、使用import隐藏类
import java.sql.{Date=>_,_}//其中的Date被隐藏
import java.util._
8、起别名
type UtilDate = java.util.Date
val a : UtilDate = new UtilDate()
```

### 10、双亲委派

```
启动类加载器：加载jdk自带的类
扩展类加载器：java扩展类库
应用类加载器：classpath下的类
```

### 11、class和object区别

```
object编译时：产生一个当前类文件（伴生类），还有一个单例的类文件（用来模仿静态）（伴生对象）
class编译时：只会产生当前类文件
```

### 12、空集合

```
val arr = Nil
```

### 13、set

```
  val value = mutable.Set(1,2,3,4,55,6,7,8,9,0,1)
  value.add(22)
//  删除数据（找到指定元素就删除）
  value.update(3,false)
//  添加数据（找到指定元素就添加）
  value.update(33,true)
//  删除数据（找到指定元素删除）
  value.remove(3)
```

### 14、wordcount

```
object ATest12 extends App{
  private val stream = new FileInputStream("E:\\idea\\workspace\\student0213\\test0525\\input\\word.txt")

  private val stream1 = new BufferedReader(new InputStreamReader(stream))

  val a = new ListBuffer[String]()

  Breaks.breakable{
    while(true){
      val str = stream1.readLine()
      if(str != null){
        a.append(str)
      }else{Breaks.break()}
    }
  }
  val b = a.flatten(_.split(" "))
  println(a)
  println("***********************")
  println(b)
  println("***********************")
  val c: Map[String, ListBuffer[String]] = b.groupBy(s => s)
  println(c)
  println("***********************")


  val d = c.map(kv => (kv._1,kv._2.length))
  println(d)
  val e = d.toList.sortBy(kv => -kv._2)
  println(e.take(3))
  stream.close()
```

```
filter过滤



  val b = List("chensiqi","zhangyubo","lijieng","mawenyan","yangjingjign","sunyangping")

  def fun(s : String) : Boolean = {
    if(s.startsWith("s")){true}
    else{false}
  }

  println(b.filter(fun))//过滤
```

```
sortBy排序


  val b = List("a","z" ,"d" ,"h")
  println(b.sortBy(s => s))
          // scala中的元组自动比较大小。
        // 先比较第一个元素，再比较第二个元素，异常类推
        println(list.sortBy(user => {
            (user.age, user.name)
        }))
  
  
```

```
flatMap数据扁平化

  val c = List("zhang zou","chen si")
  def fun2(s : String) : Array[String] = {
    s.split(" ")
  }

  println(c.flatMap(fun2))
  println(c.flatMap(_.split(" ")))//List(zhang, zou, chen, si)
  
  flatmap的逻辑是先执行map 再执行flatten

```

```
map批量转换，批量处理

  val a = List(1,2,3,4)
  println(a.map(_ * 2))//List(2,4,6,8)
```

```
flatten扁平化


    val xs = List(
               Set(1, 2, 3),
               Set(1, 2, 3)
             ).flatten// xs == List(1, 2, 3, 1, 2, 3)
    val ys = Set(
               List(1, 2, 3),
               List(3, 2, 1)
             ).flatten// ys == Set(1, 2, 3)
```

```
groupBy分组

  val a = List(1,2,3,4)
  def fun(num : Int) : Int = {num % 2}
  private val intToInts: Map[Int, List[Int]] = a.groupBy(fun)
  println(intToInts)//Map(1 -> List(1, 3), 0 -> List(2, 4))
  
  val a = List("a","a","e","e","e")
  private val stringToStrings: Map[String, List[String]] = a.groupBy(word=>word)
  println(stringToStrings)//Map(e -> List(e, e, e), a -> List(a, a))
```

```
下划线

1.参数出现一次
2.当做函数整体
```

### 15、方法

```
// 头 => 1
        //println(list.head)
        // 尾 => List(2,3,4)
        // 尾 -> 尾 -> 尾
        //println(list.tail)

        // 最后一个 => 4
        //println(list.last)
        // 初始 => List(1,2,3)
        //println(list.init)

        // 反转
        //println(list.reverse)
       // println(list.reverse.head)

        // 判断数据是否存在
       // println(list.contains(5))

        // 数据去重
        //println(list.toSet)
        //println(list.distinct)

        // 取(拿)数据
        //println(list.take(3))
        // 从右边拿
        // 1,2,3,4
        // 4, 3, 2
        // 2, 3, 4
        //println(list.takeRight(3))

        // 丢弃数据
        // 1,2,3,4
        println(list.drop(2))
        println(list.dropRight(2))
```

### 16、方法链

```
val dataList = List("hello scala", "hello spark", "hive hadoop")
val result = dataList
            .flatMap(_.split(" "))
            .groupBy(word=>word) // 不简化
            .map( kv => ( kv._1, kv._2.size ) ) // 不简化
            .toList
            .sortBy(_._2)(Ordering.Int.reverse)
            .take(3)
```

#### 17、不同的数据类型怎么扁平化

```
//        val list: List[Any] = List( 1,2, List(3,4) )
//        list.flatMap(
//            data => {
//                // 模式匹配
//                if ( data.isInstanceOf[List] ) {
//                    data.asInstanceOf[List]
//                } else {
//                    List(data)
//                }
//            }
//        )
```

### 18、sortWith怎么用

```
println(ints2.sortWith(
            (left, right) => {

                // 升序
                // TODO 当满足你的排序要求时，你就返回true
                // TODO 当不满足你的排序要求时，你就返回false

                //left._1 > right._1 // 降序
                //left._1 < right._1 // 升序
                if ( left._1 > right._1  ) {
                    // true, false
                    true
                } else if ( left._1 == right._1 ) {
                    left._2 < right._2
                } else {
                    // true. false
                    false
                }
            }
        ))
说明：left 和 right代表相邻两个数据
```

19、不同省份的销量排名

```
// TODO 不同省份(当中)商品点击排行
        // (item count) => (word, count)
        val dataList = List(
            ("zhangsan", "河北", "鞋"),
            ("lisi", "河北", "衣服"),
            ("wangwu", "河北", "鞋"),
            ("zhangsan", "河南", "鞋"),
            ("lisi", "河南", "衣服"),
            ("wangwu", "河南", "鞋"),
            ("zhangsan", "河南", "鞋"),
            ("lisi", "河北", "衣服"),
            ("wangwu", "河北", "鞋"),
            ("zhangsan", "河北", "鞋"),
            ("lisi", "河北", "衣服"),
            ("wangwu", "河北", "帽子"),
            ("zhangsan", "河南", "鞋"),
            ("lisi", "河南", "衣服"),
            ("wangwu", "河南", "帽子"),
            ("zhangsan", "河南", "鞋"),
            ("lisi", "河北", "衣服"),
            ("wangwu", "河北", "帽子"),
            ("lisi", "河北", "衣服"),
            ("wangwu", "河北", "电脑"),
            ("zhangsan", "河南", "鞋"),
            ("lisi", "河南", "衣服"),
            ("wangwu", "河南", "电脑"),
            ("zhangsan", "河南", "电脑"),
            ("lisi", "河北", "衣服"),
            ("wangwu", "河北", "帽子")
        )

        //  TODO 数据会存在多余的内容，应该将数据进行清洗
        // ("wangwu", "河北", "帽子") => ("河北", "帽子") => ("河北-帽子")
        val list: List[String] = dataList.map(
            t => {
                (t._2 +"-"+t._3)
            }
        )

        // TODO 应该在统计数据时，根据省份和商品同时进行分组
        // group ("河北-帽子") => ("河北-帽子", count)
        val dataToListMap: Map[String, List[String]] = list.groupBy(data=>data)
        // ("河北-帽子", count)
        val dataToCountMap: Map[String, Int] = dataToListMap.mapValues(_.size)

        // TODO 将分组聚合后的数据进行结构的转换
        // 如果改变数据结构时，可能会导致key重复，那么不要使用map结构
        // ("河北-帽子", count) => ( 河北, (帽子，count) )
        // ("河北-衣服", count) => ( 河北, (衣服，count) )
        val prvToItemAndCountList: List[(String, (String, Int))] = dataToCountMap.toList.map(
            kv => {
                val k = kv._1
                val count = kv._2
                // 将key进行拆分
                val ks = k.split("-")
                (ks(0), (ks(1), count))
            }
        )

        // TODO 将分组聚合后的数据根据省份进行分组
        // ( 河北, List[(帽子，count),(鞋，count),(衣服，count) ])
        val groupMap: Map[String, List[(String, (String, Int))]] = prvToItemAndCountList.groupBy(_._1)

        // TODO 将分组后的数据进行排序：降序
        val result: Map[String, List[(String, Int)]] = groupMap.mapValues(list => {
            val itemToCountList: List[(String, Int)] = list.map(_._2)
            itemToCountList.sortWith(
                (left, right) => {
                    left._2 > right._2
                }
            )
        })

        println( result )
        
        
        
        
val a = List(
      ("zhangsan", "河北", "鞋"), ("lisi", "河北", "衣服"), ("wangwu", "河北", "鞋"),
      ("zhangsan", "河南", "鞋"), ("lisi", "河南", "衣服"), ("wangwu", "河南", "鞋"),
      ("zhangsan", "河南", "鞋"), ("lisi", "河北", "衣服"), ("wangwu", "河北", "鞋"),
      ("zhangsan", "河北", "鞋"), ("lisi", "河北", "衣服"), ("wangwu", "河北", "帽子"),
      ("zhangsan", "河南", "鞋"), ("lisi", "河南", "衣服"), ("wangwu", "河南", "帽子"),
      ("zhangsan", "河南", "鞋"), ("lisi", "河北", "衣服"), ("wangwu", "河北", "帽子"),
      ("lisi", "河北", "衣服"), ("wangwu", "河北", "电脑"), ("zhangsan", "河南", "鞋"),
      ("lisi", "河南", "衣服"), ("wangwu", "河南", "电脑"), ("zhangsan", "河南", "电脑"),
      ("lisi", "河北", "衣服"), ("wangwu", "河北", "帽子"))
    /*
    需求数据河南->((鞋，6),(衣服,5),(帽子,2)),河北->
     */
    val b = a.map(s =>(s._2,s._3))
    println(b)
    val c = b.groupBy(s => s._1)
    println(c)
    val d = c.map(s=> (s._1,s._2.map(h => h._2)))
    println(d)
    val e = d.map(s => (s._1,s._2.groupBy(h =>h)))
    println(e)
    val f = e.map(s => (s._1,s._2.map(h=>(h._1,h._2.length)).toList.sortBy(j=>j._2).reverse))
    println(f)//Map(河南 -> List((鞋,6), (衣服,3), (电脑,2), (帽子,1)), 河北 -> List((衣服,6), (鞋,4), (帽子,3), (电脑,1)))
```

### 19、scala快速使用

```
1、安装jdk1.8
2、解压Scala2.12
3、window中配置环境变量
4、idea中在plugins安装Scala插件
5、新建maven工程
6、添加Scala支持
```

### 20、变量

```
1、可变：var name : String = "张玉博"
2、不可变：val name : String = "chen"
3、必须显示初始化
4、不可变变量类似于final，不可重新被赋值
5、标识符可以使用字母，数字，下划线，$符，符号等，不能以数字开头，不能是关键字或保留字
```

### 21、字符串

```
1、传值字符串（按照指定格式输出字符串变量值，使用如下）

    var name = "zhangyubo"
    printf("name=%s", name)//name=zhangyubo
    
2、插值字符串
var name = "afe"
println(s"name=$name")//name=afe

3、多行字符串
    var name = "afe"
    println(
      s"""
         |name=$name
         |age=19
         |""".stripMargin)
输出格式为：
name=afe
age=19
```

### 22、数据类型

```
any
	anyval
		unit和各种基本数据对象
	anyref
		各种类，集合，数组，方法对象
			null
nothing是所有类的子类，特指抛出异常

自动类型转换：Byte转Short转Int转Long转Float转Double、Char转Int
强转：toInt等方法

```

### 23、运算符

```
1、算数，关系，赋值，逻辑，位等运算符
2、运算符本质是方法
```

### 24、流程控制

#### if else

```
        val b = true
        if ( b ) {
            println("true")
} else {
    println("false")
}

```

#### for

普通的循环

```
    for(i <- Range(0,5)){print(i)}
    print("\n")
    for(i <- 0 to 5){print(i)}
    print("\n")
    for(i <- 0 until 5)(print(i))

返回值：
val result = for(i <- Range(0,5)){i*2};print(result)//unit
val result2 = for(i <- Range(1,5)) yield {i*2};print(result2)//Vector(2, 4, 6, 8)

输出：
01234
012345
01234
```

循环守卫

```
for(i <- Range(0,5) if i != 3){print(i)}//0124
```

循环步长

```
for(i <- Range(0,5,2)){print(i)}//024
for(i <- 0 to 5 by 2){print(i)}//024
```

嵌套循环

```
    for(i <- Range(0,5,2)){
      for(j <- Range(0,3)){print(i + j)}
    }//012234456
    
    for(i <- Range(0,5);j <- Range(0,5)){print(i + j)}//0123412345234563456745678
```

#### while do

```
var i = 5
while(i > 0){print(i);i += -1}//54321

```

#### do while

```
var i = 5
do{print(i);i += -1}while(i > 0)//54321
```

#### 循环中断

```
中断：
使用Breaks.breakable{语句体 + 终止条件{Breaks.break}}的方式
例如：
Breaks.breakable{
  for(i <- Range(0,5)){if(i == 3){Breaks.break};print(i)}
}
```

### 25、函数

#### 定义:

```

def main(args: Array[String]): Unit = {
    fun1()
    fun2("chen")
    fun3("zhang")
    fun4("zhang",10)
    fun5("chen",23)
    fun6("chen","zhang")
    fun7("yang")
  }
  def fun1() ={println("a")}
  def fun2(name : String) = {println(name)}
  def fun3(name : String) : String = {println(name);name}
  def fun4(name : String,age : Int) = {println(s"name=$name,age=$age")}
  def fun5(name : String,age : Int) : String = {println(s"name=$name,age=$age");name + age}
  def fun6(s : String*){print(s)}
  def fun7(name : String,age : Int = 19) = {println(s"name=$name,age=$age")}
  
  输出：
  a
chen
zhang
name=zhang,age=10
name=chen,age=23
WrappedArray(chen, zhang)
name=yang,age=19
```

#### 函数至简原则

```
return，花括号，返回值类型，等号，参数列表
匿名函数val b = ()=>{println("zhang")}
    b//代表函数本身，不会执行函数体
    b()//使用函数，会 执行函数体
    
    输出：
    zhang
```

#### 函数的值及函数本身

```
    def fun(){println("chen")}
    fun//会执行
    fun _//函数本身
    val b = fun//会执行
    val c = fun _//将函数本身赋值注意空格下划线

    println(b)
    println(c)
    
    输出：
    chen
	chen
	()
	com.atguigu.scala.STest$$$Lambda$6/1556595366@b97c004
```

#### 函数作为参数

```
def fun(f : Int=>Int){f(3)}

fun((i : Int) => {println(i)})

输出：3
函数作为参数一般声明为：f : Int=>Int
					f : ()=>String
					冒号前是参数名，冒号后第一个是函数参数，后一个是返回值
					
```

#### 闭包

```
函数嵌套
内部函数对外部函数作用域里对象的引用
外部函数返回内部函数对象
```

#### 柯里化函数

```
def fun6(i:Int)(j:Int) = {i * j}
```

#### 控制抽象

```
    def fun(f : => Any){f}

    fun(
      if(true){println("chen")}
    )
说明，即可以将一段代码作为参数传入fun函数
```

#### 函数递归

```
求n！

递归函数如下：
def fun(n : Int) : Int = {
	if(i = 1){1}
	else{n*fun(n - 1)}
}
```

#### 惰性函数

```
lazy val a = fun9()

将函数赋值给变量时不会执行，只有首次取值时才执行
```

### 26、面向对象

包：

```

1、Scala中包和类的路径没有关系
2、Scala中package可以嵌套使用
3、子包可以直接访问父包的内容而不需要import
4、package也可以当做对象，声明属性和方法，供所有子包使用
5、Scala中导包可以在任意地方使用import导入
6、Scala中导包使用下划线代替java中的*号导入包下所有
7、Scala中同一行可以多次导入如import java.util.{List,ArrayList}
8、Scala中可以在导入时屏蔽包中的某个类如import java.util.{Date=>_,Array=>_,_}(导入util包下除了Date类和Array类的其他所有类)
9、Scala中导入时可以起别名如import java.util.{ArrayList=>AL}
10、Scala中默认导入的包有：
import java.lang._
import scala._
import scala.Predef._
```

类：

```
伴生类和伴生对象

```

## 二、扩展

### 1、插值字符串

```
object ScalaString {
    def main(args: Array[String]): Unit = {
        // 插值字符串
        // 将变量值插入到字符串
        println(s"name=$name")
    }
}
```

### 2、多行字符串

```
//封装隔行的字符串，命令等
val name = "zhangyubo"
    val margin =
      s"""
         |select * from table1
         |where name = ${name};
    """.stripMargin
    println(margin)
```

### 3、输入输出

```
从键盘：val a = StdIn.readLine()
    println(a)
从文件：val a = Source.fromFile("").getLines()
    println(a)
输出同java
```

### 4、网络

```
客户端：val socket = new Socket("localhost",9999)
服务端：val serverSocket = new ServerSocket(9999)
       val socket = serverSocket.accept()
```

### 5、数据类型

```
Any代表所有
AnyVal代表基本数据类型
AnyRef代表对象类型
```

### 6、== 和eq

```
scala中==代表equals的意思
eq是比较地址值
```

### 7、强大的for循环

```
for(i <- Range(1,10,2);j = i * i){
      println(j)
    }
```

### 8、函数参数有默认值

```
def fun(name : String = "chen",password : String): Unit ={
      println(name + "," + password)
    }
    fun(password = "0000000")
```

### 9、匿名函数

```
val a = () =>{println("我是匿名函数")}
a()
```

### 10、函数作为参数

```
def fun(f : Int => Int) = {println("我需要一个函数作为参数" + f(5))}
    val b = fun(i => i * i)
    b
```

### 11、包嵌套，包对象

```
1、包嵌套的时候，子包可以访问父包的内容，不用import，在java中不行，
2、包对象中可以声明属性，方法，包下面所有都可以访问
3、使用import java.util._的方式导入包中所有内容，java中使用的是*
4、导入的时候可以写一行import java.util.{List,ArrayList}
5、导入可以屏蔽些类import java.sql.{Date=>_,Array=>_,_}
```

### 12、万能的下划线_

```
1、导包时代表所有
2、导包的时候可以屏蔽
3、可以将函数作为函数本身，而不执行
4、属性默认初始化
5、函数之后一个参数时，还可以代表参数
6、标识符
```

### 13、构造方法

```
class User(){
  var name: String = _
  def this(name : String) = {
    this()
    this.name = name
  }
}
```

### 14、使用apply构建对象

```
object User{
  def apply(): User = new User()
}
class User(){}
```

### 15、特质

```
初始化从左往右，继承优先，功能实现从右往左
```

### 16、枚举类和应用类

```
object Color extends Enumeration{val RED = Value(1,“red”)}
object User extends App{print("我可以直接运行")}
```

### 17、泛型的协变与逆变

```
1、代码：
//大师
class Master

//专家
class Professor extends Master

//讲师
class Teacher04 extends Professor

//这个是协变，Professor是Master的子类，此时Card[Professor]也会是Card[Master]的子类
class Card[+T]

class CovariantDemo {
  def enterMeet(card: Card[Master]): Unit = {
    println()
  }
}

  def main(args: Array[String]): Unit = {
    val masterCard=new Card[Master]
    val professorCard=new Card[Professor]
    val teacherCard=new Card[Teacher04]

    val demo = new CovariantDemo
//当参数是Card[Master]时 ，并且class Card[+T]
    demo.enterMeet(master)
    demo.enterMeet(professor)
    demo.enterMeet(teacher)
//当参数是Card[Teacher04]时,并且class Card[-T]

  }
```

18、