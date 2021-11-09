# Docker安装

#### [官网链接][https://docs.docker.com/]

#### 主机：ubuntu 20.04 focal

### 使用docker存储库+清华源

1. 卸载原来docker

   ```shell
   sudo apt-get remove docker docker-engine docker.io containerd runc
   sudo rm -rf /var/lib/docker
   sudo rm -rf /var/lib/containerd
   ```

   然后还需要手动卸载原来配置文件（如果有的话）

2. 更新apt && 安装https相关组件
   ```shell
    sudo apt-get update
    sudo apt-get install \
       ca-certificates \
       curl \
       gnupg \
       lsb-release
   ```

3. 添加清华源GPG密钥

   ```shell
   curl -fsSL https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
   ```

4. 设置稳定存储库

   ```shell
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu \
     $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   ```

5. 安装

   1. 最新版

      ```shell
      sudo apt-get update
      sudo apt-get install docker-ce docker-ce-cli containerd.io
      ```

   2. 特定版本

      ```shell
      # 查看有哪些版本
      apt-cache madison docker-ce
      # 挑选一个安装即可
      sudo apt-get install docker-ce=<VERSION_STRING> docker-ce-cli=<VERSION_STRING> containerd.io
      ```

6. 来！试试看！

   ```shell
   sudo docker run hello-world
   ```

   可能的报错：

   ```sh
   WARNING: Error loading config file: /home/user/.docker/config.json -
   stat /home/user/.docker/config.json: permission denied
   ```

   解决：直接`rm -rf ~/.docker/`，或者：

   ```shell
   sudo chown "$USER":"$USER" /home/"$USER"/.docker -R
   sudo chmod g+rwx "$HOME/.docker" -R
   ```

# docker管理

### 非Root用户管理Docker

Docker的守护进程默认绑定到Unix socket上而不是TCP端口，所以只能Root身份来管理，使用的时候如果不想在命令前面都加sudo，就需要创建一个Unix组叫`docker`并向其中添加用户。使用的时候Docker就会创建一个由这个组的成员都可以访问的Unix Socket。

1. 创建`docker`组

   ```shell
   sudo groupadd docker
   ```

2. 将用户添加进`docker`组

   ```shell
   sudo usermod -aG docker $USER
   ```

3. 重新登录或重启，也可以直接激活组更改，输入：

   ```shell
   newgrp docker 
   ```

4. 检查

   ```shell
   docker run hello-world
   ```

### docker开机启动

启用`enable`

停用直接`disable`

```shell
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

	### 配置 Docker 守护进程在哪里监听连接

默认情况下，Docker守护进程侦听Unix Socket，要Docker接受来自远程主机的请求需要配置为侦听 IP 地址和端口以及 UNIX 套接字。更多详细信息，请查看[Docker CLI 参考](https://docs.docker.com/engine/reference/commandline/dockerd/)文章的“将[Docker](https://docs.docker.com/engine/reference/commandline/dockerd/)绑定到另一个主机/端口或 unix 套接字”部分。

将Docker 配置为同时使用`systemd`单元文件和`daemon.json` 文件来侦听连接会导致冲突，从而阻止 Docker 启动。

### 使用`systemd`单元文件配置远程访问

1. 使用该命令在文本编辑器中`sudo systemctl edit docker.service`打开覆盖文件`docker.service`。

2. 添加或修改以下几行，替换您自己的值。

   ```systemd
   [Service]
   ExecStart=
   ExecStart=/usr/bin/dockerd -H fd:// -H tcp://127.0.0.1:2375
   ```

3. 保存文件。

4. 重新加载`systemctl`配置。

   ```
    sudo systemctl daemon-reload
   ```

5. 重启 Docker。

   ```
   sudo systemctl restart docker.service
   ```

6. 通过查看`netstat`以确认`dockerd`正在侦听配置的端口的输出来检查更改是否得到遵守。

   ```
   sudo netstat -lntp | grep dockerd
   ```

### 使用配置远程访问`daemon.json`

1. 设置连接到UNIX套接字的`hosts`数组`/etc/docker/daemon.json`和IP地址，如下：

   ```
   {
     "hosts": ["unix:///var/run/docker.sock", "tcp://127.0.0.1:2375"]
   }
   ```

2. 重启 Docker。

3. 通过查看`netstat`以确认`dockerd`正在侦听配置的端口的输出来检查更改是否得到遵守。

   ```
   sudo netstat -lntp | grep dockerd
   ```

# 常见错误

## `Cannot connect to the Docker daemon`

如果您看到如下错误，则您的 Docker 客户端可能被配置为连接到不同主机上的 Docker 守护程序，而该主机可能无法访问。

```none
Cannot connect to the Docker daemon. Is 'docker daemon' running on this host?
```

要查看您的客户端配置为连接到哪个主机，请检查`DOCKER_HOST`您环境中变量的值。

```
$ env | grep DOCKER_HOST
```

如果此命令返回一个值，则 Docker 客户端将设置为连接到在该主机上运行的 Docker 守护程序。如果未设置，则 Docker 客户端将设置为连接到在本地主机上运行的 Docker 守护程序。如果设置错误，请使用以下命令取消设置：

```
$ unset DOCKER_HOST
```

您可能需要在文件中编辑环境，例如`~/.bashrc`或 `~/.profile`以防止`DOCKER_HOST`错误设置变量。

如果`DOCKER_HOST`按预期设置，请验证 Docker 守护程序是否正在远程主机上运行，并且防火墙或网络中断未阻止您进行连接。

### IP转发问题

如果使用手动配置你的网络`systemd-network`有`systemd` 219或更高版本，Docker容器可能无法访问您的网络。从`systemd`版本 220开始，给定网络 ( `net.ipv4.conf.<interface>.forwarding`)的转发设置默认为*off*。此设置可防止 IP 转发。它还与 Docker`net.ipv4.conf.all.forwarding`在容器内启用设置的行为相冲突。

要在 RHEL、CentOS 或 Fedora 上解决此问题，请`<interface>.network` 在`/usr/lib/systemd/network/`Docker 主机（例如：）上编辑文件`/usr/lib/systemd/network/80-container-host0.network`并在该`[Network]`部分中添加以下块。

```systemd
[Network]
...
IPForward=kernel
# OR
IPForward=true
```

此配置允许按预期从容器转发 IP。

## `DNS resolver found in resolv.conf and containers can't use it`

使用 GUI 的 Linux 系统通常运行一个网络管理器，它使用`dnsmasq`在环回地址（例如`127.0.0.1`或 ） 上运行的 实例`127.0.1.1`来缓存 DNS 请求，并将此条目添加到 `/etc/resolv.conf`. 该`dnsmasq`服务可加快 DNS 查找速度并提供 DHCP 服务。此配置不拥有自己的网络命名空间的码头工人容器内工作，因为多克尔容器做出决议回环如地址`127.0.0.1`到 **自身**，这是很不可能的运行在自己的回送地址的DNS服务器。

如果泊坞窗检测中引用没有DNS服务器`/etc/resolv.conf`是一个全功能的DNS服务器，下面的警告出现和码头工人使用由谷歌在提供的公共DNS服务器`8.8.8.8`和`8.8.4.4`DNS解析。

```none
WARNING: Local (127.0.0.1) DNS resolver found in resolv.conf and containers
can't use it. Using default external servers : [8.8.8.8 8.8.4.4]
```

如果您看到此警告，请首先检查您是否使用`dnsmasq`：

```
$ ps aux |grep dnsmasq
```

如果您的容器需要解析网络内部的主机，则公共名称服务器是不够的。你有两个选择：

- 您可以指定一个 DNS 服务器供 Docker 使用，**或者**
- 您可以`dnsmasq`在 NetworkManager 中禁用。如果您这样做，NetworkManager 会将您真正的 DNS 名称服务器添加到`/etc/resolv.conf`，但您将失去`dnsmasq`.

**您只需要使用这些方法之一。**

### 为 Docker 指定 DNS 服务器

配置文件的默认位置是`/etc/docker/daemon.json`. 您可以使用`--config-file` 守护程序标志更改配置文件的位置。下面的文档假设配置文件位于`/etc/docker/daemon.json`.

1. 创建或编辑Docker守护进程配置文件，默认为 `/etc/docker/daemon.json`file，控制Docker守护进程配置。

   ```
   $ sudo nano /etc/docker/daemon.json
   ```

2. 添加`dns`具有一个或多个 IP 地址作为值的键。如果文件已有内容，您只需添加或编辑该`dns`行。

   ```
   {
     "dns": ["8.8.8.8", "8.8.4.4"]
   }
   ```

   如果您的内部 DNS 服务器无法解析公共 IP 地址，请至少包含一个可以解析的 DNS 服务器，以便您可以连接到 Docker Hub 并且您的容器可以解析 Internet 域名。

   保存并关闭文件。

3. 重新启动 Docker 守护进程。

   ```
   $ sudo service docker restart
   ```

4. 通过尝试拉取镜像来验证 Docker 是否可以解析外部 IP 地址：

   ```
   $ docker pull hello-world
   ```

5. 如有必要，请验证 Docker 容器是否可以通过 ping 来解析内部主机名。

   ```
   $ docker run --rm -it alpine ping -c4 <my_internal_host>
   
   PING google.com (192.168.1.2): 56 data bytes
   64 bytes from 192.168.1.2: seq=0 ttl=41 time=7.597 ms
   64 bytes from 192.168.1.2: seq=1 ttl=41 time=7.635 ms
   64 bytes from 192.168.1.2: seq=2 ttl=41 time=7.660 ms
   64 bytes from 192.168.1.2: seq=3 ttl=41 time=7.677 ms
   ```

#### 禁用 `dnsmasq`

##### Ubuntu

如果您不想更改 Docker 守护程序的配置以使用特定 IP 地址，请按照这些说明`dnsmasq`在 NetworkManager 中禁用。

1. 编辑`/etc/NetworkManager/NetworkManager.conf`文件。

2. 通过`dns=dnsmasq`在行`#`的开头添加一个字符来注释掉该行。

   ```none
   # dns=dnsmasq
   ```

   保存并关闭文件。

3. 重新启动 NetworkManager 和 Docker。作为替代方案，您可以重新启动系统。

   ```
   $ sudo systemctl restart network-manager
   $ sudo systemctl restart docker
   ```

##### RHEL、CentOS 或 Fedora

要`dnsmasq`在 RHEL、CentOS 或 Fedora 上禁用：

1. 禁用`dnsmasq`服务：

   ```
   $ sudo systemctl stop dnsmasq
   $ sudo systemctl disable dnsmasq
   ```

2. 使用[Red Hat 文档](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Deployment_Guide/s1-networkscripts-interfaces.html)手动配置 DNS 服务器 。

### 允许通过防火墙访问远程 API 

如果您在运行 Docker 的同一主机上运行防火墙，并且希望从另一台主机访问 Docker Remote API 并且启用了远程访问，则需要配置防火墙以允许 Docker 端口上的传入连接，默认为`2376`if TLS 加密传输已启用或`2375` 以其他方式启用。

两种常见的防火墙守护进程是 [UFW（Uncomplicated Firewall）](https://help.ubuntu.com/community/UFW)（常用于 Ubuntu 系统）和[firewalld](https://firewalld.org/)（常用于基于 RPM 的系统）。请参阅您的操作系统和防火墙的文档，但以下信息可能会帮助您入门。这些选项相当宽松，您可能希望使用不同的配置来更多地锁定您的系统。

- **UFW**：`DEFAULT_FORWARD_POLICY="ACCEPT"`在您的配置中设置。

- **firewalld**：将类似于以下内容的规则添加到您的策略中（一种用于传入请求，一种用于传出请求）。确保接口名称和链名称正确。

  ```
  <direct>
    [ <rule ipv="ipv6" table="filter" chain="FORWARD_direct" priority="0"> -i zt0 -j ACCEPT </rule> ]
    [ <rule ipv="ipv6" table="filter" chain="FORWARD_direct" priority="0"> -o zt0 -j ACCEPT </rule> ]
  </direct>
  ```

## `Your kernel does not support cgroup swap limit capabilities`

在 Ubuntu 或 Debian 主机上，使用映像时，您可能会看到类似于以下内容的消息。

```none
WARNING: Your kernel does not support swap limit capabilities. Limitation discarded.
```

在基于 RPM 的系统上不会出现此警告，默认情况下会启用这些功能。

如果您不需要这些功能，则可以忽略警告。您可以按照这些说明在 Ubuntu 或 Debian 上启用这些功能。即使 Docker 没有运行，内存和交换计算也会产生大约 1% 的总可用内存开销和 10% 的整体性能下降。

1. 以具有`sudo`特权的用户身份登录 Ubuntu 或 Debian 主机。

2. 编辑`/etc/default/grub`文件。添加或编辑该`GRUB_CMDLINE_LINUX`行以添加以下两个键值对：

   ```none
   GRUB_CMDLINE_LINUX="cgroup_enable=memory swapaccount=1"
   ```

   保存并关闭文件。

3. 更新 GRUB。

   ```
   $ sudo update-grub
   ```

   如果您的 GRUB 配置文件的语法不正确，则会发生错误。在这种情况下，请重复步骤 2 和 3。

   更改在系统重新启动时生效。
