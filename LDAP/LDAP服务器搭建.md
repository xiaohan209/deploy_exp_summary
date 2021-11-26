# LDAP服务器搭建

> 参考文章：
>
> - https://blog.csdn.net/qq_37951246/article/details/113247358
> - https://www.cnblogs.com/wilburxu/p/9174353.html
>
> 入门指南：https://www.openldap.org/doc/admin26/quickstart.html
>
> 仓库：https://www.openldap.org/software/download/OpenLDAP/openldap-release/
>
> 
>
> SLAPD使用参数手册：https://www.openldap.org/software/man.cgi?query=slapd
>
> OpenLDAP文档：https://www.openldap.org/doc/

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

open-ldap版本：



### 软件前情提要



第一步：修改hostname

127.0.0.1       localhost

127.0.1.1       ldap.ldapdomain.com alternative

192.168.5.180   ldap.ldapdomain.com

注：在 Debain 里安装 OpenLDAP 时，Debian 会提示给 LDAP 的 admin 用户设置一个密码，然后就自动地创建了一个默认的数据库，这个默认的数据库使用了一个默认的 base DN，默认情况下，Debian 会使用本机的域名来作为 base DN，比如如果我的域名是 ldapdomain.com，那么 Debian 就会使用 dc=ldapdomain,dc=com 作为我的默认 base DN




配置/etc/ldap/ldap.conf, 添加BASE 和 URI. 这里的BASE为dc=ldapdomain,dc=com  URI为ldap://192.168.5.180:389

BASE     dc=ldapdomain,dc=com

URI     ldap://192.168.5.180:389







修改openldap配置
从openldap2.4.23版本开始，所有配置保存在/etc/openldap/slapd.d目录下的cn=config文件夹内，不再使用slapd.conf作为配置文件。配置文件的后缀为 ldif，且每个配置文件都是通过命令自动生成的，任意打开一个配置文件，在开头都会有一行注释，说明此为自动生成的文件，请勿编辑。因此修改信息需要使用ldapmodify命令进行修改



安装openldap后，会有三个命令用于修改配置文件，分别为ldapadd, ldapmodify, ldapdelete，顾名思义就是添加，修改和删除。而需要修改或增加配置时，则需要先写一个ldif后缀的配置文件，然后通过命令将写的配置更新到slapd.d目录下的配置文件中去，完整的配置过程如下，跟着我做就可以了：

```shell
# 生成管理员密码,记录下这个密码，后面需要用到
slappasswd
# 按要求输入密码后会生成出{SSHA}15TXO5dtVyvBlU6MFV8kr2Cn5phgxTLB，记录下来

# 新增修改密码文件,ldif为后缀，文件名随意，不要在/etc/ldap/slapd.d/目录下创建类似文件
# 生成的文件为需要通过命令去动态修改ldap现有配置，如下，我在家目录下，创建文件
mkdir -p ~/ldap/myself-config/
vim ~/ldap/myself-config/chrootpwd.ldif
----------------------------------------------------------------------
dn: olcDatabase={0}config,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}15TXO5dtVyvBlU6MFV8kr2Cn5phgxTLB
----------------------------------------------------------------------
# 这里解释一下这个文件的内容：
# 第一行执行配置文件，这里就表示指定为 cn=config/olcDatabase={0}config 在/etc/openldap/slapd.d/目录下
# 第二行 changetype 指定类型为修改
# 第三行 add 表示添加 olcRootPW 配置项
# 第四行指定 olcRootPW 配置项的值
# 在执行下面的命令前，你可以先查看原本的olcDatabase={0}config文件，里面是没有olcRootPW这个项的，执行命令后，你再看就会新增了olcRootPW项，而且内容是我们文件中指定的值加密后的字符串

# 执行命令，修改ldap配置，通过-f执行文件
ldapadd -Y EXTERNAL -H ldapi:/// -f ~/ldap/myself-config/chrootpwd.ldif
```















## 安装软件

1. 利用`apt-get`包管理软件，进行下载安装软件

   ```shell
   sudo apt-get install slapd ldap-utils
   ```

   并根据后续软件提示进行初始化

2. 可以运行以下命令

3. b

4. c

5. 如果修改配置后需要重新加载

   ```shell
   sudo dpkg-reconfigure slapd
   ```

   

6. 添加基础的schema

   ```sh
   ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif
   ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif
   ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif
   ```

6. 











## LDAP相关管理工具

### PHP

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

3. 



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





#### 对服务器进行操作
