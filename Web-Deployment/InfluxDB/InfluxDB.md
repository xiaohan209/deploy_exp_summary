# InfluxDB

## 介绍

## 安装

### apt安装

> 官方下载指南: <https://portal.influxdata.com/downloads/>

**系统版本：Ubuntu 20.04 focal**

1. 获取官方gpg密钥：
   
   ```bash
   # 下载密钥
   wget -q https://repos.influxdata.com/influxdata-archive_compat.key
   # 验证并添加密钥
   echo '393e8779c89ac8d958f81f942f9ad7fb82a25e133faddaf92e15b16e6ac9ce4c influxdata-archive_compat.key' | sha256sum -c && cat influxdata-archive_compat.key | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/influxdata-archive_compat.gpg > /dev/null
   
   ```
   
2. 添加官方存储库：
   
   ```bash
   # 添加到sources.list.d中
   echo 'deb [signed-by=/etc/apt/trusted.gpg.d/influxdata-archive_compat.gpg] https://repos.influxdata.com/ubuntu stable main' | sudo tee /etc/apt/sources.list.d/influxdata.list
   
   ```
   
3. 下载最新版本influxdb：
   
   ```bash
   sudo apt-get update && sudo apt-get install influxdb2
   ```

## 使用

