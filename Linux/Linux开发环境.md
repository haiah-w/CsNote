# 更换源

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

# 固定IP

查看网关：

```shell
route -n
```

网络配置文件：/etc/netplan/xxx.yaml

```yaml
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    eth0: 
      dhcp4: no 
      dhcp6: no
      addresses: [172.25.0.7/24]
      gateway4: 172.25.0.1
      nameservers:
        addresses: [8.8.8.8, 114.114.114.114]
```

使配置生效：

```shell
netplan apply
```

# 允许Root远程登录

```shell
vim /etc/ssh/sshd_config
------------------------
PermitRootLogin yes
------------------------
# 重启ssh
systemctl restart ssh
```

# Kubernetes集群

增加Docker、k8s源：

```shell
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] http://mirrors.aliyun.com/docker-ce/linux/debian (lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg 
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] [kubernetes-apt安装包下载_开源镜像站-阿里云](https://mirrors.aliyun.com/kubernetes/apt/) kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
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

## 虚拟机配置root

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
sed -i 's/.*swap.*/#&/' /etc/fstab    # 永久关
# 查看 swap为0 即可
free
```

## 节点间host映射

```shell
#  固定主机名
sudo vi /etc/hostname
# 添加节点映射
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

## Docker

新建/etc/apt/sources.list.d/docker.list,添加docker源：

```shell
deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable
```

更新并添加公钥：

```shell
apt-get update
apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv [pub key]
# 再次更新
apt-get update
```

安装docker-ce、containerd.io

```shell
apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

## kubectl/kubeadm/kubelet

前置依赖：

```shell
apt-get install -y apt-transport-https ca-certificates curl
```

下载Google公钥、添加apt源：

```shell
curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

更新并安装组件：

```shell
apt-get update
apt-get install -y kubectl kubeadm kubelet
```

# DevelopEnv

## Go

[Download and install - The Go Programming Language](https://go.dev/doc/install)

```shell
# 下载一个版本，解压，配置环境变量即可
rm -rf /opt/go && tar -C /opt -xzf go1.18.9.linux-amd64.tar.gz
export PATH=$PATH:/opt/go/bin
go version
----------
go version go1.18.9 linux/amd64
```

简单配置

```shell
go env -w GOPROXY=https://goproxy.io
go env -w GO111MODULE=on
```

## Rust

Ubuntu20.04版本

GCC:[编译器、链接器和库简介 –learncpp.com](https://www.learncpp.com/cpp-tutorial/introduction-to-the-compiler-linker-and-libraries/)

```shell
sudo apt install make build-essential gcc curl
```

官网安装：[Rusts](https://www.rust-lang.org/zh-CN/tools/install)

```shell
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

速度慢，代理：[RsProxy](https://rsproxy.cn/)

错误：error: linking with `cc` failed: exit status: 1

更改rustup使用的toolchain

```shell
rustup install nightly-2019-06-01-x86_64-unknown-linux-gnu
rustup default nightly-2019-06-01-x86_64-unknown-linux-gnu
```

错误：supported edition values are `2015` or `2018`, but `2021` is unknown

```shell
rustup show
rustup update
```

## NodeJs

官网：[Download | Node.js (nodejs.org)](https://nodejs.org/en/download/)

下载二进制文件

```shellag-0-1ggsang4hag-1-1ggsang4h
tar -xvf node-v18.12.0-linux-x64.tar.xz
```

解压后，做软连接到/usr/bin/下

```shell
sudo ln -s home/haiah/dev/node-v18.12.0-linux-x64/node /usr/bin/node
sudo ln -s /home/haiah/dev/node-v18.12.0-linux-x64/lib/node_modules/npm/bin/npm-cli.js /usr/bin/npm
sudo ln -s /home/haiah/dev/node-v18.12.0-linux-x64/lib/node_modules/npm/bin/npx-cli.js /usr/bin/npx
```

# MySQL

ubuntu安装mysql-server 8.0

查看可安装版本：

```shell
will@master:~$ sudo apt list | grep mysql-server
default-mysql-server-core/focal 1.0.5ubuntu2 all
default-mysql-server/focal 1.0.5ubuntu2 all
mysql-server-8.0/focal-updates,now 8.0.31-0ubuntu0.20.04.2 amd64 [installed,automatic]
mysql-server-core-8.0/focal-updates,now 8.0.31-0ubuntu0.20.04.2 amd64 [installed,automatic]
mysql-server/focal-updates,now 8.0.31-0ubuntu0.20.04.2 all [installed]
```

安装最新版mysql-server

```shell
sudo apt-get install mysql-server
```

开机自启动：

```shell
sudo systemctl enable mysql
sudo systemctl start mysql
```

登录并修改root密码：

```shell
sudo mysql --user=root mysql
-----------------------------
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'root';
flush privileges;
```

开放端口远程访问：

```shell
sudo ufw allow 3306
sudo iptables -I INPUT -p tcp --dport 3306 -j ACCEPT
sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
------------------------------------------
bind-address = 0.0.0.0
```
