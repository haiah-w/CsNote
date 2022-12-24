# 服务器

三台ubuntu：master、slave0、slave1

## 修改主机名：

```shell
sudo vi /etc/hosts
sudo vi /etc/hostname
```

# 节点间host映射

```shell
sudo vi /etc/hosts
----------------------
192.168.1.4 master
192.168.1.4 slave0
192.168.1.5 slave1
```

# 时间同步

```shell
sudo apt-get install ntpdate
```

# 主机间免密

1、安装openssh-server

```shell
sudo apt-get -y install openssh-server
```

2、生成ssh公私钥

```shell
ssh-keygen-t rsa
```

3、添加公钥到其他节点的`~.ssh/authorized_keys`

# Docker/Kubernetes

前置依赖：

```shell
sudo apt-get update
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release \
    apt-transport-https
```

国内源：/etc/apt/sources.list

```shell
deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable
deb http://mirrors.ustc.edu.cn/kubernetes/apt kubernetes-xenial main
```

## Docker Engine

[Install Docker Engine on Debian](https://docs.docker.com/engine/install/debian/)

```shell
sudo apt-get remove docker docker-engine docker.io containerd runc
```

安装

```shell
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

## Kubectl/kubeadm/kubelet

[在 Linux 系统中安装并设置 kubectl](https://kubernetes.io/zh-cn/docs/tasks/tools/install-kubectl-linux/)

update并安装：

```shell
sudo apt-get update
sudo apt-get install -y kubectl kubeadm kubelet
```

# 配置

## docker镜像加速

配置docker镜像加速(阿里云加速)，没有则新建：

/etc/docker/daemon.json

```json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "registry-mirrors":["https://xmgeb39x.mirror.aliyuncs.com"]
}
```

## 服务启动

```shell
sudo service --status-all # 查看所有服务
sudo service docker start # 启动服务
sudo service docker restart # 重启服务
```

## 非root操作docker

1、赋予用户root权限

2、添加用户到docker组

docker安装时默认创建docker组

```shell
# 查看docker组
sudo cat /etc/group  |grep docker 

# 查看docker.sock所在组
ll /var/run/docker.sock 

# 添加will用户到docker组
sudo gpasswd -a will docker  

# 刷新用户组
newgrp docker

# 查看当前用户所在群组
id
```

# Troubleshooting

在更新源时，可能出现公钥不可用的情况：

> Err:1 http://mirrors.ustc.edu.cn/kubernetes/apt kubernetes-xenial InRelease
>   The following signatures couldn't be verified because the public key is not available: NO_PUBKEY B53DC80D13EDEF05 NO_PUBKEY FEEA9169307EA071

```shell
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv [pubkey]
```
