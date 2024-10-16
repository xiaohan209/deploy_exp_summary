# Grafana

>[官方下载页面](https://grafana.com/grafana/download)
>
>[官方插件页面](https://grafana.com/grafana/plugins/)
>
>[官方Dashboard显示界面分享论坛](https://grafana.com/grafana/dashboards/)
>
>[官方Grafana配置说明](https://grafana.com/docs/grafana/latest/setup-grafana/configure-grafana/)

## 介绍

Grafana开源软件可以进行查询、预警和跟踪各种仪表指标等，可以将时间序列数据库 (TSDB) 数据转化为具有洞察力的图表和可视化数据。

通过模板变量，可以创建可重复用于多种不同用例的仪表盘。这些模板不会对值进行硬编码，因此，例如，如果你有一个生产服务器和一个测试服务器，就可以在这两个服务器上使用相同的仪表盘。

例如Grafana可以有这样的图表：

![](./complex-dashboard-example.png)

## 安装

> [官方下载页面](https://grafana.com/grafana/download)
>
> [清华源下载安装帮助页面](https://mirrors.tuna.tsinghua.edu.cn/help/grafana/)

### APT包管理

操作系统：ubuntu 22.04 jammy

1. 安装其他依赖：

   ```shell
   sudo apt-get update
   sudo apt-get install wget gpg apt-transport-https
   ```

2. 添加GPG Key：

   ```shell
   wget -qO - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/grafana.key > /dev/null
   ```

3. 添加仓库：

   1. 添加官方稳定版，则为：

      ```shell
      # 官方镜像源
      echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/trusted.gpg.d/grafana.key] https://mirrors.tuna.tsinghua.edu.cn/grafana/apt/ stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
      # 华为镜像
      echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://mirrors.huaweicloud.com/redis/ $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list
      ```

   2. 添加Beta不稳定最新版，则为：

      ```shell
      echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/trusted.gpg.d/grafana.key] https://mirrors.tuna.tsinghua.edu.cn/grafana/apt/ beta main" | sudo tee /etc/apt/sources.list.d/grafana.list
      ```

4. 安装Grafana

   ```shell
   sudo apt-get update
   sudo apt-get install grafana
   ```

5. 修改配置：

   一般在/etc/grafana/目录下可以看到自定义的配置文件，可以对http_port等进行修改，以及ldap配置文件进行修改

6. 检查是否安装成功

   ```shell
   # 检查是否已经下载好grafana二进制文件
   grafana --version
   # 查看当前服务状态
   service grafana-server status
   ```

### Deb包直接安装

1. 进入[下载页面](https://grafana.com/grafana/download)，选择合适的系统和软件版本

2. 安装其他依赖

   ```bash
   sudo apt-get install -y adduser libfontconfig1
   ```

3. 获取文件

   ```bash
   wget https://dl.grafana.com/enterprise/release/grafana-enterprise_10.1.0_amd64.deb
   ```

4. 使用dpkg安装

   ```bash
   sudo dpkg -i grafana-enterprise_10.1.0_amd64.deb
   ```

   





## 使用

### 数据源

1. 点击首页的左上角导航侧边栏
2. 选择Connections
3. 点击"Add new data source"并填写配置以添加数据源，或者点击下方显示的已有数据源重新配置

### 插件

> [官方插件页面](https://grafana.com/grafana/plugins/)

### Dashboard

> [官方Dashboard显示界面分享论坛](https://grafana.com/grafana/dashboards/)

1. 进入Dashboard分享界面，搜索已有的前端显示布局
2. 找到适合的界面，点击下载json配置文件或者复制作品ID
3. 进入已经部署的Grafana前端，进入Dashboard界面
4. 点击New按钮下拉选择Import选项
5. 复制ID或者上传配置文件即可引入该前端布局
