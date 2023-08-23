# Gitlab-Runner

> [官方说明](https://docs.gitlab.com/runner/)
>
> [官方下载教程](https://docs.gitlab.com/runner/install/index.html)
>
> [官方配置说明及示例](https://docs.gitlab.com/runner/configuration/advanced-configuration.html)
>
> [deb包下载版本库](https://gitlab-runner-downloads.s3.amazonaws.com/latest/index.html)
>
> 参考博客：
>
> - https://blog.csdn.net/weixin_43878297/article/details/119865646
> - https://www.jianshu.com/p/df433633816b

## 介绍





## 安装

### 存储库安装包方式安装

操作系统：ubuntu 20.04 focal

1. 下载基础依赖包：

   ```shell
   sudo apt update
   sudo apt install curl
   ```

2. 添加官方存储库：

   1. 直接使用官方脚本添加

      ```shell
      curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | sudo bash
      ```
      
   2. 添加第三方存储库（清华源为例）
   
      先添加存储库gpg公钥：
   
      ```shell
      curl https://packages.gitlab.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/gitlab-runner.gpg > /dev/null
      ```
   
      将以下内容写入：
   
      ```shell
      echo deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/trusted.gpg.d/gitlab-runner.gpg] https://mirrors.tuna.tsinghua.edu.cn/gitlab-runner/ubuntu $(lsb_release -c -s) main | sudo tee /etc/apt/sources.list.d/gitlab-runner.list > /dev/null
      ```


3. 固定存储库（可选）：

   ```shell
   cat <<EOF | sudo tee /etc/apt/preferences.d/pin-gitlab-runner.pref
   Explanation: Prefer GitLab provided packages over the Debian native ones
   Package: gitlab-runner
   Pin: origin packages.gitlab.com
   Pin-Priority: 1001
   EOF
   ```

4. 通过apt安装：

   1. 安装最新版：

      ```shell
      sudo apt-get install gitlab-runner
      ```

   2. 安装特定版本：

      ```shell
      # 查看当前库中包含的gitlab-runner版本
      apt-cache madison gitlab-runner
      # 选定版本后，例如为10.0.0，则可以指定版本进行安装
      sudo apt-get install gitlab-runner=10.0.0
      ```
      
   3. 如果更新期间存在问题请查看[更新相关内容](#更新相关)

5. 到此安装完成，可以使用了，可以查看一下当前版本是否正确：

   ```shell
   gitlab-runner -v
   ```



### 手动下载安装包

这种方法没有办法自动更新，需要每次找到新版本手动下载安装包，按照安装时同样的方法进行更新

操作系统：ubuntu 20.04 focal

1. 寻找版本：
   登录发布在Amazon的S3存储上的[deb包下载版本库](https://gitlab-runner-downloads.s3.amazonaws.com/latest/index.html)，找到需要的版本后复制连接进行下载

2. 下载安装包：

   ```shell
   # 如果确定了自己的链接则替换下面指令中的${YOUR_LINK}为复制好的链接
   curl -LJO ${YOUR_LINK}
   # 如果在Linux上不确定应该下载哪个版本也可以执行以下指令
   curl -LJO "https://gitlab-runner-downloads.s3.amazonaws.com/latest/deb/gitlab-runner_$(dpkg --print-architecture).deb"
   ```

3. 手动安装：

   ```shell
   # 上一步默认方式下载的文件即可直接进行安装
   sudo dpkg -i gitlab-runner_$(dpkg --print-architecture).deb
   # 如果上述指令不行，也可以找到下载的文件进行安装即可
   sudo dpkg -i ${YOUR_FILE}
   ```



### 二进制文件直接导入

这种方法简单粗暴，但是每次都需要自己进行手动更新，且需要自己管理

#### 安装方式：

1. 直接下载二进制到`/usr/local/bin`中：

   ```shell
   sudo curl -L --output /usr/local/bin/gitlab-runner "https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-$(dpkg --print-architecture)"
   ```

2. 赋予执行权限：

   ```shell
   sudo chmod +x /usr/local/bin/gitlab-runner
   ```

3. 创建`Gitlab Runner`用户：

   ```shell
   sudo useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash
   ```

4. 安装并作为服务运行：

   ```shell
   sudo gitlab-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner
   sudo gitlab-runner start
   ```

#### 更新方式：

1. 停止服务：

   ```shell
   sudo gitlab-runner stop
   ```

2. 下载新二进制文件：

   ```shell
   sudo curl -L --output /usr/local/bin/gitlab-runner "https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-$(dpkg --print-architecture)"
   ```

3. 赋予执行权限：

   ```shell
   sudo chmod +x /usr/local/bin/gitlab-runner
   ```

4. 再次启动服务：

   ```shell
   sudo gitlab-runner start
   ```

### Docker容器安装

1. 安装Docker
2. a







## 使用

### 更新相关

#### 替换新签名

如果出现签名过期的情况：

```shell
# The following signatures couldn’t be verified because the public key is not available: NO_PUBKEY 3F01618A51312F3F
# https://packages.gitlab.com/gitlab/gitlab-ee/el/7/x86_64/repodata/repomd.xml: [Errno -1] repomd.xml signature could not be verified for gitlab-ee
# W: 校验数字签名时出错。此仓库未被更新，所以仍然使用此前的索引文件。GPG 错误：https://packages.gitlab.com/runner/gitlab-runner/ubuntu bionic InRelease: 下列签无效： EXPKEYSIG 3F01618A51312F3F GitLab B.V. (package repository signing key) <packages@gitlab.com>
# W: 无法下载 https://packages.gitlab.com/runner/gitlab-runner/ubuntu/dists/bionic/InRelease  下列签名无效： EXPKEYSIG 3F01618A51312F3F GitLab B.V. (package repitory signing key) <packages@gitlab.com>
```

需要替换签名：

```shell
curl https://packages.gitlab.com/gpg.key 2> /dev/null | sudo apt-key add - &>/dev/null
```





