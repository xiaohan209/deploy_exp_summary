# WordPress

> - 官方支持手册页面：
>   - https://wordpress.org/support/
>   - 若需要中文版，可进入https://cn.wordpress.org/support/
>
> - 安装过程参考页面：
>   - https://wordpress.org/support/article/installing-multiple-blogs/
>   - https://wordpress.org/support/article/how-to-install-wordpress/
>   - https://blog.csdn.net/u011781521/article/details/82807776

## 介绍

WordPress是基于PHP和MySQL的免费开源内容管理系统（CMS）。

WordPress始于2003年，最开始仅为一款简单的博客系统，但现已发展成为具有数千款插件，小工具和主题功能完整的CMS系统。它是根据开源协议通用公共许可证（GPLv2或更高版本）进行授权。

## 安装要求

- PHP版本7.4+以上
- MySQL版本5.6+或者MariaDB版本10.1+以上
- Nginx或带mod_rewrite模块的Apache
- HTTPS加密访问支持
- 空间至少需要100MB
- 数据库至少占用20MB

## 安装过程

**系统：ubuntu 20.04 focal**

**WordPress：5.8.2**



1. 安装必要的系统工具

   ```sh
   sudo apt-get install wgetc tar
   ```

2. 下载运行环境，可以使用LAMP一键安装包或LNMP一键安装包，也可以单独安装：

   1. Mysql-5.6以上版本或MariaDB-10.1以上版本
   2. Nginx或Apache
   3. PHP-7.4以上版本

3. 下载解压WordPress包

   ```sh
   # 下载最新版WordPress包
   wget https://wordpress.org/latest.tar.gz -O - | tar -xzv
   # 如果访问速度较慢可以换链接下载
   wget https://cn.wordpress.org/latest-zh_CN.tar.gz -O - | tar -xzv
   # 下载指定版本，登录https://cn.wordpress.org/download/releases/进行查看，选定版本后替换链接即可
   wget https://cn.wordpress.org/wordpress-<$your_version_number>-zh_CN.tar.gz | tar -xzv
   ```

4. 创建数据库

   1. 先对数据库进行初始化，设置root密码
   2. 创建wordpress所使用的数据库，示例命名为example_wordpress
   3. 建立新用户，示例名为test，密码password
   4. 将example_wordpress的权限授予新用户

5. 对WordPress进行初步配置（也可以跳过，在安装脚本中进行配置）：

   在wordpress文件夹中编辑或新建`wp-config.php`，并写入内容，可以参考[示例配置](https://wordpress.org/support/article/editing-wp-config-php/)

   ```php
   
   /** wordpress要使用的数据库名称 */
   define( 'DB_NAME', 'example_wordpress' );
   /** MySQL 登录的用户名 */
   define( 'DB_USER', 'test' );
   /** MySQL 登录用户的密码 */
   define( 'DB_PASSWORD', 'password' );
   /** MySQL 所在的主机地址 */
   define( 'DB_HOST', 'localhost' );
   /** MySQL 使用的字符集,不知道如何配置可以不添加进去 */
   define( 'DB_CHARSET', 'utf8' );
   /** MySQL 排序规则通常应留空，不知道如何配置可以不添加进去 */
   define( 'DB_COLLATE', '' );
   
   ```

   

6. 设置根目录及反向代理：
   示例中将根目录设置为/var/www/html/wordpress

   ```sh
   # 移动wordpress文件夹到/var/www/html/下
   sudo mv ~/wordpress /var/www/html/
   ```

   在nginx或apache中设置代理，根文件夹为`/var/www/html/wordpress`

7. 访问页面运行安装脚本
   
8. 完成安装

## 设置WordPress







## 迁移WordPress

> - https://zhuanlan.zhihu.com/p/111896376

#### 更换服务器，不换域名

1. 备份原服务器wordpress文件夹下所有内容
2. 进入phpmyadmin备份全部mysql数据库文件
3. 在新站点上传备份文件并恢复备份的数据库文件

#### 更换域名

1. 进入phpmyadmin后台

2. 执行如下代码

   ```mysql
   UPDATE wp_options SET option_value = replace(option_value, 'www.old.com','www.new.com') ;    
   UPDATE wp_posts SET post_content = replace(post_content, 'www.old.com','www.new.com') ;    
   UPDATE wp_posts SET guid = replace(guid, 'www.old.com','www.new.com') ;    
   UPDATE wp_comments SET comment_content = replace(comment_content, 'www.old.com', 'www.new.com') ;    
   UPDATE wp_comments SET comment_author_url = replace(comment_author_url, 'www.old.com', 'www.new.com') ;
   ```

   其中`www.old.com`为原域名，`www.new.com`为新域名

3. 