### `操作步骤`
```
一、更新系统内核（全部节点）

二、基础环境设置（全部节点）
1、修改 Host
2、修改 Hostname
3、主机时间同步
4、关闭防火墙服务
5、关闭并禁用SELinux
6、禁用 Swap 设备
7、设置内核参数
8、配置 IPVS 模块
9、配置资源限制
10、安装依赖包以及相关工具

三、安装Docker（全部节点）
1、移除之前安装过的Docker
2、更换 Docker 的 yum 源
3、显示 docker 所有可安装版本：
4、安装指定版本 docker
5、配置 Docker 参数和镜像加速器
6、启动 docker 并设置 docker 开机启动

四、安装 kubelet、kubectl、kubeadm（全部节点）
1、配置可用的国内 yum 源
2、安装 kubelet、kubectl、kubeadm
3、启动 kubelet 并配置开机启动

五、重启服务器（全部节点）

六、kubeadm 安装 kubernetes（Master 节点）

七、工作节点加入集群（Work Node 节点）

八、部署网络插件（Master 节点）
1、部署 Calico 网络插件
2、查看 Pod 是否成功启动

九、配置 Kubectl 命令自动补全（Master 节点）

十、查看是否开启 IPVS（Master 节点）

十一、部署 Kubernetes Dashboard
1、创建 Dashboard 部署文件
2、创建监控信息 kubernetes-metrics-scraper
3、创建 Dashboard ServiceAccount
```

-------------
系统环境：
```
•   CentOS 版本：7.5+
•   Docker 版本：18.09.9-3
•   Calico 版本：v3.10
•   Kubernetes 版本：1.16.3
•   Kubernetes Newwork 模式：IPVS
•   Kubernetes Dashboard 版本：dashboard:v2.0.0-beta6
```

服务器信息：

|地址|  主机名| 内存&CPU | 角色|
|:-:|:-:|:-:|:-:|
|192.168.2.11   | k8s-master | 4C & 4G |master|
|192.168.2.12   | k8s-node-01 |16c & 64G |node|
|192.168.2.13   | k8s-node-02 |16c & 64G |node|

-----------
`注意事项：`
kubeadm 的证书有效期是一年，在一年内执行任意一次更新集群命令就可与再次刷新证书为一年时间，如果想改成永久，可与网上查找修改 kubeadm 源码方法，手动编译，然后执行创建 kubernetes 集群操作。

## 一、更新系统内核（全部节点）
由于 Docker 对系统内核有一定的要求，所以我们最好使用 yum 来更新系统软件及其内核。
#### 备份本地 yum 源
```
$ mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo_bak 
```

#### 获取阿里 yum 源配置文件
```
$ wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo 
```

#### 清理 yum
```
$ yum clean all
```

#### 更新软件版本并且更新现有软件
```
$ yum -y update
```

## 二、基础环境设置（全部节点）
Kubernetes 需要一定的环境来保证正常运行，如各个节点时间同步，主机名称解析，关闭防火墙等等。

#### 1、修改 Host
分布式系统环境中的多主机通信通常基于主机名称进行，这在 IP 地址存在变化的可能 性时为主机提供了固定的访问人口，因此一般需要有专用的 DNS 服务负责解析各节点主机。考虑到此处部署的是测试集群，因此为了降低系复杂度，这里将基于 hosts 的文件进行主机名称解析。
```
$ vim /etc/hosts
```
加入下面内容：
```
192.168.2.11   k8s-master
192.168.2.12   k8s-node-01
192.168.2.13   k8s-node-02
```

#### 2、修改 Hostname
kubernetes 中会以各个服务的 hostname 为其节点命名，所以需要进入不同的服务器修改 hostname 名称。
```
#修改 192.168.2.11 服务器，设置 hostname，然后将 hostname 写入 hosts
$ hostnamectl  set-hostname  k8s-master
$ echo "127.0.0.1   $(hostname)" >> /etc/hosts

#修改 192.168.2.12 服务器，设置 hostname，然后将 hostname 写入 hosts
$ hostnamectl  set-hostname  k8s-node-01
$ echo "127.0.0.1   $(hostname)" >> /etc/hosts

#修改 192.168.2.13 服务器，设置 hostname，然后将 hostname 写入 hosts
$ hostnamectl  set-hostname  k8s-node-02
$ echo "127.0.0.1   $(hostname)" >> /etc/hosts
```

#### 3、主机时间同步
将各个服务器的时间同步，并设置开机启动同步时间服务。
```
$ systemctl start chronyd.service && systemctl enable chronyd.service
```

#### 4、关闭防火墙服务
关闭防火墙，并禁止开启启动。
```
$ systemctl stop firewalld && systemctl disable firewalld
```

#### 5、关闭并禁用SELinux
关闭SELinux，并编辑／etc/sysconfig selinux 文件，以彻底禁用 SELinux
```
$ setenforce 0
$ sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/' /etc/selinux/config
```
查看selinux状态
```
$ getenforce 
```

#### 6、禁用 Swap 设备
关闭当前已启用的所有 Swap 设备：
```
$ swapoff -a && sysctl -w vm.swappiness=0
```
编辑 fstab 配置文件，注释掉标识为 Swap 设备的所有行：
```
$ vi /etc/fstab

#/dev/mapper/centos-swap swap  swap  defaults  0 0
```

#### 7、设置内核参数
配置内核参数：
```
$ cat <<EOF > /etc/sysctl.d/k8s.conf

net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.tcp_keepalive_time = 600
net.ipv4.tcp_keepalive_intvl = 30
net.ipv4.tcp_keepalive_probes = 10
EOF
```
使配置生效:
```
#挂载 br_netfilter
$ modprobe br_netfilter

#使配置生效
$ sysctl -p /etc/sysctl.d/k8s.conf

#查看是否生成相关文件
$ ls /proc/sys/net/bridge
```

#### 8、配置 IPVS 模块
由于ipvs已经加入到了内核的主干，所以为kube-proxy开启ipvs的前提需要加载以下的内核模块：
•   ip_vs
•   ip_vs_rr
•   ip_vs_wrr
•   ip_vs_sh
•   nf_conntrack_ipv4
```
$ cat > /etc/sysconfig/modules/ipvs.modules <<EOF

#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4

EOF
```

执行脚本并查看是否正常加载内核模块：
```
#修改脚本权限
$ chmod 755 /etc/sysconfig/modules/ipvs.modules 

#执行脚本
$ bash /etc/sysconfig/modules/ipvs.modules 

#查看是否已经正确加载所需的内核模块
$ lsmod | grep -e ip_vs -e nf_conntrack_ipv4
```

安装 ipset
```
$ yum install -y ipset
```

#### 9、配置资源限制
```
echo "* soft nofile 65536" >> /etc/security/limits.conf
echo "* hard nofile 65536" >> /etc/security/limits.conf
echo "* soft nproc 65536"  >> /etc/security/limits.conf
echo "* hard nproc 65536"  >> /etc/security/limits.conf
echo "* soft memlock  unlimited"  >> /etc/security/limits.conf
echo "* hard memlock  unlimited"  >> /etc/security/limits.conf
```

#### 10、安装依赖包以及相关工具
```
$ yum install -y epel-release
$ yum install -y yum-utils device-mapper-persistent-data lvm2 net-tools conntrack-tools wget vim ntpdate libseccomp libtool-ltdl
```

## 三、安装Docker（全部节点）
#### 1、移除之前安装过的Docker
```
$ sudo yum -y remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-ce-cli \
                  docker-engine
```
查看还有没有存在的 Docker 组件，一定要确保删除干净：
```
$ rpm -qa | grep docker
```
有则通过命令 yum -y remove XXX 来删除，比如：
```
$ yum remove docker-ce-cli
```

#### 2、更换 Docker 的 yum 源
由于官方下载速度比较慢，所以需要更改 Docker 安装的 yum 源，这里推荐用阿里镜像源：
```
$ yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

#### 3、显示 docker 所有可安装版本：
```
$ yum list docker-ce --showduplicates | sort -r

docker-ce.x86_64            3:19.03.0-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.9-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.8-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.7-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.6-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.5-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.4-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.3-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.2-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.1-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.0-3.el7                     docker-ce-stable
docker-ce.x86_64            18.06.3.ce-3.el7                    docker-ce-stable
docker-ce.x86_64            18.06.2.ce-3.el7                    docker-ce-stable
docker-ce.x86_64            18.06.1.ce-3.el7                    docker-ce-stable
docker-ce.x86_64            18.06.0.ce-3.el7                    docker-ce-stable
docker-ce.x86_64            18.03.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            18.03.0.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.12.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.12.0.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.09.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.09.0.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.06.2.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.06.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.06.0.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.03.3.ce-1.el7                    docker-ce-stable
docker-ce.x86_64            17.03.2.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.03.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.03.0.ce-1.el7.centos             docker-ce-stable
```
#### 4、安装指定版本 docker
> 注意：安装前一定要提前查询将要安装的 Kubernetes 版本是否和 Docker 版本对应。

```
$ yum install -y docker-ce-18.09.9-3.el7
```

设置镜像存储目录，找到大点的挂载的目录进行存储
```
$ vi /lib/systemd/system/docker.service

#找到这行，往后面加上存储目录，例如这里是 --graph /apps/docker
ExecStart=/usr/bin/docker --graph /apps/docker
```

#### 5、配置 Docker 参数和镜像加速器
Kubernetes 推荐的一些 Docker 配置参数，这里配置一下。还有就是由于国内访问 Docker 仓库速度很慢，所以国内几家云厂商推出镜像加速下载的代理加速器，这里也需要配置一下。
创建 Docker 配置文件的目录：
```
$ mkdir -p /etc/docker
```

添加配置文件：
```
$ cat > /etc/docker/daemon.json << EOF

{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "registry-mirrors": [
      "https://dockerhub.azk8s.cn",
      "http://hub-mirror.c.163.com",
      "https://registry.docker-cn.com"
  ],
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m",
    "max-file":"5"
  }
}

EOF
```

#### 6、启动 docker 并设置 docker 开机启动
启动 Docker：
```
$ systemctl start docker && systemctl enable docker
```
如果 Docker 已经启动，则需要重启 Docker：
```
$ systemctl daemon-reload
$ systemctl restart docker
```

## 四、安装 kubelet、kubectl、kubeadm（全部节点）
>> 实际kubeadm是init k8s master的核心组件编译和安装

#### 1、配置可用的国内 yum 源
```
$ cat <<EOF > /etc/yum.repos.d/kubernetes.repo

[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg

EOF
```

#### 2、安装 kubelet、kubectl、kubeadm
•   kubelet: 在集群中的每个节点上用来启动 pod 和 container 等。
•   kubectl: 用来与集群通信的命令行工具。
•   kubeadm: 用来初始化集群的指令。

>> 注意安装顺序，一定不要先安装 kubeadm，因为 kubeadm 会自动安装最新版本的 kubelet 与 kubectl，导致版本不一致问题。

```
#安装 kubelet
$ yum install -y kubelet-1.16.3-0

#安装 kubectl
$ yum install -y kubectl-1.16.3-0

#安装 kubeadm
$ yum install -y kubeadm-1.16.3-0
```

#### 3、启动 kubelet 并配置开机启动
```
$ systemctl start kubelet && systemctl enable kubelet
```

>> 检查状态时会发现 kubelet 是 failed 状态，等初 master 节点初始化完成后即可显示正常。

## 五、重启服务器（全部节点）
为了防止发生某些未知错误，这里我们重启下服务器，方便进行后续操作
```
$ reboot
```

## 六、kubeadm 安装 kubernetes（Master 节点）
创建 kubeadm 配置文件 kubeadm-config.yaml，然后需要配置一些参数：
•   配置 localAPIEndpoint.advertiseAddress 参数，调整为你的 Master 服务器地址。
•   配置 imageRepository 参数，调整 kubernetes 镜像下载地址为阿里云。
•   配置 networking.podSubnet 参数，调整为你要设置的网络范围。

`kubeadm-config.yaml`
```
$ cat > kubeadm-config.yaml << EOF

apiVersion: kubeadm.k8s.io/v1beta2
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 192.168.2.11
  bindPort: 6443
nodeRegistration:
  taints:
  - effect: PreferNoSchedule
    key: node-role.kubernetes.io/master
---
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
imageRepository: registry.aliyuncs.com/google_containers
kubernetesVersion: v1.16.3
networking:
  podSubnet: 10.244.0.0/16
---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: ipvs

EOF
```

kubeadm 初始化 kubernetes 集群
```
$ kubeadm init --config kubeadm-config.yaml
```
部署日志信息：
```
......
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.2.11:6443 --token 4udy8a.f77ai0zun477kx0p \
    --discovery-token-ca-cert-hash sha256:4645472f24b438e0ecf5964b6dcd64913f68e0f9f7458768cfb96a9ab16b4212
```
在此处看日志可以知道，可以通过下面命令，添加 kubernetes 相关环境变量：
```
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

#### 七、工作节点加入集群（Work Node 节点）
根据上面 Master 节点创建 Kubernetes 集群时的日志信息，可以知道在各个节点上执行下面命令来让工作节点加入主节点：
```
$ kubeadm join 192.168.2.11:6443 --token 4udy8a.f77ai0zun477kx0p \
    --discovery-token-ca-cert-hash sha256:4645472f24b438e0ecf5964b6dcd64913f68e0f9f7458768cfb96a9ab16b4212
```

#### 八、部署网络插件（Master 节点）
Kubernetes 中可以部署很多种网络插件，不过比较流行也推荐的有两种：
•   `Flannel`： Flannel 是基于 Overlay 网络模型的网络插件，能够方便部署，一般部署后只要不出问题，一般不需要管它。
•   `Calico`： 与 Flannel 不同，Calico 是一个三层的数据中心网络方案，使用 BGP 路由协议在主机之间路由数据包，可以灵活配置网络策略。
这两种网络根据环境任选其一即可，这里使用的是 Calico，可以按下面步骤部署：
#### 1、部署 Calico 网络插件
下载 Calico 部署文件，并替换里面的网络范围为上面 kubeadm 中 networking.podSubnet 配置的值。
```
#下载 calico 部署文件
$ wget https://docs.projectcalico.org/v3.10/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml 

#替换 calico 部署文件的 IP 为 kubeadm 中的 networking.podSubnet 参数 10.244.0.0。
$ sed -i 's/192.168.0.0/10.244.0.0/g' calico.yaml

#部署 Calico 插件
$ kubectl apply -f calico.yaml
```

#### 2、查看 Pod 是否成功启动
```
$ kubectl get pod -n kube-system

NAME                                       READY   STATUS    RESTARTS   AGE
calico-kube-controllers-6b64bcd855-jn8pz   1/1     Running   0          2m40s
calico-node-5wssd                          1/1     Running   0          2m40s
calico-node-7tw94                          1/1     Running   0          2m40s
calico-node-xzfp4                          1/1     Running   0          2m40s
coredns-58cc8c89f4-hv4fn                   1/1     Running   0          21m
coredns-58cc8c89f4-k97x6                   1/1     Running   0          21m
etcd-k8s-master                            1/1     Running   0          20m
kube-apiserver-k8s-master                  1/1     Running   0          20m
kube-controller-manager-k8s-master         1/1     Running   0          20m
kube-proxy-9dlpz                           1/1     Running   0          14m
kube-proxy-krd5n                           1/1     Running   0          14m
kube-proxy-tntpr                           1/1     Running   0          21m
kube-scheduler-k8s-master                  1/1     Running   0          20m
```

可以看到所以 Pod 都已经成功启动。

## 九、配置 Kubectl 命令自动补全（Master 节点）
```
$ yum install -y bash-completion
```
添加补全配置
```
$ source /usr/share/bash-completion/bash_completion
$ source <(kubectl completion bash)
$ echo "source <(kubectl completion bash)" >> ~/.bashrc
```
添加完成就可与通过输入 kubectl 后，按补全键（一般为 tab）会自动补全对应的命令。

## 十、查看是否开启 IPVS（Master 节点）
上面全部组件都已经部署完成，不过还需要确认是否成功将网络模式设置为 IPVS，可以查看 kube-proxy 日志，在日志信息中查找是否存在 IPVS 关键字信息来确认。
```
$ kubectl get pod -n kube-system | grep kube-proxy

kube-proxy-9dlpz                           1/1     Running   0          42m
kube-proxy-krd5n                           1/1     Running   0          42m
kube-proxy-tntpr                           1/1     Running   0          49m
```
选择其中一个 Pod ，查看该 Pod 中的日志信息中是否存在 ipvs 信息：
```
$ kubectl logs kube-proxy-9dlpz -n kube-system

I1120 18:13:46.357178       1 node.go:135] Successfully retrieved node IP: 192.168.2.13
I1120 18:13:46.357265       1 server_others.go:176] Using ipvs Proxier.
W1120 18:13:46.358005       1 proxier.go:420] IPVS scheduler not specified, use rr by default
I1120 18:13:46.358919       1 server.go:529] Version: v1.16.3
I1120 18:13:46.359327       1 conntrack.go:100] Set sysctl 'net/netfilter/nf_conntrack_max' to 131072
I1120 18:13:46.359379       1 conntrack.go:52] Setting nf_conntrack_max to 131072
I1120 18:13:46.359426       1 conntrack.go:100] Set sysctl 'net/netfilter/nf_conntrack_tcp_timeout_established' to 86400
I1120 18:13:46.359452       1 conntrack.go:100] Set sysctl 'net/netfilter/nf_conntrack_tcp_timeout_close_wait' to 3600
I1120 18:13:46.359626       1 config.go:313] Starting service config controller
I1120 18:13:46.359685       1 shared_informer.go:197] Waiting for caches to sync for service config
I1120 18:13:46.359833       1 config.go:131] Starting endpoints config controller
I1120 18:13:46.359889       1 shared_informer.go:197] Waiting for caches to sync for endpoints config
I1120 18:13:46.460013       1 shared_informer.go:204] Caches are synced for service config 
I1120 18:13:46.460062       1 shared_informer.go:204] Caches are synced for endpoints config 
```
如上，在日志中查到了 IPVS 字样，则代表使用了 IPVS 模式。

## 十一、部署 Kubernetes Dashboard
接下来我们将部署 Kubernetes 的控制看板，由于集群为 1.16.3，而 Dashboard 的稳定版本还是基于 Kubernetes 1.10 版本，所以我们直接使用 Kubernetes Dashboard 2.0.0 Bate 版本，虽然为测试版本，不过它比较新，对新版本的兼容还是比旧版本好些，坐等它稳定，这里先部署尝尝鲜。

#### 1、创建 Dashboard 部署文件
`k8s-dashboard-deploy.yaml`

```
$ cat > k8s-dashboard-deploy.yaml << EOF

apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
rules:
  # Allow Dashboard to get, update and delete Dashboard exclusive secrets.
  - apiGroups: [""]
    resources: ["secrets"]
    resourceNames: ["kubernetes-dashboard-key-holder", "kubernetes-dashboard-certs", "kubernetes-dashboard-csrf"]
    verbs: ["get", "update", "delete"]
    # Allow Dashboard to get and update 'kubernetes-dashboard-settings' config map.
  - apiGroups: [""]
    resources: ["configmaps"]
    resourceNames: ["kubernetes-dashboard-settings"]
    verbs: ["get", "update"]
    # Allow Dashboard to get metrics.
  - apiGroups: [""]
    resources: ["services"]
    resourceNames: ["heapster", "dashboard-metrics-scraper"]
    verbs: ["proxy"]
  - apiGroups: [""]
    resources: ["services/proxy"]
    resourceNames: ["heapster", "http:heapster:", "https:heapster:", "dashboard-metrics-scraper", "http:dashboard-metrics-scraper"]
    verbs: ["get"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
rules:
  # Allow Metrics Scraper to get metrics from the Metrics server
  - apiGroups: ["metrics.k8s.io"]
    resources: ["pods", "nodes"]
    verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: kubernetes-dashboard
subjects:
  - kind: ServiceAccount
    name: kubernetes-dashboard
    namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-dashboard
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: kubernetes-dashboard
subjects:
  - kind: ServiceAccount
    name: kubernetes-dashboard
    namespace: kube-system
---
apiVersion: v1
kind: Secret
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-certs
  namespace: kube-system
type: Opaque
---
apiVersion: v1
kind: Secret
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-csrf
  namespace: kube-system
type: Opaque
data:
  csrf: ""
---
apiVersion: v1
kind: Secret
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-key-holder
  namespace: kube-system
type: Opaque
---
kind: ConfigMap
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-settings
  namespace: kube-system
---
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  type: NodePort
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 30001
  selector:
    k8s-app: kubernetes-dashboard
---
kind: Deployment
apiVersion: apps/v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: kubernetes-dashboard
  template:
    metadata:
      labels:
        k8s-app: kubernetes-dashboard
    spec:
      containers:
        - name: kubernetes-dashboard
          image: kubernetesui/dashboard:v2.0.0-beta6
          ports:
            - containerPort: 8443
              protocol: TCP
          args:
            - --auto-generate-certificates
            - --namespace=kube-system          #设置为当前namespace
          volumeMounts:
            - name: kubernetes-dashboard-certs
              mountPath: /certs
            - mountPath: /tmp
              name: tmp-volume
          livenessProbe:
            httpGet:
              scheme: HTTPS
              path: /
              port: 8443
            initialDelaySeconds: 30
            timeoutSeconds: 30
      volumes:
        - name: kubernetes-dashboard-certs
          secret:
            secretName: kubernetes-dashboard-certs
        - name: tmp-volume
          emptyDir: {}
      serviceAccountName: kubernetes-dashboard
      tolerations:
        - key: node-role.kubernetes.io/master
          effect: NoSchedule
          
EOF
```

部署 `Kubernetes Dashboard`
```
$ kubectl apply -f k8s-dashboard-deploy.yaml
```

#### 2、创建监控信息 kubernetes-metrics-scraper
`k8s-dashboard-metrics.yaml`
```
$ cat > k8s-dashboard-metrics.yaml << EOF

kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-metrics-scraper
  name: dashboard-metrics-scraper
  namespace: kube-system
spec:
  ports:
    - port: 8000
      targetPort: 8000
  selector:
    k8s-app: kubernetes-metrics-scraper
---
kind: Deployment
apiVersion: apps/v1
metadata:
  labels:
    k8s-app: kubernetes-metrics-scraper
  name: kubernetes-metrics-scraper
  namespace: kube-system
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: kubernetes-metrics-scraper
  template:
    metadata:
      labels:
        k8s-app: kubernetes-metrics-scraper
    spec:
      containers:
        - name: kubernetes-metrics-scraper
          image: kubernetesui/metrics-scraper:v1.0.1
          ports:
            - containerPort: 8000
              protocol: TCP
          livenessProbe:
            httpGet:
              scheme: HTTP
              path: /
              port: 8000
            initialDelaySeconds: 30
            timeoutSeconds: 30
      serviceAccountName: kubernetes-dashboard
      tolerations:
        - key: node-role.kubernetes.io/master
          effect: NoSchedule
          
EOF
```

部署 Dashboard Metrics
```
$ kubectl apply -f k8s-dashboard-metrics.yaml
```

#### 3、创建 Dashboard ServiceAccount
访问 Kubernetes Dashboard 时会验证身份，需要提前在 Kubernetes 中创建一个一定权限的 ServiceAccount，然后获取其 Token 串，这里为了简单方便，直接创建一个与管理员绑定的服务账户，获取其 Token 串。
`k8s-dashboard-rbac.yaml`
```
$ cat > k8s-dashboard-token.yaml << EOF

kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: admin
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name: admin
  namespace: kube-system
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin
  namespace: kube-system
  labels:
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
    
EOF
```

部署访问的 ServiceAccount：
```
$ kubectl apply -f k8s-dashboard-rbac.yaml
```
获取 Token：
```
$ kubectl describe secret/$(kubectl get secret -n kube-system |grep admin|awk '{print $1}') -n kube-system
```
token 信息如下：
```
eyJhbGciOiJSUzI1NiIsImtpZCI6IlZBZWNPbTNaQ2hLUWRhRFFUWGdUY3YtSl9PN0d0Y3lvZi1RVExaUXd0RjAifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi10b2tlbi1jcGdueCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJhZG1pbiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6ImViZGI2M2YxLTEyOTktNDBiMi1iZmM4LTZiMDk2Mzc4ODNkZiIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTphZG1pbiJ9.bLDN5ju77w8_T1s9saZ-gll2wqFOyKbyieISDcW6yk_UW8AdFqSHc9eLNGnOBgUJYJENl_ovluwHPJ4be-Junnopf5rXxMWHIN-wvjB-sTUa8P8vw8R4BD_y5slHXrmvuh9Rr5_1Ts2g76CenQQ2bNJHZrOAuHivMYxY3_iEe9l42R-Dfk-00uajn719-cAWk5bUawHkRs5jblLjoTLtRyycfNHasKuS7WGTglw27o2yAWTAcFvc0WHOnwYLlrZMYvk1AJ67vOG2O_Cz6mkzKKBHSlLgdIj6tsQqAy8p4O9WWSx-Njq0HxehWoBWrpBRqU0ylzi7lgUNjyI2yZvXKw
```
然后输入 Kubernetes 集群任意节点地址配置上面配置的 Service 的 NodePort 30001 来访问看板。输入地址 `https://192.168.2.11:30001` 进入看板页面，输入上面获取的 Token 串进行验证登录。
>> 提醒一下，由于谷歌浏览器访问自签名证书的网址时，可能会被拒绝访问，所以，这里推荐最好使用火狐浏览器访问。
 
—FIN—







