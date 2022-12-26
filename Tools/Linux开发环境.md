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

Go

[Download and install - The Go Programming Language](https://go.dev/doc/install)

# Rust

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

# NodeJs

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
