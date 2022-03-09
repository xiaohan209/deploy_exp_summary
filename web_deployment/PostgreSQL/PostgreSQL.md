# PostgreSQL

> [PostgreSQL官网](https://www.postgresql.org/)

## 介绍

## 安装

> [官网安装教程](https://www.postgresql.org/download/)

**操作系统：ubuntu 20.04 focal**

**PostgreSQL版本：12**

1. 安装前置工具依赖
   
   ```shell
   sudo apt-get update
   sudo apt-get install wget curl ca-certificates gnupg
   ```

2. 添加官方存储库签名
   
   ```shell
   # 使用wget
   wget -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
   # 或使用curl
   curl https://www.postgresql.org/media/keys/ACCC4CF8.asc | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/apt.postgresql.org.gpg >/dev/null
   ```

3. 添加存储库
   
   ```shell
   # 官方存储库地址
   sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
   # 清华源地址
   sudo sh -c 'echo "deb http://mirrors.tuna.tsinghua.edu.cn/postgresql/repos/apt/ $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
   ```

4. 更新并下载PostgreSQL
   
   ```shell
   sudo apt-get update
   ```

可以不用手动执行上述过程，使用官方提供的脚本即可完成操作：

```shell
sudo apt install postgresql-common
sudo sh /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh
sudo apt-get update
```

## 使用

> [官方sql命令说明](https://www.postgresql.org/docs/current/sql-commands.html)

### 初始化登录

```shell
local   all                postgres                                trust

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local    all                all                                        peer
# IPv4 local connections:
host    all                all                127.0.0.1/32            scram-sha-256
# IPv6 local connections:
host    all                all                ::1/128                    scram-sha-256
# Allow replication connections from localhost, by a user with the
# replication privilege.
local   replication     all                                        peer
host    replication     all                127.0.0.1/32            scram-sha-256
host    replication     all                ::1/128                    scram-sha-256
```

### 创建并管理角色

> [添加新角色](https://www.postgresql.org/docs/current/sql-createrole.html)
> 
> [修改角色](https://www.postgresql.org/docs/current/sql-alterrole.html)
> 
> [改变当前会话角色](https://www.postgresql.org/docs/current/sql-set-role.html)
> 
> [删除角色](https://www.postgresql.org/docs/current/sql-droprole.html)
> 
> [添加用户](https://www.postgresql.org/docs/current/sql-createuser.html)
> 
> [修改用户](https://www.postgresql.org/docs/current/sql-alteruser.html)
> 
> [删除用户](https://www.postgresql.org/docs/current/sql-dropuser.html)
> 
> [添加用户组](https://www.postgresql.org/docs/current/sql-creategroup.html)
> 
> [修改用户组](https://www.postgresql.org/docs/current/sql-altergroup.html)
> 
> [删除用户组](https://www.postgresql.org/docs/current/sql-dropgroup.html)
> 
> 密码相关：
> 
> - <https://www.modb.pro/db/40292>

SET ROLE 允许登录的用户切换直接或间接所属的各种角色

用户与角色区别：用户不会继承用户，但是用户与角色，角色与角色之间可以进行继承，用户是一种特殊的角色。一般将用作用户的角色添加NOINHERIT属性，使其不能被继承，而对于普通的角色添加INHERIT属性，可以被继承。

角色属性`LOGIN`、`SUPERUSER`、`CREATEDB`和`CREATEROLE`可以被认为是特殊权限，不会像其他权限被直接继承，而是以普通用户身份登录，然后采用`SET ROLE`的方式进行角色切换后得到权限。

命令行工具：

```shell
createuser ${name}
dropuser ${name}
```

进入psql：

```sql
# 查看所有角色
SELECT rolname FROM pg_roles;
# 创建新的角色
CREATE ROLE ${name} LOGIN;
# CREATE USER 默认添加权限LOGIN，CREATE ROLE需要指定权限
CREATE USER ${name};
# 删除用户
DROP ROLE ${name};
```

#### 改密码

```sql
# 将${password}按照系统默认加密方式进行加密后写入数据库
ALTER USER ${name} WITH PASSWORD '${password}';
# 直接将${password}写入密码中，需要提前将真正的密码加密后再写入
ALTER USER ${name} with ENCRYPTED PASSWORD '${password}';
```

查看密码：

```sql
# pg_shadow与pg_authid中显示加密后的密码
SELECT usename,passwd FROM pg_shadow;
SELECT rolname,rolpassword FROM pg_authid;
# pg_user和pg_roles中的密码是不可见的
SELECT usename,passwd FROM pg_user;
SELECT rolname,rolpassword FROM pg_roles;
```

### 配置

一般在`/etc/postgresql/`目录下，会有不同的

### 管理工具

#### PgAdmin4 & PgAgent

> [工具官网下载页面](https://www.pgadmin.org/download/)
> 
> [docker部署方式](https://www.pgadmin.org/docs/pgadmin4/latest/container_deployment.html)

##### 以Ubuntu为例进行安装：

1. 安装前置依赖
   
   ```shell
   sudo apt-get update
   sudo apt install curl
   ```

2. 添加apt-key：
   
   ```shell
   sudo curl https://www.pgadmin.org/static/packages_pgadmin_org.pub | sudo apt-key add
   ```

3. 添加仓库地址：
   
   ```shell
   sudo sh -c 'echo "deb https://ftp.postgresql.org/pub/pgadmin/pgadmin4/apt/$(lsb_release -cs) pgadmin4 main" > /etc/apt/sources.list.d/pgadmin4.list'
   ```

4. 更新并安装
   
   ```shell
   sudo apt update
   # 安装完整版
   sudo apt install pgadmin4
   # 只安装桌面版
   sudo apt install pgadmin4-desktop
   # 只安装网页版
   sudo apt install pgadmin4-web
   ```

5. 桌面版直接打开进行操作，如果下载的是网页版，需要进行配置：
   
   ```shell
   sudo /usr/pgadmin4/bin/setup-web.sh
   ```

其中web版默认开启Apache，因此如果存在冲突可以使用docker

##### docker部署：

###### 直接容器运行：

1. 拉取镜像：
   
   ```shell
   docker pull dpage/pgadmin4
   ```

2. 由镜像运行容器：
   
   ```shell
   # 其中5050为宿主机的访问端口，80为容器端口，可以进行修改
   # 以下两参数是必须填的，作为注册初始登录用户
   docker run -p 5050:80 \
       -e "PGADMIN_DEFAULT_EMAIL=user@domain.com" \
       -e "PGADMIN_DEFAULT_PASSWORD=yoursecret" \
       -d dpage/pgadmin4
   ```
   
   其余参数可以查看官方教程[docker部署方式](https://www.pgadmin.org/docs/pgadmin4/latest/container_deployment.html)

3. 进行登录访问即可

###### docker-compose

1. 下载docker-compose

2. 找到一个用来存放配置的路径，这里以`/etc/pgadmin4/`为例：
   
   ```shell
   sudo mkdir /etc/pgadmin4
   ```

3. 新建并填写compose.yml：
   
   ```yaml
   version: '3.1'
   services:
           pgadmin:
                   image: dpage/pgadmin4:latest
                   container_name: pgadmin_dc
                   restart: always
                   environment:
                           PGADMIN_DEFAULT_EMAIL: youremail@yourdomain #在此填写pgAdmin登录账户邮箱
                           PGADMIN_DEFAULT_PASSWORD: yourpasswd #在此填写pgAdmin密码
                   ports:
                           - "5050:80"
   ```

4. 在此目录下启动：
   
   ```shell
   sudo docker-compose up -d
   ```

5. 


