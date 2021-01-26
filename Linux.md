# 	Linux

## 概念

### 1、安装VMware Workstation

```
步骤：双击——》接受——》自定义路径——》输入密钥——》完成打开
```

### 2、安装CentOS

```
步骤：打开VMware——》新建——》自定义类型——》下一步——》稍后安装操作系统——》选择Linux和centOS7 64位——》命名并指定位置建议固态硬盘中——》选择CPU数量：8核就2X4——》虚拟机内存2G以上——》网络NAT——》推荐——》推荐——》新建虚拟硬盘——》指定容量50G,将虚拟机拆分成多个文件——》磁盘文件位置为前面的安装位置——》装机完成
```

### 3、安装Linux

```
步骤：
1、cd/dvd双击选择映像文件E:\BaiduNetdiskDownload\04_hadoop生态\02_尚硅谷大数据技术之Linux\02_尚硅谷大数据技术之Linux\02_尚硅谷大数据技术之Linux\2.资料\01_安装包\linux7.5镜像\CentOS-7.5-x86_64-DVD-1804

2、开启虚拟机

3、键盘上下键选择install CentOS7

4、中文

5、日期时间修改中国上海

6、软件选择——》GNOME桌面

7、安装位置自动配置分区

说明：如果是手动，可以点进去选择我要配置分区
/root 1G 文件系统ex4
swap 2G(与内存一致) 文件系统swap
/ (50 - 1 - 2) = 47G 文件系统ex4

8、KDUMP中不勾选

9、网络打开并设置主机名

10、安装（10分钟左右）

11、完成时设置root密码，谨慎，新建用户不管

12、配置（2-3分钟）

13、完成时重启

14、接受协议完成配置
 	 
15、进入欢迎界面选择语言和时区设置密码

16、注销后在未列出输入root,输入密码，进入，设置，进入
```

### 4、vi和vim

文档编辑器

```
1、右键中间空白处打开终端，pwd 显示当前所在路径 ls 显示当前路径下的内容 vi或vim 文件名进入文件

2、yy y2y dd d2d p yw dw x shift+^ shift+$ shift+g 2+shift+g i a o I A O :q :w :! :set nu :set nonu :%s/old/new :%s/old/new/g / 要查的东西

```

### 5、Linux中的文件目录有哪些

```
1、主目录是/

2、次级目录介绍：
/bin:存放的是经常使用的命令；
/home:存放普通用户的资料，Linux中每一个用户都有自己的目录，一般目录名是以账号名命名；
/root:该目录为系统管理员，超级权限者的主目录；
/lib:开机所需要的动态链接共享库，几乎所有的程序都需要用到这些共享库；
/etc:所有系统管理相关的配置文件和子目录；
/user：所有用户程序和文件都在这个目录下，类似Windows里的program files目录；
别动：/boot /proc /srv /sys；
/media:U盘，光驱等识别后挂载在这个目录下，CentOS7转移到/run/media下；
/mnt:让用户临时挂载别的文件系统；
/opt:用户安装的mysql等应用可以放在这个目录下；
/var:不断变化的，如日志文件；
```

### 6、网络设置

```
1、ifconfig；查看本机网络及ip

2、ping 192.168.75.101;查看连接性

3、修改本机ip为固定ip:vim /etc/sysconfig/network-scripts/ifcfg-ens33
说明：修改配置文件内容如下：
BOOTPROTO="static"
增加：
IPADDR=192.168.75.100
GATEWAY=192.168.75.2
DNS1=192.168.75.2
修改完成配置文件后执行 service network restart重启网络，或者执行reboot重启虚拟机
```

### 7、主机名

```
查看：hostname
修改：vim /etc/hostname
```

### 8、主机名与ip映射

```
修改配置文件：vim /etc/hosts
增加如下内容:
192.168.75.100 zhangyubo
192.168.75.101 hadoop101
192.168.75.102 hadoop102
192.168.75.103 hadoop103
192.168.75.104 hadoop104
192.168.75.105 hadoop105
说明：自己练习的话尽量名字和ip统一起来，好用又好记

修改Windows7的ip映射：
进入 C:\Windows\System32\drivers\etc\hosts修改配置文件增加
192.168.75.100 zhangyubo
192.168.75.101 hadoop101
192.168.75.102 hadoop102
192.168.75.103 hadoop103
192.168.75.104 hadoop104
192.168.75.105 hadoop105

修改Windows10的ip映射：
复制C:\Windows\System32\drivers\etc\hosts文件至桌面，修改它，增加
192.168.75.100 zhangyubo
192.168.75.101 hadoop101
192.168.75.102 hadoop102
192.168.75.103 hadoop103
192.168.75.104 hadoop104
192.168.75.105 hadoop105
用修改好的hosts文件覆盖原路径下的hosts文件
```

### 9、物理机防火墙

```
网络——>网络和Internet设置——>Windows防火墙——>关闭防火墙
```

### 10、虚拟机防火墙

```
systemctl disable firewalld.service 开机自关闭
systemctl enable firewalld.service 开机自开启
systemctl status firewalld 查看防火墙状态
systemctl stop firewalld 关闭
systemctl start firewalld 打开
systemctl restart firewalld 重启
```

### 11、后台服务管理

```
systemctl list-unit-files         （功能描述：查看服务开机启动状态）
systemctl disable 服务名  （功能描述：关掉指定服务的自动启动）
systemctl enable 服务名   （功能描述：开启指定服务的自动启动）

```

### 12、进程运行级别

```
常用3和5：使用vim /etc/inittab进入查看默认运行级别

multi-user.target: 等价于原运行级别3（多用户有网，无图形界面）
graphical.target: 等价于原运行级别5（多用户有网，有图形界面）
```

### 13、关机

```
sync：同步数据至硬盘
halt：关机
reboot:重启
shutdown -h now 立马关机
shutdown -h 1 一分钟后关机
shutdown -r 2 一分钟后重启
```

### 14、远程登录工具

```
Xshell, SSH Secure Shell, SecureCRT,FinalShell
```

### 15、常用命令

| man [命令或配置文件]                           | 获得帮助信息                         |
| ---------------------------------------------- | ------------------------------------ |
| Ctrl + c                                       | 停止进程                             |
| Ctrl + l                                       | 清屏                                 |
| Ctrl + alt                                     | Linux 和windows之间切换              |
| pwd                                            | 显示当前工作目录的绝对路径           |
| ls -al                                         | 显示全部文件，包括隐藏的             |
| cd                                             | 切换路径                             |
| mkdir -p /etc/zhangyubo/java                   | 创建多层文件目录                     |
| rmdir -r java                                  | 递归的删除一个文件目录               |
| touch /etc/zhangyubo/java/abc.txt              | 创建文件                             |
| cp -r /etc /zhangyubo/java /root               | 递归地复制一个文件至root下           |
| rm -rf /etc/zhangyubo                          | 递归地强制地删除文件或目录           |
| mv /etc /zhangyubo/java /etc /zhangyubo/javaee | 重命名或者剪切                       |
| cat -n abc.txt                                 | 带行号地显示文件内容（小文件）       |
| more abc.txt                                   | 分屏查看文件内容（大文件）           |
| less abc.txt                                   | 分屏查看文件内容（大文件）           |
| echo -e "hello\tworld"                         | 在控制台输出hello         world      |
| head -n 2 smartd.conf                          | 显示文档头两行                       |
| tail -n 1 smartd.conf                          | 查看文档最后一行                     |
| tail -f smart.conf                             | 显示文件最新追加的内容，监视文件变化 |
| ls -l  > 文件                                  | 列表的内容写入文件a.txt中（覆盖）    |
| ls -al  >> 文件                                | 列表的内容写入文件a.txt中（追加）    |
| cat 文件1 > 文件2                              | 将文件1的内容覆盖到文件2             |
| echo “内容” >> 文件                            | 讲内容写入文件中（追加）             |
| ln -s xiyou/dssz/houge.txt ./houzi             | 创建快捷方式houzi                    |
| history                                        | 查看历史命令                         |

```
rz，sz是Linux/Unix同Windows进行ZModem文件传输的命令行工具

sz：将选定的文件发送（send）到本地机器
rz：运行该命令会弹出一个文件选择窗口，从本地选择文件上传到Linux服务器
find -type f -size +100M 产看大于100M的文件
du -sh . 查看当前目录大小
du -sh * 查看目录下所有文件及大小

安装：yum install lrzsz

```



### 16、日期

```
date： 	显示当前时间
date +%Y	显示当前年
date +%m 	显示当前月
date +%d	显示当前日
date "+%Y-%m-%d %H:%M:%S" 显示年月日时分秒
date -s "2017-06-19 20:52:18" 设置时间
cal		查看当月日历
cal 2017	查看某年的日历
```

### 17、用户

```
useradd tangseng 	创建新用户tangseng
useradd -g 组名 用户名  		将·新用户添加到指定组中
passwd tangseng		设置用户密码
id tangseng		查看是否有用户tangseng
cat  /etc/passwd		查看所有用户
userdel  用户名 		删除用户保留主目录
userdel -r 用户名 		删除用户及主目录
su 用户名 			切换用户（获得执行权限，没有获得环境变量）
su - 用户名 			切换用户（获得执行权限，并获得环境变量）
groupadd 组名  	添加组
groupdel 组名  	删除组
groupmod -n 新组名 老组名  	重命名
cat  /etc/group  		查看所有组

vi /etc/sudoers 修改文件
## Allow root to run any commands anywhere
root      ALL=(ALL)     ALL
atguigu   ALL=(ALL)     NOPASSWD:ALL
将用户atguigu的权限提升至root权限，并且不需要输入密码，然后用atguigu用户sudo mkdir module创建文件
说明：必须在命令前增加sudo
```

### 18、文件权限

```
d---rwxrwx. 15 zhangyubo zhangyubo 4096 4月   2 09:30 zhangyubo
说明：文件属性10位表示，用角标0-9表示
0首位：文件类型：-表示文件，d表示目录，l代表链接文档
1-3位：文件所有者拥有的权限
4-6位：同组的其他用户拥有的权限
7-9位：其他用户拥有的权限

权限解释：
作用于文档时：r,查看，w，文档内增删改查，x,可被执行
作用于目录时：r,可ls,w,目录内增删改查，x,可进入目录

修改权限：chmod  [mode=421 ]  [文件或目录]
说明：r=4 w=2 x=1        rwx=4+2+1=7 组合填写即可

修改文件所有者：chown [-R] [最终用户] [文件或目录]将文件或目录递归地改为最终用户所有

修改文件所属组：chgrp [最终用户组] [文件或目录]
```

### 19、搜索

```
find:
find xiyou/ -name *.txt		根据名称查找/目录下的filename.txt文件
find xiyou/ -user atguigu		查找/opt目录下，用户名称为atguigu的文件
find /home -size +204800 		在/home目录下查找大于200m的文件（+n 大于  -n小于   n等于）

locate:
updatedb	第一次使用需创建查询数据库	
locate tmp	查询tmp文件夹

| grep 模糊条件  		过滤信息（提示信息）
```

### 20、压缩解压缩

```
tar -zcvf xiyou.tar.gz xiyou/ xiyou/ xiyou/mingjie/ xiyou/dssz/houge.txt压缩文件至xiyou.tar.gz内

tar -zxvf houma.tar.gz 解压至当前目录

tar -zxvf xiyou.tar.gz -C /opt 解压至opt目录下

```

### 21、磁盘

```
df -h:显示磁盘容量
fdisk -l:查看分区
lsblk -f:查看设备挂载情况
mount/umount 挂载/卸载
```

### 22、进程

```
ps -aux | grep xxx		（功能描述：查看系统中所有进程）
ps -ef | grep xxx		（功能描述：可以查看子父进程之间的关系）

kill  [选项] 进程号（功能描述：通过进程号杀死进程）
killall 进程名称（功能描述：通过进程名称杀死进程，也支持通配符，这在系统因负载过大而变得很慢时很有用）

kill -9 5102 强制杀死进程5102

pstree 进程树

top 查看进程排名
```

### 21、定时任务

```
crontab -e
*/1 * * * * /bin/echo ”11” >> /root/bailongma.txt

说明：-e创建定时任务，并进入编辑界面，-l查询定时任务，-r删除定时任务
```

| 项目      | 含义                 | 范围                    |
| --------- | -------------------- | ----------------------- |
| 第一个“*” | 一小时当中的第几分钟 | 0-59                    |
| 第二个“*” | 一天当中的第几小时   | 0-23                    |
| 第三个“*” | 一个月当中的第几天   | 1-31                    |
| 第四个“*” | 一年当中的第几月     | 1-12                    |
| 第五个“*” | 一周当中的星期几     | 0-7（0和7都代表星期日） |

| 特殊符号 | 含义                                                         |
| -------- | ------------------------------------------------------------ |
| *        | 代表任何时间。比如第一个“*”就代表一小时中每分钟都执行一次的意思。 |
| ，       | 代表不连续的时间。比如“0 8,12,16 * * * 命令”，就代表在每天的8点0分，12点0分，16点0分都执行一次命令 |
| -        | 代表连续的时间范围。比如“0 5  *    *  1-6命令”，代表在周一到周六的凌晨5点0分执行命令 |
| */n      | 代表每隔多久执行一次。比如“*/10  *    *  *  *  命令”，代表每隔10分钟就执行一遍命令 |

| 时间              | 含义                                       |
| ----------------- | ------------------------------------------ |
| 45 22 * * * 命令  | 在22点45分执行命令                         |
| 0 17 * * 1 命令   | 每周1 的17点0分执行命令                    |
| 0 5 1,15 * * 命令 | 每月1号和15号的凌晨5点0分执行命令          |
| 40 4 * * 1-5 命令 | 每周一到周五的凌晨4点40分执行命令          |
| */10 4 * * * 命令 | 每天的凌晨4点，每隔10分钟执行一次命令      |
| 0 0 1,15 * 1 命令 | 每月1号和15号，每周1的0点0分都会执行命令。 |

### 23、软件包下载安装

```
rpm -qa	|grep xxx			（功能描述：查询所安装的所有rpm软件包）
rpm -e firefox 卸载
rpm -ivh firefox-45.0.1-1.el6.centos.x86_64.rpm 安装

网上下载：
yum install wget 	从网上下载wget
cd /etc/yum.repos.d/ 进入文件目录
cp CentOS-Base.repo   CentOS-Base.repo.backup 复制文件备份
wget http://mirrors.aliyun.com/repo/Centos-7.repo 下载阿里云
wget http://mirrors.163.com/.help/CentOS7-Base-163.repo 下载网易
mv CentOS7-Base-163.repo   CentOS-Base.repo 使用网易替换原来的文件
yum clean all 清理缓存
yum makecache 缓存新数据

测试：yum list | grep firefox
yum -y install firefox.x86_64
```

### 24、克隆

```
提示：设置好网络，ip,名字，防火墙，yum下载源等后再进行克隆，减少工作量，最好设置好两台样机，专门用来克隆
步骤：右键主机——>管理——>克隆——>从当前状态——>创建完整克隆——>名字——>完成——>关闭——>修改ip和主机名
```

### 25、面试题1

```
Linux常用命令：
find、df、tar、ps、top、netstat
```

### 26、面试题2

```
Linux查看内存、磁盘存储、io 读写、端口占用、进程等命令：
1、查看内存：top
2、查看磁盘存储情况：df -h
3、查 看磁盘IO读写情况：iotop（需要安装一下：yum install iotop）、iotop -o（直接查看输出比较高的磁盘读写程序）
4、查看端口占用情况：netstat -tunlp | grep 端口号
5、查看进程：ps -aux
```

### 27、软硬连接

```
硬链接：ln aaa.txt /test/aaa.txt_hard_link
查看：ls -il

软连接：ln -s aaa.txt /test/aaa.txt_soft_link
查看：ls -il

区别：硬链接两个文件完全相同，类似文件备份，源文件丢失，硬链接还在
软连接相当于创建了快捷方式，源文件丢失，软连接就失效了
```

