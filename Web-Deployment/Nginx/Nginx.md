# Nginx

>[nginx官网](http://nginx.org/)
>
>[官方中文文档](https://www.nginx.cn/doc/index.html)
>
>参考博客：
>
>- https://blog.csdn.net/weixin_42167759/article/details/85049546
>- https://www.cnblogs.com/hanyinglong/p/5141504.html



## 介绍

nginx是一个高性能的HTTP和反向代理服务器，也是一个IMAP/POP3/SMTP服务器，源代码符合BSD开源

特点为：

1. 占用内存少
2. 并发能力强
3. 模块化的结构
4. 处理静态文件,索引文件以及自动索引，打开文件描述符缓冲
5. 无缓存的反向代理加速，简单的负载均衡和容错

## 安装

推荐使用LAMP一键安装包或LNMP一键安装包安装

如果确定想手动安装，则可参考[安装教程](https://nginx.org/en/linux_packages.html#Ubuntu)

对于Linux Ubuntu发行版过程如下：

1. 下载必要的系统依赖：

   ```shell
   sudo apt install curl gnupg2 ca-certificates lsb-release ubuntu-keyring
   ```

2. 引入nginx官方签名：

   ```shell
   curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor \
       | sudo tee /etc/apt/trusted.gpg.d/nginx-archive-keyring.gpg >/dev/null
   ```

3. 确认下载的签名正确：

   ```shell
   gpg --dry-run --quiet --import --import-options import-show /etc/apt/trusted.gpg.d/nginx-archive-keyring.gpg
   ```

   确认输出为：

   ```shell
   pub   rsa2048 2011-08-19 [SC] [expires: 2024-06-14]
         573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62
   uid                      nginx signing key <signing-key@nginx.com>
   ```

4. 添加官方仓库：

   稳定版仓库输入以下命令：

   ```shell
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/trusted.gpg.d/nginx-archive-keyring.gpg] \
   http://nginx.org/packages/ubuntu `lsb_release -cs` nginx" \
       | sudo tee /etc/apt/sources.list.d/nginx.list
   ```

   如果需要添加最新开发版，即Mainline Package，则为：

   ```shell
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/trusted.gpg.d/nginx-archive-keyring.gpg] \
   http://nginx.org/packages/mainline/ubuntu `lsb_release -cs` nginx" \
       | sudo tee /etc/apt/sources.list.d/nginx.list
   ```

5. 固定存储库为nginx官方发行版，防止下载apt存储库的落后版本：

   ```shell
   echo -e "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900\n" \
       | sudo tee /etc/apt/preferences.d/99nginx
   ```

6. 更新源并安装nginx

   ```shell
   sudo apt update
   sudo apt install nginx
   ```

6. 



## Nginx配置

nginx目录下的nginx.conf为nginx的主配置文件，分为：

1. main（全局配置）：
2. server（主机配置）：继承main的配置
3. location（URL匹配特定位置的设置）：继承server配置
4. upstream（负载均衡服务器设置）：不会继承其它设置也不会被继承

```nginx
##代码块中的events、http、server、location、upstream等都是块配置项##
##块配置项可以嵌套。内层块直接继承外层快，例如：server块里的任意配置都是基于http块里的已有配置的##
 
##Nginx worker进程运行的用户及用户组 
#语法：user username[groupname]    默认：user nobody nobody
#user用于设置master进程启动后，fork出的worker进程运行在那个用户和用户组下。当按照"user username;"设置时，用户组名与用户名相同。
#若用户在configure命令执行时，使用了参数--user=usergroup 和 --group=groupname,此时nginx.conf将使用参数中指定的用户和用户组。
user  nobody;

##Nginx worker进程个数：其数量直接影响性能。
#每个worker进程都是单线程的进程，他们会调用各个模块以实现多种多样的功能。如果这些模块不会出现阻塞式的调用，那么，有多少CPU内核就应该配置多少个进程，反之，有可能出现阻塞式调用，那么，需要配置稍多一些的worker进程。
worker_processes  1;
 
##ssl硬件加速。
#用户可以用OpneSSL提供的命令来查看是否有ssl硬件加速设备：openssl engine -t
#ssl_engine device;
 
##守护进程(daemon)。是脱离终端在后台允许的进程。它脱离终端是为了避免进程执行过程中的信息在任何终端上显示。这样一来，进程也不会被任何终端所产生的信息所打断。##
##关闭守护进程的模式，之所以提供这种模式，是为了放便跟踪调试nginx，毕竟用gdb调试进程时最繁琐的就是如何继续跟进fork出的子进程了。##
##如果用off关闭了master_proccess方式，就不会fork出worker子进程来处理请求，而是用master进程自身来处理请求
#daemon off;   #查看是否以守护进程的方式运行Nginx 默认是on 
#master_process off; #是否以master/worker方式工作 默认是on
 
##error日志的设置#
#语法： error_log /path/file level;
#默认： error_log / log/error.log error;
#当path/file 的值为 /dev/null时，这样就不会输出任何日志了，这也是关闭error日志的唯一手段；
#leve的取值范围是debug、info、notice、warn、error、crit、alert、emerg从左至右级别依次增大。
#当level的级别为error时，error、crit、alert、emerg级别的日志就都会输出。大于等于该级别会输出，小于该级别的不会输出。
#如果设定的日志级别是debug，则会输出所有的日志，这一数据量会很大，需要预先确保/path/file所在的磁盘有足够的磁盘空间。级别设定到debug，必须在configure时加入 --with-debug配置项。
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;
 
##pid文件（master进程ID的pid文件存放路径）的路径
pid        logs/nginx.pid;
 
 
events {
 #仅对指定的客户端输出debug级别的日志： 语法：debug_connection[IP|CIDR]
 #这个设置项实际上属于事件类配置，因此必须放在events{……}中才会生效。它的值可以是IP地址或者是CIRD地址。
 	#debug_connection 10.224.66.14;  #或是debug_connection 10.224.57.0/24
 #这样，仅仅以上IP地址的请求才会输出debug级别的日志，其他请求仍然沿用error_log中配置的日志级别。
 #注意：在使用debug_connection前，需确保在执行configure时已经加入了--with-debug参数，否则不会生效。
	worker_connections  1024;
}
 
##核心转储(coredump):在Linux系统中，当进程发生错误或收到信号而终止时，系统会将进程执行时的内存内容(核心映像)写入一个文件(core文件)，以作为调试只用，这就是所谓的核心转储(coredump).
 
http {
##嵌入其他配置文件 语法：include /path/file
#参数既可以是绝对路径也可以是相对路径（相对于Nginx的配置目录，即nginx.conf所在的目录）
    include       mime.types;
    default_type  application/octet-stream;
 
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';
 
    #access_log  logs/access.log  main;
 
    sendfile        on;
    #tcp_nopush     on;
 
    #keepalive_timeout  0;
    keepalive_timeout  65;
 
    #gzip  on;
 
    server {
##listen监听的端口
#语法：listen address:port [ default(deprecated in 0.8.21) | default_server | [ backlog=num | rcvbuf=size | sndbuf=size | accept_filter=filter | deferred | bind | ssl ] ]
#default_server: 如果没有设置这个参数，那么将会以在nginx.conf中找到的第一个server块作为默认server块
				listen       8080;
 
#主机名称：其后可以跟多个主机名称，开始处理一个HTTP请求时，nginx会取出header头中的Host，与每个server中的server_name进行匹配，以此决定到底由那一个server来处理这个请求。有可能一个Host与多个server块中的server_name都匹配，这时会根据匹配优先级来选择实际处理的server块。server_name与Host的匹配优先级见文末。
	 			server_name  localhost;
 
        #charset koi8-r;
 
        #access_log  logs/host.access.log  main;
 
        #location / {
        #    root   html;
        #    index  index.html index.htm;
        #}
 
##location 语法： location [=|~|~*|^~] /uri/ { ... }
# location的使用实例见文末。
#注意：location时有顺序的，当一个请求有可能匹配多个location时，实际上这个请求会被第一个location处理。
        location / {
        		proxy_pass http://192.168.1.60;
        }
 
        #error_page  404              /404.html;
 
        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
 
        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}
 
        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}
 
        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }
 
 
 
    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;
 
    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}
 
 
    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;
 
    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;
 
    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;
 
    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;
 
    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}
 
}
```

