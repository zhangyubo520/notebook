# Nginx

### 1、概念

```
高性能的web服务器
```

### 2、安装

```
1、依赖安装
sudo yum -y install    openssl openssl-devel pcre pcre-devel    zlib zlib-devel gcc gcc-c++

2、解压 tar -zxvf /opt/software/nginx-1.12.2.tar.gz -C /opt/module/

3、解压后得到文件夹nginx-1.12.2
进入文件夹：cd /opt/module/nginx-1.12.2/

4、执行：./configure   --prefix=/opt/module/nginx    
执行：make && make install

5、放开80端口限制（nginx占用80端口，默认情况下非root用户不允许使用1024以下端口）
执行：sudo setcap cap_net_bind_service=+eip /opt/module/nginx/sbin/nginx

6、查看80端口占用情况：netstat -ntulp | grep 80 发现httpd在占用80端口，并且kill不掉

7、停用httpd服务：sudo systemctl stop httpd

8、开机禁启httpd服务：sudo systemctl disable httpd

9、启动Nginx服务：/opt/module/nginx/sbin/nginx

10、重启Nginx服务：/opt/module/nginx/sbin/nginx -s reload

11、停止Nginx服务：/opt/module/nginx/sbin/nginx -s stop

12、启动后可以用浏览器输入：hadoop102/ 查看到welcome

13、更改配置文件实现代理三台虚拟机：
vim /opt/module/nginx/conf/nginx.conf    修改如下内容（注意大括号的范围）
http{
   ..........
    upstream logserver{
      server    hadoop102:8090 weight=1;  
      server    hadoop103:8090 weight=1;
      server    hadoop104:8090 weight=1;
 
    }
    server {
        listen       80;
        server_name  logserver;
 
        location / {
            root   html;
            index  index.html index.htm;
            proxy_pass http://logserver;
            proxy_connect_timeout 10;
 
         }
   ..........
}

14、重启服务并浏览器查看发现报错，正常
```

### 3、正向代理

```
概念：内网服务器主动去请求外网的服务的一种行为

理解1：VPN的原理大体上也类似于一个正向代理，也就是需要访问外网的电脑，发起一个访问外网的请求，通过本机上的VPN去寻找一个可以访问国外网站的代理服务器，代理服务器向外国网站发起请求，然后把结果返回给本机

理解2：正向代理其实就是说客户端无法主动或者不打算完成主动去向某服务器发起请求，而是委托了nginx代理服务器去向服务器发起请求，并且获得处理结果，返回给客户端

理解3：自己做不了的事情或者自己不打算做的事情，委托或依靠别人来完成

正向代理配置：
 server {
         #指定DNS服务器IP地址  
         resolver 114.114.114.114;   
         #指定代理端口    
         listen 8080;  
         ocation / {
             #设定代理服务器的协议和地址（固定不变）    
             proxy_pass http://$http_host$request_uri; 
         }
```

### 4、反向代理

```
概念：用代理服务器来接受客户端发来的请求，然后将请求转发给内网中的上游服务器，上游服务器处理完之后，把结果通过nginx返回给客户端

理解1：反向代理是对于来自外界的请求，先通过nginx统一接受，然后按需转发给内网中的服务器，并且把处理请求返回给外界客户端，此时代理服务器对外表现的就是一个web服务器，客户端根本不知道“上游服务器”的存在

配置：
server{
    #监听端口
    listen 80;
    #服务器名称，也就是客户端访问的域名地址
    server_name  a.xxx.com;
    #nginx日志输出文件
    access_log  logs/nginx.access.log  main;
    #nginx错误日志输出文件
    error_log  logs/nginx.error.log;
    root   html;
    index  index.html index.htm index.php;
    location / {
        #被代理服务器的地址
        proxy_pass  http://localhost:8081;
        #对发送给客户端的URL进行修改的操作
        proxy_redirect     off;
        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
        proxy_max_temp_file_size 0;
   }
}

```

### 5、透明代理

```
概念：做简单代理，意思客户端向服务端发起请求时，请求会先到达透明代理服务器，代理服务器再把请求转交给真实的源服务器处理，也就是是客户端根本不知道有代理服务器的存在

理解1：它的用法有点类似于拦截器，如某些制度严格的公司里的办公电脑，无论我们用电脑做了什么事情，安全部门都能拦截我们对外发送的任何东西，这是因为电脑在对外发送时，实际上先经过网络上的一个透明的服务器，经过它的处理之后，才接着往外网走，而我们在网上冲浪时，根本没有感知到有拦截器拦截我们的数据和信息
```

### 6、负载均衡

```
负载均衡的集中模式：

轮询：每个请求按时间顺序逐一分配到不同的后端服务器，也是nginx的默认模式。轮询模式的配置很简单，只需要把服务器列表加入到upstream模块中即可

配置：
upstream serverList {
    server 1.2.3.4;
    server 1.2.3.5;
    server 1.2.3.6;
    }
ip_hash：每个请求按访问IP的hash结果分配，同一个IP客户端固定访问一个后端服务器。可以保证来自同一ip的请求被打到固定的机器上，可以解决session问题；负载中有三台服务器，当请求到达时，nginx优先按照ip_hash的结果进行分配，也就是同一个IP的请求固定在某一台服务器上，其它则按时间顺序把请求分配给三台服务器处理

配置：
upstream serverList {
	ip_hash
    server 1.2.3.4;
    server 1.2.3.5;
    server 1.2.3.6;
    }
url_hash：按访问url的hash结果来分配请求，相同的url固定转发到同一个后端服务器处理。

配置：
upstream serverList {
    server 1.2.3.4;
    server 1.2.3.5;
    server 1.2.3.6;
    hash $request_uri; 
    hash_method crc32;
    }
    
fair：按后端服务器的响应时间来分配请求，响应时间短的优先分配

配置：
upstream serverList {
    server 1.2.3.4;
    server 1.2.3.5;
    server 1.2.3.6;
    fair;
    }


说明：每种模式下每台服务器都有如下参数：
down: 当前服务器暂不参与负载

weight: 权重，值越大，服务器的负载量越大。

max_fails：允许请求失败的次数，默认为1。

fail_timeout:max_fails次失败后暂停的时间。

backup：备份机， 只有其它所有的非backup机器down或者忙时才会请求backup机器。
```

### 7、静态服务器

```
配置：
server {
        listen       80;                                                         
        server_name  www.xxx.com;                                               
        client_max_body_size 1024M;
        location / {
               root   /var/www/xxx_static;
               index  index.html;
           }
    }
```

### 8、简单使用

```

```

### 9、