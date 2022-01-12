# OverLeaf

>[官方代码仓库](https://github.com/overleaf/overleaf/)
>
>[官方工具包](https://github.com/overleaf/toolkit/)
>
>[官方新版教程](https://github.com/overleaf/toolkit/blob/master/doc/quick-start-guide.md)：采用自动化脚本部署，对安装部分较为详细
>
>[官方老版教程](https://github.com/overleaf/toolkit/blob/master/doc/quick-start-guide.md)：单独拉取镜像，并对安装后的管理员有所设置
>
>参考博客：
>
>- https://www.jianshu.com/p/549901fe73a5
>- https://www.zhihu.com/question/62943097/answer/2217087254
>- https://www.ivdone.cn/article/292.html

## 介绍



## 安装

1. 安装docker

2. 安装docker-compose

3. 下载overleaf toolkit启动脚本包：

   ```shell
   git clone https://github.com/overleaf/toolkit.git ./overleaf
   ```

4. 初始化配置文件：

   ```shell
   cd ~/overleaf
   ./bin/init
   ```

   如果不能执行脚本，可能是由于没有执行权限造成的，可以添加权限：

   ```shell
   cd ~/overleaf/bin
   chmod +x *
   ```

   初始化后所有配置文件复制到了`overleaf/config`文件夹中，应该包含：

   1. `overleaf.rc`：启动镜像的脚本所需的各种配置文件
   2. `variables.env`：添加到docker容器中的环境变量
   3. `version`：使用的docker镜像版本

5. 启动镜像：

   ```shell
   ~/overleaf/bin/up
   ```

   此时会有docker容器的日志输出到屏幕上，`CTRL-C`终止后容器也会停止。如果采用`./bin/start`的脚本启动，则启动后不会停留在输出日志阶段，docker容器在后台运行，如果仍然有需要也可以采用`./bin/docker-compose`的方式来直接对docker进行控制。

6. a

7. a

8. 

```bash
docker exec sharelatex tlmgr option repository https://mirrors.tuna.tsinghua.edu.cn/CTAN/systems/texlive/tlnet
docker exec sharelatex tlmgr update --self --all
docker exec sharelatex tlmgr install scheme-full
docker commit sharelatex sharelatex/sharelatex:with-texlive-full

docker exec 容器id号 /bin/bash -c "cd /var/www/sharelatex; grunt user:create-admin --email=joe@example.com"
docker exec sharelatex /bin/bash -c "cd /var/www/sharelatex; grunt user:create-admin --email=joe@example.com"
```

## 使用

### 配置管理员

1. 网页配置：

   1. 浏览器中打开`http://your-domain/launchpad`，根据需要可以使用https协议，以及修改`your-domain`为自己服务器的ip或域名
   2. 填入管理员的用户名密码，进行注册
   3. 注册成功登录后就会显示成为管理员了

2. 进入docker的容器进行配置：

   1. 执行

      ```shell
      # 其中exec后面填入的为容器名，一般第一次修改时为sharelatex，如果有需要请填写当前需要进入的容器名
      # 请将create-admin中${your_admin}改为正确的邮箱用户名，example.com改为正确的邮箱域名
      docker exec sharelatex /bin/bash -c "cd /var/www/sharelatex; grunt user:create-admin --email=${your_admin}@example.com"
      ```

      

3. 

4. 

   ```shell
   
   ```

   

   

   `http://localhost/`如果您已将端口转发更新为类似的内容`ports: - 8080:80`，上述命令将始终生成一个 URL ，您应该使用正确的端口访问密码确认页面：`http://localhost:8080/user/password/set?passwordResetToken=<token>`. 另一种选择是将`SHARELATEX_SITE_URL`环境变量设置为`http://localhost:8080`or `http://sharelatex.mydomain.com`。

### 创建用户

1. 完成[管理员配置](# 配置管理员)
2. 使用管理员身份登录`http://your-domain/admin/register`进行成员添加，其中your-domain为真实使用的ip地址或域名

### 配置LDAP

> 参考：
>
> - <https://github-wiki-see.page/m/overleaf/overleaf/wiki/Server-Pro:-LDAP-Config>
> - <https://github.com/smhaller/ldap-overleaf-sl>
> - <https://github.com/worksasintended/overleaf_ldap>
> - <https://github.com/overleaf/toolkit/blob/master/doc/ldap.md>
> - <https://github.com/overleaf/overleaf/wiki/Server-Pro%3A-LDAP-Config>

### 配置SMTP

>参考：
>
>- https://github.com/overleaf/overleaf/wiki/Configuring-SMTP-Email