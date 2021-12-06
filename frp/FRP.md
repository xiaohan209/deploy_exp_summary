# FRP

> [FRP仓库地址](https://github.com/fatedier/frp)
>
> [FRP官方文档](https://gofrp.org/docs/)
>
> 参考教程：
>
> - <https://www.appinn.com/frp/>
> - https://www.cnblogs.com/shook/p/12790532.html
> - https://www.cnblogs.com/sanduzxcvbnm/p/8509150.html
> - https://www.cnblogs.com/chywx/p/10939966.html



## 配置

当前系统版本：linux ubuntu-20.04 focal

FRP版本：0.38.0

1. 保证有下载文件和解压的工具链

   ```shell
   sudo apt update
   sudo apt install tar wget dpkg
   ```

   

2. 下载release文件
   随便打开一个目录，以用户根目录`~`为例，在目录下下载文件，建议先打开release网站查看适合本机的版本后复制连接下载，例如：

   ```shell
   # 进入用户目录
   cd ~/
   # 下载并解压
   wget -c https://github.com/fatedier/frp/releases/download/v0.38.0/frp_0.38.0_linux_$(dpkg --print-architecture).tar.gz -O - | tar -xz
   ```

   

3. 修改`frpc.ini`或`frps.ini`

   1. frpc.ini：

      ```shell
      [common]											#指定common不能变，代表通用配置
      server_addr = 6.6.6.6       	#要连接的远程服务器地址
      server_port = 7000            #远程服务器的frps.ini配置的可连接端口
      
      [ssh]													#随便一个名字，但不能与其他要连到服务器的frpc.ini相同
      type = tcp										#要转发的协议类型，可以是tcp,udp
      local_ip = 127.0.0.1					#客户端机器目前子网可以达到的某个主机A
      local_port = 22								#连接需要转发到A的端口
      remote_port = 6000						#与服务主机通信的端口
      ```

      最详细配置参考：<https://github.com/fatedier/frp/blob/dev/conf/frpc_full.ini>

   2. frps.ini：

      ```shell
      [common]											#指定common字段不能变，代表通用配置
      bind_port = 7000							#服务器开放的端口
      ```

      最详细的配置参考：<https://github.com/fatedier/frp/blob/dev/conf/frps_full.ini>

   比较常用的几种情况[FRP仓库地址](https://github.com/fatedier/frp)的README已经列出来了，直接查看即可

4. 加入service方便管理FRP功能

   ```shell
   # 可以根据本机为客户端or服务端选择frpc或frps加入
   
   # 在/etc/下新建frp文件夹
   sudo mkdir -p /etc/frp/
   
   # 将可执行文件加入/usr/bin
   sudo cp frpc /usr/bin/
   sudo cp frps /usr/bin/
   
   # 将修改好的ini文件
   sudo cp frpc.ini /etc/frp/
   sudo cp frps.ini /etc/frp/
   
   # 将service文件加入
   sudo cp systemd/frpc.service /etc/systemd/system/
   sudo cp systemd/frps.service /etc/systemd/system/
   ```

5. 开机自动启动

   ```shell
   # 自动启动frpc客户端
   systemctl enable frpc
   # 自动启动frps服务器端
   systemctl enable frps
   ```

   如果不想重启也可以直接启动服务，有两种方法，以frpc为例：

   1. ```shell
      systemctl start frpc
      ```

   2. ```shell
      service frpc start
      ```

   其中重启可以采用`service frpc restart`命令，查看服务状态可以采用`service frpc status`命令

6. 启动成功

   + 注意应该现在服务端主机启用frps后，再在客户端主机启用frpc，否则可能连接不上
   + FRP连接上以后如果服务器端服务需要重启，则客户端frpc会尝试进行tcp重连，因此不需要再重启客户端（当然进行重启客户端服务也没有任何问题）

## 使用

### FRP配置解释

当需要内网穿透时，一般先使用一个具有公网IP的主机S当做服务器，假设它的IP为`server_ip`，并且在frps.ini中开放端口`Server_port`供客户端连接。

对需要穿透的内网中的所有主机可以分别在所有主机上配置frpc，将自己ip以及端口暴露给主机S，也可以利用一台主机将内网所有需要穿透的主机及其端口配置到主机A中去。

配置中每一项都有`local_ip`，`local_port`，`remote_port`，后文对其使用时以`${x}`的形式表述



### ssh命令

继续沿用上面的例子，使用以下两种命令都可以：

1. ```shell
   # ssh -p ${remote_port} username@${server_ip}
   ssh -p 6000 username@6.6.6.6
   ```

2. ```shell
   # ssh -o Port=${remote_port} username@${server_ip}
   ssh -o Port=6000 username@6.6.6.6
   ```

   

输入密码之后即可进入内网主机终端，然后就可以为所欲为了~