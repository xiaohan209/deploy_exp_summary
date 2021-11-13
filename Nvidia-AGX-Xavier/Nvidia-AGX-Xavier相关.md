# Nvidia-AGX-Xavier

## 配置

### 物理配件：

#### 开箱硬件

- 3根电源线（允许的电流大小不同）
- 1个电源适配器
- 1根USB转type-C线（用于刷机）
- 1根USB**母头**转type-C 线（用于连接外设，如鼠标）
- Nvidia AGX Xavier开发板

[中国标准三孔插头规格][https://wenku.baidu.com/view/b7bad5635a1b6bd97f192279168884868762b8ea.html?fixfr=lDT8TAXAOZMNSYWwHsuYrQ%253D%253D&fr=income2-wk_go_search-search]：长头21mm，短头18mm



#### Xavier配置

[硬件接口相关文档][https://developer.download.nvidia.com/assets/embedded/secure/jetson/xavier/docs/Jetson_AGX_Xavier_Developer_Kit_Carrier_Board_Specification_SP-09778-001_v2.1.pdf?kekOKvzM7NSLqwKLMBbW4lIhWsKlVyQP8GaqY2IFCXfTfPQ4KY8yfnvsTDx5mP9RVjrjNVGG-ZcSwMxdjqWKXBVJYZuXT2mbWbkC0EU4YWHBybMo3hcEUv4fZ8VYEM-WFKJdh8t3_ocW-AKQEelPqwmvVbjlCN4SZaPTpH5RtdjCNyUJc3flrzL_pyifTZuHaIGY3SkRc7Hw9sH0xiufU4c1w3t3-Jr4obZCr6rZubuMUfVKV-EdV14f]



## 装系统

##### 准备以下：

- 1个显示器
- 1根HDMI 线
- 1根网线（或者无线网卡）
- 鼠标和键盘

##### 可参考文章：

- [装机示例][https://blog.csdn.net/FSKEps/article/details/106558205]
- **[AGX Xavier官方使用手册][https://developer.download.nvidia.cn/assets/embedded/secure/jetson/xavier/docs/nv_jetson_agx_xavier_developer_kit_user_guide.pdf?NRjGjYZKMfDd3TXmnOT-_gO9fQA52Nmp5MkF4ipDGwr7qRSoCkysFqHd-QwAlD5tXdp8oGJNYMoptIHcox5oRZbASwKOyP2D1ffJ4NKVBOVd-RWMSHy4Dwcy_xIbpqOhU1MSLfmZun1HbWH0U1mz-GvQI2vTIop8gsJsTnICzRW-0fpYIsnOaG5OIRvwUii2Ehpy9iBl-VI2pQ]**



### SDK Manager安装

[SDK Manager官方文档][https://docs.nvidia.com/sdk-manager/index.html]



### jetpack

[jetpack官方文档][https://docs.nvidia.com/jetson/jetpack/index.html]



###  L4T平台

[R32.6.1版本说明][https://developer.nvidia.com/embedded/linux-tegra-r3261]

[L4T使用指南][https://docs.nvidia.com/jetson/archives/l4t-archived/l4t-3261/index.html]





## 调整系统设置

### 换源

```shell
# 备份一下之前的源
sudo cp /etc/apt/sources.list /etc/apt/sources.list.origin.save
# 编辑现有源
sudo vim /etc/apt/sources.list

# 在vim下按esc后进入正常模式输入以下命令进行全局替换
:%s/ports.ubuntu.com/mirrors.tuna.tsinghua.edu.cn/g
```

最终文件内容为：

```shell
# See http://help.ubuntu.com/community/UpgradeNotes for how to upgrade to
# newer versions of the distribution.
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic main restricted
# deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic main restricted

## Major bug fix updates produced after the final release of the
## distribution.
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-updates main restricted
# deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-updates main restricted

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu
## team. Also, please note that software in universe WILL NOT receive any
## review or updates from the Ubuntu security team.
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic universe
# deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-updates universe
# deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-updates universe

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu
## team, and may not be under a free licence. Please satisfy yourself as to
## your rights to use the software. Also, please note that software in
## multiverse WILL NOT receive any review or updates from the Ubuntu
## security team.
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic multiverse
# deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-updates multiverse
# deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-updates multiverse

## N.B. software from this repository may not have been tested as
## extensively as that contained in the main release, although it includes
## newer versions of some applications which may provide useful features.
## Also, please note that software in backports WILL NOT receive any review
## or updates from the Ubuntu security team.
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-backports main restricted universe multiverse
# deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-backports main restricted universe multiverse

## Uncomment the following two lines to add software from Canonical's
## 'partner' repository.
## This software is not part of Ubuntu, but is offered by Canonical and the
## respective vendors as a service to Ubuntu users.
# deb http://archive.canonical.com/ubuntu bionic partner
# deb-src http://archive.canonical.com/ubuntu bionic partner

deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-security main restricted
# deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-security main restricted
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-security universe
# deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-security universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-security multiverse
# deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-security multiverse
```



### 性能调整

#### 风扇相关

原文链接：https://blog.csdn.net/weixin_42743099/article/details/107231462

```shell
sudo nvpmodel --query  							 #查看当前的模式，默认为2, 功耗15W
sudo nvpmodel -m 0   			  			     #设置满功率运行，MAX
sudo jetson_clocks 		  			         #强制风扇启
sudo jetson_clocks --show   	  			   #查看设置
sudo jetson_clocks --store   	   				 #默认保存配置文件到/home/$User/l4t_dfs.conf，可以命令后加指定路径保存
sudo vim /home/$USER/l4t_dfs.conf        #在l4t_dfs.conf最后面修改原有配置或加入下面这段，速度（0-250）
		/sys/devices/pwm-fan/target_pwm:200 		     #200的转速不会太吵而且散热也快
sudo jetson_clocks --restore   			 		 #执行完这个命令就可以开启风扇
#设置开机自动开启风扇
sudo vim ~/.bashrc
#在最后面插入下面这个命令
sudo jetson_clocks --restore
```





