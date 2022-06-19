# InfluxDB

## 介绍

## 安装

### apt安装

**系统版本：Ubuntu 20.04 focal**

1. 获取官方gpg密钥：
   
   ```bash
   wget -qO- https://repos.influxdata.com/influxdb.key | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/influxdb.gpg > /dev/null
   ```

2. 添加官方存储库：
   
   ```bash
   # bash用法
   export DISTRIB_ID=$(lsb_release -si); export DISTRIB_CODENAME=$(lsb_release -sc)
   echo "deb [signed-by=/etc/apt/trusted.gpg.d/influxdb.gpg] https://mirrors.tuna.tsinghua.edu.cn/influxdata/${DISTRIB_ID,,} ${DISTRIB_CODENAME} stable" | sudo tee /etc/apt/sources.list.d/influxdb.list > /dev/null
   # 非bash用法直接使用系统版本如ubuntu focal来替换上述环境变量
   echo "deb [signed-by=/etc/apt/trusted.gpg.d/influxdb.gpg] https://mirrors.tuna.tsinghua.edu.cn/influxdata/ubuntu focal stable" | sudo tee /etc/apt/sources.list.d/influxdb.list > /dev/null
   ```

3. 下载最新版本influxdb：
   
   ```bash
   sudo apt-get update && sudo apt-get install influxdb2
   ```

## 使用


