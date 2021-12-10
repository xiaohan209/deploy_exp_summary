# LNMP

## 介绍

LNMP = Linux + Nginx + Mysql + PHP，是网站服务器的一种应用架构

LNMP结构里php会启动php-fpm服务，Nginx会把用户的动态请求交给php服务去处理，php服务就会去和数据库进行交互；用户的静态请求Nginx会直接处理。Nginx处理静态请求的速度要比Apache快很多性能上要好，承受的并发量要比Apache大，可以承受好几万的并发量。

## 安装

> - https://help.aliyun.com/document_detail/97251.html
> - https://blog.51cto.com/u_13525470/2117048
> - https://lnmp.org/install.html
> - https://lnmp.org/

### 一键安装包

#### 简介

一键安装包是一个用Linux Shell编写的可以为CentOS/RHEL/Fedora/Aliyun/Amazon、Debian/Ubuntu/Raspbian/Deepin/Mint Linux VPS或独立主机安装LNMP(Nginx/MySQL/PHP)、LNMPA(Nginx/MySQL/PHP/Apache)、LAMP(Apache/MySQL/PHP)生产环境的Shell程序。

它无需一个一个的输入命令进行编译安装，无需值守。它可以在编译安装时优化编译参数，提高性能，解决不必要的软件间依赖，特别针对配置自动优化。

#### 软硬件安装要求

- CentOS/RHEL/Fedora/Debian/Ubuntu/Raspbian/Deepin/Aliyun/Amazon/Mint Linux发行版
- 需要5GB以上硬盘剩余空间，MySQL 5.7或MariaDB 10都至少需要9GB剩余空间
- 需要128MB以上内存(128MB小内存VPS,Xen需有SWAP,OpenVZ至少要有128MB以上的vSWAP或突发内存)，注意小内存请勿使用64位系统！
- **安装MySQL 5.6或5.7及MariaDB 10必须1G以上内存，更高版本至少要2G内存!**
- **安装PHP 7及以上版本必须1G以上内存!**
- VPS或服务器必须设置好可用的yum或apt-get源并确保能正常工作，[离线安装](https://lnmp.org/install.html#offline)需要增加 CheckMirror=n 参数！
- Linux下区分大小写，输入命令时请注意！
- 如有通过yum或apt-get安装的MySQL/MariaDB请自行备份数据等相关文件！
- CentOS 5、6,Debian 6及之前版本其官网已经结束支持无法直接使用，需自行更换vault或archive源！
- CentOS 6请用lnmp 1.8+版本进行安装！
- Ubuntu 18+,Debian 9+,Mint 19+,Deepin 15.7+及所有新的Linux发行版只能使用1.7+进行安装！
- PHP 7.1.*以下版本不支持Ubuntu 19+、Debian 10等等非常新的Linux发行版！
- [阿里云](https://www.vpser.net/go/aliyun)Ubuntu 14.04系统模版有问题不要用！！！
- PHP 7.4升级或安装必须CentOS 7+,Debian 8+,Ubuntu 16.04+且必须使用1.7+！！！
- MySQL 8.0.23以下版本升级或安装必须CentOS 8+,Debian 9+,Ubuntu 16.04+且必须使用1.7+！！！
- MySQL 8.0.24以上版本升级或安装必须Debian 11+,Ubuntu 20.04+,Fedora 33+且必须使用1.8！！！

#### 使用方式

**操作系统：ubuntu20.04 focal**

**lnmp version：1.8**

1. 安装系统工具

   ```sh
   sudo apt-get install wget tar
   ```

2. 下载：

   1. 先下载脚本后运行脚本下载：

      ```sh
      wget http://soft.vpser.net/lnmp/lnmp1.8.tar.gz -O - | tar -xzv
      ```

   2. 完整版包含二进制文件下载：

      ```sh
      wget http://soft.vpser.net/lnmp/lnmp1.8-full.tar.gz -O - | tar -xzv
      ```

3. 对安装包进行配置（不需要可以跳过）：在解压缩文件夹中编辑`lnmp.conf`

   |       参数名称        |           参数介绍            |                             例子                             |
   | :-------------------: | :---------------------------: | :----------------------------------------------------------: |
   |    Download_Mirror    |           下载镜像            | 一般默认，如异常可[修改下载镜像](https://lnmp.org/faq/download-url.html) |
   | Nginx_Modules_Options |  添加Nginx模块或其他编译参数  |                —add-module=第三方模块源码目录                |
   |  PHP_Modules_Options  |     添加PHP模块或编译参数     |           —enable-exif 有些模块需提前安装好依赖包            |
   |    MySQL_Data_Dir     |      MySQL数据库目录设置      |                   默认/usr/local/mysql/var                   |
   |   MariaDB_Data_Dir    |     MariaDB数据库目录设置     |                  默认/usr/local/mariadb/var                  |
   |  Default_Website_Dir  |   默认虚拟主机网站目录位置    |                  默认/home/wwwroot/default                   |
   | Enable_Nginx_Openssl  |   Nginx是否使用新版openssl    |           默认 y，建议不修改，y是启用并开启到http2           |
   |  Enable_PHP_Fileinfo  | 是否安装开启php的fileinfo模块 |         默认n，根据自己情况而定，安装启用的话改成 y          |
   |   Enable_Nginx_Lua    |    是否为Nginx安装lua支持     |       默认n，安装lua可以使用一些基于lua的waf网站防火墙       |

   [下载节点](https://lnmp.org/faq/download-url.html)：

   1. 又拍云国内外CDN：http://soft.vpszt.com
   2. 国内下载节点：http://upyun.vpser.net、http://soft1.vpser.net、http://soft3.vpser.net
   3. 国外下载节点：http://soft2.vpser.net、http://soft4.vpser.net、http://soft6.vpser.net

4. 安装所需要的软件：

   ```sh
   # 打开解压缩后的lnmp文件夹
   cd lnmp1.8
   # install.sh可以输入各种参数，如lnmpa、lnmp、lamp，从1.5开始也支持nginx、db、mphp参数安装
   ./install.sh lnmp
   ```

   安装后的大部分软件都在`/usr/local`下，例如nginx的各文件位于`/usr/local/nginx/`下

5. 根据后续提示在命令行输入参数即可，其中版本以及其他组件的安装可以参见[安装手册](https://lnmp.org/install.html)

6. 对已有的软件进行更新以及其他组件下载，以及已下载的软件目录等信息参见[配置手册](https://lnmp.org/faq/lnmp-software-list.html)



#### TIPS

安装后自动部署进`/etc/systemd/system`当中的service，可以直接用service进行启动，即`service nginx start`来启动nginx

安装后进行[管理](https://lnmp.org/faq/lnmp-status-manager.html)：

1. 对lnmp整体进行管理

   ```sh
   lnmp {start|stop|reload|restart|kill|status}
   ```

2. 对各个程序进行管理

   ```sh
   lnmp {nginx|mysql|mariadb|php-fpm|pureftpd} {start|stop|reload|restart|kill|status}
   ```

3. 多PHP版本状态管理

   ```sh
   # 5.5为版本号，请根据实际替换
   /etc/init.d/php-fpm5.5 {start|stop|quit|restart|reload|logrotate} 
   ```

### 单独安装

1. Nginx
2. Mysql 或 MariaDB
3. PHP