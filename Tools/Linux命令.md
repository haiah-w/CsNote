ubuntu 20.04

# 服务相关命令

查看启动的所有服务

```shell
sudo service --status-all
```

启动服务

不看看咋打的团 追得上？

```shell
sudo service [service_name] start
```

# 端口命令

netstat命令

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
