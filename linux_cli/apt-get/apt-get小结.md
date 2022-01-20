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

