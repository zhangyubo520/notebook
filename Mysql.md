# Mysql

### 一、概念

#### 问题1、sql分类

答：分三种，分别为DDL,数据定义语言；DML，数据管理语言；DCL,数据控制语言；

1、DDL:

CREATE,创建数据库DATABASE;创建TABLE等；

ALTER,更新表中的字段；

DROP,删除某一列，通过列名，删除索引；

2、DML:

select 查询表中数据；

update 更改表中数据；

delete 删除表中数据；

insert 添加数据；

3、DCL:

了解，用于控制语句执行：GRANT`、`REVOKE`、`COMMIT`、`ROLLBACK`、`SAVEPOINT

#### 问题2、空值运算

答：空值参与运算后得到的是空值

### 二、代码

#### 1、查询

```
select distinct 列名1，列名2，单行函数1 别名1
from 表名
where 查询条件（>,<,>=,<=,=,!=,<>,between a and b,in(a,b,c,d),is null,is not null,like，&&，and,or,||,not,xor,+,-,*,/,div,%,mod);
说明：
1、别名使用双引号，
2、distinct用于筛选重复数据，
3、where 用于定义查询条件，
4、区分符号时使用转义字符\,或者自定义转义字符escape ‘$‘
```

#### 2、显示表结构

```
desc 表名
```

#### 3、排序

```
select 信息
from 表名
order by 列名1，列名2 asc or desc;
说明：
1、比较的是某一列的值大小，String，date，数值型均可以比大小，
2、默认升序asc,可以自定义desc为降序，
3、排序操作通常在语句的末尾，将前面处理好的数据排序显示，
4、可以使用多个列排序，
5、要显示的信息，和排序根据可以不一样，即可以使用不在select中列的信息排序
```

#### 4、分页显示

```
select * from table limit 0,10；
说明：
1、已知当前页数的情况下，显示范围：当前页数减一乘以公差，公差
2、放在order by 的语句之后
```

#### 5、多表查询

```
select *
from table1,table2
where 连接条件
说明：
1、连接n个表至少需要n-1个连接条件
2、where 一般用外键等于内键的方式
```

#### 6、内外连接

```
select *
from table1
left join table2
on 连接条件1
left join table3
on 连接条件2
说明：
1、内连接不包含不对应的值，外连接包含不对应的值（用null)填充
```

#### 7、求内外连接后的并集

```
SELECT * 
FROM t_emp A LEFT JOIN t_dept B 
ON A.deptId = B.id
UNION
SELECT * 
FROM t_emp A RIGHT JOIN t_dept B 
ON A.deptId = B.id
说明：
1、union用在两个查询到的集合之间，求并集，包含去重操作；
2、union all 也是用在两个查询到的集合之间，求并集，不包含去重操作，效率高，但是有重复的交集数据；
```

#### 8、单行函数

```
y = f(n1,n2,n3.....)
1、字符串函数（包含增删改查去空格等操作）
2、大小写控制（lower(),upper())
3、数值函数（包含绝对值随机数等操作）
4、日期函数（包含now()，curdate()，curtime()等操作）
5、流程函数（包含if等操作）
```

#### 9、流程函数

```
1、IF(value,t ,f)如果value是真，返回t，否则返回f
2、IFNULL(value1, value2)如果value1不为空，返回value1，否则返回value2
3、CASE WHEN 条件1 THEN result1 WHEN 条件2 THEN result2 .... [ELSE resultn] END相当于Java的if...else if...else...
4、CASE  expr WHEN 常量值1 THEN 值1 WHEN 常量值1 THEN 值1 .... [ELSE 值n] END相当于Java的switch...case...
说明：
1、三元运算符问题用if(value,t,f),if-else问题用case when 条件 then 结果，switch-case问题用case 值 when 常量 then 结果
```

#### 10、分组函数

```
select avg(),min(),max(),sum(),count()
from table
group by 列名
说明：
1、使用count()计算的是非空的数据，要想知道所有数据条数，使用count(1)或者count(*)
```

#### 11、having和where

```
1、having用在分组函数中，表示满足分组的将被显示，效率低
2、where紧跟from做第一轮搜索，减小搜索范围，提高后续操作效率，一定要紧跟，
```

#### 12、查询通用结构模板

```
1、select a,b,c,d.....
from table1,table2....
where 多表连接条件
and 限制条件
group by 分组依据	
having 分组显示条件
order by 排序依据
limit 分页显示
*************************************************
2、select a,b,c,d.....
from table1
join table2
on 连接条件
where 限制条件1
and/or 限制条件2
group by 分组依据	
having 分组显示条件
order by 排序依据
limit 分页显示
```

#### 13、子查询

```
select*
from table1
where 条件一

其中条件一为：
列名 = > != in any all(
select*
from table2
where 条件二
)
。。。。。。
无限的找条件表达式的右边，左边通常是我们要根据什么显示题目的信息，比如我们要显示某一员工的信息，我们想通过id找，就可以把id，放左边，里面搜索满足条件的id
例如，求工资最低的员工信息
select *
from table
where id = (
            select id
            from table
            where salary <= all(
                                select salary
                                from table
                                )
            )
```

#### 14、相关子查询

```
说明：相当于循环嵌套，主查询中的每一条都当做子查询的条件去执行子查询，满足就返回该条数据
```

##### 题目1：排序

```
查询员工的id,salary,按照department_name 排序
select id,salary
from table1 t1
order by (
            select department_name
            from table2 t2
            where t1.id = t2.id
            )
```

##### 题目2：更新

```
使用相关子查询依据一个表中的数据更新另一个表的数据。
ALTER TABLE employees
ADD department_name VARCHAR(30)

UPDATE employees e
SET department_name = (
			SELECT department_name
			FROM departments d
			WHERE e.`department_id` = d.`department_id`
			)
```

#### 15、创建和管理表

```
1、创建数据库
create database 数据库名
说明：show databases,显示所有数据库

2、创建表
create table 表名（字段1 类型（大小），字段1 类型（大小），字段2 类型（大小），字段3 类型（大小）...）；
create table emp1 as select * from employees;
create table emp2 as select * from employees where 1=2; --创建的emp2是空表
说明：show tables from database名,显示某数据库中所有的表

3、删除数据库，表，列都可以用drop
drop database 库名
drop table 表名
drop column 列名

4、重命名表
rename table 原名
to 现名
或者是
alter table 原名
rename 现名

5、追加一个列
ALTER TABLE dept80 
ADD job_id varchar(15)

6、修改一个列
ALTER TABLE	dept80
MODIFY (last_name VARCHAR(30))
说明：可以修改列的数据类型, 尺寸和默认值

7、ALTER TABLE  dept80
CHANGE department_name dept_name varchar(15);
说明 使用change 原列名 新列名 类型（长度）

8、清空表
TRUNCATE TABLE detail_dept;不能回滚
delete from 表名;能回滚， 先设置set autocommit = false;然后使用rollback回滚到上一次提交之后
```

#### 16、操作数据

```
1、插入
insert into 表名(列名1，列名2....)
values(数值1，数值2.....),(数值1，数值2.....),(数值1，数值2.....)

2、拷贝
INSERT INTO sales_reps(id, name, salary, commission_pct)
SELECT employee_id, last_name, salary, commission_pct
FROM   employees
WHERE  job_id LIKE '%REP%';

3、更新
UPDATE 	表名
SET    	department_id = 110;

4、删除
delete from 表名
where 条件
```

#### 17、约束

```
1、NOT NULL，非空约束，规定某个字段不能为空
2、UNIQUE，唯一约束，规定某个字段在整个表中是唯一的
3、PRIMARY KEY，主键(非空且唯一)约束
4、FOREIGN KEY，外键约束
5、CHECK，检查约束
6、DEFAULT，默认值约束

说明：每个表只有一个主键，主键唯一且非空，多列组合为主键，这些列均非空，且唯一
外键约束：
CREATE TABLE dept(
dept_id INT AUTO_INCREMENT PRIMARY KEY,
dept_name VARCHAR(20)
);

CREATE TABLE emp(
emp_id INT AUTO_INCREMENT PRIMARY KEY,
last_name VARCHAR(15),
dept_id INT,
CONSTRAINT emp_dept_id_fk FOREIGN KEY(dept_id) 
REFERENCES dept(dept_id)
);
```

#### 18、MySQL utf8超过字节数

```
MySQL的utf8编码最多存储3个字节，当数据中存在表情号、特色符号时会占用超过3个字节数的字节，那么会出现错误 Incorrect string value: '\xF0\x9F\x91\x91\xE5\xB0...'
解决办法：将utf8修改为utf8mb4

1、首先修改库的基字符集和数据库排序规则

2、再使用 SHOW VARIABLES LIKE '%char%'; 命令查看参数

3、确保这几个参数的value值为utf8mb4 如果不是使用set命令修改
如：set character_set_server = utf8mb4;
```

#### 19、mysql的隔离级别

```
1、Read Uncommitted(读取未提交内容)
所有事务都可以查看到其他未提交事务的结果，读取未提交数据称为脏读

2、Read Committed(读取提交内容)
大多数会数据库默认的隔离级别（但不是MySQL默认的），一个事务只能看到已经提交的事务所做的改变，这种隔离级别也支持不可重复读

3、Repeatable Read(可重复读)
这是MySQL默认的隔离级别，确保同一事务的多个实例在并发读取数据时，会看到同样的数据行，但是会导致幻读

4、Serializable(串行化)
强制事务排序，使他们之间不可能相互冲突，解决了幻读问题

脏读：

不可重读：

幻读：
```

#### 20、查询优化

```
对查询进行优化，要尽量避免全表扫描，首先应考虑在 where 及 order by 涉及的列上建立索引。

1、应尽量避免在 where 子句中对字段进行 null 值判断，否则将导致引擎放弃使用索引而进行全表扫描

2、应尽量避免在 where 子句中使用 != 或 <> 操作符，否则引擎将放弃使用索引而进行全表扫描。

3、应尽量避免在 where 子句中使用 or 来连接条件，如果一个字段有索引，一个字段没有索引，将导致引擎放弃使用索引而进行全表扫描

4、in 和 not in 也要慎用，否则会导致全表扫描

5、like模糊全匹配也将导致全表扫描
```

#### 21、聚集索引和非聚集索引

```
聚集索引类似字典中的拼音索引a,b

非聚集索引类似字典中的偏旁索引

实际上，您可以把索引理解为一种特殊的目录。微软的SQL SERVER提供了两种索引：聚集索引（clustered index，也称聚类索引、簇集索引）和非聚集索引（nonclustered index，也称非聚类索引、非簇集索引）。下面，我们举例来说明一下聚集索引和非聚集索引的区别：

其实，我们的汉语字典的正文本身就是一个聚集索引。比如，我们要查“安”字，就会很自然地翻开字典的前几页，因为“安”的拼音是“an”，而按照拼音排序汉字的字典是以英文字母“a”开头并以“z”结尾的，那么“安”字就自然地排在字典的前部。如果您翻完了所有以“a”开头的部分仍然找不到这个字，那么就说明您的字典中没有这个字；同样的，如果查“张”字，那您也会将您的字典翻到最后部分，因为“张”的拼音是“zhang”。也就是说，字典的正文部分本身就是一个目录，您不需要再去查其他目录来找到您需要找的内容。我们把这种正文内容本身就是一种按照一定规则排列的目录称为“聚集索引”。

如果您认识某个字，您可以快速地从自动中查到这个字。但您也可能会遇到您不认识的字，不知道它的发音，这时候，您就不能按照刚才的方法找到您要查的字，而需要去根据“偏旁部首”查到您要找的字，然后根据这个字后的页码直接翻到某页来找到您要找的字。但您结合“部首目录”和“检字表”而查到的字的排序并不是真正的正文的排序方法，比如您查“张”字，我们可以看到在查部首之后的检字表中“张”的页码是672页，检字表中“张”的上面是“驰”字，但页码却是63页，“张”的下面是“弩”字，页面是390页。很显然，这些字并不是真正的分别位于“张”字的上下方，现在您看到的连续的“驰、张、弩”三字实际上就是他们在非聚集索引中的排序，是字典正文中的字在非聚集索引中的映射。我们可以通过这种方式来找到您所需要的字，但它需要两个过程，先找到目录中的结果，然后再翻到您所需要的页码。我们把这种目录纯粹是目录，正文纯粹是正文的排序方式称为“非聚集索引”。

通过以上例子，我们可以理解到什么是“聚集索引”和“非聚集索引”。进一步引申一下，我们可以很容易的理解：每个表只能有一个聚集索引，因为目录只能按照一种方法进行排序。

选择依据：
```

### 22、事务

```
ACID
A:原子性
C:一致性
I:隔离性
D:持久性
```

### 23、存储引擎

```
MYISAM\innoDB\MEMORY\ARCHIVE

MYISAM:B+树，表锁，插入查询块，不支持事务

innoDB:B+树，行锁，支持事务，内存消耗高，支持外键，支持自增列

memory:内存，速度快，可靠性低

archive：归档的历史数据，不支持索引，

```

