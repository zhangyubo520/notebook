# Shell

### 一、概念

#### 1、Shell解析器

```
查看：cat /etc/shells
1、/bin/sh 底层还是bash
2、/bin/bash  默认的解析器
```

#### 2、Shell运行方式

```
方式1、bash 文件绝对路径或者相对路径

方式2、sh 文件绝对路径或者相对路径

方式3、提升权限chmod 744 文件，之后使用“绝对路径”执行脚本

方式4、提升权限chmod 744 文件，之后使用“. 文件名”执行脚本

方式5、提升权限chmod 744 文件，之后使用“./文件名”执行脚本
```

#### 3、Shell变量

```
常用的几个：$HOME、$PWD、$SHELL、$PATH
查看所有的变量：set
自定义变量A=5
撤销变量unset A
声明静态变量readonly A

说明：
1、变量名由数字、字母、下划线组成，不能以数字开头，环境变量建议大写
2、等号两边不能有空格
3、在bash中，默认都是字符串，不能直接运算，通过$取得值后进行计算
4、变量的值有空格需要使用“”或者‘’
5、使用export 变量名可以将变量提升至全局变量供其他shell使用
```

#### 4、特殊变量

```
$n:$0代表该脚本的名称，$1-$9代表1-9个参数，10以上的参数为${10}

$#：获取输入参数的个数

$*：代表全部参数，看成一个整体

$@：代表全部参数，每个区别对待

$?:返回上一个命令执行的返回值，若为0执行正确，若为其他，执行错误
```

#### 5、加减乘除

```
echo $[3 +4]:没有空格要求

echo $((4 *4)):没有空格要求

expr `expr 2 + 3` \* 4 :有空格要求

说明：用飘号保存一次的计算结果，也可以用echo $[(2+3) * 4]没有太多要求
```

#### 6、条件判断

```
-lt 小于（less than）			-le 小于等于（less equal）
-eq 等于（equal）				-gt 大于（greater than）
-ge 大于等于（greater equal）	   -ne 不等于（Not equal）
-r 有读的权限（read）			-w 有写的权限（write）
-x 有执行的权限（execute）
-f 文件存在并且是一个常规的文件（file）
-e 文件存在（existence）		-d 文件存在并是一个目录（directory）

说明：[ condition ]（注意condition前后要有空格），条件非空即为true，[ atguigu ]返回true，[] 返回false。
```

#### 7、if判断

```
if  [ 条件判断式 ];then 
执行语句
fi

或者

if  [ 条件判断式 ] 
then 
执行语句
elif [ 条件判断 ]
then
执行语句
else
执行语句
fi

说明：[ 条件判断式 ]，中括号和条件判断式之间必须有空格，if后要有空格，以fi结束，if或者elif后末尾是then

例子：

```

#### 8、case判断

```
基本语法：
case $变量名 in 
  "值1"） 
    如果变量的值等于值1，则执行程序1 
    ;; 
  "值2"） 
    如果变量的值等于值2，则执行程序2 
    ;; 
  …省略其他分支… 
  *） 
    如果变量的值都不是以上的值，则执行此程序 
    ;; 
esac

说明：
1)case行尾必须为单词“in”，每一个模式匹配必须以右括号“）”结束。
2)双分号“;;”表示命令序列结束，相当于java中的break。
3)最后的“*）”表示默认模式，相当于java中的default。
```

#### 9、for循环

```
for ((初始值;循环控制条件;变量变化)) 
do 
执行语句 
done

或者
for 变量名 in 值1 值2 值3
do
执行语句
done

说明：for后面有空格,以done结束
```

#### 10、while

```
while  [ 条件判断式 ] 
do 
执行语句
done

说明：while后面有空格，以done结束
```

#### 11、read

读取控制台的输入，等待时间10秒，提示为”请输入：“，将读取内容写入文件test.txt

```
#!/bin/bash
read -t 10 -p "请输入：" massager
echo $massager
touch /root/shell/test.txt
echo "$massager" >> test.txt
```

#### 12、basename和dirname

```
basename:找到最后一个/输出之后的部分，常用来获取文件名
dirname:找到最后一个/输出之前的部分，常用来获取文件的根目录
```

#### 13、自定义函数

```
function 函数名()
{函数体}

说明：
1、可以有返回值，函数返回值，只能通过$?系统变量获得，可以显示加：return 返回值，如果不加，将以最后一条命令运行结果，作为返回值
2、先声明后调用才有效
```

#### 15、cut

```
cut 文件名
说明：
1、使用-f,指明列号，列好从1开始
2、使用-d,指明分割符是什么，默认是制表符，如果既没有制表符又没有指明分隔符，那仅有一列
3、多个列可以表示为-f 1,2,3
4、之后多个列可以表示为-f 2-(2列后多个列，包括第二列)
5、文件内容没有变
```

#### 16、sed

```
sed '执行的正则表达式' 文件名
说明：
1、-e，'多个执行正则表达式'时在表达式前使用，例如：sed -e '1a mei mei' -e '2d' test.txt
2、命令a，在下一行新增a后面的内容，如sed '1a mei mei' test.txt
3、命令b，删除，如sed '/wo/d' sed.txt，全局删
4、命令s，替换，默认只替换第一个，全部替换末尾加上g,如：sed 's/wo/ni/g' sed.txt
```

#### 17、awk

```
awk [选项参数] ‘pattern1{action1}  pattern2{action2}...’ filename
pattern：表示AWK在数据中查找的内容，就是匹配模式
action：在找到匹配内容时所执行的一系列命令

参数说明：
-F：指定分割符
-v:赋值一个用户变量

注意：
1、只有匹配了pattern的行才会执行action
2、BEGIN 在所有数据读取行之前执行；END 在所有数据执行之后执行，如：awk -v i=1 -F: '{print $3+i}' test.txt
```

#### 18、sort

```
sort 文件名
参数说明：
1、-n,按数值大小，默认升序
2、-r,按降序排序
3、-k,指定需要排序的列
4、-t,指定分割符
```

#### 19、wc

```
wc 文件名
参数说明：
1、-l,统计文件行数
2、-m,统计字符数
3、-c,统计字节数
4、-w,统计单词数
```

### 二、代码

#### 1、脚本输出Hello World！

```
1、创建shell文件:
mkdir /root/shell
touch /root/shell/HelloWorld.sh

2、编辑shell文件
vim /root/shell/HelloWorld.sh
编辑如下内容：
#！/bin/bash
echo "Hello World!"

3、运行shell文件
sh HelloWorld.sh
```

#### 2、计算输入10个参数的和

```
#!/bin/bash
echo $[$1 + $2 + $3 + $4 + $5 + $6 + $7 + $8 + $9 + ${10}]
```

#### 3、if条件判断

如果输入的参数是1个就输出陈思思，若为2个就输出杨静静，否则就输出小甜甜

```
count=$#
if [ $count  -eq 1 ];then
echo "陈思思"
elif [ $count -eq 2 ];then
echo "杨静静"
else
echo "小甜甜"
fi
```

#### 9、case条件判断

仅能输入一个参数，若为1，输出陈思思，若为2，输出杨静静，其他值，输出小甜甜

```
#!/bin/bash
if [ $# -lt 1 ];then
echo "没有参数"
fi
case $1 in
"1")
echo "陈思思"
;;
"2")
echo "杨静静"
;;
*)
echo "小甜甜"
;;
esac
```

#### 10、计算1到100连加和

```
#!/bin/bash
sum=0
for ((i=1;i<=100;i++))
do
sum=$[$i+$sum]
done
echo $sum
```

#### 11、获取一个数字

文件内容是：

1 2 3
4 5 6
7 8 9

获取数字5打印至控制台

```

```

#### 12、面试1

使用Linux命令查询file1中空行所在的行号

```
awk '/^$/{print NR}' file1
```

#### 13、面试2

张三 40

李四 50

王五 60

计算第二列的和并输出

```
cat chengji.txt | awk -F " " '{sum+=$2} END{print sum}'
```

#### 14、面试3

Shell脚本里如何检查一个文件是否存在？如果不存在该如何处理？

```
#!/bin/bash

if [ -f file.txt ]; then
   echo "文件存在!"
else
   echo "文件不存在!"
fi
```

#### 15、面试4

用shell写一个脚本，对文本中无序的一列数字排序

```
sort -n test.txt
```

#### 15、面试5

请用shell脚本写出查找当前文件夹（/home）下所有的文本文件内容中包含有字符”shen”的文件名称

```
grep -r "shen" /home | cut -d ":" -f 1
```

# shell脚本攻略

### 1、echo 和printf

#### echo

```
echo hello world
echo "hello world"
echo 'hello world'
echo "hello world！"
echo 'hello world！'

说明：
1、三种方式输出的都是 hello world
2、两个命令之间用分号隔开，所以第一种方式中不能出现分号
3、双引号会解释特殊字符，所以要想输出 ！等特殊字符需要加 \ 进行转义
4、单引号不会解释特殊字符，所以可以正常输出特殊字符
5、默认情况下，echo会在输出文本的尾部追加一个换行符。可以使用选项-n来禁止这种行为
6、在使用转义序列时，需要使用echo -e "包含转义序列的字符串"这种形式
```

#### printf

```
printf hello world
printf "hello world"
printf 'hello world'
printf "\n" hello world
printf  "%-5s %-10s %-4.2f\n" 1 Sarath 80.3456

说明：
1、第一个输出的是hello,后两个输出的是hello world
2、printf命令接受引用文本或由空格分隔的参数。
3、%s、%c、%d和%f都是格式替换符（format substitution character），它们定义了该如何打印后 续参数。%-5s指明了一个格式为左对齐且宽度为5的字符串替换（-表示左对齐）。如果不指明-， 字符串就采用右对齐形式。宽度指定了保留给某个字符串的字符数量。对Name而言，其保留宽 度是10。因此，任何Name字段的内容都会被显示在10字符宽的保留区域内，如果内容不足10个 字符，余下的则以空格填充。 对于浮点数，可以使用其他参数对小数部分进行舍入（round off）。 
对于Mark字段，我们将其格式化为%-4.2f，其中.2指定保留两位小数。注意，在每行的格 式字符串后都有一个换行符（\n）
```

#### 彩色输出

文本

```
文本颜色是由对应的色彩码来描述的。其中包括：重置=0，黑色=30，红色=31，绿色=32， 黄色=33，蓝色=34，洋红=35，青色=36，白色=37。 
要打印彩色文本，可输入如下命令： 
echo -e "\e[1;31m This is red text \e[0m" 
其中\e[1;31m是一个转义字符串，可以将颜色设为红色，\e[0m将颜色重新置回。只需要将31替 换成想要的色彩码就可以了。 
```

文本背景

```
对于彩色背景，经常使用的颜色码是：重置=0，黑色=40，红色=41，绿色=42，黄色=43， 蓝色=44，洋红=45，青色=46，白色=47。  
要设置彩色背景的话，可输入如下命令： 
echo -e "\e[1;42m Green Background \e[0m" 
```

转义序列介绍

```
man console_codes
```

### 2、变量

查看

```
查看环境变量：env或printenv
查看某个进程的：cat /proc/$PID/environ 
获取进程id: pgrep
```

赋值

```
1、varname=value
varName是变量名，value是赋给变量的值。如果value不包含任何空白字符（例如空格）， 那么就不需要将其放入引号中，否则必须使用单引号或双引号。 

2、注意，var = value不同于var=value。把var=value写成var = value 是一个常见的错误。两边没有空格的等号是赋值操作符，加上空格的等号表示的 是等量关系测试。 在变量名之前加上美元符号$就可以访问变量的值
$varname或者${varname}

3、变量长度：${#varname}
```

### 3、数学运算

Bash shell使用let、(( ))和[]执行基本的算术操作。工具expr和bc可以用来执行高级操作。

let

```
测试：var=1;echo $var;let var++;echo $var 输出1和2
解释：
1、let var++;自增1
2、let var--;自减1
3、let var+=6;自增6
```

[  ]

```
测试：var=1;echo $var;result=$[ $var + 1 ];echo $result 输出1和2
```

((  ))

```
测试：var=1;echo $var;result=$(( $var + 1 ));echo $result 输出1和2
```

expr

```
测试：var=1;echo $var;result=$(expr $var + 1);echo $result 输出1和2
```

前面这些都是整数运算

bc

```
测试：echo "4 * 0.56" | bc 
设定精度：echo "scale=2;22/7" | bc
进制转换：echo "obase=2;2" | bc 
```

4、重定向

">"

```
将内容输入到文件，会清空文件之前的内容
```

">>"

```
将内容输入到文件，不会清空，只会追加
```

说明

```
 0 —— stdin （标准输入）
 1 —— stdout（标准输出）
 2 —— stderr（标准错误）

1、将stdout和stderr输入到同一个文件中的命令：2>&1 exmple.txt
2、>等同于1>；对于>>来说，情况也类似（即>>等同于1>>）
3、处理错误时，来自stderr的输出被倾倒入文件/dev/null中。./dev/null是一个特殊的设备文件， 它会丢弃接收到的任何数据。null设备通常也被称为黑洞，因为凡是进入其中的数据都将一去不返。
```

