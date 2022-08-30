# Rust 语言

> [Rust官网](https://www.rust-lang.org/)
> [官网文档](https://doc.rust-lang.org/)
> [菜鸟教程](https://www.runoob.com/rust/rust-tutorial.html)
> [Rust Play在线运行平台](https://play.rust-lang.org/)
> 

## 简介

Rust 语言是一种高效、可靠的通用高级语言。其高效不仅限于开发效率，它的执行效率也是令人称赞的，是一种少有的兼顾开发效率和执行效率的语言。

Rust 语言由 Mozilla 开发，最早发布于 2014 年 9 月。Rust 的编译器是在 MIT License 和 Apache License 2.0 双重协议声明下的免费开源软件

Rust语言特点：

- 高性能: Rust 速度惊人且内存利用率极高。由于没有运行时和垃圾回收，它能够胜任对性能要求特别高的服务，可以在嵌入式设备上运行，还能轻松和其他语言集成。

- 可靠性: Rust 丰富的类型系统和所有权模型保证了内存安全和线程安全，让您在编译期就能够消除各种各样的错误。

- 生产力: Rust 拥有出色的文档、友好的编译器和清晰的错误提示信息，还集成了一流的包管理器和构建工具，智能地自动补全和类型检验的多编辑器支持，以及自动格式化代码等等。

## 安装

### Windows Powershell使用Rustup工具包

> 参考博客：
> - https://blog.csdn.net/weixin_43882409/article/details/87616268

安装过程:

1. 从官网下载[rustup-init.exe](https://static.rust-lang.org/rustup/dist/x86_64-pc-windows-msvc/rustup-init.exe)

2. 安装rustup:
   1. 选择C++前置环境: 选择MSVC环境则自动安装微软C++库，GNU环境需要手动下载GNU套件
   2. 定制安装选项: 具体可查看[Rustup设置说明](#rustup设置)
   3. 

3. 配置环境变量:

    CARGO_HOME设置为.cargo

    RUSTUP_HOME设置为.  

4. 换源(如果需要的话):
    ```bash
    # 设置环境变量
    RUSTUP_DIST_SERVER=https://mirrors.ustc.edu.cn/rust-static
    RUSTUP_UPDATE_ROOT=https://mirrors.ustc.edu.cn/rust-static/rustup
    ```

### Windows WSL使用Rustup工具包

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

### Linux


## 使用

### VSCode开发配置过程


### Rustup设置

rustup的配置文件在用户目录下的`.rustup/settings.toml`中

#### default_host_triple


#### default_toolchain

#### profile



#### version

目前安装的版本号，安装时无法设置

#### modify_path_variable

只有在安装的时候才会显示的选项，代表着是否自动对环境变量进行修改，将cargo的可执行文件添加到环境变量的PATH中，一般保持yes即可
