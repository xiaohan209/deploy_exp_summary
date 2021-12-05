# LDAP服务器搭建

> 参考文章：
>
> - <https://blog.csdn.net/qq_37951246/article/details/113247358>
> - <https://www.cnblogs.com/wilburxu/p/9174353.html>
>
> 入门指南：<https://www.openldap.org/doc/admin26/quickstart.html>
>
> 仓库：<https://www.openldap.org/software/download/OpenLDAP/openldap-release/>
>
> SLAPD使用参数手册：<https://www.openldap.org/software/man.cgi?query=slapd>
>
> OpenLDAP文档：<https://www.openldap.org/doc/>

## LDAP简介

LDAP（Light Directory Access Portocol），是基于X.500标准的轻量级目录访问协议。

目录服务是为查询、浏览和搜索而优化的数据库，它成树状结构组织数据，类似文件目录一样。

目录数据库和关系数据库不同，它有优异的读性能，很适合查询，但写性能差，并且没有事务处理、回滚等复杂功能，不适于存储修改频繁的数据。

OpenLDAP是这个协议的一个开源实现

slapd是OpenLDAP的服务端名称

ldap-utils是OpenLDAP的命令行工具集，包含ldapsearch，ldapmodify，ldapadd等

| 关键字  |      英文全称      |                             含义                             |
| :-----: | :----------------: | :----------------------------------------------------------: |
| **dc**  |  Domain Component  | 域名的部分，其格式是将完整的域名分成几部分，如域名为example.com变成dc=example,dc=com（一条记录的所属位置） |
| **uid** |      User Id       |              用户ID songtao.xu（一条记录的ID）               |
| **ou**  | Organization Unit  | 组织单位，组织单位可以包含其他各种对象（包括其他组织单元），如“oa组”（一条记录的所属组织） |
| **cn**  |    Common Name     |       公共名称，如“Thomas Johansson”（一条记录的名称）       |
| **sn**  |      Surname       |                          姓，如“许”                          |
| **dn**  | Distinguished Name | “uid=songtao.xu,ou=oa组,dc=example,dc=com”，一条记录的位置（唯一） |





## LDAP服务器安装

操作系统：ubuntu 20.04 focal

open-ldap版本：2.4.49+dfsg-2ubuntu1.8



### 软件前情提要

#### hostname

安装时，自动使用本机域名作为Base DN创建默认数据库，如果要修改的话可以输入命令重新进行命令行界面的配置，也可以直接修改`/etc/hosts`将`127.0.0.1`后的名称修改为需要的域名



#### 修改openldap配置

从openldap2.4.23版本开始，所有配置保存在`/etc/ldap/slapd.d`目录下的`cn=config`文件夹内，不再使用`slapd.conf`作为配置文件

每一个配置文件都是通过命令行自动生成的，打开后可以看到第一行注释说明了请勿直接编辑本文件，后缀类型为`.ldif`

因此修改目录中的内容需要使用`ldapmodify`进行修改



## 安装软件

1. 利用`apt-get`包管理软件，进行下载安装软件

   ```shell
   sudo apt-get install slapd ldap-utils
   ```

2. 根据后续软件提示进行初始化，输入初始密码等

5. 如果修改配置后需要重新加载，输入：

   ```shell
   sudo dpkg-reconfigure slapd
   ```

6. 添加基础的schema，不用添加也可以，默认启动以下三个schema

   ```sh
   ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif
   ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif
   ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif
   ```

6. 根据后续的管理工具，愉快地进行数据的管理吧！



## LDAP相关管理工具

### [PHP Ldap Admin](http://phpldapadmin.sourceforge.net/wiki/index.php/Main_Page)

1. 如果需要安装软件进行web端管理，可以安装php相关工具

   ```shell
   sudo apt-get install phpldapadmin
   ```

2. 对配置文件`/etc/phpldapadmin/config.php`进行修改：

   ```php
   $servers = new Datastore();
   $servers->newServer('ldap_pla');
   $servers->setValue('server','name','your_company LDAP Server');
   $servers->setValue('server','host','your_server_ip');#修改为某个内网可访问的IP地址
   $servers->setValue('server','base',array('dc=your_company,dc=com'));#修改为baseDN，这里修改为dc=ldapdomain,dc=com
   $servers->setValue('login','auth_type','session');#修改为baseDN下的admin, cn=admin,dc=ldapdomain,dc=com
   $servers->setValue('login','bind_id','cn=admin,dc=your_company,dc=com');
   $config->custom->appearance['hide_template_warning'] = false #false修改为true
   ```

   如果有防火墙可以进行放行：

   ```shell
   ufw allow "Apache"
   ufw allow "Apache Full"
   ufw allow "Apache Secure"
   # <SERVER_IP>填写自己服务器的IP即可
   ufw allow proto tcp from any to <SERVER_IP> port 389
   ufw allow proto tcp from any to <SERVER_IP> port 636 
   ```

   之后，重启服务

   ```shell
   /etc/init.d/apache2 restart 
   ```

   通过

   ```shell
   curl http://<IP-Address>/phpldapadmin
   ```

   或浏览器输入网址可以访问页面



### GO-LDAP

参考博客：

- https://blog.csdn.net/Mr_rsq/article/details/118937775
- https://studygolang.com/articles/19346
- https://www.jianshu.com/p/59ecd84118ce
- https://blog.csdn.net/weixin_39594447/article/details/87804225

官方文档：https://pkg.go.dev/gopkg.in/ldap.v3#section-readme

仓库地址：https://github.com/go-ldap/ldap

#### 第三方包下载

```go
 go get github.com/go-ldap/ldap/v3
```

#### 连接

```go
func NewLDAPService(serverAddr string) (*ldap.Conn, error) {
	// 纯tcp连接访问 一般为389端口
  ldapConnection, err := ldap.Dial("tcp", serverAddr)
	if err != nil {
		return nil, err
	}
  // TLS连接访问 一般为636端口
	tlsConfig := &tls.Config{InsecureSkipVerify: true}
	ldapsConnection, err := ldap.DialTLS("tcp", serverAddr, tlsConfig)
	if err != nil {
		return nil, err
	}
  // 根据连接形式选择一种进行连接即可
  return ldap_connection,err
}
```

#### 绑定认证

```go
func NewLDAPService(serverConnection )

err = conn.Bind(config.BindUserName, config.BindPassword)
	if err != nil {
		return nil, err
	}
berror := conn.Bind("uid=admin,dc=wjq,dc=com", "openldap")
```





#### LDAP Admin

windows专属软件，可浏览、搜索、修改、创建和删除LDAP服务器上的对象，也支持更复杂操作，例如远程服务器之间的目录复制和移动。软件有预定的对象种类设置，支持特定的对象类型，以及该类型常见的属性名。

###### 优点：开箱即用，windows下很好用

###### 缺点：最后一次更新与2017年，版本较老



### CLI

>参考博客：
>
>- <https://www.cnblogs.com/xzkzzz/p/9269714.html>
>- <https://www.cnblogs.com/qiuxiangmuyu/p/6437937.html>
>- https://blog.csdn.net/liumiaocn/article/details/83990960

#### ldap命令中通用的可选option参数：

| 命令 |   后面跟的参数   |                         说明                         |
| :--: | :--------------: | :--------------------------------------------------: |
|  -D  |     登录的DN     |                   所绑定服务器的DN                   |
|  -W  |        -         | 进行操作认证，但在命令输入后的下一行根据提示输入密码 |
|  -w  | 登录DN对应的密码 |          进行操作的认证，直接命令行指定密码          |
|  -h  |      主机名      |       LDAP服务器的ip或者域名，不能与-H同时使用       |
|  -p  |      端口号      |        LDAP服务器使用的端口，不能与-H同时使用        |
|  -H  |       URI        |    完整的URI直接指定服务器地址，协议为ldap或ldaps    |
|  -x  |        -         |                     使用简单认证                     |
|  -I  |        -         |                   使用SASL会话方式                   |
|  -y  |     文件路径     |                   从文件中读取密码                   |
|  -v  |        -         |                     显示详细信息                     |
|  -e  | 具体内容参照提示 |                    设置客户端证书                    |
|  -E  | 具体内容参照提示 |                    设置客户端私钥                    |
|  -c  |        -         |                   出现错误时不退出                   |



#### ldapadd

##### 可选option详解

| 命令 |   参数   |             说明             |
| :--: | :------: | :--------------------------: |
|  -f  | 文件路径 | 使用ldif文件内容进行条目添加 |

##### 例子

```sh
# 命令行添加条目
ldapadd -x -D "cn=root,dc=starxing,dc=com" -w secret 
# 使用文件添加条目
ldapadd -x -D "cn=root,dc=starxing,dc=com" -w secret -f ~/test.ldif
# ~/test文件内容为
#--------------------------------------------------------
dn: uid=test,ou=test,dc=example,dc=com
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
homeDirectory: /home/test
userPassword: {SSHA}15TXO5dtVyvBlU6MFV8kr2Cn5phgxTLB
loginShell: /bin/bash
cn: user_test
uidNumber: 1234
gidNumber: 1000
sn: test
mail: test@example.com
mobile: 18211111111
#--------------------------------------------------------
```



#### ldapdelete

##### 可选option详解

| 命令 | 参数 |   说明   |
| :--: | :--: | :------: |
|  -r  |  -   | 递归删除 |

##### 例子

```sh
ldapdelete -x -D "cn=admin,dc=example,dc=com" -w 123456 "uid=user,ou=test,dc=example,dc=com"
```



#### ldappassword

##### 可选option详解

| 命令 |  参数  |               说明               |
| :--: | :----: | :------------------------------: |
|  -a  | 旧密码 |  根据之前的旧密码随机生成新密码  |
|  -s  | 新密码 |       直接使用输入的新密码       |
|  -S  |   -    | 后续从命令行中根据提示属于新密码 |

`ldappassword`在上述的`option`输入完成后还需要输入需要修改的DN名称

##### 例子

```sh
# -s 指定密码
ldappasswd -x -D "cn=admin,dc=example,dc=com" -w 123456 "uid=user,ou=test,dc=example,dc=com" -s 123456
# -S 交互式需要输入新密码
ldappasswd -x -D "cn=admin,dc=example,dc=com" -w 123456 "cn=user,ou=test,dc=example,dc=com" -S
# -a 根据旧密码产生随机密码
ldappasswd -x -D "cn=admin,dc=example,dc=com" -w 123456 "uid=user,ou=test,dc=example,dc=com" -a 123456
# 不指定 产生随机密码
ldappasswd -x -D "cn=admin,dc=example,dc=com" -w 123456 "uid=user,ou=test,dc=example,dc=com"
```



#### ldapmodify

##### 可选option详解

| 命令 |   参数   |            说明            |
| :--: | :------: | :------------------------: |
|  -a  |    -     | 设置为追加条目，否则为替换 |
|  -f  | 文件路径 |  使用ldif文件内容进行修改  |

##### 例子

可以用`ldapmodify`实现`ldapadd`的功能，即使用添加的方式进行修改：

```sh
ldapmodify -x -D "cn=admin,dc=suixingpay,dc=com" -w 123456 -f ~/mod.ldif
# ~/mod.ldif内容如下
#-----------------------------------
dn: cn=test user,dc=example,dc=com
changetype: add
objectClass: inetOrgPerson
cn: test user
cn: test user
sn: user
title: this is the test user
mail: test_user@example.com
uid: test01
#-----------------------------------
```

当忘记管理员密码时也可以用`ldapmodify`进行修改，过程如下

```shell
# 生成管理员密码
slappasswd
# 按要求输入密码后会生成出一个加密内容，此处示例为{SSHA}15TXO5dtVyvBlU6MFV8kr2Cn5phgxTLB，记录下来

# 新增修改密码文件,ldif为后缀，文件名随意，不要在/etc/ldap/slapd.d/目录下创建类似文件
# 例如在~/ldap/myself-config路径下
mkdir -p ~/ldap/myself-config/
vim ~/ldap/myself-config/chrootpwd.ldif
#----------------------------------------------------- 以下为文件填写的内容
dn: olcDatabase={0}config,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}15TXO5dtVyvBlU6MFV8kr2Cn5phgxTLB
#------------------------------------------------------
# 这里解释一下这个文件的内容：
# 第一行执行配置文件，这里就表示指定为 cn=config/olcDatabase={0}config 在/etc/openldap/slapd.d/目录下
# 第二行 changetype 指定类型为修改
# 第三行 add 表示添加 olcRootPW 配置项
# 第四行指定 olcRootPW 配置项的值
# 在执行下面的命令前，你可以先查看原本的olcDatabase={0}config文件，里面是没有olcRootPW这个项的，执行命令后，你再看就会新增了olcRootPW项，而且内容是我们文件中指定的值加密后的字符串

# 执行命令，修改ldap配置，通过-f执行文件
ldapmodify -Y EXTERNAL -H ldapi:/// -f ~/ldap/myself-config/chrootpwd.ldif
```

因此`ldapmodify`可以根据`changetype`的不同以及后续文件内指令的不同完成各种指令的功能



#### ldapmordn

##### 可选option详解

| 命令 |   参数   |           说明           |
| :--: | :------: | :----------------------: |
|  -f  | 文件路径 | 使用ldif文件内容进行修改 |
|  -r  |    -     |  删除旧的DN，否则为新增  |

在输入`option`后还需要输入需要修改的原`DN`以及需要修改为的`RDN`

##### 例子

```sh
# 命令行直接输入参数修改
ldapmodrdn -x -D "cn=admin,dc=suixingpay,dc=com" -w 123456 -r "uid=user,ou=test,dc=example,dc=com" "uid=new"

# 使用文件修改
ldapmodrdn -x -D "cn=admin,dc=suixingpay,dc=com" -w 123456 -f ~/test.ldif
# test.ldif文件内容如下
#————————————————————————————————————————————————————————
dn: uid=user,ou=test,dc=example,dc=com
changetype: modrdn
newrdn: uid=new
deleteoldrnd: 1
#————————————————————————————————————————————————————————
```



#### ldapsearch

##### 可选option详解

| 命令 |  参数  |                    说明                    |
| :--: | :----: | :----------------------------------------: |
|  -b  | 一条DN | 基于输入的DN作为查询的根，查询根下所有内容 |
|  -L  |   -    |             使用LDIFv1格式输出             |
| -LL  |   -    |               不输出注释信息               |
| -LLL |   -    |           不输出版本以及注释信息           |

##### 例子

```sh
#显示所有uid的条目
ldapsearch -x -LLL uid
#指定uid显示
ldapsearch -x -LLL uid=test
# "+"显示隐藏属性
ldapsearch -x -LLL uid=test +
```



#### slapd与其他服务器端程序

请参考`openldap`官方文档的内容：https://www.openldap.org/doc/admin26/runningslapd.html

服务器端还有其他的CLI程序，请参考官方文档的其他内容
