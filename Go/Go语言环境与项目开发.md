# Go语言环境与项目开发

菜鸟教程：<https://www.runoob.com/go/>

参考博客：

- <https://www.jianshu.com/p/760c97ff644c>

- <https://zhuanlan.zhihu.com/p/103534192/>

- <https://blog.csdn.net/benben_2015/article/details/82227338>

- <https://www.cnblogs.com/wl-blog/p/14978415.html>

## 环境安装

系统：ubuntu-20.04 focal

Go版本：1.17.3



安装包下载地址：

1. <https://golang.org/dl/>
2. <https://golang.google.cn/dl/>

### Windows

1. 直接去官网下载.msi文件并进行安装
2. 





### linux

#### 官网二进制压缩文件安装

1. 去官网查看要下载的版本，下载安装包并解压到某路径，以`/usr/local/go`路径及1.17.3版本为例：

   ```shell
   
   ```

2. 添加到环境变量

   ```sh
   # GOPATH可以自己创建一个并指定
   export GOPATH=/opt/go
   export GOROOT=/usr/local/go
   export GOBIN=$GOROOT/bin/
   export GOTOOLDIR=$GOROOT/pkg/tool/
   export PATH=$PATH:$GOBIN:$GOTOOLDIR
   ```

#### apt-get包管理安装（不建议采用）

一般apt-get包管理的版本落后于go的官方版本，如要采用可以：

```shell
# 最老版本，默认go1.13
sudo apt-get install golang
# 指定版本，但也较为落后
sudo apt-get install golang-1.16
```

且安装好后仍然需要建立`GOPATH`文件夹并修改环境变量

## Go mod介绍与使用

1. 初始化，需要起一个模块名

   ```go
   go mod init <模块名>
   ```

2. 下载modules到本地

   ```go
   go mod download
   ```

   默认下载到`${GOPATH}/pkg/mod`和`${GOPATH}/pkg/sum`下

3. 文本模式打印依赖需求图

   ```go 
   go mod graph
   ```

4. 删除错误或者不使用的modules

   ```go
   go mod tidy
   ```

5. 查看vendor目录

   ```go
   go mod vendor
   ```

   一般在项目部署时有不能下载的部分依赖，可以将项目的依赖库下载到项目内部的vendor目录中，作为项目一部分

6. 验证依赖正确性

   ```go
   go mod verify
   ```

7. 查找依赖

   ```go
   go mod why
   ```

8. 用新的仓库地址替代引用的依赖包

   ```shell
   # go mod edit -replace=<原包地址>@<原版本>=<新包地址>@<新版本>
   go mod edit -replace=golang.org/x/crypto@v0.0.0=github.com/golang/crypto@latest
   ```

9. 清理module缓存

   ```go
   go clean -modcache
   ```

10. 查看可下载版本

    ```shell
    # go list -m -versions <包名>
    go list -m -versions github.com/gogf/gf
    ```

```go
module Test
go 1.16

require (
  github.com/gin-gonic/gin v1.6.3
)

replace (
    golang.org/x/crypto v0.0.0-20190313024323-a1f597ede03a => github.com/golang/crypto v0.0.0-20190313024323-a1f597ede03a
)
```



```sh
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.cn,https://goproxy.io,direct
```





















## go get下载项目依赖

#### 下载包命令

```text
go get <包名>@<版本>
```

例如：

1. 下载最新版本

   ```sh
   go get golang.org/x/text@latest
   ```

2. 指定分支为main

   ```sh
   go get golang.org/x/text@main
   ```

3. 指定版本，可以用哈希值和版本号

   ```sh
   # 版本号
   go get golang.org/x/text@v0.3.2
   # 哈希值
   go get golang.org/x/text@342b2e
   ```

#### 更新版本

```sh
go get -u
```

