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
>- https://evanzj.com/2020/02/04/ShareLatexDocker/
>- https://lisz.me/tech/docker/sharelatex.html

## 介绍



## 安装

1. 安装docker

2. 安装docker-compose

3. 下载overleaf toolkit启动脚本包：

   ```shell
   git clone https://github.com/overleaf/toolkit.git ./overleaf
   ```

   下载好后，`overleaf`文件夹下有`doc`文件夹，内部为此版本的官方脚本教程，可以优先参考

   ```shell
   # 以下为overleaf/doc的目录结构
   .
   ├── ce-upgrading-texlive.md
   ├── configuration.md
   ├── dependencies.md
   ├── docker-compose.md
   ├── getting-server-pro.md
   ├── img
   │   └── upgrade-demo.gif
   ├── ldap.md
   ├── overleaf-rc.md
   ├── overview.md
   ├── persistent-data.md
   ├── quick-start-guide.md
   ├── README.md
   ├── saml.md
   ├── sandboxed-compiles.md
   ├── the-doctor.md
   ├── tls-proxy.md
   └── upgrading.md
   ```

4. 初始化配置文件：

   ```shell
   cd ~/overleaf
   # 不需要TLS连接
   ./bin/init
   # 如果需要TLS连接则执行
   ./bin/init --tls
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
   3. `nginx`文件夹：所有TLS相关的证书等配置

5. 修改配置文件：

   1. 修改overleaf.rc：
      打开`config/overleaf.rc`之后：

      ```shell
      #### Overleaf RC ####
      # 可以修改为自己想要的名字
      PROJECT_NAME=overleaf
      
      # Sharelatex container
      SHARELATEX_DATA_PATH=data/sharelatex
      SERVER_PRO=false
      # 修改为最终浏览器访问时输入的hostname，如果不确定可以直接使用0.0.0.0监听所有连接
      SHARELATEX_LISTEN_IP=0.0.0.0
      # 默认HTTP采用80端口，如果有需要可以采用其他端口
      SHARELATEX_PORT=80
      
      # Sibling Containers
      SIBLING_CONTAINERS_ENABLED=false
      DOCKER_SOCKET_PATH=/var/run/docker.sock
      
      # Mongo configuration
      MONGO_ENABLED=true
      MONGO_DATA_PATH=data/mongo
      
      # Redis configuration
      REDIS_ENABLED=true
      REDIS_DATA_PATH=data/redis
      
      # TLS proxy configuration (optional)
      # See documentation in doc/tls-proxy.md
      # 如果不需要开启TLS则NGINX_ENABLED需要设置为false，否则会检查配置文件及证书等
      NGINX_ENABLED=false
      NGINX_CONFIG_PATH=config/nginx/nginx.conf
      # 修改为NGINX需要使用的PORT
      NGINX_HTTP_PORT=80
      # 修改为NGINX需要监听的IP，注意NGINX监听的IP与Port不能与SHARELATEX监听的IP和Port完全一致
      NGINX_HTTP_LISTEN_IP=127.0.1.1
      NGINX_TLS_LISTEN_IP=127.0.1.1
      
      TLS_PRIVATE_KEY_PATH=config/nginx/certs/overleaf_key.pem
      TLS_CERTIFICATE_PATH=config/nginx/certs/overleaf_certificate.pem
      TLS_PORT=443
      ```

      其中主要修改`SHARELATEX_LISTEN_IP`以及`SHARELATEX_PORT`为外部访问当前主机需要的host和port

      如果需要使用Nginx容器，则先要部署Nginx的证书等，然后再对以上文件的NGINX部分进行配置

   2. <span id="env">修改variables.env</span>

      打开`config/variables.env`之后：

      ```shell
      SHARELATEX_APP_NAME=Our Overleaf Instance
      
      ENABLED_LINKED_FILE_TYPES=project_file,project_output_file
      
      # Enables Thumbnail generation using ImageMagick
      ENABLE_CONVERSIONS=true
      
      # Disables email confirmation requirement
      EMAIL_CONFIRMATION_DISABLED=true
      
      # temporary fix for LuaLaTex compiles
      # see https://github.com/overleaf/overleaf/issues/695
      TEXMFVAR=/var/lib/sharelatex/tmp/texmf-var
      
      ## Nginx
      # NGINX_WORKER_PROCESSES=4
      # NGINX_WORKER_CONNECTIONS=768
      
      ## Set for TLS via nginx-proxy
      # SHARELATEX_BEHIND_PROXY=true
      # SHARELATEX_SECURE_COOKIE=true
      # 可以修改为自己设置的网站地址，用于后续发放修改密码等连接使用
      SHARELATEX_SITE_URL=http://overleaf.example.com
      SHARELATEX_NAV_TITLE=Our Overleaf Instance
      # SHARELATEX_HEADER_IMAGE_URL=http://somewhere.com/mylogo.png
      # SHARELATEX_ADMIN_EMAIL=support@example.com
      
      # SHARELATEX_LEFT_FOOTER=[{"text":"Powered by Overleaf © 2021", "url": "https://www.overleaf.com"}, {"text": "Contact your support team", "url": "mailto:support@example.com"} ]
      # SHARELATEX_RIGHT_FOOTER=[{"text":"Hello I am on the Right"}]
      
      # SHARELATEX_EMAIL_FROM_ADDRESS=team@example.com
      
      # SHARELATEX_EMAIL_AWS_SES_ACCESS_KEY_ID=
      # SHARELATEX_EMAIL_AWS_SES_SECRET_KEY=
      
      # SHARELATEX_EMAIL_SMTP_HOST=smtp.example.com
      # SHARELATEX_EMAIL_SMTP_PORT=587
      # SHARELATEX_EMAIL_SMTP_SECURE=false
      # SHARELATEX_EMAIL_SMTP_USER=
      # SHARELATEX_EMAIL_SMTP_PASS=
      # SHARELATEX_EMAIL_SMTP_NAME=
      # SHARELATEX_EMAIL_SMTP_LOGGER=false
      # SHARELATEX_EMAIL_SMTP_TLS_REJECT_UNAUTH=true
      # SHARELATEX_EMAIL_SMTP_IGNORE_TLS=false
      # SHARELATEX_CUSTOM_EMAIL_FOOTER=This system is run by department x
      
      
      TEX_LIVE_DOCKER_IMAGE=quay.io/sharelatex/texlive-full:2020.1
      ALL_TEX_LIVE_DOCKER_IMAGES=quay.io/sharelatex/texlive-full:2020.1,quay.io/sharelatex/texlive-full:2019.1
      ```

      可以修改`SHARELATEX_SITE_URL`为自己的网站地址，为了后续发放修改密码等链接来进行使用，否则overleaf的网站实例生成URL时会使用`localhost`且不会有端口号，需要在输入链接时自己补全所有的host以及port

6. 启动镜像：

   ```shell
   # 容器日志输出到标准输出上
   ~/overleaf/bin/up
   # 不输出日志，在后台运行
   ~/overleaf/bin/up -d
   ```

   `bin/up`脚本有创建容器(如果当前容器不存在)功能，创建后若使用`CTRL-C`停止，容器依旧存在，但并不运行。

   容器采用`bin/start`的脚本启动时，只是对停止的容器进行启动，并在后台运行，若未创建容器则启动失败。

   如果仍然有需要也可以采用`bin/docker-compose`的方式来直接对docker进行控制。

7. 到此就可以进入界面，但此时的容器仍然为精简版，因此需要更新：

   1. 查看当前容器的tlmgr版本及设置：

      ```shell
      # sharelatex 可以根据需要调整为当前使用的容器名
      # 查看tlmgr版本
      docker exec sharelatex tlmgr --version
      # 查看当前tlmgr设置
      docker exec sharelatex tlmgr option
      ```

   2. 如果网络环境不好，需要更换源，则：

      ```shell
      # sharelatex为当前容器名，以清华源为示例，可以更换其他源
      docker exec sharelatex tlmgr option repository https://mirrors.tuna.tsinghua.edu.cn/CTAN/systems/texlive/tlnet
      ```

   3. 进行自更新及安装完整版：

      ```shell
      # sharelatex 可以根据需要调整为当前使用的容器名
      docker exec sharelatex tlmgr update --self --all
      docker exec sharelatex tlmgr install tikzlings tikzmarmots tikzducks
      docker exec sharelatex tlmgr install scheme-full
      ```

      

8. 根据需要可以保存完整版为新镜像：

   ```shell
   docker commit sharelatex sharelatex/sharelatex:with-texlive-full
   ```

   如果需要使用新镜像创建容器，则需要新添加`lib/docker-compose.override.yml`：

   ```yaml
   ---
   # verison需要与docker-compose.base.yml中一致
   version: '2.2'
   services:
       sharelatex:
           image: sharelatex/sharelatex:with-texlive-full
   ```

9. 可以重启镜像进行使用了：

   ```shell
   bin/stop
   bin/start
   ```

   如果需要删除原来创建的`sharelatex`容器，则可以在`overleaf`文件夹路径下执行：

   ```shell
   bin/docker-compose rm -f sharelatex
   ```

   

## 使用

### 更新

在`overleaf`文件夹执行`/bin/upgrade`并跟随命令行提示进行操作

需要注意的是upgrade时脚本会检测当前的docker镜像版本，如果有新版本则会询问是否替换，替换过后会生成`config/__old-version`文件，留作副本。

另外替换完新版本会询问是否停止当前容器，并启动新容器。此时可以取消这一步手动停止并备份文件等；也可以继续，并将停止的原容器中需要的数据导出。

过程大致如下：

![Demonstration of the upgrade script](./img/upgrade-demo.gif)

### 配置管理员

1. 网页配置：

   1. 浏览器中打开`http://your-domain/launchpad`，根据自己的配置可以使用https协议，以及修改`your-domain`为自己服务器的ip+port或域名
   2. 填入管理员的用户名密码，进行注册
   3. 注册成功登录后就会显示成为管理员了

2. 进入docker的容器进行配置：

   1. 执行增添管理员的命令

      ```shell
      # 其中exec后面填入的为容器名，一般第一次修改时为sharelatex，如果有需要请填写当前需要进入的容器名
      # 请将create-admin中${your_admin}改为正确的邮箱用户名，example.com改为正确的邮箱域名
      docker exec sharelatex /bin/bash -c "cd /var/www/sharelatex; grunt user:create-admin --email=${your_admin}@example.com"
      ```

   2. 填写密码：
      docker容器会执行指令并最终返回一个URL用来设置刚刚创建的管理员密码，打开URL并填写密码，若URLhostname等设置有误，请参考[variable设置](#env)

   

### 用户管理

#### 添加

1. 完成[管理员配置](# 配置管理员)
2. 使用管理员身份登录`http://your-domain/admin/register`进行成员添加，其中your-domain为真实使用的ip地址或域名

#### 删除

```shell
docker exec sharelatex /bin/bash -c "cd /var/www/sharelatex; grunt user:delete --email=${delete_user}@example.com"
```

#### 查看

```shell
# 进入数据库
sudo docker exec -it mongo bash
# 列出sharelatex的用户
mongoexport -d sharelatex -c users -f email
```



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