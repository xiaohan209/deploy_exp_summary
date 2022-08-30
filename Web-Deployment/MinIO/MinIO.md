# MinIO

> [官网](https://min.io/)
>
> [旧版官方文档](https://docs.min.io/)
>
> [新官方文档](https://docs.min.io/minio/baremetal/)
>
> [官方下载页面](https://min.io/download)
>
> 参考博客：
>
> - http://www.cloudbin.cn/?p=2917
> - https://www.jianshu.com/p/f39e7255805b
> - https://www.jianshu.com/p/c2b43ff67df0
> - https://www.digitalocean.com/community/tutorials/how-to-set-up-an-object-storage-server-using-minio-on-ubuntu-18-04
> - https://www.cnblogs.com/zaevn00001/p/12877161.html
> - https://www.cnblogs.com/newz/p/12598899.html
> - https://www.cnblogs.com/erdongx/p/11838717.html





## 介绍

对象存储服务OSS（Object Storage Service）是一种海量、安全、低成本、高可靠的云存储服务，**适合存放任意类型的文件**。容量和处理能力弹性扩展，多种存储类型供选择，全面优化存储成本。

MinIO对象存储系统是为海量数据存储、人工智能、大数据分析而设计，基于Apache License v2.0开源协议的对象存储系统，它完全兼容Amazon S3接口，单个对象最大可达5TB，适合存储海量图片、视频、日志文件、备份数据和容器/虚拟机镜像等。

MinIO 是一个云原生应用程序，旨在以可持续的方式在多租户环境中扩展。不仅仅是将单体应用程序改造到基于现代容器的计算环境中，云原生一词围绕着将应用程序部署为微服务的想法，使得云原生应用程序在设计上具有可移植性和弹性，并且微服务可以通过简单的复制进行水平扩展。

MinIO 建立在云原生前提之上，提供纠删码、分布式和共享设置等功能，由于容器提供了隔离的应用程序执行环境，只需为每个用户复制 MinIO 实例并添加隔离的存储环境即可对其进行扩展。

MinIO主要采用Golang语言实现，带有命令行客户端和浏览器界面，并支持[高级消息队列协议 (AMQP)](https://www.digitalocean.com/community/tutorials/an-advanced-message-queuing-protocol-amqp-walkthrough)、[Elasticsearch](https://www.digitalocean.com/community/tutorials/how-to-install-elasticsearch-logstash-and-kibana-elastic-stack-on-ubuntu-18-04)、[Redis](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-redis-on-ubuntu-18-04)、[NATS](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-nats-on-ubuntu-16-04)和[PostgreSQL](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-18-04)目标的简单队列服务，整个系统都运行在操作系统的用户态空间，客户端与存储服务器之间采用http/https通信协议。



### MinIO涉及的概念

对象：类似于hash表中的表项：它的名字相当于关键字，它的内容相当于“值”。

桶：是若干个对象的逻辑抽象，是盛装对象的容器。

租户：用于隔离存储资源。在租户之下可以建立桶、存储对象。

用户：在租户下面创建的用于访问不同桶的账号。可以使用MinIO提供的mc命令设置不用用户访问各个桶的权限。





## 安装部署

> 
>
> [server配置说明](http://docs.minio.org.cn/docs/master/minio-server-configuration-guide)
>
> [普通用户客户端命令行使用说明](https://docs.min.io/docs/minio-client-complete-guide.html)
>
> [管理员客户端命令行使用说明](https://docs.min.io/docs/minio-admin-complete-guide.html)

MinIO为云原生程序打造，因此建议使用docker部署



### Docker部署

1. 

```shell
docker run \
  -p 9000:9000 \
  -p 9001:9001 \
  --name minio1 \
  -v ~/minio/data:/data \
  -e "MINIO_ROOT_USER=AKIAIOSFODNN7EXAMPLE" \
  -e "MINIO_ROOT_PASSWORD=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY" \
  quay.io/minio/minio server /data --console-address ":9001"
  

```



要覆盖 MinIO 的自动生成的密钥，您可以将密钥和访问密钥显式地作为环境变量传递。MinIO 服务器还允许常规字符串作为访问密钥和密钥。



### 二进制文件

#### 服务端部署

1. 安装前置依赖：

   ```shell
   sudo apt-get update
   sudo apt-get install wget
   ```

2. 准备好文件目录，一般手动安装的包放到`/usr/local`下：

   ```shell
   # 新建minio文件夹
   mkdir /usr/local/minio
   # 新建可执行文件
   mkdir /usr/local/minio/bin
   # 新建配置目录
   mkdir /usr/local/minio/etc
   # 新建存储目录
   mkdir /usr/local/minio/data
   ```

3. 新建MinIO管理用户：

   ```shell
   # -r 表示为系统用户
   # -s 设置为nologin使其不能进行登录
   # -U 建立同名组
   sudo useradd -r minio -s /sbin/nologin -U
   ```

4. 下载MinIO二进制文件：

   ```shell
   wget https://dl.min.io/server/minio/release/linux-$(dpkg --print-architecture)/minio
   ```

   可以前往[发布界面](https://dl.min.io/server/minio/release/)查看当前支持的系统架构

5. 将其放入正确文件及并修改权限：

   ```shell
   # 设置minio的可执行权限
   sudo chmod 750 minio
   sudo mv minio /usr/local/minio/bin
   sudo chown -R minio:minio /usr/local/minio
   ```

   

6. 创建MinIO配置文件，一般将其位置设定为`/usr/local/minio/etc/minio.conf`，可以自选其他位置如在`/etc/default/minio`

7. 编辑环境配置文件：

   ```shell
   # 访问 Minio 浏览器界面的初始用户名，修改为自己的用户名即可
   MINIO_ROOT_USER="user"
   # 这将设置您将用于在 Minio 界面中初始用户登录的私钥，修改为自己的密码即可
   MINIO_ROOT_PASSWORD="password"
   # 存储MinIO产生的数据的文件路径，也就是为存储桶而创建的存储目录
   MINIO_VOLUMES="/usr/local/minio/data"
   # -C 之后参数填入应该使用的MinIO配置文件目录
   # --address 之后参数填入监听的IP地址和端口（如果不指定 IP 地址，Minio 会绑定到服务器上配置的每个地址）
   # 一般address端口用9000,console_adrress端口用9001或9090
   MINIO_OPTS="-C /usr/local/minio/etc --address ${your_server_ip}:${port} --console-address ${your_server_ip}:${console_port}"
   ```

   其中`MINIO_ACCESS_KEY`和`MINIO_SECRET_KEY`变量是老版本中对应的`MINIO_ROOT_USER`和`MINIO_ROOT_PASSWORD`

   同时也支持将其设置为文件的形式，使用`MINIO_ROOT_USER_FILE`变量和`MINIO_ROOT_PASSWORD_FILE`变量

   `MINIO_OPTS`中主要填写的是MinIO启动时传入的命令行参数，具体可见[MinIO命令行参数](#MinIO命令行参数)

   

8. 下载Systemd启动服务脚本

   ```shell
   curl -O https://raw.githubusercontent.com/minio/minio-service/master/linux-systemd/minio.service
   # 将其移动到systemd存放服务脚本文件夹中
   sudo mv minio.service /etc/systemd/system
   ```

9. 对上述刚创建的`/etc/systemd/system/minio.service`进行配置：

   ```shell
   [Unit]
   Description=MinIO
   Documentation=https://docs.min.io
   Wants=network-online.target
   After=network-online.target
   # 修改为自己下载的minio二进制文件存放路径
   AssertFileIsExecutable=/usr/local/minio/bin/minio
   
   [Service]
   # 服务的工作目录
   WorkingDirectory=/usr/local/
   
   # 修改为刚才创建的启动MinIO的用户
   User=minio
   # 修改为需要的组，没有特殊情况也与启动MinIO用户名一致
   Group=minio
   ProtectProc=invisible
   
   # 修改为自己创建的MinIO环境配置文件路径
   EnvironmentFile=/usr/local/minio/etc/minio.conf
   ExecStartPre=/bin/bash -c "if [ -z \"${MINIO_VOLUMES}\" ]; then echo \"Variable MINIO_VOLUMES not set in /etc/default/minio\"; exit 1; fi"
   # 修改启动minio程序的路径为二进制文件存放的路径
   ExecStart=/usr/local/minio/bin/minio server $MINIO_OPTS $MINIO_VOLUMES
   
   # Let systemd restart this service always
   Restart=always
   
   # Specifies the maximum file descriptor number that can be opened by this process
   LimitNOFILE=1048576
   
   # Specifies the maximum number of threads this process can create
   TasksMax=infinity
   
   # Disable timeout logic and wait until process is stopped
   TimeoutStopSec=infinity
   SendSIGKILL=no
   
   [Install]
   WantedBy=multi-user.target
   
   # Built for ${project.name}-${project.version} (${project.name})
   ```

   

10. 加载脚本到系统服务中：

    ```shell
    # 重新加载所有systemd单元
    sudo systemctl daemon-reload
    # 启用监控minio单元
    sudo systemctl enable minio
    ```

    

11. 启动服务：

    ```shell
    # 启动服务
    sudo systemctl start minio
    # 可以查看当前minio服务状态
    sudo systemctl status minio
    ```







MinIO Server 带有一个基于 Web 的嵌入式对象浏览器。将您的 Web 浏览器指向 [http://127.0.0.1:9000](http://127.0.0.1:9000/) 以确保您的服务器已成功启动。



> 注意：默认情况下，MinIO 在随机端口上运行控制台，如果您希望选择特定端口，请使用 `--console-address` 来选择特定接口和端口。

MinIO浏览器会将访问的请求转发到控制台

可以配置环境变量`MINIO_BROWSER_REDIRECT_URL`为浏览器转发请求的目的地址

配置环境变量使得浏览器监听想要的IP地址和端口





#### 客户端部署



1. 下载前置依赖：

   ```shell
   sudo apt update
   sudo apt install wget
   ```

2. 下载客户端：

   ```shell
   sudo wget https://dl.minio.io/client/mc/release/linux-$(dpkg --print-architecture)/mc
   ```

3. 将命令行客户端mc变成可执行程序：

   ```shell
   # 添加可执行权限
   sudo chmod +x mc
   sudo mv mc /usr/local/minio/bin
   
   # 编辑用户环境变量
   sudo vim /etc/profile
   # 在/etc/profile文件末尾添加以下内容
   export PATH=/usr/local/minio/bin:$PATH
   ```

   如果不希望放到`/usr/local/minio/bin`中也可以直接放入`/usr/local/bin`中，此时一般情况下不用再次添加环境变量：

   ```shell
   chmod +x mc
   sudo mv mc /usr/local/bin
   ```

4. 测试是否可以使用：

   ```shell
   # 查看客户端版本
   mc -v
   ```

5. 给mc客户端配置基础的别名

   ```shell
   # 以下两条命令可以查看当前有的所有别名
   mc alias list
   mc config host list
   # 修改${your_alias} ${minio_url} ${user} ${password}为自己的设置
   # 对于minio_url根据自身设置的协议、域名或IP地址、端口号来进行填写
   mc alias set ${your_alias} ${minio_url} ${user} ${password} --api "s3v4"
   # 也可以直接使用下方命令，之后手动在命令行中输入登录用户及密码
   mc alias set ${your_alias} ${minio_url}
   ```

6. 配置完成就直接可以用别名进行后续操作了：

   ```shell
   # 创建bucket
   mc mb ${your_alias}/test
   # 查看当前某个服务的所有用户
   mc admin user list ${your_alias}
   # 查看某个服务的所有组
   mc admin group list ${your_alias}
   # 查看某个服务的所有策略
   mc admin policy list ${your_alias}
   # 查看某个服务的具体信息
   mc admin info ${your_alias}
   ```

   具体的详细操作可以看[普通用户客户端命令行使用说明](https://docs.min.io/docs/minio-client-complete-guide.html)和[管理员客户端命令行使用说明](https://docs.min.io/docs/minio-admin-complete-guide.html)，或直接输入`mc --help`查看帮助





## 使用

### MinIO命令行参数

格式为：  

```shell
minio server [FLAGS] DIR1 [DIR2..]
minio server [FLAGS] DIR{1...64}
minio server [FLAGS] DIR{1...64} DIR{65...128}
```

FLAGS主要有：

1. --address：之后填入**一个**监听地址，用作，端口号不填则默认9000
2. --listeners：之后填入整数，表明每一个地址有几个监听进程，默认为1
3. --console-address：之后填入**一个**监听地址，用作管理端即web端的监听，此命令不填则管理口监听所有本机地址并选择动态端口
4. --certs-dir value：
5. --config-dir：可简写为-C，之后填写启动之后的配置文件目录，可能包括证书及json形式的配置文件

例如：

```shell
# 在/home/shared目录上启动单节点MinIO服务
minio server /home/shared
# 启动单节点MinIO服务，有"/mnt/data1"到"/mnt/data64"64个目录
minio server /mnt/data{1...64}
# Start distributed MinIO服务在32 node setup with 32 drives each, run following command on all the nodes
minio server http://node{1...32}.example.com/mnt/export{1...32}
# Start distributed minio server in an expanded setup, run the following command on all the nodese超过
minio server http://node{1...16}.example.com/mnt/export{1...32} \
            http://node{17...64}.example.com/mnt/export{1...64}

```





### 用户管理

> [官方用户管理说明](https://docs.min.io/minio/baremetal/security/minio-identity-management/user-management.html)

