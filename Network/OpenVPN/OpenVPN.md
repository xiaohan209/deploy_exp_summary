# OpenVPN

>[官方介绍网站](https://openvpn.net/)
>
>[OpenVPN老版本内核仓库地址](https://github.com/OpenVPN/openvpn)
>
>[官方社区下载页面](https://openvpn.net/index.php/download/community-downloads.html)
>
>[OpenVPN老版本指南](https://openvpn.net/community-resources/how-to)
>
>[OpenVPN完整 CLI 指令说明](https://build.openvpn.net/man/openvpn-2.5/openvpn.8.html)
>
>
>
>[OpenVPN3内核仓库地址](https://github.com/OpenVPN/openvpn3/)
>
>[OpenVPN3 linux客户端仓库地址](https://github.com/OpenVPN/openvpn3-linux)
>
>[OpenVPN Access Server下载说明](https://openvpn.net/download-open-vpn/)
>
>[OpenVPN3 linux客户端说明](https://openvpn.net/cloud-docs/openvpn-3-client-for-linux/)
>
>[OpenVPN3 linux客户端wiki页面](https://community.openvpn.net/openvpn/wiki/OpenVPN3Linux)
>
>
>
>参考博客：
>
>- http://alanhou.org/openvpn-notes/
>- https://www.bianchengquan.com/article/645358.html
>- http://www.gimoo.net/t/1804/5adaa93263e14.html

## 介绍

**OpenVPN** 是 **James Yonan** 的开源 VPN 守护程序，旨在提供 IPSec 的许多关键功能，现在已经成为了一个通用的 VPN 工具，具有高度灵活性，且占用空间较小。

OpenVPN 支持 SSL/TLS 安全、以太网桥接、通过代理或 NAT 的 TCP 或 UDP 隧道传输、对动态 IP 地址和 DHCP 的支持、对成百上千用户的可扩展性以及对大多数主要操作系统平台的可移植性。

OpenVPN 与 OpenSSL 库紧密绑定，并从中派生出大部分加密功能。

OpenVPN 支持使用客户端和服务器证书使用预共享密钥 **（静态密钥模式）**或公钥安全性**（SSL/TLS 模式）**的传统加密，以及非加密 TCP/UDP 隧道。

OpenVPN 旨在与大多数平台上存在的**TUN/TAP虚拟网络接口**一起使用。





[OpenVPN 3 Linux 项目](https://github.com/OpenVPN/openvpn3-linux/)是建立在[OpenVPN 3 核心库](https://github.com/OpenVPN/openvpn3/)上的新客户端，它也用于各种 OpenVPN Connect 客户端和 Android 版 OpenVPN（需要通过应用程序中的设置页面启用）。

[OpenVPN 网站](https://openvpn.net/)上有更多文档和示例

## 安装部署

操作系统：ubuntu 20.04 focal

请根据需要的版本安装OpenVPN，有[OpenVPN2](#OpenVPN2)以及[OpenVPN3](#OpenVPN3)



### OpenVPN2

#### APT软件包+官方存储库



OPENVPN 2

OpenVPN 2.2版本及以上有Debian和RPM软件包提供，可以去官方软件包下载说明中找到

[官方存储库说明](https://community.openvpn.net/openvpn/wiki/OpenvpnSoftwareRepos)中使用官方存储库与APT软件包管理进行安装的过程如下：

1. 安装前置的一些依赖：

   ```shell
   sudo apt-get update && sudo apt-get install wget
   ```

2. 添加gpg key：

   ```shell
   # swupdate.openvpn.net网站有可能国内访问不了，可以开代理，目前没有找到国内镜像源
   wget -O - https://swupdate.openvpn.net/repos/repo-public.gpg | sudo apt-key add -
   ```

3. 添加存储库：

   ```shell
   # 源应该为deb http://build.openvpn.net/debian/openvpn/${version} $(lsb_release -cs) main
   # 其中${version}以stable为例
   echo "deb http://build.openvpn.net/debian/openvpn/stable $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/openvpn-aptrepo.list > /dev/null
   ```

   `${version}`的值说明：

   - **stable**: 稳定版
   - **testing**: 最新发布的测试版，包括alphas/betas/RCs
   - **release/2.3**: 以2.3开头的所有版本
   - **release/2.4**: 以2.4开头的所有版本
   - **release/2.5**: 以2.5开头的所有版本

4. 安装：

   ```shell
   sudo apt-get update && apt-get install openvpn
   ```

   

#### 源代码编译

1. 

2. 安装依赖：

   1. lzo：

      ```shell
      #安装lzo压缩模块
      wget http://www.oberhumer.com/opensource/lzo/download/lzo-2.06.tar.gz
      tar zxf lzo-2.06.tar.gz
      cd lzo-2.06
      ./configure
      make
      make install
      cd ../
      ```

   2. openssl：

      ```shell
      # 安装OpenSSL
      yum install openssl* -y
      ```

      

3. 登录[官方下载页面](https://openvpn.net/index.php/download/community-downloads.html)查看需要的版本

4. 下载文件：

   ```shell
   # 以gzip的压缩包为例，将$<version>替换为需要的版本即可
   wget https://swupdate.openvpn.org/community/releases/openvpn-$<version>.tar.gz -O - | sudo tar -zxv
   # 或者直接git方式下载源代码
   git clone git@github.com:OpenVPN/openvpn.git
   ```

5. 编译并安装：

   ```shell
   cd openvpn-2.5.5
   sudo ./configure
   make
   make install
   
   ./configure --with-lzo-headers=/usr/local/include --with-lzo-lib=/usr/local/lib
   make
   make install
   cd ../
   ```

   

6. a

7. a









VPN时间同步

```shell
#手动执行
/usr/sbin/ntpdate pool.ntp.org (ntpdate未安装请自行安装)
#设置定时任务自动执行
echo '#sync time' >> /var/spool/cron/root
echo '*/5 * * * * /usr/sbin/ntpdate pool.ntp.org > /dev/null 2>&1' >> /var/spool/cron/root
```



签名:

```shell
wget -O security-openvpn-net.asc https://keys.openpgp.org/vks/v1/by-fingerprint/F554A3687412CFFEBDEFE0A312F5F7B42F2B01E7
gpg --import security-openvpn-net.asc
# 下载开源文件及其签名，放在同一位置，运行验证
gpg [.asc 文件]
```



### OpenVPN3 Linux



1. 安装依赖：

   ```shell
   sudo apt update && sudo apt install apt-transport-https curl
   ```

2. 获取官方gpg key：

   ```shell
   curl -fsSL https://swupdate.openvpn.net/repos/openvpn-repo-pkg-key.pub | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/openvpn-repo-pkg-keyring.gpg
   ```

3. 获取官方存储库：

   ```shell
   sudo echo "https://swupdate.openvpn.net/community/openvpn3/repos/openvpn3-$(lsb_release -cs).list" | xargs curl -fsSL | sudo tee /etc/apt/sources.list.d/openvpn3.list > /dev/null
   ```

4. 更新存储库并下载openvpn3：

   ```shell
   sudo apt update && sudo apt install openvpn3
   ```

   

### OpenVPN Access Server









## 使用

1. 登录openvpn主页，注册账户
2. 注册后根据提示在官网中选择免费版进行订阅
3. 获得订阅版的