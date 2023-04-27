# MySQL & MariaDB



## MySQL介绍

## MariaDB介绍

## MariaDB安装

> - [官方下载二进制运行文件页面](https://mariadb.org/download/)
>
> - [官方指定仓库网址](https://mariadb.org/download/?t=repo-config)

###### **当前示例系统版本：Ubuntu 20.04 focal**

1. 安装必要依赖：

   ```bash
   sudo apt-get install apt-transport-https curl gpg
   ```

2. 添加官方存储库密钥：

   ```bash
   sudo curl -o /etc/apt/trusted.gpg.d/mariadb_release_signing_key.asc 'https://mariadb.org/mariadb_release_signing_key.asc'
   ```

3. 添加存储库：

   建议去[官方指定仓库网址](https://mariadb.org/download/?t=repo-config)根据自己的需要进行仓库链接的生成，对生成的文件写入`/etc/apt/sources.list.d/mariadb.list`中，文件示例：

   ```bash
   # https://mariadb.org/download/
   deb https://mirrors.neusoft.edu.cn/mariadb/repo/11.0/ubuntu focal main
   # deb-src https://mirrors.neusoft.edu.cn/mariadb/repo/11.0/ubuntu focal main
   ```

4. 换源：

   ```bash
   # 将链接url地址中的repo及以前的字段进行替换，保留后面的版本信息
   # 例如将https://mirrors.neusoft.edu.cn/mariadb/repo替换为以下内容
   https://mirrors.tuna.tsinghua.edu.cn/mariadb/repo
   ```

   如果遇到`Skipping acquire of configured file 'main/binary-i386/Packages' as repository ....... doesn't support architecture 'i386'`类似的警告信息，在`/etc/apt/sources.list.d/mariadb.list`的

5. 下载并进行安装

   ```bash
   sudo apt-get update
   sudo apt-get install mariadb-server mariadb-common
   ```

   





## 使用

### MysqlDump

> 参考博客：
>
> - https://www.cnblogs.com/markLogZhu/p/11398028.html
> - https://www.cnblogs.com/chenmh/p/5300370.html
> - https://www.runoob.com/mysql/mysql-database-export.html
> - https://blog.csdn.net/mengzuchao/article/details/81674460
> - https://blog.51cto.com/aklaus/1677766
> - http://c.biancheng.net/view/7373.html
> - https://segmentfault.com/a/1190000000621104

`mysqldump` 是 `MySQL` 自带的逻辑备份工具，工作流程大致为：

1. 连接数据库，校验账户，密码，IP
2. 进入`INFORMATION_SCHEMA`库，获取要备份的数据库的信息，包含存储过程，视图，表
3. 进入`INFORMATION_SCHEMA`库，获取每个表的字段名称，字段类型等信息
4. 查询每个表的数据，`select SQL_NO_CACHE from tbname`
5. 将查询出的数据拼接为DDL SQL语句
6. 写入备份文件

#### 命令使用形式

1. 导出单个数据库：

   ```mysql
   mysqldump [OPTIONS] DB [tables] > table.db
   ```

   DB为需要导出的数据库名，tables为需要导出的多个表名

2. 导出指定的多个数据库：

   ```mysql
   mysqldump [OPTIONS] --databases [OPTIONS] DB1 [DB2 DB3...] > databases.db
   ```

   DB为需要导出的多个数据库名

3. 导出全部数据库：

   ```mysql
   mysqldump [OPTIONS] --all-databases [OPTIONS] > all.db
   ```

   

#### OPTION参数说明

|             参数名              |                   含义                   |
| :-----------------------------: | :--------------------------------------: |
|          --host 或 -h           |               服务器IP地址               |
|          --port 或 -p           |               服务器端口号               |
|          --user 或 -u           |               MySQL 用户名               |
|         --pasword 或 -p         |                MySQL 密码                |
|        --databases 或 -B        |            指定要备份的数据库            |
|      --all-databases 或 -A      |      备份mysql服务器上的所有数据库       |
|     --all-tablespaces 或 -Y     |          备份某个数据库的所有表          |
|            --compact            |         压缩模式，产生更少的输出         |
|         --skip-comments         |        取消注释信息，默认添加注释        |
|        --complete-insert        |            输出完成的插入语句            |
|          --lock-tables          | 备份前，锁定所有数据库表，完成后自动解锁 |
| --no-create-db/--no-create-info |     只导出数据，不添加创建数据库语句     |
|            --no-data            |      不导出数据，只导出数据库表结构      |
|             --force             |       当出现错误时仍然继续备份操作       |
|     --default-character-set     |              指定默认字符集              |
|           --add-locks           |        备份数据库表时锁定数据库表        |
|         --ignore-table=         |    忽略某些表，`=`后面填写忽略的表名     |
|    --skip-add-drop-database     |  数据库创建前不添加drop语句，默认为添加  |
|      --skip-add-drop-table      |    表创建前不添加drop语句，默认为添加    |
|            --replace            |     使用REPLACE INTO 取代INSERT INTO     |
|        --routines 或 -R         |        导出存储过程以及自定义函数        |

#### 实例

1. 备份所有数据库：

   ```java
   mysqldump -uroot -p --all-databases > all.db
   ```

2. 备份指定的多个数据库：

   ```java
   mysqldump -uroot -p --databases db1 db2 test1 test2 > test.db
   ```

3. 备份指定数据库的多个表

   ```java
   mysqldump -uroot -p  mysql db event user > tables.db
   ```

4. 备份指定数据库排除某些表

   ```java
   mysqldump -uroot -p test --ignore-table=test.t1 --ignore-table=test.t2 > tables2.db
   ```



### 导入数据还原数据库

>- https://www.runoob.com/mysql/mysql-database-import.html

1. Shell命令行形式

   ```sh
   mysqladmin -uroot -p create db_name 
   mysql -uroot -p  db_name < /backup/mysqldump/db_name.db
   
   注：在导入备份数据库前，db_name如果没有，是需要创建的； 而且与db_name.db中数据库名是一样的才可以导入。
   ```

2. Mysql数据库客户端命令行形式

   ```mysql
   use db_name;
   source /backup/mysqldump/db_name.db;
   ```

3. 





### 重置root密码

> - <http://blog.wuxu92.com/reset-mysql-root-password/>

1. 开启安全免密登录模式

   ```sh
   # 关闭mysql服务
   sudo service mysql stop
   # 必须使用sudo，否则没有权限打开mysql文件夹
   sudo mysqld_safe --skip-grant-tables
   ```

   如果遇到`mysqld_safe Directory '/var/run/mysqld' for UNIX socket file don't exists`，则需要手动创建`/var/run/mysqld`并手动赋予权限

2. 启动mysql

   ```sh
   sudo service mysql start
   ```

3. 再次使用命令行登录

   ```sh
   mysql -u root
   ```

4. 切换到mysql的用户管理数据库，在mysql控制台中输入：

   ```mysql
   use mysql;
   ```

5. 更改密码

   ```mysql
   # 需要替换为自己的新密码
   update user set authentication_string = password('yourpassword') where user='root';
   ```

6. 刷新数据库

   ```mysql
   flush privileges;
   ```

7. 退出mysql控制台

   ```mysql
   quit
   ```

8. 重启mysql服务并连接

   ```sh
   # 第一种方式：直接杀死mysqld
   sudo kill `cat /var/run/mysqld/mysqld.pid`
   # 第二种方式先查看原来的mysqld_safe进程号
   ps -a | grep mysql
   # 第二种方式再根据进程号杀死原来进程
   sudo kill -9 <$your_pid>
   # 最后重启服务
   sudo service mysql restart
   ```

9. 

### 常见错误

#### ERROR1524

错误提示为：ERROR 1524 (HY000): Plugin 'auth_socket' is not loaded

解决办法：

```sh
sudo /etc/init.d/mysql stop
sudo /etc/init.d/mysql start
```

#### [ERROR] InnoDB: Missing MLOG_CHECKPOINT

> - <https://blog.csdn.net/w1346561235/article/details/107046170>
>
> - <https://blog.51cto.com/853056088/2483244>

##### 原因

在Mysql 5.7之后的版本，checkpoint完成后会在redo log写一个一字节的MLOG_CHECKPOINT 标记，用来标记在此之前的redo都已checkpoint完成，没有找到的话整个log文件都会被忽略

##### 恢复方法

如果没有备份，则以下恢复过程可能会有数据丢失

1. 在当前mysql数据库使用的my.cnf配置中设置以下参数

   ```mysql
   # 设置innodb的检查
   innodb_log_checksums = ON
   # 避免客户端连接进来操作数据
   skip_networking=ON
   ```

2. 移除在日志同级目录下(即运行时mysql文件夹)`ib_logfile`开头的文件：

   ```sh
   sudo mv ib_logfile0 ib_logfile0.save
   sudo mv ib_logfile1 ib_logfile1.save
   ```

3. 尝试启动一下数据库，如果直接启动则正常运行，到第5步进行再次备份即可；如果仍然出错则需要继续第4步调整mysql设置

4. 若出现`[ERROR] InnoDB: Page log sequence number is in the future`，是因为mysql writer线程按照配置的时间间隔以page为单位刷新buffer数据到磁盘，当数据刷新到磁盘的时候，新写入磁盘的page包含了较新的lsn，此时系统system表空间头的lsn并没有同步更新，通常这是检查点线程的工作。在正常的崩溃恢复中，mysql可以借助redo日志来进行前滚和回滚，但是此时redo日志已经被我们删掉了，mysql无法进行恢复操作，需要设置：

   ```mysql
   innodb_force_recovery=3
   ```

   每次错误后提升`innodb_force_recovery`数值再进行重试

5. 如果需要备份，使用`mysqldump`导出备份即可

   ```mysql
   mysqldump -uroot -p --single-transaction --master-data=2 --flush-privileges --routines --triggers --events --all-databases > fullbackup.sql
   ```

   可以指定--ignore-table参数跳过有问题的表，或使用--ignore-error参数忽略指定的错误号

6. 新备份恢复到数据库中

#### [ERROR] Incorrect definition of table mysql.proc/mysql.event

> - <https://segmentfault.com/q/1010000017210657/a-1020000017284905>
>
> - <https://dev.mysql.com/doc/relnotes/mysql/5.7/en/news-5-7-23.html>

原错误信息：

```sh
[ERROR] Incorrect definition of table mysql.proc: expected column 'sql_mode' at position 14 to have type set('REAL_AS_FLOAT','PIPES_AS_CONCAT','ANSI_QUOTES','IGNORE_SPACE','NOT_USED','ONLY_FULL_GROUP_BY','NO_UNSIGNED_SUBTRACTION','NO_DIR_IN_CREATE','POSTGRESQL','ORACLE','MSSQL','DB2','MAXDB','NO_KEY_OPTIONS','NO_TABLE_OPTIONS','NO_FIELD_OPTIONS','MYSQL323','MYSQL40','ANSI','NO_AUTO_VALUE_ON_ZERO','NO_BACKSLASH_ESCAPES','STRICT_TRANS_TABLES','STRICT_ALL_TABLES','NO_ZERO_IN_DATE','NO_ZERO_DATE','INVALID_DATES','ERRO
```

或：

```sh
[ERROR] Incorrect definition of table mysql.event: expected column 'sql_mode' at position 14 to have type set('REAL_AS_FLOAT','PIPES_AS_CONCAT','ANSI_QUOTES','IGNORE_SPACE','NOT_USED','ONLY_FULL_GROUP_BY','NO_UNSIGNED_SUBTRACTION','NO_DIR_IN_CREATE','POSTGRESQL','ORACLE','MSSQL','DB2','MAXDB','NO_KEY_OPTIONS','NO_TABLE_OPTIONS','NO_FIELD_OPTIONS','MYSQL323','MYSQL40','ANSI','NO_AUTO_VALUE_ON_ZERO','NO_BACKSLASH_ESCAPES','STRICT_TRANS_TABLES','STRICT_ALL_TABLES','NO_ZERO_IN_DATE','NO_ZERO_DATE','INVALID_DATES','ERROR_FOR_DIVISION_BY_ZERO','TRADITIONAL','NO_AUTO_CREATE_USER','HIGH_NOT_PRECEDENCE','NO_ENGINE_SUBSTITUTION','PAD_CHAR_TO_FULL_LENGTH'), found type set('REAL_AS_FLOAT','PIPES_AS_CONCAT','ANSI_QUOTES','IGNORE_SPACE','IGNORE_BAD_TABLE_OPTIONS','ONLY_FULL_GROUP_BY','NO_UNSIGNED_SUBTRACTION','NO_DIR_IN_CREATE','POSTGRESQL','ORACLE','MSSQL','DB2','MAXDB','NO_KEY_OPTIONS','NO_TABLE_OPTIONS','NO_FIELD_OPTIONS','MYSQL323','MYSQL40','ANSI','NO_AUTO_VALUE_ON_ZERO','NO_BACKSLASH_ESCAPES','STRICT_TRANS_TABLES','STRICT_A
```

##### 可能原因

有可能是不同版本的mysql.proc或mysql.event表结构不兼容引起的

##### 恢复方法

1. 先直接执行`mysql_upgrade`看看升级后是否能够修复：

   ```sh
   mysql_upgrade
   # 如果有指定的数据库可以调整以下参数进行更新
   mysql_upgrade --host="127.0.0.1" --port=3306 --user="root" --password="password"
   ```

2. 如果不能修复则启动服务端：

   ```sh
   /usr/sbin/mysqld --skip-grant --general-log &
   ```

3. 修改表项：

   ```mysql
   ALTER TABLE mysql.proc MODIFY sql_mode 
   # 将报错信息中前面应该使用的表项填入
   SET (...)
   ALTER TABLE mysql.event MODIFY sql_mode 
   # 将报错信息中应该使用的表项信息填入
   SET (...)
   ```

4. 恢复默认登录选项：

   ```mysql
   flush privileges;
   ```

   

