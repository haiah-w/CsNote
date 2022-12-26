ubuntu 20.04

# 服务相关命令

查看启动的所有服务

```shell
sudo service --status-all
```

查看指定服务：

```shell
sudo systemctl -a | grep mysql
sudo service --status-all | grep mysql
```

启动服务

```shell
sudo service [service] start
sudo systemctl start [service]
```

# 防火墙

查看防火墙状态：

```shell
sudo ufw status
--------------
Status: inactive
Status: active
--------------
sed -i 's/.*swap.*/#&/' /etc/fstab    # 注释掉swap那一行
```

开启/关闭：

```shell
sudo ufw enable
sudo ufw disable
```

允许/拒绝指定端口：

```shell
sudo ufw allow 3306
sudo ufw deny 3306
```

删除已创建的防火墙规则：

```shell
ufw delete allow/deny 3306
```

# 端口命令

## iptables

开放指定端口，重启失效：

```shell
sudo iptables -I INPUT -p tcp --dport 3306 -j ACCEPT
```

持久化端口开放策略：

```shell
sudo apt-get install iptables-persistent
-----------------------------------------
sudo netfilter-persistent save
sudo netfilter-persistent reload
```

## netstat

- -a 或--all：显示所有连线中的Socket；

- -p 或--programs：显示正在使用Socket的程序识别码和程序名称；

- -t 或--tcp：显示TCP传输协议的连线状况；

- -u 或--udp：显示UDP传输协议的连线状况；

- -l 或--listening：显示listen中的服务器的Socket；

- -x 或--unix：此参数的效果和指定"-A unix"参数相同；

- -l 或--listening：显示listen中的服务器的Socket；

# 权限/用户/组

给用户添加root权限：

```shell
sudo vi /etc/sudoers
----------------------
root    ALL=(ALL:ALL) ALL
will   ALL=(ALL:ALL) ALL     # 有root权限 sudo需要密码
will    ALL=(ALL:ALL) NOPASSWD: ALL # sudo免密
```

查看所有/过滤群组：

```shell
sudo cat /etc/group |grep docker
```

添加用户到群组：

```shell
sudo gpasswd -a [用户] [群组]
-----------------------------
sudo gpasswd -a will docker
```

查看当前用户的用户组

```shell
id
newgrp docker # 刷新用户组 
```

# scp文件传输

```shell
scp [args] [src] [user@ip:/dst]
```

从本地传输到远端：

```shell
scp -r k8s-images.tar will@192.168.1.4:/home/will
```
