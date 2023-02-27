# MongoDB

## 介绍



## 安装

> [MongoDB官方手册](https://www.mongodb.com/docs/manual/)

### apt 使用官方存储库方式安装

> [官网Ubuntu安装方式指南](https://www.mongodb.com/docs/manual/tutorial/install-mongodb-on-ubuntu/)

**系统版本：Ubuntu 20.04 focal**

1. apt下载需要安装的前置功能包：
   ```bash
   sudo apt-get install gnupg wget
   ```

2. 获取指定版本官方gpg密钥：

   ```bash
   wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add -
   ```

   如果要其他版本的可以到官方存储库网站https://pgp.mongodb.com/

3. 添加官方存储库：

   ```bash
   # 添加官方存储库
   echo "deb [arch=amd64] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list > /dev/null
   ```

4. 下载最新版本influxdb：

   ```bash
   sudo apt-get update && sudo apt-get install mongodb-org
   ```



### 直接使用二进制文件安装

> [官方下载页面](https://www.mongodb.com/try/download/community)

根据提示进行选择版本、平台、文件格式的选择，一键下载并安装



### docker容器安装





## 使用

