[Ubuntu下搭建Kubernetes集群(3)--k8s部署 - 走看看 (zoukankan.com)](http://t.zoukankan.com/xl2432-p-10933022.html)

[(81条消息) ubuntu部署k8s_信安成长日记的博客-CSDN博客_ubuntu安装k8s](https://blog.csdn.net/SHELLCODE_8BIT/article/details/122192034)

[(81条消息) Debian11之基于kubeadm安装K8S(v1.26.0) 集群_大能嘚吧嘚的博客-CSDN博客](https://blog.csdn.net/qq_30818545/article/details/128056996)

# Install

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

# k8s集群部署

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

找一个能下载的镜像，再修改tag为：registry.k8s.io

```shell
docker pull k8simage/kube-apiserver:v1.26.0-rc.1
docker pull k8simage/kube-controller-manager:v1.26.0-rc.1
docker pull k8simage/kube-scheduler:v1.26.0-rc.1
docker pull k8simage/kube-proxy:v1.26.0-rc.1
docker pull k8simage/pause:3.9
docker pull k8simage/etcd:3.5.6-0
docker pull k8simage/coredns:v1.9.3
-----------------------------------------
docker tag [image_id] registry.k8s.io/kube-apiserver:v1.26.0
```

## 集群初始化

```shell
kubeadm init \
--pod-network-cidr=10.0.0.0/24 \
--kubernetes-version=v1.26.0 \
--apiserver-advertise-address=10.0.2.15 \
--node-name=master
```

- pod-network-cidr：子群子网

- kubernetes-version：k8s版本，必须和导入镜像一致

- apiserver-advertise-address：
