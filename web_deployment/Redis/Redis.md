# Redis

> [Redis官网](https://redis.io/)
>
> 参考博客：
>
> - <https://www.cnblogs.com/qqflying/p/9192331.html>
> - <https://blog.csdn.net/suifeng3051/article/details/38657613>
> - <https://blog.csdn.net/wsdc0521/article/details/106765856>

## 介绍

Redis 是一个开源（BSD 许可）的、使用C语言编写的、支持网络交互的、可基于内存也可持久化的数据结构存储，用作数据库、缓存和消息代理。Redis 提供数据结构，例如字符串、散列、列表、集合、具有范围查询的排序集合、位图、超日志、地理空间索引和流。Redis 具有内置复制、Lua 脚本、LRU 驱逐、事务和不同级别的磁盘持久性，并通过 Redis Sentinel 和 Redis Cluster 自动分区提供高可用性



## 安装

> [官方下载说明](https://redis.io/download)
>
> [官方快速入门安装说明](https://redis.io/topics/quickstart)
>
> [稳定版源码在线版](http://download.redis.io/redis-stable/)

### APT包管理

操作系统：ubuntu 20.04 focal

Redis版本：



1. 安装其他依赖：

   ```shell
   sudo apt-get update
   sudo apt-get install curl gpg
   ```

   

2. 添加GPG Key：

   ```shell
   sudo curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg
   ```

   

3. 添加仓库：

   1. 添加官方稳定版，则为：

      ```shell
      # 官方镜像源
      echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list
      # 华为镜像
      echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://mirrors.huaweicloud.com/redis/ $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list
      ```

   2. 添加PPA个人存储库最新版，则为：

      ```shell
      sudo add-apt-repository ppa:redislabs/redis
      ```
   
4. 安装Redis

   ```shell
   sudo apt-get update
   sudo apt-get install redis
   ```
   
5. 测试安装结果

   ```shell
   # 检查server是否已经启动
   redis-server -v
   # 检查命令行工具以及其连接情况
   redis-cli ping
   ```

   

6. 



### 源码编译

1. 下载需要的依赖：

   ```shell
   sudo apt update
   sudo apt install wget
   ```

2. 下载源码：

   ```shell
   # 官方为当前最新稳定版本
   wget http://download.redis.io/redis-stable.tar.gz
   tar xvzf redis-stable.tar.gz
   cd redis-stable
   ```

3. 编译安装

   1. 进行`make`

   2. 可以使用

   3. 编译完成之后在src目录中可以看到一下内容：
      - **redis-server**：Redis 服务器本身
      - **redis-sentinel**：Redis Sentinel 可执行文件，用来监控和故障转移
      - **redis-cli**：Redis命令行界面实用程序
      - **redis-benchmark**：用于检查 Redis 性能
      - **redis-check-aof**和**redis-check-rdb**(在3.0及以下版本中为**redis-check-dump**)：用于数据文件损坏的检查恢复

   4. 将`redis-server`和`redis-cli`复制到需要的目录下，例如：

      ```shell
      sudo cp src/redis-server /usr/local/bin/
      sudo cp src/redis-cli /usr/local/bin/
      ```

   当然以上步骤可以直接执行`sudo make install`来自动进行编译安装

4. 启动redis并测试

   ```shell
   # 直接启动，采用默认配置
   redis-server
   # 启动时加载配置文件/etc/redis.conf
   redis-server /etc/redis.conf
   ```

   可以将下载下来的文件目录中的`redis.conf`复制到`/etc/redis.conf`中，具体配置过程请见[配置Redis数据库](#配置Redis数据库)

   测试cli命令：

   ```shell
   redis-cli ping
   ```

   



## 使用

> [命令行工具说明](https://redis.io/topics/rediscli)
>
> https://redis.io/commands

### 配置Redis数据库

> https://redis.io/topics/config
>
> [官方设置配置说明](https://redis.io/commands/config-set)
>
> [官方获取配置说明](https://redis.io/commands/config-get)









### 添加用户及密码权限

Redis有两种设置用户验证的方式：

1. configfile：`/etc/redis/redis.config`文件中直接配置
2. aclfile：在外部文件中配置，一般位于`/etc/redis/users.acl`，具体文件路径可在`redis.config`中指定

以上两种方式的配置命令和语法一致，均采用DSL(Domain specific language)定义的，该DSL描述了用户能够执行的操作，其始终从上到下，从左到右应用，因为规则的顺序对于理解用户的实际权限很重要

以上两种配置不能同时出现，只要在`redis.config`中指定`aclfile`，则`redis.config`当中的其他用户验证设置，如`requirepass`以及`user`开头的acl规则均失效，优先采用`aclfile`中的验证规则

使用aclfile模式之后不能使用`redis-cli -a ${password}`登陆，必须使用`redis-cli --user {user} --pass {password}`来登陆



#### config文件

##### requirepass

在config文件中找到requirepass修改此行，或者直接添加：

```shell
# 直接以明文形式将${password}替换为自己的密码输入到config文件
requirepass ${password}
```



##### ACL规则

与ACLFILE中使用DSL语言规范一致，只是可以写在config文件中，在使用Redis时对用户规则的改动与外部ACLFILE方式有所区别，具体请见[两种方式配置的对比](# 两种方式配置的对比)



#### 外部ACLFILE

##### DSL语言用法

> 以下原文链接：https://blog.csdn.net/wsdc0521/article/details/106765856

以下对于`{}`中包含的部分均表示变量，而不是关键字

###### 启用和禁用用户

- `on`：启用用户，可以以该用户身份进行认证。
- `off`：禁用用户，不再可以使用此用户进行身份验证，但是已经通过身份验证的连接仍然可以使用。

###### 允许和禁止调用命令

- `+{command}`：将命令添加到用户可以调用的命令列表中。
- `-{command}`：将命令从用户可以调用的命令列表中移除。
- `+@{category}`：允许用户调用`{category}`类别中的所有命令，有效类别为`@admin`，`@set`，`@sortedset`等，可通过调用`ACL CAT`命令查看完整列表。特殊类别`@all`表示所有命令，包括当前和未来版本中存在的所有命令。
- `-@{category}`：禁止用户调用`{category}` 类别中的所有命令。
- `+{command}|subcommand`：允许使用已禁用命令的特定子命令。
- `allcommands`：`+@all`的别名，包括当前存在的命令以及将来通过模块加载的所有命令。
- `nocommands`：`-@all`的别名，禁止调用所有命令。

###### 允许或禁止访问某些Key

- `~{pattern}`：添加可以在命令中提及的键模式，例如`~*`和`* allkeys`允许所有键。
- `resetkeys`：使用当前模式覆盖所有允许的模式，如：`~foo:* ~bar:* resetkeys ~objects:*`，客户端只能访问匹配`object:*` 模式的键。

###### 为用户配置有效密码

- `>{password}`：将此密码添加到用户的有效密码列表中，例如`>mypass`表示将“mypass”添加到有效密码列表中，该命令会清除用户的nopass标记。**每个用户可以有任意数量的有效密码。**
- `<{password}`：从有效密码列表中删除此密码，若该用户的有效密码列表中没有此密码则会返回错误信息。
- `#{hash}`：将`{hash}`这个SHA-256哈希值添加到用户的有效密码列表中，该哈希值将与为ACL用户输入的密码的哈希值进行比较。允许用户将哈希存储在users.acl文件中，而不是存储明文密码。仅接受SHA-256哈希值，因为密码哈希必须为64个字符且小写的十六进制字符。
- `!{hash}`：从有效密码列表中删除该哈希值（当不知道哈希值对应的明文是什么时很有用）
- `nopass`：移除该用户已设置的所有密码，并将该用户标记为nopass无密码状态，即任何密码都可以登录。`resetpass`命令可以清除`nopass`这种状态。
- `resetpass`：情况该用户的所有密码列表，而且移除`nopass`状态。`resetpass`之后用户没有关联的密码同时也无法使用无密码登录，因此`resetpass`之后必须添加密码或改为`nopass`状态才能正常登录。
- `reset`：重置用户状态为初始状态，相当于执行以下操作：`resetpass`，`resetkeys`，`off`，`-@all`





#### 两种方式配置的对比

|                   |     redis.conf     |  users.acl   |
| :---------------: | :----------------: | :----------: |
|   **配置方式**    |        DSL         |     DSL      |
|  **加载ACL配置**  |   重启Redis服务    | ACL LOAD命令 |
| **持久化ACL配置** | CONFIG REWRITE命令 | ACL SAVE命令 |

