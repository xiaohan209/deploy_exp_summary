# Netplan

> [Netplan官网](https://netplan.io/)
>
> 参考博客：
>
> - <https://www.jianshu.com/p/174656635e74>
> - <https://www.jianshu.com/p/1378e6abd94d>
> - <https://blog.csdn.net/terence_csdn/article/details/106450541>
> - <https://blog.51cto.com/qu87w/2454285>
> - <https://blog.csdn.net/djstavaV/article/details/112261818>

## 介绍

/etc/netplan/*.yaml 中有 Snappy、Server、Client、MaaS、cloud-init 的中央网络配置文件。所有安装程序只生成这样的文件，不再生成 /etc/network/interfaces。还有一个netplan命令行工具来驱动一些操作。

系统在早期引导期间使用“网络渲染器”进行配置，该渲染器读取 /e/n/*.yaml 并将配置写入 /run 以将设备控制权移交给指定的网络守护程序。

- Wifi 和 WWAN 由 [NetworkManager 管理](https://wiki.ubuntu.com/NetworkManager)
- 默认情况下，任何其他配置的设备都由 networkd 处理，除非明确标记为由特定管理器（[NetworkManager](https://wiki.ubuntu.com/NetworkManager)）管理
- 网络配置未涵盖的设备根本不会被触及



Ubuntu服务器18.04版本修改了IP地址配置程序, Ubuntu和Debian的软件架构师删除了以前的ifup/ifdown命令和`/etc/network/interfaces`配置文件, 改为使用`/etc/netplan/01-netcfg.yaml`和`sudo netplay apply`命令管理IP地址

## 使用

在`/etc/netplan/`使用`.yaml`扩展名（例如`/etc/netplan/config.yaml`）保存配置文件，然后运行`sudo netplan apply`. 此命令解析配置并将其应用于系统。写入磁盘的配置`/etc/netplan/`将在重新启动之间保持不变。


```yaml
network:
	version: 2
	renderer: networkd # 填入 networkd 或 NetworkManager
	
	ethernets:
		# 静态的ip设置
		ens33: # 修改为自己网卡信息
			addresses: [192.168.1.102/24]
			gateway4: 192.168.1.101
			nameservers:
				addresses: [192.168.1.103]     
			dhcp4: no
			optional: no
		# 动态的ip设置
		ens34: # 修改为自己网卡信息
			dhcp4: true
			dhcp6: true
			addresses: []
  # 无线网
	wifis:
		# 开放网络
		wlan0:
			dhcp4: yes
			dhcp6: yes
			access-points:
				opennetwork: {} # 填入接入点名称
  	# WPA 个人无线网
		wlan0: 
			dhcp4: yes
			dhcp6: yes
			access-points:
 				"ssid": # 填入wifi的ssid
 					password: "password" # 填入password
		# WPA-EAP + TTLS
		wlan1:
			dhcp4: yes
			dhcp6: yes
			access-points:
				workplace: # 接入点名称
					auth:
						key-management: eap
						method: ttls
						anonymous-identity: "@internal.example.com"
						identity: "user@internal.example.com"
						password: "password"
```

更多配置文件请参考[官方配置](https://netplan.io/examples/)

```sh
# 直接启动
sudo netplan apply
# 使用debug模式
sudo netplan --debug apply
```

```sh
# 打印网络设备的摘要信息
networkctl list
# 打印每一个IP地址的状态
networkctl status
```

