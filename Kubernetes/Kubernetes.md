# Kubernetes

## 介绍











## 下载步骤

> - [官方下载页面](https://kubernetes.io/releases/download/)
> - 清华源：https://mirrors.tuna.tsinghua.edu.cn/help/kubernetes/

### 软件仓库版(推荐)

1. 查看版本
   ```bash
   # 一般以大版本为主即前两个数字为版本号v1.31.1即v1.31
   curl -Ls https://dl.k8s.io/release/stable.txt
   ```

   

2. 添加公钥认证

   ```bash
   # 需要更改v1.31为需要的版本号
   curl -fsSL "https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key" | gpg --dearmor -o /etc/apt/keyrings/k8s.gpg
   ```

   

3. 添加仓库，选择其中一个即可，可以把剩余的注释掉随时更换，目前看阿里云仓库版本号更新更频繁：

   ```bash
   # 使用官方源
   echo "deb [signed-by=/etc/apt/keyrings/k8s.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /" | sudo tee -a /etc/apt/sources.list.d/k8s.list
   # 选择阿里源
   echo "deb [signed-by=/etc/apt/keyrings/k8s.gpg] https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.31/deb/ /" | sudo tee -a /etc/apt/sources.list.d/k8s.list
   # 选择清华源
   echo "deb [signed-by=/etc/apt/keyrings/k8s.gpg] https://mirrors.tuna.tsinghua.edu.cn/kubernetes/core:/stable:/v1.31/deb/ /" | sudo tee -a /etc/apt/sources.list.d/k8s.list
   echo "deb [signed-by=/etc/apt/keyrings/k8s.gpg] https://mirrors.tuna.tsinghua.edu.cn/kubernetes/addons:/cri-o:/stable:/v1.31/deb/ /" | sudo tee -a /etc/apt/sources.list.d/k8s.list
   ```

   

4. 下载k8s相关的软件
   ```bash
   sudo apt update
   sudo apt install kubelet kubeadm kubectl -y
   ```

5. 



### 容器版

1. 列出需要安装的容器名称
   ```shell
   curl -Ls "https://sbom.k8s.io/$(curl -Ls https://dl.k8s.io/release/stable.txt)/release" | grep "SPDXID: SPDXRef-Package-registry.k8s.io" |  grep -v sha256 | cut -d- -f3- | sed 's/-/\//' | sed 's/-v1/:v1/'
   ```

   

### 二进制版

Kubectl控制器

1. 单独下载Kubectl控制端

   ```bash
   curl -Ls "https://sbom.k8s.io/$(curl -Ls https://dl.k8s.io/release/stable.txt)/release" | grep "SPDXID: SPDXRef-Package-registry.k8s.io" |  grep -v sha256 | cut -d- -f3- | sed 's/-/\//' | sed 's/-v1/:v1/'
   ```

   

2. 下载Kubeadm

3. 





## 部署集群方式1：容器+Kubeadm方式部署集群（采用软件仓库方式下载）

> [官网Kubeadm在不使用互联网下创建集群](https://kubernetes.io/zh-cn/docs/reference/setup-tools/kubeadm/kubeadm-init/#without-internet-connection)
>
> [官网Kubeadm创建集群方式](https://kubernetes.io/zh-cn/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)
>
> 其他网站参考
>
> - https://zhuanlan.zhihu.com/p/321795209
> - https://blog.csdn.net/tiny_du/article/details/123823093
> - https://cloud.tencent.com/developer/article/2160663
> - https://blog.csdn.net/TheRulingPower/article/details/139440330
> - https://www.cnblogs.com/Sunzz/p/15184167.html





> 系统环境：Ubuntu 24.04 LTS
>
> Kubernetes: 1.31.1
>
> Docker-CE: 27.3.1



### 1. 安装其他软件依赖

1. 容器运行时
   安装containerd，

   ```bash
   # 编辑containerd配置文件
   sudo vim /etc/containerd/config.toml 
   # 找到以下内容，注释该行，启用cri插件
   # disabled_plugins = ["cri"]
   
   # 退出后重新运行containerd
   sudo systemctl restart containerd
   ```

   

2. 

### 2. 预准备

1. 修改各个节点上的hosts域名

2. 关闭防火墙
    ```bash
    # 关闭防火墙
    sudo systemctl stop firewalld
    sudo systemctl disable firewalld
    ```

3. 关闭selinux

    ```bash
    # 关闭selinux
    sestatus # 查看SELinux状态
    sed -i 's/enforcing/disabled/' /etc/selinux/config
    setenforce 0
    ```

4. 关闭swap分区

    ```bash
    # 关闭swap分区
    sudo swapoff -a    # 临时关闭
    sudo vim /etc/fstab # 注释掉swap那一行，重启后可永久关闭
    ```

5. 配置内核转发网桥
    ```bash
    # 加载部分内核模块
    sudo modprobe overlay
    sudo modprobe br_netfilter
    # 如果需要ip_vs模式则加载该模块
    # sudo modprobe ip_vs
    # sudo modprobe ip_vs_rr
    # sudo modprobe ip_vs_wrr
    # sudo modprobe ip_vs_sh
    # sudo modprobe nf_conntrack_ipv4
    # 验证一下是否加载模块成功
    lsmod | egrep overlay
    lsmod | egrep br_netfilter
    # 设置开机自动加载内核模块配置文件
    cat << EOF | sudo tee /etc/modules-load.d/k8s.conf
    overlay
    br_netfilter
    EOF
    # 设置开机自动开启网桥转发配置文件
    cat << EOF | sudo tee /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
    net.ipv4.ip_forward = 1
    EOF
    # 重新加载
    sudo sysctl --system
    ```
    
6. 统一调整cgroupdriver
    需要保证

7. 

### 3. 配置master节点

1. 生成初始化文件

   ```bash
   kubeadm config print init-defaults > kubeadm-init.yaml
   # 文件内容如下，需要修改一些内容
   apiVersion: kubeadm.k8s.io/v1beta4
   bootstrapTokens:
   - groups:
     - system:bootstrappers:kubeadm:default-node-token
     token: abcdef.0123456789abcdef
     ttl: 24h0m0s
     usages:
     - signing
     - authentication
   kind: InitConfiguration
   localAPIEndpoint:
     advertiseAddress: 1.2.3.4 # 修改为本机地址
     bindPort: 6443
   nodeRegistration:
     criSocket: unix:///var/run/containerd/containerd.sock
     imagePullPolicy: IfNotPresent
     imagePullSerial: true
     name: node
     taints: null
   timeouts:
     controlPlaneComponentHealthCheck: 4m0s
     discovery: 5m0s
     etcdAPICall: 2m0s
     kubeletHealthCheck: 4m0s
     kubernetesAPICall: 1m0s
     tlsBootstrap: 5m0s
     upgradeManifests: 5m0s
   ---
   apiServer: {}
   apiVersion: kubeadm.k8s.io/v1beta4
   caCertificateValidityPeriod: 87600h0m0s
   certificateValidityPeriod: 8760h0m0s
   certificatesDir: /etc/kubernetes/pki
   clusterName: kubernetes
   controllerManager: {}
   dns: {}
   encryptionAlgorithm: RSA-2048
   etcd:
     local:
       dataDir: /var/lib/etcd
   imageRepository: registry.k8s.io  # 如果网络不通可以修改为自己需要选择的镜像源
   kind: ClusterConfiguration
   kubernetesVersion: 1.31.0
   networking:
     dnsDomain: cluster.local
     serviceSubnet: 10.96.0.0/12 # 如果有需要可以调整否则维持默认即可
     podSubnet: 10.244.0.0/16 # 需要新增此行，地址可以自行设置，一般维持默认
   proxy: {}
   scheduler: {}
   ```

   

2. 下载镜像
   ```bash
   # 使用自定义的kubeadm配置下载所需镜像，如果拉取不成功请确认是否在上步配置了可访问的镜像源
   kubeadm config images pull --config kubeadm-init.yaml
   ```

3. 执行初始化

   ```bash
   # 使用刚才创建并修改好的kubeadm配置文件进行初始化
   sudo kubeadm init --config kubeadm-init.yaml
   ```

   初始化master节点成功后弹出以下提示：

   ```bash
   ...
   Your Kubernetes control-plane has initialized successfully!
   ...
   Then you can join any number of worker nodes by running the following on each as root:
   
   kubeadm join 10.33.30.92:6443 --token abcdef.0123456789abcdef \
       --discovery-token-ca-cert-hash sha256:2883b1961db36593fb67ab5cd024f451b934fc0e72e2fa3858dda3ad3b225837
   ```

   需要复制最后一行用作下一步的命令

   如果初始化遇到问题请参照[初始化问题](#在安装过程中，kubeadm 一直等待控制平面就绪)

4. 其他节点加入master
   ```bash
   kubeadm join 10.33.30.92:6443 --token abcdef.0123456789abcdef \
       --discovery-token-ca-cert-hash sha256:2883b1961db36593fb67ab5cd024f451b934fc0e72e2fa3858dda3ad3b225837
   ```

5. 验证一下
   ```bash
   # 在master节点上使用kubectl查看所有node
   kubectl get node
   ```

   可以看到以下信息，发现node数量没问题，STATUS为`NotReady`是因为没有配置容器网络：

   ```bash
   NAME              STATUS     ROLES           AGE     VERSION
   node              NotReady   control-plane   3d9h    v1.31.1
   zeus-k8s-node-2   NotReady   <none>          2d15h   v1.31.1
   zeus-k8s-node-3   NotReady   <none>          2d15h   v1.31.1
   ```

   

### 4. 配置容器网络CNI插件

容器网络CNI插件有很多，主要负责管理pod加入网络以及相互通信的规则，最经典的就是Calico和Flannel.

以Calico为例：

```bash
# 通过tigera-operator可以让operator负责管理和安装，但是很容易卡在镜像拉取这一步上
# kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.2/manifests/tigera-operator.yaml
# kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.2/manifests/custom-resources.yaml

# 使用manifest清单安装calico
curl  -O 
kubectl 

```



通过以下指令验证一下STATUS是否变成`Ready`，变成`Ready`则容器网络配置成功：

```bash
kubectl get nodes -o wide
```



### 5. 安装Dashboard







## 部署集群方式2：二进制文件方式部署集群

> - https://www.cnblogs.com/394510636-ff/p/9501921.html
> - https://www.cnblogs.com/394510636-ff/p/9498037.html

### CA证书创建

`kubernetes` 系统各个组件需要使用`TLS`证书对通信进行加密，这里我们使用`CloudFlare`的PKI 工具集[cfssl](https://github.com/cloudflare/cfssl) 来生成Certificate Authority(CA) 证书和密钥文件， CA 是自签名的证书，用来签名后续创建的其他TLS 证书。

 

#### 安装 CFSSL



```
$ wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
$ chmod +x cfssl_linux-amd64
$ sudo mv cfssl_linux-amd64 /usr/k8s/bin/cfssl

$ wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
$ chmod +x cfssljson_linux-amd64
$ sudo mv cfssljson_linux-amd64 /usr/k8s/bin/cfssljson

$ wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
$ chmod +x cfssl-certinfo_linux-amd64
$ sudo mv cfssl-certinfo_linux-amd64 /usr/k8s/bin/cfssl-certinfo

$ export PATH=/usr/k8s/bin:$PATH
$ mkdir ssl && cd ssl
$ cfssl print-defaults config > config.json
$ cfssl print-defaults csr > csr.json
```

为了方便，将`/usr/k8s/bin`设置成环境变量，为了重启也有效，可以将上面的`export PATH=/usr/k8s/bin:$PATH`添加到`/etc/rc.local`文件中。

#### 创建CA

修改上面创建的`config.json`文件为`ca-config.json`：

```
$ cat ca-config.json
{
    "signing": {
        "default": {
            "expiry": "87600h"
        },
        "profiles": {
            "kubernetes": {
                "expiry": "87600h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth",
                    "client auth"
                ]
            }
        }
    }
}
```



- `config.json`：可以定义多个profiles，分别指定不同的过期时间、使用场景等参数；后续在签名证书时使用某个profile；
- `signing`: 表示该证书可用于签名其它证书；生成的ca.pem 证书中`CA=TRUE`；
- `server auth`: 表示client 可以用该CA 对server 提供的证书进行校验；
- `client auth`: 表示server 可以用该CA 对client 提供的证书进行验证。

修改CA 证书签名请求为`ca-csr.json`：



```
$ cat ca-csr.json
{
    "CN": "kubernetes",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "BeiJing",
            "ST": "BeiJing",
            "O": "k8s",
            "OU": "System"
        }
    ]
}
```



- `CN`: `Common Name`，kube-apiserver 从证书中提取该字段作为请求的用户名(User Name)；浏览器使用该字段验证网站是否合法；
- `O`: `Organization`，kube-apiserver 从证书中提取该字段作为请求用户所属的组(Group)；

生成CA 证书和私钥：

```
$ cfssl gencert -initca ca-csr.json | cfssljson -bare ca
$ ls ca*
$ ca-config.json  ca.csr  ca-csr.json  ca-key.pem  ca.pem
```

#### 分发证书

将生成的CA 证书、密钥文件、配置文件拷贝到所有机器的`/etc/kubernetes/ssl`目录下面：

```
$ sudo mkdir -p /etc/kubernetes/ssl
$ sudo cp ca* /etc/kubernetes/ssl
```

### etcd部署

kubernetes 系统使用`etcd`存储所有的数据，我们这里部署3个节点的etcd 集群，这3个节点直接复用kubernetes master的3个节点，分别命名为`etcd01`、`etcd02`、`etcd03`:

- etcd01：192.168.150.128
- etcd02：192.168.150.130
- etcd03：192.168.150.131

#### 定义环境变量

使用到的变量如下：

```
$ export NODE_NAME=etcd01 # 当前部署的机器名称(随便定义，只要能区分不同机器即可)
$ export NODE_IP=192.168.150.128 # 当前部署的机器IP
$ export NODE_IPS="192.168.150.128 192.168.150.130 192.168.150.131" # etcd 集群所有机器 IP
$ # etcd 集群间通信的IP和端口
$ export ETCD_NODES=etcd01=https://192.168.150.128:2380,etcd02=https://192.168.150.130:2380,etcd03=https://192.168.150.131:2380
$ # 导入用到的其它全局变量：ETCD_ENDPOINTS、FLANNEL_ETCD_PREFIX、CLUSTER_CIDR
$ source /usr/k8s/bin/env.sh
```

#### 下载etcd 二进制文件

到https://github.com/coreos/etcd/releases页面下载最新版本的二进制文件：

```
$ wget https://github.com/coreos/etcd/releases/download/v3.2.9/etcd-v3.2.9-linux-amd64.tar.gz
$ tar -xvf etcd-v3.2.9-linux-amd64.tar.gz
$ sudo mv etcd-v3.2.9-linux-amd64/etcd* /usr/k8s/bin/
```

   \#目前etcd好像是3.3.9版本了吧

#### 创建TLS 密钥和证书

为了保证通信安全，客户端(如etcdctl)与etcd 集群、etcd 集群之间的通信需要使用TLS 加密。

创建etcd 证书签名请求：

```
$ cat > etcd-csr.json <<EOF
{
  "CN": "etcd",
  "hosts": [
    "127.0.0.1",
    "${NODE_IP}"
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
EOF
```

- `hosts` 字段指定授权使用该证书的`etcd`节点IP

生成`etcd`证书和私钥：

```
$ cfssl gencert -ca=/etc/kubernetes/ssl/ca.pem \
  -ca-key=/etc/kubernetes/ssl/ca-key.pem \
  -config=/etc/kubernetes/ssl/ca-config.json \
  -profile=kubernetes etcd-csr.json | cfssljson -bare etcd
$ ls etcd*
etcd.csr  etcd-csr.json  etcd-key.pem  etcd.pem
$ sudo mkdir -p /etc/etcd/ssl
$ sudo mv etcd*.pem /etc/etcd/ssl/
```

 

#### 创建etcd 的systemd unit 文件

```
$ sudo mkdir -p /var/lib/etcd  # 必须要先创建工作目录
$ cat > etcd.service <<EOF
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target
Documentation=https://github.com/coreos

[Service]
Type=notify
WorkingDirectory=/var/lib/etcd/
ExecStart=/usr/k8s/bin/etcd \\
  --name=${NODE_NAME} \\
  --cert-file=/etc/etcd/ssl/etcd.pem \\
  --key-file=/etc/etcd/ssl/etcd-key.pem \\
  --peer-cert-file=/etc/etcd/ssl/etcd.pem \\
  --peer-key-file=/etc/etcd/ssl/etcd-key.pem \\
  --trusted-ca-file=/etc/kubernetes/ssl/ca.pem \\
  --peer-trusted-ca-file=/etc/kubernetes/ssl/ca.pem \\
  --initial-advertise-peer-urls=https://${NODE_IP}:2380 \\
  --listen-peer-urls=https://${NODE_IP}:2380 \\
  --listen-client-urls=https://${NODE_IP}:2379,http://127.0.0.1:2379 \\
  --advertise-client-urls=https://${NODE_IP}:2379 \\
  --initial-cluster-token=etcd-cluster-0 \\
  --initial-cluster=${ETCD_NODES} \\
  --initial-cluster-state=new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF
```



- 指定`etcd`的工作目录和数据目录为`/var/lib/etcd`，需要在启动服务前创建这个目录；
- 为了保证通信安全，需要指定etcd 的公私钥(cert-file和key-file)、Peers通信的公私钥和CA 证书(peer-cert-file、peer-key-file、peer-trusted-ca-file)、客户端的CA 证书(trusted-ca-file)；
- `--initial-cluster-state`值为`new`时，`--name`的参数值必须位于`--initial-cluster`列表中；

#### 启动etcd 服务

```
$ sudo mv etcd.service /etc/systemd/system/
$ sudo systemctl daemon-reload
$ sudo systemctl enable etcd
$ sudo systemctl start etcd
$ sudo systemctl status etcd
```

 \#/etc/systemd/system下面发生改变时需要执行systemctl daemon-reload 命令 重新加载配置

 \#最先启动的etcd 进程会卡住一段时间，等待其他节点启动加入集群，在所有的etcd 节点重复上面的步骤，直到所有的机器etcd 服务都已经启动。

 \#需要注意的是 我配置的都没有问题但是启动etcd的时候3个节点都显示连接超时反复排查检查证书文件都是没有问题的，最后想到的是 我这3个master机器是新建的虚拟机

  selinux没有关闭，防火墙没有放行端口2080,2379、要是个人实验可以关闭防火墙。

\#还有配置完一台的时候启动etcd肯定是启动不起来etcd服务的，因为是集群需要所有master节点都配置完etcd还能启动服务成功。

把master01上面/usr/k8s/bin/ 、/etc/kubernetes/ssl 这两个下面的所有文件都要拷贝到master02,master03机器上面对应的目录



#### 验证服务

部署完etcd 集群后，在任一etcd 节点上执行下面命令：

```
for ip in ${NODE_IPS}; do
  ETCDCTL_API=3 /usr/k8s/bin/etcdctl \
  --endpoints=https://${ip}:2379  \
  --cacert=/etc/kubernetes/ssl/ca.pem \
  --cert=/etc/etcd/ssl/etcd.pem \
  --key=/etc/etcd/ssl/etcd-key.pem \
  endpoint health; done
```



### 部署master 节点

kubernetes master 需要包含以下组件：

- kube-apiserver
- kube-scheduler
- kube-controller-manager

kube-scheduler、kube-controller-manager 和 kube-apiserver 三者的功能紧密相关； 同时只能有一个 kube-scheduler、kube-controller-manager 进程处于工作状态，如果运行多个，则需要通过选举产生一个 leader



#### 创建kubernetes 证书

所有组件之间都需要配置证书

创建kubernetes 证书签名请求：

```shell
cat > kubernetes-csr.json <<EOF
{
  "CN": "kubernetes",
  "hosts": [
    "127.0.0.1",
    "${NODE_IP}",
    "${MASTER_URL}",
    "${CLUSTER_KUBERNETES_SVC_IP}",
    "kubernetes",
    "kubernetes.default",
    "kubernetes.default.svc",
    "kubernetes.default.svc.cluster",
    "kubernetes.default.svc.cluster.local"
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
EOF
```

- 如果 hosts 字段不为空则需要指定授权使用该证书的 IP 或域名列表，所以上面分别指定了当前部署的 master 节点主机 IP 以及apiserver 负载的内部域名
- 还需要添加 kube-apiserver 注册的名为 `kubernetes` 的服务 IP (Service Cluster IP)，一般是 kube-apiserver `--service-cluster-ip-range` 选项值指定的网段的第一个IP，如 “10.254.0.1”

生成kubernetes 证书和私钥：

```
$ cfssl gencert -ca=/etc/kubernetes/ssl/ca.pem \
  -ca-key=/etc/kubernetes/ssl/ca-key.pem \
  -config=/etc/kubernetes/ssl/ca-config.json \
  -profile=kubernetes kubernetes-csr.json | cfssljson -bare kubernetes
$ ls kubernetes*
kubernetes.csr  kubernetes-csr.json  kubernetes-key.pem  kubernetes.pem
$ sudo mkdir -p /etc/kubernetes/ssl/
$ sudo mv kubernetes*.pem /etc/kubernetes/ssl/
```

[![复制代码](https://assets.cnblogs.com/images/copycode.gif)](javascript:void(0);)

#### 配置和启动kube-apiserver

##### 创建kube-apiserver 使用的客户端token 文件

kubelet 首次启动时向kube-apiserver 发送TLS Bootstrapping 请求，kube-apiserver 验证请求中的token 是否与它配置的token.csv 一致，如果一致则自动为kubelet 生成证书和密钥。

```
# 导入的 environment.sh 文件定义了 BOOTSTRAP_TOKEN 变量
$ cat > token.csv <<EOF
${BOOTSTRAP_TOKEN},kubelet-bootstrap,10001,"system:kubelet-bootstrap"
EOF
$ sudo mv token.csv /etc/kubernetes/
```

##### 创建kube-apiserver 的systemd unit文件

```
$ cat  > kube-apiserver.service <<EOF
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=network.target

[Service]
ExecStart=/usr/k8s/bin/kube-apiserver \\
  --admission-control=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota \\
  --advertise-address=${NODE_IP} \\
  --bind-address=0.0.0.0 \\
  --insecure-bind-address=${NODE_IP} \\
  --authorization-mode=Node,RBAC \\
  --runtime-config=rbac.authorization.k8s.io/v1alpha1 \\
  --kubelet-https=true \\
  --experimental-bootstrap-token-auth \\
  --token-auth-file=/etc/kubernetes/token.csv \\
  --service-cluster-ip-range=${SERVICE_CIDR} \\
  --service-node-port-range=${NODE_PORT_RANGE} \\
  --tls-cert-file=/etc/kubernetes/ssl/kubernetes.pem \\
  --tls-private-key-file=/etc/kubernetes/ssl/kubernetes-key.pem \\
  --client-ca-file=/etc/kubernetes/ssl/ca.pem \\
  --service-account-key-file=/etc/kubernetes/ssl/ca-key.pem \\
  --etcd-cafile=/etc/kubernetes/ssl/ca.pem \\
  --etcd-certfile=/etc/kubernetes/ssl/kubernetes.pem \\
  --etcd-keyfile=/etc/kubernetes/ssl/kubernetes-key.pem \\
  --etcd-servers=${ETCD_ENDPOINTS} \\
  --enable-swagger-ui=true \\
  --allow-privileged=true \\
  --apiserver-count=2 \\
  --audit-log-maxage=30 \\
  --audit-log-maxbackup=3 \\
  --audit-log-maxsize=100 \\
  --audit-log-path=/var/lib/audit.log \\
  --audit-policy-file=/etc/kubernetes/audit-policy.yaml \\
  --event-ttl=1h \\
  --logtostderr=true \\
  --v=6
Restart=on-failure
RestartSec=5
Type=notify
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF
```

[![复制代码](https://assets.cnblogs.com/images/copycode.gif)](javascript:void(0);)

- 如果你安装的是1.9.x版本的，一定要记住上面的参数`experimental-bootstrap-token-auth`，需要替换成`enable-bootstrap-token-auth`，因为这个参数在1.9.x里面已经废弃掉了
- kube-apiserver 1.6 版本开始使用 etcd v3 API 和存储格式
- `--authorization-mode=RBAC` 指定在安全端口使用RBAC 授权模式，拒绝未通过授权的请求
- kube-scheduler、kube-controller-manager 一般和 kube-apiserver 部署在同一台机器上，它们使用非安全端口和 kube-apiserver通信
- kubelet、kube-proxy、kubectl 部署在其它 Node 节点上，如果通过安全端口访问 kube-apiserver，则必须先通过 TLS 证书认证，再通过 RBAC 授权
- kube-proxy、kubectl 通过使用证书里指定相关的 User、Group 来达到通过 RBAC 授权的目的
- 如果使用了 kubelet TLS Boostrap 机制，则不能再指定 `--kubelet-certificate-authority`、`--kubelet-client-certificate` 和 `--kubelet-client-key` 选项，否则后续 kube-apiserver 校验 kubelet 证书时出现 ”x509: certificate signed by unknown authority“ 错误
- `--admission-control` 值必须包含 `ServiceAccount`，否则部署集群插件时会失败
- `--bind-address` 不能为 `127.0.0.1`
- `--service-cluster-ip-range` 指定 Service Cluster IP 地址段，该地址段不能路由可达
- `--service-node-port-range=${NODE_PORT_RANGE}` 指定 NodePort 的端口范围
- 缺省情况下 kubernetes 对象保存在`etcd/registry` 路径下，可以通过 `--etcd-prefix` 参数进行调整
- kube-apiserver 1.8版本后需要在`--authorization-mode`参数中添加`Node`，即：`--authorization-mode=Node,RBAC`，否则Node 节点无法注册
- 注意要开启审查日志功能，指定`--audit-log-path`参数是不够的，这只是指定了日志的路径，还需要指定一个审查日志策略文件：`--audit-policy-file`，我们也可以使用日志收集工具收集相关的日志进行分析。

审查日志策略文件内容如下：（/etc/kubernetes/audit-policy.yaml）

[![复制代码](https://assets.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
apiVersion: audit.k8s.io/v1beta1 # This is required.
kind: Policy
# Don't generate audit events for all requests in RequestReceived stage.
omitStages:
  - "RequestReceived"
rules:
  # Log pod changes at RequestResponse level
  - level: RequestResponse
    resources:
    - group: ""
      # Resource "pods" doesn't match requests to any subresource of pods,
      # which is consistent with the RBAC policy.
      resources: ["pods"]
  # Log "pods/log", "pods/status" at Metadata level
  - level: Metadata
    resources:
    - group: ""
      resources: ["pods/log", "pods/status"]

  # Don't log requests to a configmap called "controller-leader"
  - level: None
    resources:
    - group: ""
      resources: ["configmaps"]
      resourceNames: ["controller-leader"]

  # Don't log watch requests by the "system:kube-proxy" on endpoints or services
  - level: None
    users: ["system:kube-proxy"]
    verbs: ["watch"]
    resources:
    - group: "" # core API group
      resources: ["endpoints", "services"]

  # Don't log authenticated requests to certain non-resource URL paths.
  - level: None
    userGroups: ["system:authenticated"]
    nonResourceURLs:
    - "/api*" # Wildcard matching.
    - "/version"

  # Log the request body of configmap changes in kube-system.
  - level: Request
    resources:
    - group: "" # core API group
      resources: ["configmaps"]
    # This rule only applies to resources in the "kube-system" namespace.
    # The empty string "" can be used to select non-namespaced resources.
    namespaces: ["kube-system"]

  # Log configmap and secret changes in all other namespaces at the Metadata level.
  - level: Metadata
    resources:
    - group: "" # core API group
      resources: ["secrets", "configmaps"]

  # Log all other resources in core and extensions at the Request level.
  - level: Request
    resources:
    - group: "" # core API group
    - group: "extensions" # Version of group should NOT be included.

  # A catch-all rule to log all other requests at the Metadata level.
  - level: Metadata
    # Long-running requests like watches that fall under this rule will not
    # generate an audit event in RequestReceived.
    omitStages:
      - "RequestReceived"
```

[![复制代码](https://assets.cnblogs.com/images/copycode.gif)](javascript:void(0);)

审查日志的相关配置可以查看文档了解：https://kubernetes.io/docs/tasks/debug-application-cluster/audit/

这里解释一下审查日志策略是什么东西、起什么作用；

如果有人执行了 kube-apiserver 做了什么操作 都会有日志产生有记录的 对后续排查问题大有用处，可以把日志重定向一个文件里面来分析。

##### 启动kube-apiserver

```
$ sudo cp kube-apiserver.service /etc/systemd/system/
$ sudo systemctl daemon-reload
$ sudo systemctl enable kube-apiserver
$ sudo systemctl start kube-apiserver
$ sudo systemctl status kube-apiserver
```





#### 配置和启动kube-controller-manager

##### 创建kube-controller-manager 的systemd unit 文件



```
$ cat > kube-controller-manager.service <<EOF
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/GoogleCloudPlatform/kubernetes

[Service]
ExecStart=/usr/k8s/bin/kube-controller-manager \\
  --address=127.0.0.1 \\
  --master=http://${MASTER_URL}:8080 \\
  --allocate-node-cidrs=true \\
  --service-cluster-ip-range=${SERVICE_CIDR} \\
  --cluster-cidr=${CLUSTER_CIDR} \\
  --cluster-name=kubernetes \\
  --cluster-signing-cert-file=/etc/kubernetes/ssl/ca.pem \\
  --cluster-signing-key-file=/etc/kubernetes/ssl/ca-key.pem \\
  --service-account-private-key-file=/etc/kubernetes/ssl/ca-key.pem \\
  --root-ca-file=/etc/kubernetes/ssl/ca.pem \\
  --leader-elect=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

[![复制代码](https://assets.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

- `--address` 值必须为 `127.0.0.1`，因为当前 kube-apiserver 期望 scheduler 和 controller-manager 在同一台机器
- `--master=http://${MASTER_URL}:8080`：使用`http`(非安全端口)与 kube-apiserver 通信，需要下面的`haproxy`安装成功后才能去掉8080端口。
- `--cluster-cidr` 指定 Cluster 中 Pod 的 CIDR 范围，该网段在各 Node 间必须路由可达(flanneld保证)
- `--service-cluster-ip-range` 参数指定 Cluster 中 Service 的CIDR范围，该网络在各 Node 间必须路由不可达，必须和 kube-apiserver 中的参数一致
- `--cluster-signing-*` 指定的证书和私钥文件用来签名为 TLS BootStrap 创建的证书和私钥
- `--root-ca-file` 用来对 kube-apiserver 证书进行校验，指定该参数后，才会在Pod 容器的 ServiceAccount 中放置该 CA 证书文件
- `--leader-elect=true` 部署多台机器组成的 master 集群时选举产生一处于工作状态的 `kube-controller-manager` 进程

##### 启动kube-controller-manager

```
$ sudo cp kube-controller-manager.service /etc/systemd/system/
$ sudo systemctl daemon-reload
$ sudo systemctl enable kube-controller-manager
$ sudo systemctl start kube-controller-manager
$ sudo systemctl status kube-controller-manager
```



#### 配置和启动kube-scheduler

##### 创建kube-scheduler 的systemd unit文件



```
$ cat > kube-scheduler.service <<EOF
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/GoogleCloudPlatform/kubernetes

[Service]
ExecStart=/usr/k8s/bin/kube-scheduler \\
  --address=127.0.0.1 \\
  --master=http://${MASTER_URL}:8080 \\
  --leader-elect=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```



- `--address` 值必须为 `127.0.0.1`，因为当前 kube-apiserver 期望 scheduler 和 controller-manager 在同一台机器
- `--master=http://${MASTER_URL}:8080`：使用`http`(非安全端口)与 kube-apiserver 通信，需要下面的`haproxy`启动成功后才能去掉8080端口
- `--leader-elect=true` 部署多台机器组成的 master 集群时选举产生一处于工作状态的 `kube-controller-manager` 进程

##### 启动kube-scheduler

在这里提一句三个组件配置不是很复杂

```
$ sudo cp kube-scheduler.service /etc/systemd/system/
$ sudo systemctl daemon-reload
$ sudo systemctl enable kube-scheduler
$ sudo systemctl start kube-scheduler
$ sudo systemctl status kube-scheduler
```

现在还验证不了kube-apiserver状态 ，缺少kubectl工具



#### 配置kubectl 命令行工具

`kubectl`默认从`~/.kube/config`配置文件中获取访问kube-apiserver 地址、证书、用户名等信息，需要正确配置该文件才能正常使用`kubectl`命令。

需要将下载的kubectl 二进制文件和生产的`~/.kube/config`配置文件拷贝到需要使用kubectl 命令的机器上。

 

很多童鞋说这个地方不知道在哪个节点上执行，`kubectl`只是一个和`kube-apiserver`进行交互的一个命令行工具，所以你想安装到那个节点都想，master或者node任意节点都可以，比如你先在master节点上安装，这样你就可以在master节点使用`kubectl`命令行工具了，如果你想在node节点上使用(当然安装的过程肯定会用到的)，你就把master上面的`kubectl`二进制文件和`~/.kube/config`文件拷贝到对应的node节点上就行了。

##### 环境变量

```
$ source /usr/k8s/bin/env.sh
$ export KUBE_APISERVER="https://${MASTER_URL}:6443"
```

  注意这里的`KUBE_APISERVER`地址，因为我们还没有安装`haproxy`，所以暂时需要手动指定使用`apiserver`的6443端口，等`haproxy`安装完成后就可以用使用443端口转发到6443端口去了。

- 变量KUBE_APISERVER 指定kubelet 访问的kube-apiserver 的地址，后续被写入`~/.kube/config`配置文件

##### 下载kubectl

```
$ wget https://dl.k8s.io/v1.8.2/kubernetes-client-linux-amd64.tar.gz # 如果服务器上下载不下来，可以想办法下载到本地，然后scp上去即可
$ tar -xzvf kubernetes-client-linux-amd64.tar.gz
$ sudo cp kubernetes/client/bin/kube* /usr/k8s/bin/
$ sudo chmod a+x /usr/k8s/bin/kube*
$ export PATH=/usr/k8s/bin:$PATH
```

##### 创建admin 证书

kubectl 与kube-apiserver 的安全端口通信，需要为安全通信提供TLS 证书和密钥。创建admin 证书签名请求：



```
$ cat > admin-csr.json <<EOF
{
  "CN": "admin",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "system:masters",
      "OU": "System"
    }
  ]
}
EOF
```



- 后续`kube-apiserver`使用RBAC 对客户端(如kubelet、kube-proxy、Pod)请求进行授权
- `kube-apiserver` 预定义了一些RBAC 使用的RoleBindings，如cluster-admin 将Group `system:masters`与Role `cluster-admin`绑定，该Role 授予了调用`kube-apiserver`所有API 的权限
- O 指定了该证书的Group 为`system:masters`，kubectl使用该证书访问`kube-apiserver`时，由于证书被CA 签名，所以认证通过，同时由于证书用户组为经过预授权的`system:masters`，所以被授予访问所有API 的权限
- hosts 属性值为空列表

​    \#hosts属性为空表示不指定在那台机器上安装

生成admin 证书和私钥：



```
$ cfssl gencert -ca=/etc/kubernetes/ssl/ca.pem \
  -ca-key=/etc/kubernetes/ssl/ca-key.pem \
  -config=/etc/kubernetes/ssl/ca-config.json \
  -profile=kubernetes admin-csr.json | cfssljson -bare admin
$ ls admin
admin.csr  admin-csr.json  admin-key.pem  admin.pem
$ sudo mv admin*.pem /etc/kubernetes/ssl/
```



##### 创建kubectl kubeconfig 文件



```
# 设置集群参数
$ kubectl config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/ssl/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER}
# 设置客户端认证参数
$ kubectl config set-credentials admin \
  --client-certificate=/etc/kubernetes/ssl/admin.pem \
  --embed-certs=true \
  --client-key=/etc/kubernetes/ssl/admin-key.pem \
  --token=${BOOTSTRAP_TOKEN}
# 设置上下文参数
$ kubectl config set-context kubernetes \
  --cluster=kubernetes \
  --user=admin
# 设置默认上下文
$ kubectl config use-context kubernetes
```



- `admin.pem`证书O 字段值为`system:masters`，`kube-apiserver` 预定义的 RoleBinding `cluster-admin`将 Group `system:masters` 与 Role `cluster-admin` 绑定，该 Role 授予了调用`kube-apiserver` 相关 API 的权限
- 生成的kubeconfig 被保存到 `~/.kube/config` 文件

##### 分发kubeconfig 文件

将`~/.kube/config`文件拷贝到运行`kubectl`命令的机器的`~/.kube/`目录下去。

现在可以使用kubectl get cs查看状态了

![img](https://images2018.cnblogs.com/blog/1330567/201808/1330567-20180819173737676-1041110254.png)

get是 kubernetes api中的一个方法吧











### 配置容器网络插件

master 节点与node 节点上的Pods 通过Pod 网络通信，所以需要在master 节点上部署Flannel 网络。

部署Flannel网络









## 集群服务使用方式

> - [Service与Pod详解](https://blog.csdn.net/2301_78088515/article/details/142859492)
>
> 



### 创建Service





```yaml
Service yaml文件详解
 
apiVersion: v1
kind: Service
matadata:                                #元数据
  name: string                           #service的名称
  namespace: string                      #命名空间  
  labels:                                #自定义标签属性列表
    - name: string
  annotations:                           #自定义注解属性列表  
    - name: string
spec:                                    #详细描述
  selector: []                           #label selector配置，将选择具有label标签的Pod作为管理 
                                         #范围
  type: string                           #service的类型，指定service的访问方式，默认为 
                                         #clusterIp
  clusterIP: string                      #虚拟服务地址      
  sessionAffinity: string                #是否支持session
  ports:                                 #service需要暴露的端口列表
  - name: string                         #端口名称
    protocol: string                     #端口协议，支持TCP和UDP，默认TCP
    port: int                            #服务监听的端口号
    targetPort: int                      #需要转发到后端Pod的端口号
    nodePort: int                        #当type = NodePort时，指定映射到物理机的端口号
  status:                                #当spce.type=LoadBalancer时，设置外部负载均衡器的地址
    loadBalancer:                        #外部负载均衡器    
      ingress:                           #外部负载均衡器 
        ip: string                       #外部负载均衡器的Ip地址值
        hostname: string                 #外部负载均衡器的主机名
```



### 创建Pod

示例pod文件

```yaml
//Pod yaml文件详解
 
apiVersion: v1            #必选，版本号，例如v1
kind: Pod                #必选，Pod
metadata:                #必选，元数据
  name: string              #必选，Pod名称
  namespace: string          #必选，Pod所属的命名空间
  labels:                  #自定义标签
    - name: string            #自定义标签名字
  annotations:                #自定义注释列表
    - name: string
spec:                    #必选，Pod中容器的详细定义
  containers:              #必选，Pod中容器列表
  - name: string            #必选，容器名称
    image: string            #必选，容器的镜像名称
    imagePullPolicy: [Always | Never | IfNotPresent]    #获取镜像的策略：Alawys表示总是下载镜像，IfnotPresent表示优先使用本地镜像，否则下载镜像，Nerver表示仅使用本地镜像
    command: [string]        #容器的启动命令列表，如不指定，使用打包时使用的启动命令
    args: [string]            #容器的启动命令参数列表
    workingDir: string        #容器的工作目录
    volumeMounts:            #挂载到容器内部的存储卷配置
    - name: string              #引用pod定义的共享存储卷的名称，需用volumes[]部分定义的的卷名
      mountPath: string          #存储卷在容器内mount的绝对路径，应少于512字符
      readOnly: boolean          #是否为只读模式
    ports:                    #需要暴露的端口库号列表
    - name: string              #端口号名称
      containerPort: int      #容器需要监听的端口号
      hostPort: int              #容器所在主机需要监听的端口号，默认与Container相同
      protocol: string          #端口协议，支持TCP和UDP，默认TCP
    env:                    #容器运行前需设置的环境变量列表
    - name: string              #环境变量名称
      value: string              #环境变量的值
    resources:                #资源限制和请求的设置
      limits:                  #资源限制的设置
        cpu: string                #Cpu的限制，单位为core数，将用于docker run --cpu-shares参数
        memory: string            #内存限制，单位可以为Mib/Gib，将用于docker run --memory参数
      requests:                  #资源请求的设置
        cpu: string                #Cpu请求，容器启动的初始可用数量
        memory: string            #内存清楚，容器启动的初始可用数量
    livenessProbe:             #对Pod内个容器健康检查的设置，当探测无响应几次后将自动重启该容器，检查方法有exec、httpGet和tcpSocket，对一个容器只需设置其中一种方法即可
      exec:                    #对Pod容器内检查方式设置为exec方式
        command: [string]      #exec方式需要制定的命令或脚本
      httpGet:                #对Pod内个容器健康检查方法设置为HttpGet，需要制定Path、port
        path: string
        port: number
        host: string
        scheme: string
        HttpHeaders:
        - name: string
          value: string
      tcpSocket:            #对Pod内个容器健康检查方式设置为tcpSocket方式
         port: number
       initialDelaySeconds: 0    #容器启动完成后首次探测的时间，单位为秒
       timeoutSeconds: 0        #对容器健康检查探测等待响应的超时时间，单位秒，默认1秒
       periodSeconds: 0            #对容器监控检查的定期探测时间设置，单位秒，默认10秒一次
       successThreshold: 0
       failureThreshold: 0
       securityContext:
         privileged:false
    restartPolicy: [Always | Never | OnFailure]        #Pod的重启策略，Always表示一旦不管以何种方式终止运行，kubelet都将重启，OnFailure表示只有Pod以非0退出码退出才重启，Nerver表示不再重启该Pod
    nodeSelector: obeject        #设置NodeSelector表示将该Pod调度到包含这个label的node上，以key：value的格式指定
    imagePullSecrets:            #Pull镜像时使用的secret名称，以key：secretkey格式指定
    - name: string
    hostNetwork:false            #是否使用主机网络模式，默认为false，如果设置为true，表示使用宿主机网络
    volumes:                    #在该pod上定义共享存储卷列表
    - name: string                  #共享存储卷名称 （volumes类型有很多种）
      emptyDir: {}                  #类型为emtyDir的存储卷，与Pod同生命周期的一个临时目录。为空值
      hostPath: string              #类型为hostPath的存储卷，表示挂载Pod所在宿主机的目录
        path: string                #Pod所在宿主机的目录，将被用于同期中mount的目录
      secret:                    #类型为secret的存储卷，挂载集群与定义的secre对象到容器内部
        scretname: string  
        items:     
        - key: string
          path: string
      configMap:                #类型为configMap的存储卷，挂载预定义的configMap对象到容器内部
        name: string
        items:
        - key: string
```





### 创建Deployment

```yaml
apiVersion: extensions/v1beta1   #接口版本
kind: Deployment                 #接口类型
metadata:
  name: cango-demo               #Deployment名称
  namespace: cango-prd           #命名空间
  labels:
    app: cango-demo              #标签
spec:
  replicas: 3
  strategy:
    rollingUpdate:  ##由于replicas为3,则整个升级,pod个数在2-4个之间
      maxSurge: 1      #滚动升级时会先启动1个pod
      maxUnavailable: 1 #滚动升级时允许的最大Unavailable的pod个数
  template:         
    metadata:
      labels:
        app: cango-demo  #模板名称必填
    sepc: #定义容器模板，该模板可以包含多个容器
      containers:                                                                   
        - name: cango-demo                                                           #镜像名称
          image: swr.cn-east-2.myhuaweicloud.com/cango-prd/cango-demo:0.0.1-SNAPSHOT #镜像地址
          command: [ "/bin/sh","-c","cat /etc/config/path/to/special-key" ]    #启动命令
          args:                                                                #启动参数
            - '-storage.local.retention=$(STORAGE_RETENTION)'
            - '-storage.local.memory-chunks=$(STORAGE_MEMORY_CHUNKS)'
            - '-config.file=/etc/prometheus/prometheus.yml'
            - '-alertmanager.url=http://alertmanager:9093/alertmanager'
            - '-web.external-url=$(EXTERNAL_URL)'
    #如果command和args均没有写，那么用Docker默认的配置。
    #如果command写了，但args没有写，那么Docker默认的配置会被忽略而且仅仅执行.yaml文件的command（不带任何参数的）。
    #如果command没写，但args写了，那么Docker默认配置的ENTRYPOINT的命令行会被执行，但是调用的参数是.yaml中的args。
    #如果如果command和args都写了，那么Docker默认的配置被忽略，使用.yaml的配置。
          imagePullPolicy: IfNotPresent  #如果不存在则拉取
          livenessProbe:       #表示container是否处于live状态。如果LivenessProbe失败，LivenessProbe将会通知kubelet对应的container不健康了。随后kubelet将kill掉container，并根据RestarPolicy进行进一步的操作。默认情况下LivenessProbe在第一次检测之前初始化值为Success，如果container没有提供LivenessProbe，则也认为是Success；
            httpGet:
              path: /health #如果没有心跳检测接口就为/
              port: 8080
              scheme: HTTP
            initialDelaySeconds: 60 ##启动后延时多久开始运行检测
            timeoutSeconds: 5
            successThreshold: 1
            failureThreshold: 5
          readinessProbe:
            httpGet:
              path: /health #如果没有心跳检测接口就为/
              port: 8080
              scheme: HTTP
            initialDelaySeconds: 30 ##启动后延时多久开始运行检测
            timeoutSeconds: 5
            successThreshold: 1
            failureThreshold: 5
          resources:              ##CPU内存限制
            requests:
              cpu: 2
              memory: 2048Mi
            limits:
              cpu: 2
              memory: 2048Mi
          env:                    ##通过环境变量的方式，直接传递pod=自定义Linux OS环境变量
            - name: LOCAL_KEY     #本地Key
              value: value
            - name: CONFIG_MAP_KEY  #局策略可使用configMap的配置Key，
              valueFrom:
                configMapKeyRef:
                  name: special-config   #configmap中找到name为special-config
                  key: special.type      #找到name为special-config里data下的key
          ports:
            - name: http
              containerPort: 8080 #对service暴露端口
          volumeMounts:     #挂载volumes中定义的磁盘
          - name: log-cache
            mount: /tmp/log
          - name: sdb       #普通用法，该卷跟随容器销毁，挂载一个目录
            mountPath: /data/media    
          - name: nfs-client-root    #直接挂载硬盘方法，如挂载下面的nfs目录到/mnt/nfs
            mountPath: /mnt/nfs
          - name: example-volume-config  #高级用法第1种，将ConfigMap的log-script,backup-script分别挂载到/etc/config目录下的一个相对路径path/to/...下，如果存在同名文件，直接覆盖。
            mountPath: /etc/config       
          - name: rbd-pvc                #高级用法第2中，挂载PVC(PresistentVolumeClaim)
 
#使用volume将ConfigMap作为文件或目录直接挂载，其中每一个key-value键值对都会生成一个文件，key为文件名，value为内容，
  volumes:  # 定义磁盘给上面volumeMounts挂载
  - name: log-cache
    emptyDir: {}
  - name: sdb  #挂载宿主机上面的目录
    hostPath:
      path: /any/path/it/will/be/replaced
  - name: example-volume-config  # 供ConfigMap文件内容到指定路径使用
    configMap:
      name: example-volume-config  #ConfigMap中名称
      items:
      - key: log-script           #ConfigMap中的Key
        path: path/to/log-script  #指定目录下的一个相对路径path/to/log-script
      - key: backup-script        #ConfigMap中的Key
        path: path/to/backup-script  #指定目录下的一个相对路径path/to/backup-script
  - name: nfs-client-root         #供挂载NFS存储类型
    nfs:
      server: 10.42.0.55          #NFS服务器地址
      path: /opt/public           #showmount -e 看一下路径
  - name: rbd-pvc                 #挂载PVC磁盘
    persistentVolumeClaim:
      claimName: rbd-pvc1         #挂载已经申请的pvc磁盘
```























































































## 常用管理命令

### Kubeadm

#### init

Kubeadm init用来初始化一个Kubernetes控制平面节点，执行阶段如下：

```shell
# 运行指令查看init的执行步骤
kubeadm init --help
# ---------------
# 可以看到以下输出，如果需要分阶段运行，则可以直接输入阶段以及子阶段进行单独运行
preflight                    预检
certs                        生成证书
  /ca                          生成自签名根 CA 用于配置其他 kubernetes 组件
  /apiserver                   生成 apiserver 的证书
  /apiserver-kubelet-client    生成 apiserver 连接到 kubelet 的证书
  /front-proxy-ca              生成前端代理自签名CA(扩展apiserver)
  /front-proxy-client          生成前端代理客户端的证书（扩展 apiserver）
  /etcd-ca                     生成 etcd 自签名 CA
  /etcd-server                 生成 etcd 服务器证书
  /etcd-peer                   生成 etcd 节点相互通信的证书
  /etcd-healthcheck-client     生成 etcd 健康检查的证书
  /apiserver-etcd-client       生成 apiserver 访问 etcd 的证书
  /sa                          生成用于签署服务帐户令牌的私钥和公钥
kubeconfig                   生成建立控制平面和管理所需的所有 kubeconfig 文件
  /admin                       生成一个 kubeconfig 文件供管理员使用以及供 kubeadm 本身使用
  /super-admin                 为超级管理员生成 kubeconfig 文件
  /kubelet                     为 kubelet 生成一个 kubeconfig 文件，*仅*用于集群引导
  /controller-manager          生成 kubeconfig 文件供控制器管理器使用
  /scheduler                   生成 kubeconfig 文件供调度程序使用
etcd                         为本地 etcd 生成静态 Pod 清单文件
  /local                       为本地单节点本地 etcd 实例生成静态 Pod 清单文件
control-plane                生成建立控制平面所需的所有静态 Pod 清单文件
  /apiserver                   生成 kube-apiserver 静态 Pod 清单
  /controller-manager          生成 kube-controller-manager 静态 Pod 清单
  /scheduler                   生成 kube-scheduler 静态 Pod 清单
kubelet-start                写入 kubelet 设置并启动（或重启） kubelet
upload-config                将 kubeadm 和 kubelet 配置上传到 ConfigMap
  /kubeadm                     将 kubeadm 集群配置上传到 ConfigMap
  /kubelet                     将 kubelet 组件配置上传到 ConfigMap
upload-certs                 将证书上传到 kubeadm-certs
mark-control-plane           将节点标记为控制面
bootstrap-token              生成用于将节点加入集群的引导令牌
kubelet-finalize             在 TLS 引导后更新与 kubelet 相关的设置
  /experimental-cert-rotation  启用 kubelet 客户端证书轮换
addon                        安装用于通过一致性测试所需的插件
  /coredns                     将 CoreDNS 插件安装到 Kubernetes 集群
  /kube-proxy                  将 kube-proxy 插件安装到 Kubernetes 集群
show-join-command            显示控制平面和工作节点的加入命令
```



分阶段运行init：

```bash
# 查看特定父阶段的子阶段列表
sudo kubeadm init phase control-plane --help
# 查看某个子阶段可用选项的列表
sudo kubeadm init phase control-plane controller-manager --help
# 分阶段运行：重新生成admin.conf文件
sudo kubeadm init phase kubeconfig admin
# 重新生成token并显示加入的命令
kubeadm token create --print-join-command
```

#### config

```bash
# 打印预定义的配置文件
kubeadm config print [command]
# Command:
# init-defaults    Print default init configuration, that can be used for 'kubeadm init'
# join-defaults    Print default join configuration, that can be used for 'kubeadm join'
# reset-defaults   Print default reset configuration, that can be used for 'kubeadm reset'
# upgrade-defaults Print default upgrade configuration, that can be used for 'kubeadm upgrade'

# 列出kubeadm需要使用的镜像
kubeadm config images list
# 拉取kubeadm需要使用的镜像
sudo kubeadm config images pull

# 迁移原来的kubeadm配置文件
sudo kubeadm config migrate --old-config init-defaults.yaml

# 验证kubeadm配置文件
sudo kubeadm config validate --config init-defaults.yaml
```



### reset

```bash
sudo kubeadm reset
```





### Kubelet

```bash
# Kubelet service内容文件如下(/usr/lib/systemd/system/kubelet.service)
[Unit]
Description=kubelet: The Kubernetes Node Agent
Documentation=https://kubernetes.io/docs/
Wants=network-online.target
After=network-online.target

[Service]
ExecStart=/usr/bin/kubelet
Restart=always
StartLimitInterval=0
RestartSec=10

[Install]
WantedBy=multi-user.target


# Kubelet service环境配置文件如下(/usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf)
# Note: This dropin only works with kubeadm and kubelet v1.11+
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf"
Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"
# This is a file that "kubeadm init" and "kubeadm join" generates at runtime, populating the KUBELET_KUBEADM_ARGS variable dynamically
EnvironmentFile=-/var/lib/kubelet/kubeadm-flags.env
# This is a file that the user can use for overrides of the kubelet args as a last resort. Preferably, the user should use
# the .NodeRegistration.KubeletExtraArgs object in the configuration files instead. KUBELET_EXTRA_ARGS should be sourced from this file.
EnvironmentFile=-/etc/default/kubelet
ExecStart=
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS
```







### Kubectl

```bash
kubectl get 组件名      # 例如kubectl get pod   查看详细信息可以加上-o wide   其他namespace的指定 -n namespace名
kubectl apply -f xxx.yaml    # 例如kubectl apply -f nginx.yaml   这里是如果没有则创建，如果有则变更，比create好用
kubectl delete -f xxx.yaml    # 例如kubectl delete -f nginx.yaml
kubectl describe pod pod名     # 先用kubectl get pod查看  有异常的复制pod名使用这个命令
kubectl logs pod名     # 先用kubectl get pod查看  有异常的复制pod名使用这个命令
kubectl top 组件名     # 查看node节点或者是pod资源使用情况
kubectl exec -ti pod名 /bin/bash      # 进入pod内部

```

























































## 常见问题

### 在安装过程中，kubeadm 一直等待控制平面就绪

> https://www.cnblogs.com/panw/p/16344617.html

如果你注意到 `kubeadm init` 在打印以下行后挂起：

```console
[apiclient] Created API client, waiting for the control plane to become ready
```

这可能是由许多问题引起的。最常见的是：

- 网络连接问题。在继续之前，请检查你的计算机是否具有全部联通的网络连接。
- 容器运行时的 cgroup 驱动不同于 kubelet 使用的 cgroup 驱动。要了解如何正确配置 cgroup 驱动， 请参阅[配置 cgroup 驱动](https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/)。
- 控制平面上的 Docker 容器持续进入崩溃状态或（因其他原因）挂起。你可以运行 `docker ps` 命令来检查以及 `docker logs` 命令来检视每个容器的运行日志。 对于其他容器运行时，请参阅[使用 crictl 对 Kubernetes 节点进行调试](https://kubernetes.io/zh-cn/docs/tasks/debug/debug-cluster/crictl/)。



#### containerd与kubelet使用的驱动没有对应上

> https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/

通过`sudo journalctl -xeu kubelet`可以查看到报错信息如下：

```shell
kubelet[53074]: "Unhandled Error" err="kubernetes.io/kube-apiserver-client-kubelet: Failed while requesting a signed certificate from the control plane: cannot create certificate signing request: Post \"https://192.168.6.1:6443/apis/certificates.k8s.io/v1/certificatesigningrequests\": dial tcp 192.168.6.1:6443: connect: connection refused"
kubelet[53074]: RuntimeConfig from runtime service failed" err="rpc error: code = Unimplemented desc = unknown method RuntimeConfig for service runtime.v1.RuntimeService"
kubelet[53074]: "CRI implementation should be updated to support RuntimeConfig when KubeletCgroupDriverFromCRI feature gate has been enabled. Falling back to using cgroupDriver from kubelet config."
```

此时需要调整kubeadm的cgroup驱动和容器运行时的cgroup驱动保持一致（默认为systemd，所以尽量都调整为systemd）

##### 调整kubeadm

```yaml
# kubeadm-config.yaml
kind: ClusterConfiguration
apiVersion: kubeadm.k8s.io/v1beta4
kubernetesVersion: v1.21.0
---
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
cgroupDriver: systemd
```

增加`cgroupDriver`字段，如果没有该字段则默认为systemd为驱动



##### 调整containerd

```bash
# 首先生成默认配置
containerd config default | sudo tee /etc/containerd/config.toml > /dev/null
# 然后修改该配置sudo vim /etc/containerd/config.toml，如下带注释的是需要修改的
disabled_plugins = []
imports = []
oom_score = 0
plugin_dir = ""
required_plugins = []
root = "/var/lib/containerd"
state = "/run/containerd"
temp = ""
version = 2

[cgroup]
  path = ""

[debug]
  address = ""
  format = ""
  gid = 0
  level = ""
  uid = 0

[grpc]
  address = "/run/containerd/containerd.sock"
  gid = 0
  max_recv_message_size = 16777216
  max_send_message_size = 16777216
  tcp_address = ""
  tcp_tls_ca = ""
  tcp_tls_cert = ""
  tcp_tls_key = ""
  uid = 0

[metrics]
  address = ""
  grpc_histogram = false

[plugins]

  [plugins."io.containerd.gc.v1.scheduler"]
    deletion_threshold = 0
    mutation_threshold = 100
    pause_threshold = 0.02
    schedule_delay = "0s"
    startup_delay = "100ms"

  [plugins."io.containerd.grpc.v1.cri"]
    cdi_spec_dirs = ["/etc/cdi", "/var/run/cdi"]
    device_ownership_from_security_context = false
    disable_apparmor = false
    disable_cgroup = false
    disable_hugetlb_controller = true
    disable_proc_mount = false
    disable_tcp_service = true
    drain_exec_sync_io_timeout = "0s"
    enable_cdi = false
    enable_selinux = false
    enable_tls_streaming = false
    enable_unprivileged_icmp = false
    enable_unprivileged_ports = false
    ignore_deprecation_warnings = []
    ignore_image_defined_volumes = false
    image_pull_progress_timeout = "5m0s"
    image_pull_with_sync_fs = false
    max_concurrent_downloads = 3
    max_container_log_line_size = 16384
    netns_mounts_under_state_dir = false
    restrict_oom_score_adj = false
    sandbox_image = "harbor.zeus-netlab.com/library/pause:3.10"  # 此处需要调整为自己能用的镜像源
    selinux_category_range = 1024
    stats_collect_period = 10
    stream_idle_timeout = "4h0m0s"
    stream_server_address = "127.0.0.1"
    stream_server_port = "0"
    systemdCgroup = true   # 此处需要调整字段为systemdCgroup(原来可能是system_cgroup)，并设置为true
    tolerate_missing_hugetlb_controller = true
    unset_seccomp_profile = ""

    [plugins."io.containerd.grpc.v1.cri".cni]
      bin_dir = "/opt/cni/bin"
      conf_dir = "/etc/cni/net.d"
      conf_template = ""
      ip_pref = ""
      max_conf_num = 1
      setup_serially = false

    [plugins."io.containerd.grpc.v1.cri".containerd]
      default_runtime_name = "runc"
      disable_snapshot_annotations = true
      discard_unpacked_layers = false
      ignore_blockio_not_enabled_errors = false
      ignore_rdt_not_enabled_errors = false
      no_pivot = false
      snapshotter = "overlayfs"

      [plugins."io.containerd.grpc.v1.cri".containerd.default_runtime]
        base_runtime_spec = ""
        cni_conf_dir = ""
        cni_max_conf_num = 0
        container_annotations = []
        pod_annotations = []
        privileged_without_host_devices = false
        privileged_without_host_devices_all_devices_allowed = false
        runtime_engine = ""
        runtime_path = ""
        runtime_root = ""
        runtime_type = ""
        sandbox_mode = ""
        snapshotter = ""

        [plugins."io.containerd.grpc.v1.cri".containerd.default_runtime.options]

      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]

        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
          base_runtime_spec = ""
          cni_conf_dir = ""
          cni_max_conf_num = 0
          container_annotations = []
          pod_annotations = []
          privileged_without_host_devices = false
          privileged_without_host_devices_all_devices_allowed = false
          runtime_engine = ""
          runtime_path = ""
          runtime_root = ""
          runtime_type = "io.containerd.runc.v2"
          sandbox_mode = "podsandbox"
          snapshotter = ""

          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
            BinaryName = ""
            CriuImagePath = ""
            CriuPath = ""
            CriuWorkPath = ""
            IoGid = 0
            IoUid = 0
            NoNewKeyring = false
            NoPivotRoot = false
            Root = ""
            ShimCgroup = ""
            SystemdCgroup = true  # runc的cgroup驱动也需要修改为true

      [plugins."io.containerd.grpc.v1.cri".containerd.untrusted_workload_runtime]
        base_runtime_spec = ""
        cni_conf_dir = ""
        cni_max_conf_num = 0
        container_annotations = []
        pod_annotations = []
        privileged_without_host_devices = false
        privileged_without_host_devices_all_devices_allowed = false
        runtime_engine = ""
        runtime_path = ""
        runtime_root = ""
        runtime_type = ""
        sandbox_mode = ""
        snapshotter = ""

        [plugins."io.containerd.grpc.v1.cri".containerd.untrusted_workload_runtime.options]

    [plugins."io.containerd.grpc.v1.cri".image_decryption]
      key_model = "node"

    [plugins."io.containerd.grpc.v1.cri".registry]
      config_path = ""

      [plugins."io.containerd.grpc.v1.cri".registry.auths]

      [plugins."io.containerd.grpc.v1.cri".registry.configs]

      [plugins."io.containerd.grpc.v1.cri".registry.headers]

      [plugins."io.containerd.grpc.v1.cri".registry.mirrors]

    [plugins."io.containerd.grpc.v1.cri".x509_key_pair_streaming]
      tls_cert_file = ""
      tls_key_file = ""

  [plugins."io.containerd.internal.v1.opt"]
    path = "/opt/containerd"

  [plugins."io.containerd.internal.v1.restart"]
    interval = "10s"

  [plugins."io.containerd.internal.v1.tracing"]

  [plugins."io.containerd.metadata.v1.bolt"]
    content_sharing_policy = "shared"

  [plugins."io.containerd.monitor.v1.cgroups"]
    no_prometheus = false

  [plugins."io.containerd.nri.v1.nri"]
    disable = true
    disable_connections = false
    plugin_config_path = "/etc/nri/conf.d"
    plugin_path = "/opt/nri/plugins"
    plugin_registration_timeout = "5s"
    plugin_request_timeout = "2s"
    socket_path = "/var/run/nri/nri.sock"

  [plugins."io.containerd.runtime.v1.linux"]
    no_shim = false
    runtime = "runc"
    runtime_root = ""
    shim = "containerd-shim"
    shim_debug = false

  [plugins."io.containerd.runtime.v2.task"]
    platforms = ["linux/amd64"]
    sched_core = false

  [plugins."io.containerd.service.v1.diff-service"]
    default = ["walking"]

  [plugins."io.containerd.service.v1.tasks-service"]
    blockio_config_file = ""
    rdt_config_file = ""

  [plugins."io.containerd.snapshotter.v1.aufs"]
    root_path = ""

  [plugins."io.containerd.snapshotter.v1.blockfile"]
    fs_type = ""
    mount_options = []
    root_path = ""
    scratch_file = ""

  [plugins."io.containerd.snapshotter.v1.btrfs"]
    root_path = ""

  [plugins."io.containerd.snapshotter.v1.devmapper"]
    async_remove = false
    base_image_size = ""
    discard_blocks = false
    fs_options = ""
    fs_type = ""
    pool_name = ""
    root_path = ""

  [plugins."io.containerd.snapshotter.v1.native"]
    root_path = ""

  [plugins."io.containerd.snapshotter.v1.overlayfs"]
    mount_options = []
    root_path = ""
    sync_remove = false
    upperdir_label = false

  [plugins."io.containerd.snapshotter.v1.zfs"]
    root_path = ""

  [plugins."io.containerd.tracing.processor.v1.otlp"]

  [plugins."io.containerd.transfer.v1.local"]
    config_path = ""
    max_concurrent_downloads = 3
    max_concurrent_uploaded_layers = 3

    [[plugins."io.containerd.transfer.v1.local".unpack_config]]
      differ = ""
      platform = "linux/amd64"
      snapshotter = "overlayfs"

[proxy_plugins]

[stream_processors]

  [stream_processors."io.containerd.ocicrypt.decoder.v1.tar"]
    accepts = ["application/vnd.oci.image.layer.v1.tar+encrypted"]
    args = ["--decryption-keys-path", "/etc/containerd/ocicrypt/keys"]
    env = ["OCICRYPT_KEYPROVIDER_CONFIG=/etc/containerd/ocicrypt/ocicrypt_keyprovider.conf"]
    path = "ctd-decoder"
    returns = "application/vnd.oci.image.layer.v1.tar"

  [stream_processors."io.containerd.ocicrypt.decoder.v1.tar.gzip"]
    accepts = ["application/vnd.oci.image.layer.v1.tar+gzip+encrypted"]
    args = ["--decryption-keys-path", "/etc/containerd/ocicrypt/keys"]
    env = ["OCICRYPT_KEYPROVIDER_CONFIG=/etc/containerd/ocicrypt/ocicrypt_keyprovider.conf"]
    path = "ctd-decoder"
    returns = "application/vnd.oci.image.layer.v1.tar+gzip"

[timeouts]
  "io.containerd.timeout.bolt.open" = "0s"
  "io.containerd.timeout.metrics.shimstats" = "2s"
  "io.containerd.timeout.shim.cleanup" = "5s"
  "io.containerd.timeout.shim.load" = "5s"
  "io.containerd.timeout.shim.shutdown" = "3s"
  "io.containerd.timeout.task.state" = "2s"

[ttrpc]
  address = ""
  gid = 0
  uid = 0
```

修改后保存，并重新启动containerd，执行`sudo systemctl restart containerd`



### Calico插件TODO

```bash
# 出现以下提示
"error":"Could not resolve CalicoNetwork IPPool and kubeadm configuration: kubeadm configuration is missing required podSubnet field"
```

由于没有指定

可以直接使用kubectl修改：

```yaml
kubectl edit cm kubeadm-config -n kube-system
# 在文件中对配置进行修改
networking:
      dnsDomain: cluster.local
      serviceSubnet: 10.96.0.0/12
      podSubnet: 10.244.0.0/16   # 添加这一行，具体的值可以自定义但要和calico保持一致
```

保存并退出后重新添加插件：

```bash
# 先删除之前添加的插件
kubectl delete -f custom-resources.yaml
kubectl delete -f tigera-operator.yaml
# 再次添加
kubectl create -f tigera-operator.yaml
kubectl create -f custom-resources.yaml
```



### 当删除托管容器时 kubeadm 阻塞

如果容器运行时停止并且未删除 Kubernetes 所管理的容器，可能发生以下情况：

```shell
sudo kubeadm reset
[preflight] Running pre-flight checks
[reset] Stopping the kubelet service
[reset] Unmounting mounted directories in "/var/lib/kubelet"
[reset] Removing kubernetes-managed containers
(block)
```

一个可行的解决方案是重新启动 Docker 服务，然后重新运行 `kubeadm reset`： 你也可以使用 `crictl` 来调试容器运行时的状态。 参见[使用 CRICTL 调试 Kubernetes 节点](https://kubernetes.io/zh-cn/docs/tasks/debug/debug-cluster/crictl/)。



### kubeadm创建后各种pod重复启停

> https://blog.csdn.net/roadtohacker/article/details/134654399

可能是由于没有设置runc的cgroupdriver为systemd导致的，可以查看`/etc/containerd/config.toml`调整`SystemdCgroup`字段为true：

```toml
version = 2
[plugins]
  [plugins."io.containerd.grpc.v1.cri"]
   [plugins."io.containerd.grpc.v1.cri".containerd]
      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
          runtime_type = "io.containerd.runc.v2"
          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
            SystemdCgroup = true
```





### Kubelet修改podSubnet

> - https://blog.csdn.net/qq_36713450/article/details/109524817

需要在主节点修改集群配置：

```bash
kubectl edit cm kubeadm-config -n kube-system
```

进入修改页面后：

```yaml
data:
  ClusterConfiguration:
  	......
  	networking:
      dnsDomain: cluster.local
      serviceSubnet: 10.96.0.0/12
      podSubnet: 10.244.0.0/16  # 加入此行，具体的pod网段可以自行设置
```

修改静态pod的kube-controller-manager配置：

```bash
sudo vim /etc/kubernetes/manifests/kube-controller-manager.yaml 
# 加入以下启动参数
- --allocate-node-cidrs=true 
- --cluster-cidr=10.244.0.0/16 # 自行制定pod网段
```

如果有安装kube-proxy，则需要修改器配置为：

```bash
kube-proxy  --cluster-cidr 10.244.0.0/16
```





### kubelet设置指定的静态Internal-IP

> - https://medium.com/@kanrangsan/how-to-specify-internal-ip-for-kubernetes-worker-node-24790b2884fd
> - 

如果一个节点设置了多个IP地址，有可能会出现默认Internal-IP不是想要的IP地址，使用以下命令查看当前节点信息

```bash
# 使用该命令获取所有的节点的完整信息
kubectl get nodes -o wide
# 可以看到以下信息
NAME              STATUS     ROLES           AGE     VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
node              NotReady   control-plane   3d22h   v1.31.1   10.134.149.79    <none>        Ubuntu 24.04.1 LTS   6.8.0-45-generic   containerd://1.7.22
zeus-k8s-node-2   NotReady   <none>          3d4h    v1.31.1   10.134.148.131   <none>        Ubuntu 24.04.1 LTS   6.8.0-45-generic   containerd://1.7.22
zeus-k8s-node-3   NotReady   <none>          3d4h    v1.31.1   10.134.148.118   <none>        Ubuntu 24.04.1 LTS   6.8.0-45-generic   containerd://1.7.22
```

首先使用kubectl直接修改node信息的方式不太行：

```bash
# 需要根据上面查看的信息先指定node_name
kubectl edit node <node_name>
# 有方法是通过修改以下字段调整，但是使用后发现此种方法无效
status:
  addresses:
  - address: 10.134.149.79 # 修改这里
```

目前可行方案是通过调整kubelet配置：

```bash
# 查看Kubelet服务的信息
sudo systemctl status kubelet
# 发现Kubelet服务信息文件在/usr/lib/systemd/system/kubelet.service以及/usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf
# 打开10-kubeadm.conf设置文件看到以下内容
sudo vim /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf
# --------------------------------------------------------------------
# Note: This dropin only works with kubeadm and kubelet v1.11+
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf"
Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"
# This is a file that "kubeadm init" and "kubeadm join" generates at runtime, populating the KUBELET_KUBEADM_ARGS variable dynamically
EnvironmentFile=-/var/lib/kubelet/kubeadm-flags.env
# This is a file that the user can use for overrides of the kubelet args as a last resort. Preferably, the user should use
# the .NodeRegistration.KubeletExtraArgs object in the configuration files instead. KUBELET_EXTRA_ARGS should be sourced from this file.
EnvironmentFile=-/etc/default/kubelet
ExecStart=
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS
```

其中可以使用`$KUBELET_EXTRA_ARGS`参数，需要在**每个需要调整的节点**的`/etc/default/kubelet`中写入以下设置：

```bash
KUBELET_EXTRA_ARGS="--node-ip=192.168.6.1" # 自己指定需要调整的ip
```

最后重启该服务：

```bash
sudo systemctl restart kubelet
```

再次查看所有节点可以看到Internal-IP更改成功



### Kubernetes修改主机名与节点名称

> - https://www.cnblogs.com/dudu/p/14286983.html
