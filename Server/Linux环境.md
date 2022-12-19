包管理

1、更换国内源

```shell
mv /etc/apt/sources.list /etc/apt/sources.list.back
sudo vi /etc/apt/sources.list
```

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

2、更换dns：8.8.8.8

```shell
sudo vi /etc/resolv.conf
------------------------
[network]
generateResolvConf = false
nameserver 8.8.8.8
```

3、重启wsl网络失效问题：

(1)重置wsl网络

```shell
wsl --shutdown
netsh winsock reset
netsh int ip reset all
netsh winhttp reset proxy
ipconfig /flushdns
```

(2)防止防火墙阻止wsl

```powershell
New-NetFirewallRule -DisplayName "WSL" -Direction Inbound  -InterfaceAlias "vEthernet (WSL)"  -Action Allow
```

3、修改字体

下载字体到/usr/share/fonts,目录下执行：

```shell
sudo mkfontscale
sudo mkfontdir
sudo fc-cache -fv
```

# 虚拟机安装

镜像文件：[aliyun-ubuntu-releases-20.04](https://mirrors.aliyun.com/ubuntu-releases/20.04/)

## windowsTerminal登录

```shell
Install-Module PSReadLine -AllowPrerelease -Force
```
