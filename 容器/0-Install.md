三台ubuntu：master、slave0、slave1

# 集群准备

## 更换源

```shell
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse
# 预发布软件源
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse
```

```shell
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] http://mirrors.aliyun.com/docker-ce/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

更新并添加公钥：

```shell
apt-get update
--------------
apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv [pub-key]
```

## 配置root

```shell
sudo passwd root    # 设置root密码
sudo vim /etc/ssh/sshd_config    # 允许root登录
----------------------------
PermitRootLogin yes
----------------------------
sudo systemctl restart sshd     # 重启sshd
```

## 关闭swap

```shell
sudo swapoff -a
# 查看 swap为0 即可
free
```

## 修改主机名：

```shell
sudo vi /etc/hosts
sudo vi /etc/hostname
```

## 节点间host映射

```shell
sudo vi /etc/hosts
----------------------
192.168.1.4 master
192.168.1.4 slave0
192.168.1.5 slave1
```

## 时间同步

```shell
timedatectl set-timezone Asia/Shanghai
```

## 主机间免密

1、安装openssh-server

```shell
sudo apt-get -y install openssh-server
```

2、生成ssh公私钥

```shell
ssh-keygen-t rsa
```

3、添加公钥到其他节点的`~.ssh/authorized_keys`

# Docker

[Install Docker Engine on Debian](https://docs.docker.com/engine/install/debian/)

```shell
sudo apt-get remove docker docker-engine docker.io containerd runc
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

- image
- container
- repository

## Docker配置

配置文件：/etc/docker/daemon.json

```shell
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "registry-mirrors":[
    "http://hub-mirror.c.163.com",
    "https://docker.mirrors.ustc.edu.cn",
    "https://registry.docker-cn.com",
    "https://docker.mirrors.ustc.edu.cn",
    "https://xmgeb39x.mirror.aliyuncs.com"
  ]
}
```

# k8s

```shell
# 前置依赖
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release \
    apt-transport-https
# 安装
sudo apt-get update
sudo apt-get install -y kubectl kubeadm kubelet
sudo apt-mark hold kubelet kubeadm kubectl  # 防止自动更新
```

- kubelet：k8s的核心服务

- kubeadm：安装k8s的集成工具

- kubectl：k8s的命令行工具，部署完成之后的操作需要用到

## 镜像准备

查看kubeadm集群部署所需镜像：

```shell
$ kubeadm config images list
registry.k8s.io/kube-apiserver:v1.26.0
registry.k8s.io/kube-controller-manager:v1.26.0
registry.k8s.io/kube-scheduler:v1.26.0
registry.k8s.io/kube-proxy:v1.26.0
registry.k8s.io/pause:3.9
registry.k8s.io/etcd:3.5.6-0
registry.k8s.io/coredns/coredns:v1.9.3
```

提前拉取镜像：

```shell
kubeadm config images pull
```

## Containerd配置

1.24之后默认使用containerd runtime

配置文件：/etc/containerd/config.toml

```toml
disabled_plugins = []
```

containerd命令行：

1、查看镜像列表：

```shell
ctr images list
```

## 集群初始化

1、master主节点：

```shell
kubeadm init \
--pod-network-cidr=10.244.0.0/16 \
--kubernetes-version=v1.26.0 \
--apiserver-advertise-address=192.168.79.127 \
--node-name=master
```

- pod-network-cidr：集群子网

- kubernetes-version：k8s版本，必须和导入镜像一致

- apiserver-advertise-address：master节点IP

- image-repository：镜像仓库地址

初始化输出：

```shell
To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.79.127:6443 --token lp72n9.jgo5r7ynrtsdzltb \
        --discovery-token-ca-cert-hash sha256:19244f2c5f4c40763706c8d4d007c224cdaa74dbd332becf477349512e695874
```

2、工作节点：

```shell
kubeadm join 192.168.79.127:6443 --token lp72n9.jgo5r7ynrtsdzltb \
        --discovery-token-ca-cert-hash sha256:19244f2c5f4c40763706c8d4d007c224cdaa74dbd332becf477349512e695874
```

## 安装网络插件

下载flannel配置文件

```shell
wget https://raw.githubusercontent.com/flannel-io/flannel/v0.20.2/Documentation/kube-flannel.yml
```

安装flannel:

```shell
kubectl apply -f kube-flannel.yml
---------------------------------
namespace/kube-flannel created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds created
```
