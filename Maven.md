# Maven

### 1、概念

#### 1、管理jar包

```
用来帮助我们管理jar包的，包括添加，依赖，冲突，自动构建等
```

#### 2、环境变量

```
1、检查jdk的环境变量配置：
%JAVA_HOME%\bin;(JAVA_HOME = jdk中bin目录的根目录)

2、解压maven;

3、配置环境变量：
%M2_HOME%\bin;(M2_HOME = maven中bin目录的根目录)

4、检查：
使用mvn -v在命令行检查即可，注意空格
```

#### 3、手动创建maven工程

```
1、文件目录：
Hello //工程名（根）
	 src //主目录（主程序）
	 ——main //次目录1（主程序）
	 ————java //次次目录1(主程序源代码)
	 ————resources //次次目录2(主程序配置文件和资源文件)
	 --test //次目录2（测试程序）
	 ————java //次次目录1(测试程序源代码)
	 ————resources //次次目录2(测试程序配置文件和资源文件)
	 pom.xml //maven核心配置文件
	 
2、编写主程序和测试程序（编写主程序和测试程序各自的源代码）

3、在命令行输入mvn clean、mvn  compile、mvn  test-compile、mvn  test、mvn  package、mvn  instal进行测试并将写好的代码打包进jar包仓库供以后使用
注意：要在pom.xml文件所在目录运行以上命令
```

#### 4、配置本地仓库

```
1、settings.xml：
在解压的maven文件下找到conf\settings.xml文件

2、自定义仓库设置：
文件中找到<localRepository>E:\LocalRepository</localRepository>中间为你自己的jar仓库，以后maven下载和寻找jar都会在这个目录下

3、设置阿里云镜像提高下载速度：
<mirror>
    <id>nexus-aliyun</id>
    <mirrorOf>central</mirrorOf>
    <name>Nexus aliyun</name>
    <url>http://maven.aliyun.com/nexus/content/groups/public</url>
</mirror>
将它们写入pom.xml文件中
```

#### 5、idea中使用maven

```
1、设置maven的安装目录及本地仓库

2、设置Maven自动导入依赖的jar包

3、右键工作空间new Module ——》Maven ——》依次填入公司域名倒叙 + 项目名作为GroupId、模块名作为ArtifactId,版本号

4、next给maven命名，建议跟模块名一致，这时会自动生成手动创建maven工程时的那些目录，很方便

5、配置pom.xml文件——》编写主程序——》编写测试程序——》使用maven运行测试
```

#### 6、自动打包依赖的包

```
<build>
    <plugins>
        <plugin>
            <artifactId>maven-assembly-plugin</artifactId>
            <configuration>
                <descriptorRefs>
                    <descriptorRef>jar-with-dependencies</descriptorRef>
                </descriptorRefs>
              <archive>
                    <manifest>
                     <!-- 指定主类 -->
                        <mainClass>xxx.xxx.XXX</mainClass>
                    </manifest>
                </archive>
            </configuration>
            <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>

            </executions>
        </plugin>
    </plugins>
</build>	
添加到pom.xml文件中
```

#### 7、依赖管理

```
1、依赖范围：
1)compile：main程序和test程序均可见，也可以部署，也可依赖传递
2)test：main不可见，test可见，不可部署，不可依赖传递
3)provided：main可见，test可见，不可部署（这些提供过了），不可依赖传递
说明：默认是compile的；
可以使用<scope>test</scope>制定当前工程的依赖

2、解决jar冲突的两个原则：
路径最短
先声明优先
```

#### 8、排除依赖

```
<dependency>

    <groupId>com.atguigu.maven</groupId>
    <artifactId>OurFriends</artifactId>
    <version>1.0-SNAPSHOT</version>
    
    <!--依赖排除-->
    
    <exclusions>
        <exclusion>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
        </exclusion>
    </exclusions>
    
</dependency>

增加exclusions,想加几个就加几个
```

#### 9、统一管理版本号

```
1、定义标识：
<properties>
    <junit.version>4.5</junit.version>
</properties>

2、使用标识：
<dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
</dependency>

说明：以后改版本改一个地方就可以了
```

#### 10、maven能够自动化构建的原因

```
1、maven的生命周期定义了各个构建环节的执行顺序，有了这个清单，maven就可以自动化构建执行命令了；
2、执行生命周期中的任何一个阶段时，它前面的所有阶段都会执行，这样，maven就能够自动执行构建过程的各个环节
```

#### 11、继承

```
背景：在test中的依赖信息不能够通过依赖链传递下去，所以每一个test中都得重复定义，为了解决这一问题，引入继承

解决方法：引入继承

步骤1：建父工程，打包方式为pom;

	<groupId>com.atguigu.maven</groupId>
	<artifactId>Parent</artifactId>
	<version>1.0-SNAPSHOT</version>
	
	<packaging>pom</packaging>
说明：只保留pom文件即可，不必有代码

步骤2：在子工程里引用父工程

<parent>
    <groupId>com.atguigu.maven</groupId>
    <artifactId>Parent</artifactId>
    <version>1.0-SNAPSHOT</version>
	<relativePath>../Parent/pom.xml</relativePath>
</parent>
说明：坐标为父类坐标，连接路径为父类的相对的路径../Parent/pom.xml

步骤3、在父类中维护继承的信息
<dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>4.9</version>
                <scope>test</scope>
            </dependency>
        </dependencies>
</dependencyManagement>

说明:将Parent项目中的dependencies标签，用dependencyManagement标签括起来

步骤4、在子项目中重新指定需要的依赖，删除范围和版本号

删除自身定义的groupId:

<!--<groupId>com.guigu.maven</groupId>-->//注释掉了，这部分父类已经指定
    <artifactId>Hello</artifactId>//这是自己的
    <version>1.0-SNAPSHOT</version>//这是自己的

删除自身托管于父工程中的版本信息：

<dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
<!--        <version>${junit.version}</version>-->//注释掉了
            <scope>test</scope>
</dependency>

说明：经过上述4个步骤，就可以在父工程中统一管理版本信息了，同一个工程下肯定用的是一个版本，统一管理
```

#### 12、聚合

```
1、写完所有工程后可以来个聚合，将所有项目统一于一个父工程中，在父工程中指定模块工程的相对路径即可，如下所示：

<modules>
    <module>../MakeFriend</module>
    <module>../OurFriends</module>
    <module>../HelloFriend</module>
    <module>../Hello</module>
</modules>

2、用处：maven可以根据各个模块的关系，自动选择安装顺序，然后统一打包，很方便
```

#### 13、创建Web工程

```
idea中，进入 file -> project structure -> 选中你想添加web的module -> 加号然后下拉选中web -> 点击ok -> 在WEB-INF下创建jsp文件 ->部署
```

#### 14、资源

```
我们可以到http://mvnrepository.com/搜索需要的jar包的依赖信息。
http://search.maven.org/
```

