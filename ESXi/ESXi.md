# ESXi









## 网络

> 参考博客：
>
> - https://wenku.baidu.com/view/2fecea0f964bcf84b9d57bbd.html
> - https://blog.csdn.net/kepa520/article/details/78700708/
> - https://wenku.baidu.com/view/e9d2b4444b7302768e9951e79b89680203d86b3f.html
> - https://blog.51cto.com/andygao/817518

配置文件在`/etc/vmware/esx.conf`中

### esxcli命令

正则格式为：

```shell
esxcli [options] {namespace}+ {cmd} [cmd options]
```

#### Option说明：

|         命令          |                             说明                             |
| :-------------------: | :----------------------------------------------------------: |
| --formatter=FORMATTER | 指定后续cmd命令使用的格式，FORMATTER可以是xml，keyvalue，csv |
|        --debug        |                        开启debug模式                         |
|       --version       |                         显示当前版本                         |
|      -?或--help       |                         显示帮助界面                         |



#### Namespace说明

|   命令   |                             说明                             |
| :------: | :----------------------------------------------------------: |
|  device  |                        设备管理器指令                        |
|  esxcli  |            esxcli系统本身的命令，用于获取其他信息            |
|   fcoe   |          VMware FCoE(Fiber Channel on Ethernet)指令          |
| graphics |                        VMware图形指令                        |
| hardware |                VMKernel硬件属性和硬件配置指令                |
|  iscsi   |  VMware iSCSI(Internet Small Computer System Interface)指令  |
| network  | 维护ESX主机网络指令：包括主机的IP、DNS等网络设置配置指令，<br />以及vSwitch、端口组等虚拟网络组件的配置指令 |
|   nvme   |             VMware NVMe driver esxcli extensions             |
|   rdma   | Operations that pertain to remote direct memory access (RDMA) protocol stack on an ESX host |
|  sched   | VMKernel system properties and commands for configuring scheduling related functionality |
| software |         Manage the ESXi software image and packages          |
| storage  |                   VMware storage commands                    |
|  system  | VMKernel system properties and commands for configuring properties of the kernel core system and related system services |
|    vm    | A small number of operations that allow a user to Control Virtual Machine operations |
|   vsan   |                     VMware vSAN commands                     |





例子：

1. 查看系统版本等信息：

   ```shell
   # vmware命令可显示部分版本信息
   vmware -v
   # 使用esxcli命令则更为详细
   esxcli system version get
   ```

2. 查看系统时间：

   ```shell
   esxcli system time get
   ```

3. 修改系统时间

   ```shell
   
   esxcli system time set -y=2016 -M=9 -d=13 -H=10 -m=9
   ```

   esxcli system time set <options>

     修改系统时间，例子：

   Cmd options:

    -d|--day=<long>    Day

    -H|--hour=<long>    Hour

    -m|--min=<long>    Minute

    -M|--month=<long>   Month

    -s|--sec=<long>    Second

    -y|--year=<long>    Year

   

4. 设置开启/关闭维护模式

   ```shell
   # 设置维护模式，true为开启，false为关闭
   esxcli system maintenanceMode set --enable true/false
   # 查看当前是否为维护模式
   esxcli system maintenanceMode get
   ```

5. 系统重启/关机（必须处于维护模式，否则命令不生效）：

   ```shell
   esxcli system shutdown reboot/poweroff
   ```

6. 查看接口ipv4地址:

   ```shell
   esxcli network ip interface ipv4 get
   ```

7. 查看路由表：

   ```shell
   esxcli network ip route ipv4 list
   ```

8. 查看主机网卡列表或up-link列表：

   ```shell
   esxcli network nic list
   ```

9. 关闭/打开nic接口：

   ```shell
   esxcli network nic down/up -n=vmnic1
   ```

10. 查看磁盘列表：

    ```shell
    esxcli storage core device list
    ```

11. 







esxtop

n进入网络界面

按f，按c，将PNIC列选择后回车

查看USED-BY和TEAM-PNIC对应关系