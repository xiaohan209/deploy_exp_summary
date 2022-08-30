# apt-get相关

###### 系统：ubuntu 20.04 focal

## 换源

```shell
# 保存原来源
sudo cp /etc/apt/sources.list /etc/apt/sources.list.origin.save
# 进入保存源的文件，修改为需要的源信息
sudo vim /etc/apt/sources/list
```

### 清华源

#### x86

```sh
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse
```

#### ARM

```sh
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ focal main restricted universe multiverse
# deb-src https://https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ focal main restricted universe multiverse
deb https://https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ focal-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ focal-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ focal-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ focal-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ focal-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ focal-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ focal-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ focal-proposed main restricted universe multiverse
```

### 阿里源

#### x86

```sh
deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse 
deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse 
deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse 
deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse 
deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse 
#deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse 
#deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse 
#deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse 
#deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse 
#deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse focal
```

#### ARM

```sh
deb http://mirrors.aliyun.com/ubuntu-ports/ focal main restricted universe multiverse 
deb http://mirrors.aliyun.com/ubuntu-ports/ focal-security main restricted universe multiverse 
deb http://mirrors.aliyun.com/ubuntu-ports/ focal-updates main restricted universe multiverse 
deb http://mirrors.aliyun.com/ubuntu-ports/ focal-proposed main restricted universe multiverse 
deb http://mirrors.aliyun.com/ubuntu-ports/ focal-backports main restricted universe multiverse 
#deb-src http://mirrors.aliyun.com/ubuntu-ports/ focal main restricted universe multiverse 
#deb-src http://mirrors.aliyun.com/ubuntu-ports/ focal-security main restricted universe multiverse 
#deb-src http://mirrors.aliyun.com/ubuntu-ports/ focal-updates main restricted universe multiverse 
#deb-src http://mirrors.aliyun.com/ubuntu-ports/ focal-proposed main restricted universe multiverse 
#deb-src http://mirrors.aliyun.com/ubuntu-ports/ focal-backports main restricted universe multiverse focal
```

### 中科大源

#### x86

```sh
deb https://mirrors.ustc.edu.cn/ubuntu/ focal main restricted universe multiverse
#deb-src https://mirrors.ustc.edu.cn/ubuntu/ focal main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
#deb-src https://mirrors.ustc.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
#deb-src https://mirrors.ustc.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ focal-security main restricted universe multiverse
#deb-src https://mirrors.ustc.edu.cn/ubuntu/ focal-security main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse
#deb-src https://mirrors.ustc.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse
```

#### ARM

```sh
deb https://mirrors.ustc.edu.cn/ubuntu-ports/ focal main restricted universe multiverse
#deb-src https://mirrors.ustc.edu.cn/ubuntu-ports/ focal main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu-ports/ focal-updates main restricted universe multiverse
#deb-src https://mirrors.ustc.edu.cn/ubuntu-ports/ focal-updates main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu-ports/ focal-backports main restricted universe multiverse
#deb-src https://mirrors.ustc.edu.cn/ubuntu-ports/ focal-backports main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu-ports/ focal-security main restricted universe multiverse
#deb-src https://mirrors.ustc.edu.cn/ubuntu-ports/ focal-security main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu-ports/ focal-proposed main restricted universe multiverse
#deb-src https://mirrors.ustc.edu.cn/ubuntu-ports/ focal-proposed main restricted universe multiverse
```

### 网易源

#### x86

```sh
deb http://mirrors.163.com/ubuntu/ focal main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ focal-security main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ focal-updates main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ focal-proposed main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ focal-backports main restricted universe multiverse
#deb-src http://mirrors.163.com/ubuntu/ focal main restricted universe multiverse
#deb-src http://mirrors.163.com/ubuntu/ focal-security main restricted universe multiverse
#deb-src http://mirrors.163.com/ubuntu/ focal-updates main restricted universe multiverse
#deb-src http://mirrors.163.com/ubuntu/ focal-proposed main restricted universe multiverse
#deb-src http://mirrors.163.com/ubuntu/ focal-backports main restricted universe multiverse
```

#### ARM

```sh
deb http://mirrors.163.com/ubuntu-ports/ focal main restricted universe multiverse
deb http://mirrors.163.com/ubuntu-ports/ focal-security main restricted universe multiverse
deb http://mirrors.163.com/ubuntu-ports/ focal-updates main restricted universe multiverse
deb http://mirrors.163.com/ubuntu-ports/ focal-proposed main restricted universe multiverse
deb http://mirrors.163.com/ubuntu-ports/ focal-backports main restricted universe multiverse
#deb-src http://mirrors.163.com/ubuntu-ports/ focal main restricted universe multiverse
#deb-src http://mirrors.163.com/ubuntu-ports/ focal-security main restricted universe multiverse
#deb-src http://mirrors.163.com/ubuntu-ports/ focal-updates main restricted universe multiverse
#deb-src http://mirrors.163.com/ubuntu-ports/ focal-proposed main restricted universe multiverse
#deb-src http://mirrors.163.com/ubuntu-ports/ focal-backports main restricted universe multiverse
```

## 使用

### 重置远程仓库

```shell
sudo apt-get clean
sudo rm -rf /var/lib/apt/lists/
```

### apt-get 参数设置

```shell
# 强制apt-get走IPv4
sudo apt-get -o Acquire::ForceIPv4=true update
# 将your_proxy替换为apt的http报文使用的代理地址
sudo apt -o Acquire::http::Proxy="${your_proxy}" update 
```

### HTTPS报错

对于出现类似以下情况的报错

```shell
Certificate verification failed: The certificate is NOT trusted. 
The certificate issuer is unknown. Could not handshake: Error in the certificate verification. 
The certificate is NOT trusted. The certificate issuer is unknown. Could not handshake: Error in the certificate verification.
```

可能是客户端证书认证的问题：可以尝试以下步骤：

1. 安装`ca-certificates`包：
   
   ```shell
   sudo apt-get install ca-certificates
   ```

2. 重装`ca-certificates`包：
   
   ```shell
   sudo apt-get install --reinstall ca-certificates
   ```

3. 手动下载最新`ca-certificates`包：
   
   1. 前往官方寻找deb包：
      
      ```shell
      # 官方网站
      https://pkgs.org/download/ca-certificates
      # 打不开可以在ubuntu镜像源寻找
      http://archive.ubuntu.com/ubuntu/pool/main/c/ca-certificates/
      # 清华源
      https://mirrors.tuna.tsinghua.edu.cn/ubuntu/pool/main/c/ca-certificates/
      # 以上为x86架构的源，如果要arm源则为
      https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/pool/main/c/ca-certificates/
      ```
      
      在镜像源中选取自身ubuntu版本以及架构的**all.deb**的包
   
   2. 下载包：
      
      ```shell
      wget https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/pool/main/c/ca-certificates/ca-certificates_20210119~20.04.2_all.deb
      ```
   
   3. 安装：
      
      ```shell
      sudo dpkg -i ./ca-certificates_20210119~18.04.2_all.deb
      ```

4. 再次尝试update：
   
   ```shell
   sudo apt-get update
   ```
