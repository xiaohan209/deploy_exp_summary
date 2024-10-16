# 容器网络插件



## Calico

### 介绍







### 下载





### 部署

> - https://docs.tigera.io/archive/v3.7/maintenance/troubleshooting#configure-networkmanager
> - https://cloud.tencent.com/developer/article/2353434
> - [通过清单文件安装3.23版本](https://www.cnblogs.com/RidingWind/p/16301622.html)
> - [官网自建实例的部署方式](https://docs.tigera.io/calico/latest/getting-started/kubernetes/self-managed-onprem/onpremises)



描述: Calico 提供两种部署方式有`operator `部署和 `manifests `清单方式部署。

- `Calico operator` : Calico 由 operator安装，该operator负责管理Calico集群的安装、升级和一般生命周期。operator作为Deployment直接安装在集群上，并通过一个或多个自定义Kubernetes API资源进行配置。
- `Calico manifests` ：Calico 也可以使用原始清单作为operator的替代品进行安装。清单包含在Kubernetes集群中的每个节点上安装Calico所需的资源。不建议使用清单，因为它们不能像operator那样自动管理Calico的生命周期。然而，清单可能对需要对底层Kubernetes资源进行高度特定修改的集群有用。

温馨提示: 当前【2023年9月15日 16:35:05】最新Calico版本为 3.26.1 , 你总是可以在其Github项目[https://github.com/projectcalico/calico/releases]中获取最新版本。

1. 

2. 如果网络配置是由Network Manager管理的，则需要创建文件`/etc/NetworkManager/conf.d/calico.conf`并写入以下配置：
   ```shell
   [keyfile]
   unmanaged-devices=interface-name:cali*;interface-name:tunl*
   ```

3. 





## Flannel




## 