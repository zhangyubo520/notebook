# jvm

### 1、概念

```
jvm内存介绍：
方法区（Method Area）方法区主要是放一下类似类定义、常量、编译后的代码、静态变量等，在JDK1.7中，HotSpot VM的实现就是将其放在永久代中，这样的好处就是可以直接使用堆中的GC算法来进行管理，但坏处就是经常会出现内存溢出，即PermGen Space异常，所以在JDK1.8中，HotSpot VM取消了永久代，用元空间取而代之，元空间直接使用本地内存，理论上电脑有多少内存它就可以使用多少内存，所以不会再出现PermGen Space异常。

堆（Heap）几乎所有对象、数组等都是在此分配内存的，在JVM内存中占的比例也是极大的，也是GC垃圾回收的主要阵地，平时我们说的什么新生代、老年代、永久代也是指的这片区域，至于为什么要进行分代后面会解释。

虚拟机栈（Java Stack）当JVM在执行方法时，会在此区域中创建一个栈帧来存放方法的各种信息，比如返回值，局部变量表和各种对象引用等，方法开始执行前就先创建栈帧入栈，执行完后就出栈。

本地方法栈（Native Method Stack）和虚拟机栈类似，不过区别是专门提供给Native方法用的。

程序计数器（Program Counter Register）占用很小的一片区域，我们知道JVM执行代码是一行一行执行字节码，所以需要一个计数器来记录当前执行的行数。
```

### 2、参数

```
-Xms：设置初始分配大小，默认为物理内存的1/64
-Xmx：最大分配内存，默认为物理内存的1/4
-XX:+PrintGCDetails：输出详细的GC处理日志
-XX:+PrintGCTimeStamps：输出GC的时间戳信息
-XX:+PrintGCDateStamps：输出GC的时间戳信息（以日期的形式）
-XX:+PrintHeapAtGC：在GC进行处理的前后打印堆内存信息
-Xloggc:(SavePath)：设置日志信息保存文件
在堆内存的调整策略中，基本上只要调整两个参数：-Xms和-Xmx

-XX:NewSize新生代的最小值
-XX:MaxNewSize新生代的最大值
-XX:NewRatio设置新生代与老年代在堆空间的大小
-XX:SurvivorRatio新生代中Eden所占区域的大小

-XX:MaxPermSize 永久代的大小
-XX:MaxTenuringThreshold,设置将新生代对象转到老年代时需要经过多少次垃圾回收，但是仍然没有被回收

-XX:UseTLAB 默认开启，使用tlab,占用伊甸园区的1%
```

### 3、逃逸分析

```
对象的声明周期在方法内部时，就可以将对象分配在栈上

逃逸的情况：
1、新建的对象，是否在方法外被调用

能用局部变量的，就不要在方法外声明

优化：
栈上分配：没有逃逸就分配在栈上
同步省略：没有逃逸的对象锁会被消除
分离对象或者标量替换：没有逃逸的对象，可以将对象的属性换成标量
```

### 4、jvm的三个主要子系统

```
类加载子系统，运行时数据区，执行引擎

类加载子系统：加载、链接、初始化
			加载：启动类加载器、扩展类加载器、应用类加载器
			链接：校验、准备、解析
 	 			校验：验证字节码是否正确
 	 			准备：分配内存并赋默认值给所有静态变量
 	 			解析：符号引用转换为方法区的直接引用
			初始化：静态变量被赋初始值
			
运行时数据区：方法区、堆、运行时栈、PC、本地方法栈
			方法区：类级别数据，类、静态变量、静态方法、常量、成员方法，线程不安全，会有GC
			堆：对象和对应的实例变量，数组，线程不安全，会有GC
			运行时栈:线程私有，局部变量、对象引用、线程
			PC:当前指令的地址
			本地方法栈：本地方法信息，一个线程一个

执行引擎：解释器、编译器、垃圾回收器
收集并删除未引用的对象。可以通过调用"System.gc()"来触发垃圾回收，但并不保证会确实进行垃圾回收。JVM的垃圾回收只收集哪些由new关键字创建的对象。所以，如果不是用new创建的对象，你可以使用finalize函数来执行清理。

说明：另外的两个
Java本地接口 (JNI): JNI 会与本地方法库进行交互并提供执行引擎所需的本地库。
本地方法库:它是一个执行引擎所需的本地库的集合。
```

### 5、堆

```
新生代、老年代、元空间
新生代包含：伊甸园区、幸存者1区和幸存者2区
```

### 6、GC算法

```
标记清除算法：
内存中的对象构成一棵树，当有效的内存被耗尽的时候，程序就会停止，做两件事，第一：标记，标记从树根可达的对象（途中水红色），第二：清除（清楚不可达的对象）。标记清除的时候有停止程序运行，如果不停止，此时如果存在新产生的对象，这个对象是树根可达的，但是没有被标记（标记已经完成了），会清除掉。

缺点：递归效率低、释放空间连续、有碎片、会stw(停止程序运行)

复制算法：
把内存分成两块区域：空闲区域和活动区域，第一还是标记（标记谁是可达的对象），标记之后把可达的对象复制到空闲区，将空闲区变成活动区，同时把以前活动区对象1，4清除掉，变成空闲区。

缺点：速度快但是浪费空间

标记整理算法：
标记谁是活跃对象，整理，会把内存对象整理成一课树一个连续的空间，

jvm的策略：
新生代：采用复制算法
老年代： 标记／整理算法，GC后会执行压缩，整理到一个连续的空间，这样就维护着下一次分配对象的指针，下一次对象分配就可以采用碰撞指针技术，将新对象分配在第一个空闲的区域

增量算法：边GC边执行应用程序，
缺点：浪费线程切换和上下文转换的性能消耗

分区算法：分区进行垃圾回收

分代收集算法
分代收集算法就是目前虚拟机使用的回收算法，它解决了标记整理不适用于老年代的问题，将内存分为各个年代。一般情况下将堆区划分为老年代（Tenured Generation）和新生代（Young Generation），在堆区之外还有一个代就是永久代（Permanet Generation）。
在不同年代使用不同的算法，从而使用最合适的算法，新生代存活率低，可以使用复制算法。而老年代对象存活率搞，没有额外空间对它进行分配担保，所以只能使用标记清除或者标记整理算法。
```

### 7、引用

```
强引用：new出的对象之类的引用，只要强引用还在，永远不会回收

软引用：引用但非必须的对象，内存溢出异常之前，回收

弱引用：非必须的对象，对象能生存到下一次垃圾收集发生之前。

虚引用：对生存时间无影响，在垃圾回收时得到通知。
```

### 8、垃圾收集器

```
     1.Serial GC 。从名字上看，串行GC意味着是一种单线程的，所以它要求收集的时候所有的线程暂停。这对于高性能的应用是不合理的，所以串行GC一般用于Client模式的JVM中。

     2.ParNew GC 。是在SerialGC的基础上，增加了多线程机制。但是如果机器是单CPU的，这种收集器是比SerialGC效率低的。

     3.Parrallel Scavenge GC 。这种收集器又叫吞吐量优先收集器，而吞吐量=程序运行时间/(JVM执行回收的时间+程序运行时间),假设程序运行了100分钟，JVM的垃圾回收占用 1分钟，那么吞吐量就是99%。Parallel Scavenge GC由于可以提供比较不错的吞吐量，所以被作为了server模式JVM的默认配置。

     4.ParallelOld 是老生代并行收集器的一种，使用了标记整理算法，是JDK1.6中引进的，在之前老生代 只能使用串行回收收集器。

     5.Serial Old 是老生代client模式下的默认收集器，单线程执行，同时也作为CMS收集器失败后的备用收集器。

     6.CMS 又称响应时间优先回收器，使用标记清除算法。他的回收线程数为(CPU核心数+3)/4，所以当CPU核心数为2时比较高效些。CMS分为4个过程：初始标记、并发标记、重新标记、并发清除。

     7.GarbageFirst（G1） 。比较特殊的是G1回收器既可以回收Young Generation，也可以回收Tenured Generation。它是在JDK6的某个版本中才引入的，性能比较高，同时注意了吞吐量和响应时间。
         
     Young代回收算法：
因为Young代都是短命对象，一次YoungGC会回收大部分对象，只有少量会剩下来。ParNew采用多线程，Stop The World的方式，采用复制算法进行回收。


特点是：

Stop The Workd
多线程并发回收
复制算法，内存回收后方便分配
Old代采用CMS回收算法：

 回收步骤：

初始化清除：Stop The World 标记由根可以直达的对象
并发标记：并发标记可达对象
重新标记：Stop The World并发查找前一阶段新生代晋升或者新分配，被更新的对象
并发清理
线程重置
优点：

低延迟
缺点：

因为是基于标记清除，容易产生碎片 
```

### 9、fullGC触发条件

```
1.Tenured Space空间不足以创建打的对象或者数组，会执行FullGC，并且当FullGC之后空间如果还不够，那么会OOM:java heap space。

     2.Permanet Generation的大小不足，存放了太多的类信息，在非CMS情况下回触发FullGC。如果之后空间还不够，会OOM:PermGen space。

     3.CMS GC时出现promotion failed和concurrent mode failure时，也会触发FullGC。promotion failed是在进行Minor GC时，survivor space放不下、对象只能放入旧生代，而此时旧生代也放不下造成的；concurrent mode failure是在执行CMS GC的过程中同时有对象要放入旧生代，而此时旧生代空间不足造成的。

     4.判断MinorGC后，要晋升到TenuredSpace的对象大小大于TenuredSpace的大小，也会触发FullGC。

     可以看出，当FullGC频繁发生时，一定是内存出问题了。
```

### 10、虚拟机栈

```
1.Java虚拟机栈是线程私有的，它的生命周期与线程相同。

每一个方法被调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中从入栈到出栈的过程。
虚拟机栈是执行Java方法的内存模型(也就是字节码)服务：每个方法在执行的同时都会创建一个栈帧，用于存储 局部变量表、操作数栈、动态链接、方法出口等信息。

局部变量表：32位变量槽，存放了编译期可知的各种基本数据类型、对象引用、returnAddress类型。

操作数栈：基于栈的执行引擎，虚拟机把操作数栈作为它的工作区，大多数指令都要从这里弹出数据、执行运算，然后把结果压回操作数栈。

动态连接：每个栈帧都包含一个指向运行时常量池（方法区的一部分）中该栈帧所属方法的引用。持有这个引用是为了支持方法调用过程中的动态连接。Class文件的常量池中有大量的符号引用，字节码中的方法调用指令就以常量池中指向方法的符号引用为参数。这些符号引用一部分会在类加载阶段或第一次使用的时候转化为直接引用，这种转化称为静态解析。另一部分将在每一次的运行期间转化为直接应用，这部分称为动态连接

方法出口：返回方法被调用的位置，恢复上层方法的局部变量和操作数栈，如果无返回值，则把它压入调用者的操作数栈。
局部变量表所需的内存空间在编译期间完成分配，当进入一个方法时，这个方法需要在帧中分配多大的局部变量空间是完全确定的。

在方法运行期间不会改变局部变量表的大小。主要存放了编译期可知的各种基本数据类型、对象引用 （reference类型）、returnAddress类型）。

java虚拟机栈,规定了两种异常状况：

如果线程请求的深度大于虚拟机所允许的深度，将抛出StackOverflowError异常。
如果虚拟机栈动态扩展，而扩展时无法申请到足够的内存，就会抛出OutOfMemoryError异常。
```

### 11、本地方法栈

```
可通过参数 栈容量可由-Xss设置

虚拟机栈为虚拟机执行Java方法（也就是字节码）服务。
本地方法栈则是为虚拟机使用到的Native方法服务。有的虚拟机（譬如Sun HotSpot虚拟机）直接就把本地方法栈和虚拟机栈合二为一
```

### 12、方法区

```
可通过参数-XX:MaxPermSize设置

线程共享内存区域，用于储存已被虚拟机加载的类信息、常量、静态变量，即编译器编译后的代码，方法区也称持久代（Permanent Generation）。

虽然Java虚拟机规范把方法区描述为堆的一个逻辑部分，但是它却有一个别名叫做Non-Heap（非堆），目的应该是与Java堆区分开来。

如何实现方法区，属于虚拟机的实现细节，不受虚拟机规范约束。

方法区主要存放java类定义信息，与垃圾回收关系不大，方法区可以选择不实现垃圾回收,但不是没有垃圾回收。

方法区域的内存回收目标主要是针对常量池的回收和对类型的卸载。

运行时常量池，也是方法区的一部分，虚拟机加载Class后把常量池中的数据放入运行时常量池。

 ①方法区存储的是每个class的信息:
（1）类加载器引用(ClassLoader)
（2）运行时常量池：包含所有常量、字段引用、方法引用、属性
（3）字段数据：每个字段的名字、类型(如类的全路径名、类型或接口) 、修饰符（如public、abstract、final）、属性
（4）方法数据：每个方法的名字、返回类型、参数类型(按顺序)、修饰符、属性
（5）方法代码：每个方法的字节码、操作数栈大小、局部变量大小、局部变量表、异常表和每个异常处理的开始位置、结束位置、代码处理在程序计数器中的偏移地址、被捕获的异常类的常量池索引
```

### 13、运行时常量池

```
JDK1.6之前字符串常量池位于方法区之中。JDK1.7字符串常量池已经被挪到堆之中。

可通过参数-XX:PermSize和-XX:MaxPermSize设置

常量池（Constant Pool）：常量池数据编译期被确定，是Class文件中的一部分。存储了类、方法、接口等中的常量，当然也包括字符串常量。
字符串池/字符串常量池（String Pool/String Constant Pool）：是常量池中的一部分，存储编译期类中产生的字符串类型数据。

运行时常量池（Runtime Constant Pool）：方法区的一部分，所有线程共享。虚拟机加载Class后把常量池中的数据放入到运行时常量池。常量池：可以理解为Class文件之中的资源仓库，它是Class文件结构中与其他项目资源关联最多的数据类型。
常量池中主要存放两大类常量：字面量（Literal）和符号引用（Symbolic Reference）。

字面量：文本字符串、声明为final的常量值等。

符号引用：类和接口的完全限定名（Fully Qualified Name）、字段的名称和描述符（Descriptor）、方法的名称和描述符。
```

### 14、直接内存

```
可通过-XX:MaxDirectMemorySize指定，如果不指定，则默认与Java堆的最大值（-Xmx指定）一样。

直接内存（Direct Memory）并不是虚拟机运行时数据区的一部分，也不是Java虚拟机规范中定义的内存区域，但是这部分内存也被频繁地使用，而且也可能导致OutOfMemoryError异常出现。
```

### 15、堆

```
可通过参数 -Xms 和-Xmx设置

Java堆是被所有线程共享,是Java虚拟机所管理的内存中最大的一块 Java堆在虚拟机启动时创建
Java堆唯一的目的是存放对象实例，几乎所有的对象实例和数组都在这里
Java堆为了便于更好的回收和分配内存，可以细分为：新生代和老年代；再细致一点的有Eden空间、From Survivor空间、To Survivor区
新生代：包括Eden区、From Survivor区、To Survivor区，系统默认大小Eden:Survivor=8:1。
老年代：在年轻代中经历了N次垃圾回收后仍然存活的对象，就会被放到年老代中。因此，可以认为年老代中存放的都是一些生命周期较长的对象。
java虚拟机栈(stack)

可通过参数 栈帧是方法运行期的基础数据结构栈容量可由-Xss设置

Java虚拟机栈是线程私有的，它的生命周期与线程相同。
每一个方法被调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中从入栈到出栈的过程。
虚拟机栈是执行Java方法的内存模型(也就是字节码)服务：每个方法在执行的同时都会创建一个栈帧，用于存储 局部变量表、操作数栈、动态链接、方法出口等信息

元空间的本质和永久代类似，都是对JVM规范中方法区的实现。不过元空间与永久代之间最大的区别在于：元空间并不在虚拟机中，而是使用本地内存。因此，默认情况下，元空间的大小仅受本地内存限制（具体不太清楚）
```

### 16、jvm中判断对象是否为垃圾

```
1、方法一：引用计数法（无法解决循环引用的问题）

2、方法二：可达性分析算法

3、可以作为GC Roots的对象：
虚拟机栈(栈帧中的本地变量表)中引用的对象。(可以理解为:引用栈帧中的本地变量表的所有对象)
方法区中静态属性引用的对象(可以理解为:引用方法区该静态属性的所有对象)
方法区中常量引用的对象(可以理解为:引用方法区中常量的所有对象)
本地方法栈中(Native方法)引用的对象(可以理解为:引用Native方法的所有对象)

```

18、jvm中各成分加载顺序

```
一、什么时候会加载类？
使用到类中的内容时加载：有三种情况
1.创建对象：new StaticCode();
2.使用类中的静态成员：StaticCode.num=9;  StaticCode.show();
3.在命令行中运行：Java StaticCodeDemo

二、类所有内容加载顺序和内存中的存放位置：
利用语句进行分析。
1.Person p=new Person("zhangsan",20);
该句话所做的事情：
1.在栈内存中，开辟main函数的空间，建立main函数的变量 p。
2.加载类文件：因为new要用到Person.class,所以要先从硬盘中找到Person.class类文件，并加载到内存中。
加载类文件时，除了非静态成员变量（对象的特有属性）不会被加载，其它的都会被加载。
记住：加载，是将类文件中的一行行内容存放到了内存当中，并不会执行任何语句。---->加载时期，即使有输出语句也不会执行。
静态成员变量（类变量）  ----->方法区的静态部分
静态方法              ----->方法区的静态部分
非静态方法（包括构造函数）  ----->方法区的非静态部分
静态代码块 ----->方法区的静态部分
构造代码块 ----->方法区的静态部分

注意：在Person.class文件加载时，静态方法和非静态方法都会加载到方法区中，只不过要调用到非静态方法时需要先实例化一个对象
，对象才能调用非静态方法。如果让类中所有的非静态方法都随着对象的实例化而建立一次，那么会大量消耗内存资源，
所以才会让所有对象共享这些非静态方法，然后用this关键字指向调用非静态方法的对象。


3.执行类中的静态代码块：如果有的话，对Person.class类进行初始化。
4.开辟空间：在堆内存中开辟空间，分配内存地址。
5.默认初始化：在堆内存中建立 对象的特有属性，并进行默认初始化。
6.显示初始化：对属性进行显示初始化。
7.构造代码块：执行类中的构造代码块，对对象进行构造代码块初始化。
8.构造函数初始化：对对象进行对应的构造函数初始化。
9.将内存地址赋值给栈内存中的变量p。
2.p.setName("lisi");
1.在栈内存中开辟setName方法的空间，里面有：对象的引用this，临时变量name
2.将p的值赋值给this,this就指向了堆中调用该方法的对象。
3.将"lisi" 赋值给临时变量name。
4.将临时变量的值赋值给this的name。
3.Person.showCountry();
1.在栈内存中，开辟showCountry()方法的空间，里面有：类名的引用Person。
2.Person指向方法区中Person类的静态方法区的地址。
3.调用静态方法区中的country，并输出。
  注意：要想使用类中的成员，必须调用。通过什么调用？有：类名、this、super
  
三、静态代码块、构造代码块和构造函数的区别
静态代码块：用于给类初始化，类加载时就会被加载执行，只加载一次。
构造代码块：用于给对象初始化的。只要建立对象该部分就会被执行，且优先于构造函数。
构造函数：  给对应对象初始化的，建立对象时，选择相应的构造函数初始化对象。
 创建对象时，三者被加载执行顺序：静态代码块--->构造代码块--->构造函数
 //执行顺序：（优先级从高到低。）静态代码块>mian方法>构造代码块>构造方法。
```

### 17、