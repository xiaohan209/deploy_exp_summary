# Docker-Compose

> [官方文档](https://docs.docker.com/compose/)
>
> [官方配置文件说明](https://docs.docker.com/compose/compose-file/)
>
> 参考博客：
>
> - <https://blog.csdn.net/pushiqiang/article/details/78682323>
> - <https://www.jianshu.com/p/c227dd28654c>
> - <https://www.jianshu.com/p/2217cfed29d7>
> - https://blog.csdn.net/qq_36148847/article/details/79427878

## 介绍：

`Docker Compose`是一个用来定义和运行复杂应用的Docker工具。

一个应用如果需要使用Docker中的多个容器，则可以利用Docker Compose通过一个配置文件来管理多个Docker容器，在配置文件中，所有的容器通过services来定义，然后使用docker-compose脚本来启动，停止和重启应用。

docker-compose内部不含docker，是额外的工具，因此在安装时不仅需要安装Docker-Compose，还需要安装Docker。

## 安装：

1. 访问[**Github仓库**](https://github.com/docker/compose/releases/)查看需要下载的版本

2. 下载文件到本地：

   1. 直接复制需要下载的文件链接：

      ```shell
      # 复制链接后替换${your_link}
      sudo curl -L ${your_link} -o /usr/local/bin/docker-compose
      ```

   2. 不清楚需要哪个文件可以查看版本后输入：

      ```shell
      # 需要先查看下载的版本，并替换掉下行的${your_version}
      sudo curl -L https://github.com/docker/compose/releases/download/${your_version}/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
      ```

3. 添加可执行权限：

   ```shell
   sudo chmod +x /usr/local/bin/docker-compose
   ```

4. 测试安装结果：

   ```shell
   docker-compose --version
   ```

   显示版本则为成功



## 使用：

docker-compose.yml配置文件示例，具体可以参考[官方配置文档](https://docs.docker.com/compose/compose-file/)：

```yaml
services:
  backend:
    build: backend
    secrets:
      - db-password
    depends_on:
      - db
  db:
    image: postgres
    restart: always
    secrets:
      - db-password
    volumes:
      - db-data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=example
      - POSTGRES_PASSWORD_FILE=/run/secrets/db-password
    expose:
      - 5432
    
  proxy:
    build: proxy
    ports:
      - 80:80
    depends_on: 
      - backend
volumes:
  db-data:
secrets:
  db-password:
    file: db/password.txt
```

