# 服务器

三台ubuntu：master、slave0、slave1

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

# 配置root

```shell
sudo passwd root    # 设置root密码
sudo vim /etc/ssh/sshd_config    # 允许root登录
----------------------------
PermitRootLogin yes
----------------------------
sudo systemctl restart sshd     # 重启sshd
```

# 关闭swap

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
timedatectl set-timezone Asia/Shanghai
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

# Troubleshooting

在更新源时，可能出现公钥不可用的情况：

> Err:1 http://mirrors.ustc.edu.cn/kubernetes/apt kubernetes-xenial InRelease
>   The following signatures couldn't be verified because the public key is not available: NO_PUBKEY B53DC80D13EDEF05 NO_PUBKEY FEEA9169307EA071

```shell
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv [pubkey]
```
